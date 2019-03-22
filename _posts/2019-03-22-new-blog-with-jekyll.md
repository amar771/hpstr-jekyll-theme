---
layout: post
title: "New Blog with Jekyll"
description: "Switching from Django custom blog to Jekyll."
tags: [programming, jekyll, frontend, linux, devops]
image:
  path: /images/unsplash-5.jpg
  feature: unsplash-5.jpg
  credit: Photo by Alex Knight on Unsplash
  creditlink: https://unsplash.com/photos/-4pZ_YqcSFc
---

# Intro

So couple months ago I decided to switch from a blog/CMS that I made with Django and Python to something else.

I wanted something that is easier to maintain, easier to add stuff to, and I wanted to play with something new.

# Static Site Generators

As I was researching what my options are, I stumbled upon static site generators.

At first it sounded silly and I thought it would be a horrible idea. After some looking into it I wanted to play with it and see how it works and how it really is.

Next comes the choice.

## The Choice

My main choices were:
* Jekyll
* Hugo
* Pelican
* MkDocs
* Or some javascript based one

I really didn't want to work with JS so I ignored all those.

I was really leaning towards Pelican because of my affinity towards Python and everything python related. After thinking about it for a bit, I decided against anything python since I've worked a lot with it and I wouldn't learn or mess around as much with it.

So in the end I was left with two choices Jekyll and Hugo. Hugo was really attractive since everyone was praising for it build times, but I didn't really care about that since I would never build anything larger and jekyll was really attractive since github pages runs on it and all the support behind it.

Both choices looked great but in the end I choose Jekyll since the support it had and theme I liked supported it so I went with it.

I might sometime in the future change to Hugo after some testing and playing around with it, but for now it's Jekyll.

# Switch from full-blog to static

Running full-blog is certainly nicer since I can add anything I want to it, but Jekyll is much easier to run.

## Writing posts

When I made my python blog, posts were made by writing posts with pagedown directly in the browser, then checking them and finally publishing it to public.

And with jekyll instead of writing posts in browser I write them in sublime and then push them to server. Ultimately nothing changed there, and that's the biggest thing I was doing on the blog, writing posts.

## Maintenance

With my blog I had to run a full-blown VPS that ran blog, database, web server, had https that I had to renew every couple months manually since my debian VPS wasn't working nicely with LetsEncrypt.

Sometimes nginx would just decide to stop working and I had to SSH in and restart it, other times gunicorn would crash and required restarts.

With Jekyll I did initial setup with netlify and now I just push to github and netlify handles everything else, almost no maintenance. I might add some notes to this after couple of months of running this, but right now it's pretty good.

## Price

Running jekyll with github and netlify is completely free. I could've also used my own VPS for this but netlify is enough for my size.

With my django blog I had to run my own VPS and while it wasn't expensive it certainly wasn't free.

# Process of switching

Everything I did to switch.

## Local setup

I'm running Ubuntu 18.04 Desktop VM that's used only for this Jekyll blog.

Here is everything I did to make it suitable for my needs:

Updating my machine, installing ruby, bundler and jekyll.


```shell
sudo apt update
sudo apt upgrade
sudo apt install -y ruby
sudo gem install bundler
sudo gem install jekyll
```

And a bunch of errors started popping up. Since I was doing this on a brand new install of Ubuntu I didn't standard stuff that bundler and jekyll require.

This is what I needed to install

```shell
sudo apt install -y ruby-all-dev
sudo apt install -y make
sudo apt install -y gcc
sudo apt install -y g++
```

No more errors after installing these.

## Theme

For theme I decided on <a href="https://github.com/mmistakes/hpstr-jekyll-theme">HPSTR theme</a> by mmistakes. I really enjoyed it but needed to make some changes.

I forked the repo to my github account and then cloned it to my machine.

```shell
sudo apt install -y git
git clone https://github.com/amar771/hpstr-jekyll-theme
```

After cloning I do the initial setup of bundler.

```shell
bundle install
```

And to test if everything works properly

```shell
bundle exec jekyll build
bundle exec jekyll serve
```


First I wanted was darker color scheme, so I went to <a href="https://coolors.co/">coolors.co</a> and found out color scheme I enjoy and went to edit some files.

Edited \_config.yml, about pages, deleted posts that explain how to use the theme, added info about me, and rewrote my old blog posts into kramdown and rouge.

## Images

I've always been fan of Unsplash so I've decided to use images from that site, of course all images I've used are also credited with link to the original.

Because most of the images are really large I used gimp to lower their size and quality a bit so they can load faster.

## Font

I've also used <a href="http://angrytools.com/gradient/images"></a>
After looking through googles fonts I've decided on <a href="">Inconsolata</a>

## Hosting

For hosting the site I've decided on <a href="https://www.netlify.com/">netlify</a>. I've looked through a lot of options for hosting this including github pages and hosting my own web server, but netlifys continuous deployment and instant https won me over. They also have a ton more options like forms and identity with static sites that I want to play around with in future. Plus they're free.

Once I've set everything up, connected my github account to netlify and started the build, it failed, some type of npm error that I definitely didn't have when I built locally, I didn't even install npm on my machine so it was really strange to me.

After trying a couple more options and playing around with it builds were still failling. I had to go and read the documentation on their page, and I found out that you can change environmental variables. After some more looking into it I found they have NODE_ENV variable. I've then set the NODE_ENV variable to production and build passed.

## DNS

I've used <a href="https://www.digitalocean.com/">DigitalOcean</a> for my last site so I had to change the dns servers to point to netlifys servers. Easy to change but dns propagation took around 15 hours and during that time my site was technically down since to most of the world it was pointing to my old django blog that I took down.

What I could've done to prevent that was to keep my django blog up until the propagation finished, build and served my jekyll site from the same server, or do redirection with web server, but since I didn't have a lot of visitors I've decided to do nothing.

# New blog

And that's how I got my new  blog.