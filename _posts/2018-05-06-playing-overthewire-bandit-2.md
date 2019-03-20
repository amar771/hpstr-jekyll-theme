---
layout: post
title: "OverTheWire: Bandit Part 2"
description: "Continuing through the next 10 levels of Bandit wargame"
tags: [linux, security, overthewire, bandit]
image:
  path: /images/unsplash-3.jpg
  feature: unsplash-3.jpg
  credit: Hyunwon Jang on Unsplash
  creditlink: https://unsplash.com/photos/bIkRZwv7CZg
---

## Intro

Part 1 can be found <a href="/playing-overthewire-bandit-1">here</a>.

You can play bandit by yourself <a href="http://overthewire.org/wargames/bandit/">here</a>.

# Level 10 --> Level 11

Next password is located in file data.txt that is base64 encoded. File is located in home directory.

```shell
bandit10@bandit:~$ base64 -d data.txt
The password is IFukwKGsFW8MOq3IRFqrxE1hxTNEbUPR
bandit10@bandit:~$ exit
```

```base64``` is command for encoding and decoding data to/from base64. ```-d``` flag represents decoding.

```shell
ssh bandit11@bandit.labs.overthewire.org -p 2220
```

Done.

## Level 11 --> Level 12

Password is stored in data.txt but all alphabet characters have been rotated 13 positions, basically a rot13 (also known as Caesar cipher with key 13).

```shell
bandit11@bandit:~$ cat data.txt                     
Gur cnffjbeq vf 5Gr8L4qetPEsPk8htqjhRK8XSP6x2RHh
```

We need to shift everything 13 positions for this to be usable.

```shell
bandit11@bandit:~$ cat data.txt | tr "A-Za-z" "N-ZA-Mn-za-m"
The password is 5Te8Y4drgCRfCx8ugdwuEX8KFC6k2EUu
```

```tr``` command translates a character or a set of characters into other character or set. First part ```A-Za-z``` tells the tr command to take all characters in range A-Z and a-z and translate them into ```N-ZA-Mn-za-m```, this will create the following range:

NOPQRSTUVWXYZABCDEFGHIJKLM

And same with small characters. We start with N because that is the next character after 13th one ("M").

This could've been also done with a simple Python3 script

```shell
bandit11@bandit:~$ mkdir /tmp/bandit11
bandit11@bandit:~$ vim /tmp/bandit11/rot13
```
```python
import codecs
from sys import argv
with open(arg[1], "r") as file:
    to_decode = file.read()
    print(codecs.encode(to_decode, "rot13"), end='')
```
```shell

bandit11@bandit:~$ python3 /tmp/bandit11/rot13 ~/data.txt
The password is 5Te8Y4drgCRfCx8ugdwuEX8KFC6k2EUu
```

I've done it in Python3 because I prefer it, but it could've been done in any other language, ```tr``` is still probably the easiest option for this.

## Level 12 --> Level 13

The password is stored in file data.txt that is a hexdump which was repeatedly compressed. To make this easier to work with we'll create a directory in /tmp and copy the file there so we can work easier.

```shell
bandit12@bandit:~$ mkdir /tmp/bandit12
bandit12@bandit:~$ cd /tmp/bandit12
bandit12@bandit:/tmp/bandit12$ cp ~/data.txt .
```

```mkdir``` makes the directory we want, we make one in /tmp because we don't want it to be permanent. ```cd``` changes directory to the one we want to se we don't need to type full path all the time. ```cp``` copies the file we want where we want, ```~``` indicates home directory where our file is located in, and ```.``` indicates that we want to copy it in our current working directory (in this case ```/tmp/bandit12```).

```shell
bandit12@bandit:/tmp/bandit12$ xxd -r data.txt > data.zip
```

```xxd``` gives us a hexdump of a file, if we want to reverse it we use -r flag. We take the reversed output and put it into file called data.zip with > redirection.

