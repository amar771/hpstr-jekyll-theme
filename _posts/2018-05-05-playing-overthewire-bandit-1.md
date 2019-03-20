---
layout: post
title: "OverTheWire: Bandit Part 1"
description: "Going through first 10 levels of OverTheWires Bandit wargame"
tags: [linux, security, overthewire, bandit]
image:
  path: /images/unsplash-2.jpg
  feature: unsplash-2.jpg
  credit: Alex Knight on Unsplash
  creditlink: https://unsplash.com/photos/Ys-DBJeX0nE
---

# Intro

This is my walkthrough of OverTheWires Bandit. I recently found out about it and wanted to see how it goes and if I can manage to figure everything out.

Apparently bandit is aimed at beginners and is really easy, consisting of mostly basic linux stuff.

More info about bandit <a href="http://overthewire.org/wargames/bandit/">here</a>.

## Bandit Level 0

Level 0 is really simple, just connecting to the server vias ssh. I was given port 2220 to connect to.

```shell
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

Entered bandit0 for password when prompted and that was it.

## Level 0 --> Level 1

So password for level 1 is located in home directory for user bandit0 in file called ```readme```.

```shell
bandit0@bandit:~$ ls
bandit0@bandit:~$ cat readme
boJ9jbbUNNfktd78OOpsqOltutMc3MY1
```

```ls``` is command for listing the contents of a directory and ```cat``` is simple way to read the contents of a file(```less``` or ```more``` could've also been used).

```shell
bandit0@bandit:~$ exit
```

```exit``` to leave the current ssh session.

I got my password for bandit1 and let's see if it works:

```shell
ssh bandit1@bandit.labs.overthewire.org -p 2220
```

And that was it. Basic beginning.

## Level 1 --> Level 2

Now I'm ssh-ed into user bandit1 and the password is located in file ```-``` in home directory.

Doing ```cat -``` won't work since ```-``` is considered a special character reserved for flags.

```shell
bandit1@bandit:~$ cat ./-
CV1DtqXWVFXTvM2F0k09SHz0YwRINYA9
```

```./-``` works because ./ says that we're working with file and it also indicates that file is located in current directory.

```shell
bandit1@bandit:~$ exit
ssh bandit2@bandit.labs.overthewire.org -p 2220
```

Another basic one.

```../``` is used for files one directory up.
We could've also use full path of a file for example ```/home/bandit1/-```.

## Level 2 --> Level 3

SSH-ed with user bandit2, password for next level is in file that contains spaces in home directory.

```shell
bandit2@bandit:~$ ls -l
-rw-r----- 1 bandit3 bandit2 33 Dec 28 14:34 spaces in this filename
bandit2@bandit:~$ cat "spaces in this filename"
UmHadQclWmgdLOKQ3YNgjWxGoRMb5luK
```

```-l``` flag in ```ls``` command is used for long listing, lists details about all files/directories. And we see that there is one file that contains spaces in its name while with just ls I could've thought that there are 4 files called "spaces", "in", "this", "filename" instead of just one file called "spaces in this filename".

```shell
bandit2@bandit:~$ exit
ssh bandit3@bandit.labs.overthewire.org -p 2220
```

Done. Simple.

Quotation marks ignore spaces and most special characters so they're ideal for this situation. (Tab works with quotation marks so you can do "s[tab] to finish the command for you.)

I could've also done ```cat spaces\ in\ this\ filename"``` where ```\``` is used to escape the character following it, in this case space.

## Level 3 --> Level 4

SSH-ed with user bandit3. Password is in hidden file in directory ```inhere``` in home directory.

```shell
bandit3@bandit:~$ ls -a inhere/
. .. .hidden
```

