---
layout: post
title: Setting up my Django blog
description: "How to setup my django blog that I used for over a year."
tags: [django, python, programming, linux]
image:
  path: /images/unsplash-1.jpg
  feature: unsplash-1.jpg
  credit: Felix Mittermeier on Unsplash
  creditlink: https://unsplash.com/photos/Knwea-mLGAg
---

# Stack

For server I'm using Debian 9 "Stretch".

Serving the site with nginx as reverse-proxy for gunicorn.

Database is PostgreSQL.

# Deployment

## Basic server setup

After spinning up the droplet/VM update the server, add user, give user sudo permissions, generate ssh keys for login.

```shell
ssh root@[server-ip]
apt update
apt upgrade
apt install -y sudo
apt install -y vim # or any other editor of your choice
adduser [your-user]
usermod -aG sudo [your-user]
passwd [your-user]
```

To generate SSH keys, on local machine execute:

```shell
ssh-keygen -t rsa
```

Copy the content from ```~/.ssh/id_rsa.pub``` over to server ```~/.ssh/authorized_keys``` of either root or the user depending which one you will be using. (Using root is never recommended, best thing to do is to disable root login through ssh since it's unnecessary). 

```shell
vim /etc/ssh/sshd_config
```

and set ```PasswordAuthentication no```. After doing that execute ```systemctl restart sshd```. You will be kicked off the server and you will be able to login on it again with the key.

## Firewall

For the firewall I've decided to use UFW instead of iptables since it does the job and is much easier to use.

```shell
sudo apt install -y ufw
sudo ufw status
sudo ufw app list
```

You can manage it with either ```22/tcp``` or with application name like ```OpenSSH```. UFW blocks everything by default so we only have to specify what should be allowed.

```shell
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw status
sudo ufw enable
```

And that is basic firewall configuration, if you need any other port you can add it later or if you want to delete a rule, do:

```shell
sudo ufw delete allow 22/tcp
```

# The Blog

## Database

Setting up PostgreSQL to be ready for use. Install PostgreSQL:

```shell
sudo apt install -y libpq-dev
sudo apt install -y postgresql
sudo apt install -y postgresql-contrib
```

Switch to postgres user

```shell
sudo -i -u postgres
psql
```

Create database, user, and give permissions for database to the created user:

```sql
CREATE USER user WITH PASSWORD 'secret';
CREATE DATABASE db OWNER user;
GRANT ALL PRIVILEGES ON DATABASE db TO user;
\q
```

## Python

Install basic tools necessary for Python3:

```shell
sudo apt install -y git
sudo apt install -y python3-pip
sudo apt install -y python3-dev
sudo apt install -y python3-venv
sudo pip3 install --upgrade pip
sudo pip3 install virtualenv
```

Get the blog, set up the virtual environments and install the dependencies:

```shell
mkdir blog
cd blog/
git clone https://github.com/amar771/Django-Blog/
cd Django-Blog
python3 -m venv venv/
source venv/bin/activate
pip3 install -r requirements.txt
```

## Blog settings

Making blog production ready. Setting .env file for usernames, passwords and everything else that shouldn't be kept in public repository.

```shell
cp .env.example .env
vim .env
```

Generate a strong random 50 character string for Django Secret Key, set the debug to False, add your domain name to ALLOWED HOSTS:

```
SECRET_KEY=someSuperRandomSecretKey123#
DEBUG=False
ALLOWED_HOSTS=test.com, www.test.com, localhost
```

For email settings I used sendgrid, any other provider works same. For database enter the user and database created earlier:

```
DB_NAME=db
DB_USER=user
DB_PASSWORD=secret
```

If database was on another server we'd change DBHOST and DBPORT accordingly. Skipping Googles ReCaptcha Key for now. Set OTP_TOTP_ISSUER to name you want to show up on Authenticator apps:

```
OTP_TOTP_ISSUER="My company"
```

One last thing for now is turning off Two-Factor Authentication for admin for a moment. It's gonna be reactivated later but for now it's unnecessary.

```shell
vim mysite/urls.py
```

Change ```admin.site.__class__ = OTPAdminSite```

To ```# admin.site.__class__ = OTPAdminSite```

Save and exit.

## First run

We're going to activate virtual environment, set up the database with Django models, create superuser for blog, collect the static content and run gunicorn to test if everything up until now works.

```shell
source venv/bin/activate
python manage.py migrate
python manage.py createsuperuser
python manage.py collectstatic
python -m pip install gunicorn
gunicorn --bind 0.0.0.0:8000 mysite.wsgi
```

Allow port 8000 on UFW ```sudo ufw allow 8000/tcp```.

Navigate to blog and test if everything works properly and stop gunicorn and delete the UFW rule for port 8000.

```shell
sudo ufw delete allow 8000/tcp
```

## Gunicorn in Systemd

We have to create a systemd process that will run on startup and will start gunicorn server for our blog.

```shell
sudo vim /etc/systemd/system/gunicorn.service
```

Enter this:

```
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=yourUser
Group=www-data
WorkingDirectory=/path/to/Django-Blog
ExecStart=/path/to/Django-Blog/venv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/path/to/Django-Blog/mysite.sock mysite.wsgi:application

[Install]
WantedBy=multi-user.target
```

Start it with:

```shell
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
sudo systemctl status gunicorn
```

## nginx

Configuring nginx to act as reverse proxy for gunicorn

```shell
sudo vim /etc/nginx/sites-available/blog
```

```nginx
server {
	listen 80;
	server_name test.com www.test.com;
	server_name IP_OF_SERVER;

	location = /favicon.ico {
		access_log off;
		log_not_found off;
	}

	location / {
		include proxy_params;
		proxy_pass http://unix:/path/to/Django-Blog/mysite.sock;
	}
}
```

Create a link to the nginx configuration file in sites-enabled and test if it works:

```shell
sudo ln -s /etc/nginx/sites-available/blog /etc/nginx/sites-enabled
sudo nginx -t
```

If everything works properly start nginx up:

```shell
sudo systemctl restart nginx
sudo systemctl enable nginx
```

## HTTPS

<div markdown="0"><a href="https://certbot.eff.org/lets-encrypt/debianstretch-nginx" class="btn">cerbot</a></div>

Add backports to debian repositories

```shell
sudo vim /etc/apt/sources.list.d/sources.list
```

At the end of the file add:

```
deb http://ftp.debian.org/debian stretch-backports main
```

And Save.

Update the system and install certbot for nginx.

```shell
sudo apt update
sudo apt install -y python-certbot-nginx
sudo certbot --authenticator standalone --installer nginx --pre-hook "nginx -s stop" --post-hook "nginx"
```

That should add HTTPS to your blog.

Next we need to fix our configuration of nginx for blog:

```shell
sudo vim /etc/nginx/sites-available/blog
```

And remove the line ```listen 80;```.
Save and test nginx:

```shell
sudo nginx -t
```

If everything works properly:

```shell
sudo systemctl restart nginx
```

Site should have HTTPS right now.

## Adding 2FA

Navigate to ```test.com/admin``` and log in as the superuser created earlier.

Under OTP_TOTP, click on TOTP devices. Click ADD TOTP DEVICE. Fill in everything, you're going to get link to qrcode, click on it and scan the code with your authenticator app.

Enable the device you just created so it can be used.

Edit the ```urls.py``` so that it can server admin login with 2FA.

```shell
sudo vim /path/to/Django-Blog/mysite/urls.py
```

Uncomment the line commented earlier

```# admin.site.__class__ = OTPAdminSite``` to ```admin.site.__class__ = OTPAdminSite```

Save and exit. Then restart gunicorn with:

```shell
sudo systemctl restart gunicorn
```

Go to test.com/admin and besides username and password, you will need to provide the code from the app.

## Google reCAPTCHA

I used reCAPTCHA for preventing spam on my Contact form and because of its ease of implementation.

Go to <a href="https://www.google.com/recaptcha/intro/v3.html">reCAPTCHA</a> and register your site. You will get 2 keys.

```shell
vim /path/to/Django-Blog/.env
```

Add your secret key here, and 

```shell
vim /path/to/Django-Blog/blog/templates/blog/contact.html
```

Add your data-sitekey here.

```shell
source /path/to/Django-Blog/venv/bin/activate
python /path/to/Django-Blog/manage.py collectstatic
sudo systemctl restart gunicorn
```

Contact form should have reCAPTCHA enabled now.

## Finishing touches

Edit all HTML files in ```Django-Blog/blog/templates/blog``` to match you, and after updating them run

```shell
source /path/to/Django-Blog/venv/bin/activate
python /path/to/Django-Blog/manage.py collectstatic
sudo systemctl restart gunicorn
```

to apply those changes immediately to esrver.

# Finished

And that's it, everything should work now.