```shell
bandit12@bandit:/tmp/bandit12$ file data.zip
data.zip: gzip compressed data, was "data2.bin"
```

I see here that file is gzip that was called data2.bin so I need to rename it properly to continue working.

```shell
bandit12@bandit:/tmp/bandit12$ mv data.zip data2.bin.gz
bandit12@bandit:/tmp/bandit12$ gunzip data2.bin.gz
```

```mv``` move command is used to rename the file into the proper format so gunzip can work with it. ```gunzip``` is used to unzip the files compressed with gzip.

```shell
bandit12@bandit:/tmp/bandit12$ file data2.bin
data2.bin: bzip2 compressed data, block size = 900k
bandit12@bandit:/tmp/bandit12$ mv data2.bin data2.bin.bz2
bandit12@bandit:/tmp/bandit12$ bunzip2 data2.bin.bz2
```

```bunzip2``` is used to unzip the files compressed with bz2.

```shell
bandit12@bandit:/tmp/bandit12$ file data2.bin
data2.bin: gzip compressed data, was "data4.bin"
bandit12@bandit:/tmp/bandit12$ mv data2.bin data4.bin.gz
bandit12@bandit:/tmp/bandit12$ gunzip data4.bin.gz
bandit12@bandit:/tmp/bandit12$ file data4.bin
data4.bin: POSIX tar archive (GNU)
```


After another unzipping I see that file is archived with tar next. Tar doesn't do any compression it just archives files/directories together so they can be compressed together while keeping their structure.

```shell
bandit12@bandit:/tmp/bandit12$ mv data4.bin data4.bin.tar
bandit12@bandit:/tmp/bandit12$ tar xvf data4.bin.tar
```

```tar``` is used for grouping files/directories together in an archive, ```xvf``` flags are used to unzip basically anything you can throw at it, it's really amazing and I use it really frequently.

```shell
bandit12@bandit:/tmp/bandit12$ file data5.bin
data5.bin: POSIX tar archive (GNU)
bandit12@bandit:/tmp/bandit12$ mv data5.bin data5.bin.tar
bandit12@bandit:/tmp/bandit12$ tar xvf data5.bin.tar
bandit12@bandit:/tmp/bandit12$ file data6.bin
data6.bin: bzip2 compressed data, block size = 900k
bandit12@bandit:/tmp/bandit12$ mv data6.bin data6.bin.bz2
bandit12@bandit:/tmp/bandit12$ bunzip2 data6.bin.bz2
bandit12@bandit:/tmp/bandit12$ file data6.bin
data6.bin: POSIX tar archive (GNU)
bandit12@bandit:/tmp/bandit12$ mv data6.bin data6.bin.tar
bandit12@bandit:/tmp/bandit12$ tar xvf data6.bin.tar
bandit12@bandit:/tmp/bandit12$ file data8.bin
data8.bin: gzip compressed data, was "data9.bin"
bandit12@bandit:/tmp/bandit12$ mv data8.bin data9.bin.gz
bandit12@bandit:/tmp/bandit12$ gunzip data9.bin.gz
bandit12@bandit:/tmp/bandit12$ file data9.bin
data9.bin: ASCII text
```

And finally...

```shell
bandit12@bandit:/tmp/bandit12$ cat data9.bin
The password is 8ZjyCRiBWFYkneahHwxCv3wb2a1ORpYL
bandit12@bandit:/tmp/bandit12$ exit
ssh bandit13@bandit.labs.overthewire.org -p 2220
```

Works. This was a bit tedious to do but still relatively easy.

## Level 13 --> Level 14

For this level we don't get a password but private sshkey for next level.

```shell
bandit13@bandit:~$ ls
sshkey.private
bandit13@bandit:~$ exit
scp -P 2220 bandit13@bandit.labs.overthewire.org:~/sshkey.private .
```

This will copy the file from server to our local machine and than we just need to add it to our keys.

```shell
mv sshkey.private ~/.ssh
```