```-a``` flag for ```ls``` is used to list all contents of a directory including hidden files. (Hidden files start with a dot "." and they don't show with normal ls). Common way to use ```ls``` command is to do ```ls -la``` to list all contents with long listing so we get better with of the directory.

```shell
bandit3@bandit:~$ cat inhere/.hidden
pIwrPrtPN36QITSp3EQaw936yaFoFgAB
bandit3@bandit:~$ exit
ssh bandit4@bandit.labs.overthewire.org -p 2220
```

Done.

## Level 4 --> Level 5

Password is located in only human-readable file in inhere directory located in home directory of user bandit4.

```shell
bandit4@bandit:~$ ls -l inhere/
```

A bunch of files that look same.

```shell
bandit4@bandit:~$ cat inhere/-file00
```

Well that messed my terminal up. Gotta get it back to normal now.

```shell
bandit4@bandit:~$ reset
bandit4@bandit:~$ file inhere/-file00
inhere/-file00: data
bandit4@bandit:~$ file inhere/-file01
inhere/-file01: data
bandit4@bandit:~$ file inhere/-file02
inhere/-file02: data
bandit4@bandit:~$ file inhere/-file03
inhere/-file03: data
bandit4@bandit:~$ file inhere/-file04
inhere/-file04: data
bandit4@bandit:~$ file inhere/-file05
inhere/-file05: data
bandit4@bandit:~$ file inhere/-file06
inhere/-file06: data
bandit4@bandit:~$ file inhere/-file07
inhere/-file07: ASCII text
bandit4@bandit:~$ cat inhere/-file07
koReBOKuIDDepwhWk7jZC0RTdopnAYKh
```

```file``` command tells us what the file is consisting of, it's really handy here since I would have to reset the terminal with every ```cat``` since most of the files aren't text but data.

```shell
bandit4@bandit:~$ exit
ssh bandit5@bandit.labs.overthewire.org -p 2220
```

Done, since there were only 10 files it was easy to manually go through all of them.

## Level 5 --> Level 6

Password is stored somewhere in inhere directory located in home directory of user bandit5. It has following properties:
* human-readable (ASCII-text)
* 1033 bytes in size
* not executable

```shell
bandit5@bandit:~$ ls inhere/
```
Lists a bunch of directories. Too much to go through by hand.

```shell
bandit5@bandit:~$ du -ab inhere/ | grep 1033
1033    inhere/maybehere07/.file2
```

```du``` command lists files with their size, it lists us sizes by directories so we need to use -a flag to list all files individually instead of grouping them by directories. And since it's told us that the file is 1033 "bytes" in size we use flag -b to specify that we need the size in bytes.

We get a long list of files and their sizes and we need to find the one that is 1033 bytes large so we just pipe the output of ```du``` to ```grep``` to find the right file.

```shell
bandit5@bandit:~$ cat inhere/maybehere07/.file2
DXjZPULLxYr17uwoI01bNLQbtFemEgo7
```

Looks like it breaks our terminal but if we just press enter it returns to normal so it's probably good. If we want to be sure it isn't executable but a human-readable file we can do:

```shell
bandit5@bandit:~$ file inhere//maybehere07/.file2
inhere//maybehere07/.file2: ASCII text, with very long lines
```

And yeah, it's just a text file that is padded with spaces to confuse us.

```shell
bandit5@bandit:~$ exit
ssh bandit6@bandit.labs.overthewire.org -p 2220
```

Works.

## Level 6 --> Level 7

Password is stored in file somewhere on the server and has following properties:
* owned by user "bandit7"
* owned by group "bandit6"
* 33 bytes in size

Since server has a lot of files doing this manually is painful. 

```shell
bandit6@bandit:~$ find * / -size 33c -user bandit7 -group bandit6 2>/dev/null
/var/lib/dpkg/info/bandit7.password
```

Let's dissect this command:

```find``` searches for a file in a directory, ```find * /``` searches through all files in all directories on server. * is a wildcard, basically matches everything and / is saying that we're starting to seach in the root directory of the server, it ensures that we're going to look through everything.

```-size 33c``` flag is used because one of the properties of file we're searching for is 33 bytes in size, c after 33 indicates that we're using bytes.

```-user bandit7``` is pretty self-explanatory, we're searching for all files owned by user bandit7.

```-group bandit6``` is also self-explanatory, finds files owned by group bandit6.

```2>/dev/null``` this part is a bit more complicated, it isn't necessary to use this but we're going to get a lot of permission denied messages for directories we don't have permission to access, so we're piping all errors (exit code 2) to null and thus not displaying them.

```shell
bandit6@bandit:~$ cat /var/lib/dpkg/info/bandit7.password
HKBPTKQnIay4Fw76bEy8PVxKEDQRKTzs
bandit6@bandit:~$ exit
ssh bandit7@bandit.labs.overthewire.org -p 2220
```

Works. I had to go through man page for find to figure out the size in bytes but still relatively easy level.

## Level 7 --> Level 8

Password is stored in file data.txt next to the word millionth.

```shell
bandit7@bandit:~$ ls
data.txt
bandit7@bandit:~$ cat data.txt | grep millionth
millionth       cvX2JJa4CFALtqS87jk27qwqGhBM9plV
bandit7@bandit:~$ exit
ssh bandit8@bandit.labs.overthewire.org -p 2220
```

This one was really simple, just using basic piping and grep.

## Level 8 --> Level 9

Password is stored in data.txt and is only line of text that occurs only once.

```shell
bandit8@bandit:~$ ls
data.txt
bandit8@bandit:~$ sort data.txt | uniq -c | grep "1 "
      1 UsvVyFSfZZWbi6wgC7dAFyFuR6jQQUhR
```

```sort``` sorts the file line by line so we can use uniq.

```uniq -c``` counts the amount of times a certain line is being repeated in a file, but file has to be sorted first.

```grep "1 "``` finds all lines that have 1 followed immediately by a space.

```shell
bandit8@bandit:~$ exit
ssh bandit9@bandit.labs.overthewire.org -p 2220
```

Not too complicated, there is probably a more elegant solution, I don't really use uniq or sort that much so I had to play around with them a bit to get this one to work.

## Level 9 --> Level 10

Password is stored in data.txt in one of the human-readable strings that starts with several "=" characters.

```shell
bandit9@bandit:~$ ls
data.txt
bandit9@bandit:~$ strings data.txt | grep ^==
========== theP`
========== password
========== truKLdjsbJ5g7yyJ2X2R0o3a5HQJFuLk
```

```strings``` command finds human-readable strings in data files, and we just pipe all of those to grep to find the ones that start with at least two == (^ is a regex character that finds all lines that start with anything after it, in this case all lines that start with =).

We got a couple of possible passwords but the third one looks the most promising so we're going to try that one.

```shell
bandit9@bandit:~$ exit
ssh bandit10@bandit.labs.overthewire.org -p 2220
```

And it worked. Pretty simple.

# Part 2

Solutions to next 10 levels can be found <a href="/playing-overthewire-bandit-2">here</a>.

# Part 3

Final part is <a href="/playing-overthewire-bandit-3">here</a>.