```~/.ssh``` is a default location for ssh keys so we move the ssh key there.

```shell
cd .ssh
mv sshkey.private id_rsa_bandit13
```

I renamed it to follow my naming standard for keys (starting with id_rsa).

```shell
echo "IdentityFile ~/.ssh/id_rsa_bandit13" >> config
```

Since I have a lot of keys I store them in config file, this is an easy way to append the keys to the end of the file.

```shell
sudo chmod 600 id_rsa_bandit13
```

We have to remove all unnecessary permissions for the key for it to work and we use chmod command for that. 600 means that only the owner has the permission to read and modify the file.

```shell
cd
ssh bandit14@bandit.labs.overthewire.org -p 2220
```

And password wasn't needed to connect.

## Level 14 --> Level 15

Password can be retrieved by submitting the password of the current level to port 30000 on localhost.

First we need to get the current password, they're located in ```/etc/bandit_pass/```

```shell
bandit14@bandit:~$ cat /etc/bandit_pass/bandit14
4wcYUJFw0k0XLShlDzztnTBHiqxU3b3e
bandit14@bandit:~$ exit
```

Now we can remove the ssh key for bandit13 since we won't use it anymore.

```shell
vim .ssh/config
```

Remove the key.

```shell
rm .ssh/id_rsa_bandit13
ssh bandit14@bandit.labs.overthewire.org -p 2220
```

Now we need to send the password to port 30000:

```shell
bandit14@bandit:~$ nc 127.0.0.1 30000
4wcYUJFw0k0XLShlDzztnTBHiqxU3b3e
Correct!
BfMYroe26WYalil77FoDi9qh59eK5xNr

bandit14@bandit:~$ exit
ssh bandit15@bandit.labs.overthewire.org -p 2220
```

This one is straightforward, we used ```nc``` (netcat) to open connection on localhost (127.0.0.1) on port 30000, and then we sent current password and got the reply of next password.

## Level 15 --> Level 16

Submit the current password on port 30001 on localhost but using SSL encryption this thime.

```shell
bandit15@bandit:~$ openssl s_client -connect 127.0.0.1:30001 -ign_eof
BfMYroe26WYalil77FoDi9qh59eK5xNr
Correct!
cluFn7wTiGryunymYOu4RcffSxQluehd
```

To open the SSL connection we use ```openssl s_client``` command with ```-connect``` flag to say who we're connecting to ```localhost/127.0.0.1``` and on which port 30001 [host:port].

We also have to add ```-ign_eof``` because the password we're sending begins with B, and anything that begins with B, R or Q will close the connection because they're special characters.

```shell
bandit15@bandit:~$ exit
ssh bandit16@bandit.labs.overthewire.org -p 2220
```

Done.

## Level 16 --> Level 17

Password for next level is located on port on localhost in the range 31000 to 32000 and it speaks only SSL. We have to find the right port to get the password.

```shell
bandit16@bandit:~$ nmap -p 31000-32000 127.0.0.1
PORT      STATE SERVICE
31046/tcp open  unknown
31518/tcp open  unknown
31691/tcp open  unknown
31790/tcp open  unknown
31960/tcp open  unknown
```

```nmap``` is network exploration tool and security/port scanner. In this case we used it with -p flag to specify port range we want to scan.

We didn't get that many open ports so we can check them all manually, but we can also run -sV flag on nmap to check running services on ports. Since -sV is slow we want to reduce the amount of services we're scanning as much as we can. SSL is the service we're searching for and it's in lowest intensity (1) so we can say ```--version-intensity 1``` to speed up the process as much as possible.

I'm gonna do it one by one so I get clearer output, if I had to search a lot of ports I would've set a range and redirect the output to some file and later check the file.

```shell
bandit16@bandit:~$ nmap -sV --version-intensity 1 -p 31046 127.0.0.1
31046/tcp open  unknown
```

And 1 unrecognized service, probably not this one.

```shell
bandit16@bandit:~$ nmap -sV --version-intensity 1 -p 31518 127.0.0.1
31518/tcp open  ssl/unknown
```

Looks like this is the one. Let's test it out.

```shell
bandit16@bandit:~$ openssl s_client -connect 127.0.0.1:31518 -ign_eof
cluFn7wTiGryunymYOu4RcffSxQluehd
cluFn7wTiGryunymYOu4RcffSxQluehd
```

It returned the same password as previous one, this isn't it.
Checking the next one.

```shell
bandit16@bandit:~$ nmap -sV --version-intensity 1 -p 31691 127.0.0.1
31691/tcp open  unknown
bandit16@bandit:~$ nmap -sV --version-intensity 1 -p 31691 127.0.0.1
31790/tcp open  ssl/unknown
```

Another one that speaks SSL, let's check it out:

```shell
bandit16@bandit:~$ openssl s_client -connect 127.0.0.1:31790 -ign_eof
cluFn7wTiGryunymYOu4RcffSxQluehd
Correct!
```

And I got RSA key for next level, awesome.

```shell
bandit16@bandit:~$ exit
vim .ssh/id_rsa_bandit17
echo "IdentityFile ~/.ssh/id_rsa_bandit17" >> .ssh/config
sudo chmod 600 .ssh/id_rsa_three
ssh bandit17@bandit.labs.overthewire.org -p 2220
```

Works. Let's get the password for this level from ```/etc/```

```shell
bandit17@bandit:~$ cat /etc/bandit_pass/bandit17
xLYVMN9WE5zQ5vHacb0sZEVqbrp7nBTn
```

I can now remove the ssh key for bandit17 since I don't plan to use it anymore.

## Level 17 --> Level 18

There are two files, passwords.old and passwords.new in home directory of user bandit17. The password is the only line that is different between two files.

```shell
bandit17@bandit:~$ diff passwords.old passwords.new 
42c42
< 6vcSC74ROI95NqkKaeEC2ABVMDX9TyUr
---
> kfBf3eYk5BPBRzwjqutbbfE887SVc5Yd
```

```kfBf3eYk5BPBRzwjqutbbfE887SVc5Yd``` is found in the new file and not in the old one so it's probably the password.

```shell
bandit17@bandit:~$ exit
ssh bandit18@bandit.labs.overthewire.org -p 2220
Byebye !
```

Our connection closed with byebye message, that is the problem of the next level, the password works.

## Level 18 --> Level 19

Password is stored in the readme file in the home directory of user bandit18, but we can't access it since someone modified .bashrc to log us out when we try to log in with SSH.

Since I know where the file is located I can scp it over to my local machine like:

```shell
scp -P 2220 bandit18@bandit.labs.overthewire.org:~/readme .
cat readme
IueksS7Ubh8G3DCwVzrTd8rAVOwq3M5x
```

Or I could execute commands over SSH like:

```shell
ssh bandit18@bandit.labs.overthewire.org -p 2220 ls
readme
ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"
IueksS7Ubh8G3DCwVzrTd8rAVOwq3M5x
ssh bandit19@bandit.labs.overthewire.org -p 2220
```

Done.

## Level 19 --> Level 20

To gain access to the next level, I need to use the setuid binary in the home directory of user bandit19. I need to figure out how it works and use it to gain access to password in ```/etc/bandit_pass```.

```shell
bandit19@bandit:~$ ls
bandit20-do
bandit19@bandit:~$ ./bandit20-do
Run a command as another user.
    Example: ./bandit20-do id
```

So I can use it to run commands as bandit20 user.

```shell
bandit19@bandit:~$ ./bandit20-do cat /etc/bandit_pass/bandit20
GbKksEFF4yrVs6il55v6gwY5aVje5f0j
bandit19@bandit:~$ exit
ssh bandit20@bandit.labs.overthewire.org -p 2220
```

And it works.

# Part 3

Final part is <a href="/playing-overthewire-bandit-3">here</a>