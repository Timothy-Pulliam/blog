---
layout: "../../layouts/BlogPost.astro"
title: "Over the Wire - Bandit"
description: "Writeup for Over the Wire - Bandit"
pubDate: "Dec 30 2023"
heroImage: "/capture-the-flag.jpg"
previewText: "Writeup for Bandit Over the Wire"
---

## bandit0

Start the game by SSHing into the server with password `bandit0`

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

The password is `bandit0`

## bandit0 -> bandit1

cat the readme file to get the flag

```bash
bandit0@bandit:~$ ls
readme
bandit0@bandit:~$ cat readme
NH2SXQwcBdpmTEzi3bvBHMM9H66vVXjL
```

To get to the next level, close your SSH connection and re-connect using the password from the readme file

```
exit
ssh bandit1@bandit.labs.overthewire.org -p 2220
```

## bandit1 -> bandit2

`ls` shows a file named `-`, which is usually reserved as a special character in BASH to indicate `/dev/stdin`. If we try to run `cat -` then `cat` will simply read from stdin. However, we can prevent this by prefixing the filename with a dot slash.

```bash
bandit1@bandit:~$ ls
-
bandit1@bandit:~$ cat ./-
rRGizSaX8Mk1RTb1CNQoXTcYZWU6lgzi
```

## bandit2 -> bandit3

Now there is a filename with spaces. Simply type `cat spaces` and hit TAB. BASH autocomplete will do the rest.

```bash
bandit2@bandit:~$ cat spaces\ in\ this\ filename
aBZ0W5EmUfAf7kHTQeOwd8bauFJ2lAiG
```

## bandit3 -> bandit4

In linux, files that start with a `.` are hidden from the user. Using `ls -a` shows these hidden files.

```bash
bandit3@bandit:~$ ls
inhere
bandit3@bandit:~$ cd inhere/
bandit3@bandit:~/inhere$ ls
bandit3@bandit:~/inhere$ ls -a
.  ..  .hidden
bandit3@bandit:~/inhere$ cat .hidden
2EW7BBsr6aMMoJ2HjW067dm8EgX26xNe
```

## bandit4 -> bandit5

```bash
bandit4@bandit:~$ ls
inhere
bandit4@bandit:~$ cd inhere/
bandit4@bandit:~/inhere$ ls
-file00  -file02  -file04  -file06  -file08
-file01  -file03  -file05  -file07  -file09
```

Same as in the previous level, we need to prefix each filename with `./`, as `-f` can be interpreted as a BASH command flag.

We are given the following hint

> The password for the next level is stored in the only human-readable file in the inhere directory. Tip: if your terminal is messed up, try the “reset” command.

```bash
bandit4@bandit:~/inhere$ file ./*
./-file00: data
./-file01: data
./-file02: data
./-file03: data
./-file04: data
./-file05: data
./-file06: data
./-file07: ASCII text
./-file08: data
./-file09: data
```

We can see there is only one file containing human readable text. The rest contain binary data, which is not human readable.

```bash
bandit4@bandit:~/inhere$ cat ./-file07
lrIWWI6bB37kxfiCQZqUdOIYfr6eEeqR
```

## bandit5 -> bandit6

```bash
bandit5@bandit:~$ ls
inhere
bandit5@bandit:~$ cd inhere/
bandit5@bandit:~/inhere$ ls
maybehere00  maybehere02  maybehere04  maybehere06  maybehere08  maybehere10  maybehere12  maybehere14  maybehere16  maybehere18
maybehere01  maybehere03  maybehere05  maybehere07  maybehere09  maybehere11  maybehere13  maybehere15  maybehere17  maybehere19
```

There are many directories, and we don't want have to look in them all.

We are given the hint

> The password for the next level is stored in a file somewhere under the inhere directory and has all of the following properties:
>
> human-readable
> 1033 bytes in size
> not executable

We can use these conditions in the `find` command

```bash
bandit5@bandit:~/inhere$ find ./ -readable -size 1033c -not -executable
./maybehere07/.file2
bandit5@bandit:~/inhere$ cat ./maybehere07/.file2
P4L4vucdmLnm8I7Vl7jG1ApGSfjYKqJU
```

## bandit6 -> bandit7

We are given the following hint

> The password for the next level is stored somewhere on the server and has all of the following properties:
>
> owned by user bandit7
> owned by group bandit6
> 33 bytes in size

Again, we use the find command, this time we search from the root of the filesystem, redirecting `/dev/stderr` to `/dev/null` to silence annoying permission denied warnings.

```bash
bandit6@bandit:~$ find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
/var/lib/dpkg/info/bandit7.password
bandit6@bandit:~$ cat /var/lib/dpkg/info/bandit7.password
z7WtoNQU2XfjmMtWA8u5rN4vzqu4v99S
```

## bandit7 -> bandit8

We are given the following hint

> The password for the next level is stored in the file data.txt next to the word millionth

We use `grep` to search for a line containing the string `millionth`

```bash
bandit7@bandit:~$ grep millionth data.txt
millionth	TESKZC0XvTetK0S9xNwm25STk5iWrBvP
```

## bandit8 -> bandit9

We are given the following hint

> The password for the next level is stored in the file data.txt and is the only line of text that occurs only once

First, we will sort the file so that identical lines are adjacent, then we will filter out duplicate lines.

```bash
bandit8@bandit:~$ sort data.txt | uniq -u
EN632PlfYiZbn3PhVK3XOGSlNInNE00t
```

## Bandit Level 9 → Level 10

We are given the following hint

> The password for the next level is stored in the file data.txt in one of the few human-readable strings, preceded by several ‘=’ characters.

```bash
bandit9@bandit:~$ file data.txt
data.txt: data
```

The file contains binary, non human-readable data. If we try to cat it out, it will spit out garbage and could mess up our terminal. To view the contents, we use the `xxd` command to do a hexdump of the file. We know we are looking for lines containing multiple `=` characters, so we `grep` for them.

```bash
bandit9@bandit:~$ xxd data.txt | grep ====
00000460: 785d 543d 3d3d 3d3d 3d3d 3d3d 3d20 7468  x]T========== th
000014b0: 3d3d 3d3d 3d3d 3d20 7061 7373 776f 7264  ======= password
00001b40: c8ff 3d3d 3d3d 3d3d 3d3d 3d3d 2069 73ba  ..========== is.
00003f40: 3d3d 3d3d 3d3d 3d3d 2047 3777 384c 4969  ======== G7w8LIi
```

`3d` is hexidecimal for the `=` character.

It looks like part of the password is on a different line and is getting cut off. To get the next couple of lines, we can tell grep to show us the next two lines after the matched line.

```bash
bandit9@bandit:~$ xxd data.txt | grep -A 2 ===
00000460: 785d 543d 3d3d 3d3d 3d3d 3d3d 3d20 7468  x]T========== th
00000470: 6547 2922 e929 3cd7 fd08 2147 ec80 f77b  eG)".)<...!G...{
00000480: 13a1 df58 38f6 3f13 8ad4 359d 26e3 c9f9  ...X8.?...5.&...
--
000014a0: a1a7 4f5b 970e b936 fe5e 4653 bf3d 3d3d  ..O[...6.^FS.===
000014b0: 3d3d 3d3d 3d3d 3d20 7061 7373 776f 7264  ======= password
000014c0: 6b5e a3d9 bc96 e395 cada 0031 3e9c f0d4  k^.........1>...
000014d0: 6a67 801a 53ee 3e1b 5106 9aca f2ce d335  jg..S.>.Q......5
--
00001b40: c8ff 3d3d 3d3d 3d3d 3d3d 3d3d 2069 73ba  ..========== is.
00001b50: 770b 4b1a 65cf 0840 ce26 1d2c 5d44 f1f6  w.K.e..@.&.,]D..
00001b60: 8f25 8002 76bc 0d6f 3047 9194 2cc4 b135  .%..v..o0G..,..5
--
00003f40: 3d3d 3d3d 3d3d 3d3d 2047 3777 384c 4969  ======== G7w8LIi
00003f50: 364a 336b 5462 3841 376a 394c 6772 7977  6J3kTb8A7j9Lgryw
00003f60: 7445 556c 7979 7036 730a c78e 859d 6105  tEUlyyp6s.....a.
```

We see the password is `G7w8LIi6J3kTb8A7j9LgrywtEUlyyp6s`

## Bandit Level 10 → Level 11

We are given the following hint

> The password for the next level is stored in the file data.txt, which contains base64 encoded data

```bash
bandit10@bandit:~$ ls
data.txt
bandit10@bandit:~$ file data.txt
data.txt: ASCII text
bandit10@bandit:~$ cat data.txt
VGhlIHBhc3N3b3JkIGlzIDZ6UGV6aUxkUjJSS05kTllGTmI2blZDS3pwaGxYSEJNCg==
```

This is indeed base64 encoded data (note the trailing `==` to pad out the length of the string). We can easily decode

```bash
bandit10@bandit:~$ cat data.txt | base64 -d
The password is 6zPeziLdR2RKNdNYFNb6nVCKzphlXHBM
```

## Bandit Level 11 → Level 12

We are given the following hint

> The password for the next level is stored in the file data.txt, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions

[rot13](https://en.wikipedia.org/wiki/ROT13) is a substitution cipher and a special case of the Ceaser Cipher where each letter is replaced with the the 13th letter after it. Since the english alphabet has 26 letters, rot13 cipher is it's own inverse, and applying the transformation twice will yield the original text.

We can use an [online rot13 application](https://rot13.com/), or we can implement rot13 using the `tr` command.

```bash
bandit11@bandit:~$ cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
The password is JVNBBFSmZwKKOP0XbFXOoW8chDz5yVRv
```

## Bandit Level 12 → Level 13

We are given the following hint

> The password for the next level is stored in the file data.txt, which is a hexdump of a file that has been repeatedly compressed. For this level it may be useful to create a directory under /tmp in which you can work using mkdir. For example: mkdir /tmp/myname123. Then copy the datafile using cp, and rename it using mv (read the manpages!)

We need to undo these operations. First we will undo the hexdump, then we will repeatedly uncompress the file, until we have something resembling a password.

```bash
bandit12@bandit:~$ mkdir /tmp/tim
bandit12@bandit:~$ xxd -r data.txt > /tmp/tim/data.txt
bandit12@bandit:~$ cd /tmp/tim
bandit12@bandit:/tmp/tim$ file data.txt
data.txt: gzip compressed data, was "data2.bin", last modified: Thu Oct  5 06:19:20 2023, max compression, from Unix, original size modulo 2^32 573
```

It looks like the file has been compressed with `gzip` which is a GNU tool. In order to unzip it, we need to add the `.gz` extension to the file and use `gunzip`

```bash
bandit12@bandit:/tmp/tim$ mv data.txt data.txt.gz
bandit12@bandit:/tmp/tim$ gunzip data.txt.gz
bandit12@bandit:/tmp/tim$ file data.txt
data.txt: bzip2 compressed data, block size = 900k
```

Upon inspection of the new file, it now appears it has been compressed with `bzip`, another compression tool. We will need to repeat the previous step, but add the `.bz2` extension to the file.

```bash
bandit12@bandit:/tmp/tim$ mv data.txt data.txt.bz
bandit12@bandit:/tmp/tim$ bunzip2 -d data.txt.bz
bandit12@bandit:/tmp/tim$ file data.txt
data.txt: gzip compressed data, was "data4.bin", last modified: Thu Oct  5 06:19:20 2023, max compression, from Unix, original size modulo 2^32 20480
```

Another `gzip` compressed file. It's turtles all the way down

```bash
bandit12@bandit:/tmp/tim$ mv data.txt data.txt.gz
bandit12@bandit:/tmp/tim$ gunzip data.txt.gz
bandit12@bandit:/tmp/tim$ file data.txt
data.txt: POSIX tar archive (GNU)
```

Now we have a [tar archive](https://xkcd.com/1168/). Let's untar it and hope it doesn't explode.

```bash
bandit12@bandit:/tmp/tim$ mv data.txt data.txt.tar
bandit12@bandit:/tmp/tim$ tar xvf data.txt.tar
data5.bin
bandit12@bandit:/tmp/tim$ file data5.bin
data5.bin: POSIX tar archive (GNU)
```

Another tar archive :|

Let's speed this up

```bash
bandit12@bandit:/tmp/tim$ mv data5.bin data5.bin.tar
bandit12@bandit:/tmp/tim$ tar xvf data5.bin.tar
data6.bin
bandit12@bandit:/tmp/tim$ file data6.bin
data6.bin: bzip2 compressed data, block size = 900k
bandit12@bandit:/tmp/tim$ mv data6.bin data6.bin.bz
bandit12@bandit:/tmp/tim$ bunzip2 -d data6.bin.bz
bandit12@bandit:/tmp/tim$ file data6.bin
data6.bin: POSIX tar archive (GNU)
bandit12@bandit:/tmp/tim$ mv data6.bin data6.bin.tar
bandit12@bandit:/tmp/tim$ tar xvf data6.bin.tar
data8.bin
bandit12@bandit:/tmp/tim$ file data8.bin
data8.bin: gzip compressed data, was "data9.bin", last modified: Thu Oct  5 06:19:20 2023, max compression, from Unix, original size modulo 2^32 49
bandit12@bandit:/tmp/tim$ mv data8.bin data8.bin.gz
bandit12@bandit:/tmp/tim$ gunzip data8.bin.gz
bandit12@bandit:/tmp/tim$ file data8.bin
data8.bin: ASCII text
bandit12@bandit:/tmp/tim$ cat data8.bin
The password is wbWdlBxEir4CaE8LaPhauuOo6pwRmrDw
```

## Bandit Level 13 → Level 14

We are given the following hint

> The password for the next level is stored in /etc/bandit_pass/bandit14 and can only be read by user bandit14. For this level, you don’t get the next password, but you get a private SSH key that can be used to log into the next level. Note: localhost is a hostname that refers to the machine you are working on

All we need to do is use the ssh key to log in to the next level

```bash
bandit13@bandit:~$ ssh -i sshkey.private bandit14@localhost -p 2220
The authenticity of host '[localhost]:2220 ([127.0.0.1]:2220)' can't be established.
ED25519 key fingerprint is SHA256:C2ihUBV7ihnV1wUXRb4RrEcLfXC5CXlhmAAM/urerLY.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

## Bandit Level 14 → Level 15

We are given the following hint

> The password for the next level can be retrieved by submitting the password of the current level to port 30000 on localhost.

We know from the previous level hint, the password for this level is located at `/etc/bandit_pass/bandit14`. We can telnet to localhost on port 30000 on to submit the password.

```bash
bandit14@bandit:~$ cat /etc/bandit_pass/bandit14
fGrHPx402xGC7U7rXKDaxiWFTOiF0ENq
bandit14@bandit:~$ telnet localhost 30000
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
fGrHPx402xGC7U7rXKDaxiWFTOiF0ENq
Correct!
jN2kgmIXJ6fShzhT2avhotn4Zcka6tnt

Connection closed by foreign host.
```

## Bandit Level 15 → Level 16

We are given the following hint

> The password for the next level can be retrieved by submitting the password of the current level to port 30001 on localhost using SSL encryption.
>
> Helpful note: Getting “HEARTBEATING” and “Read R BLOCK”? Use -ign_eof and read the “CONNECTED COMMANDS” section in the manpage. Next to ‘R’ and ‘Q’, the ‘B’ command also works in this version of that command…

For SSL connections, we use the `openssl` command.

```bash
bandit15@bandit:~$ openssl s_client -port 30001
CONNECTED(00000003)
depth=0 CN = localhost
verify error:num=18:self-signed certificate
verify return:1
depth=0 CN = localhost
verify error:num=10:certificate has expired
...
...
...
---
read R BLOCK
jN2kgmIXJ6fShzhT2avhotn4Zcka6tnt
Correct!
JQttfApK4SeyHwDlI9SXGR50qclOAil1

closed
```

## Bandit Level 16 → Level 17

We are given the following hint

> The credentials for the next level can be retrieved by submitting the password of the current level to a port on localhost in the range 31000 to 32000. First find out which of these ports have a server listening on them. Then find out which of those speak SSL and which don’t. There is only 1 server that will give the next credentials, the others will simply send back to you whatever you send to it.

Let's use namp to scan ports in the range of 31000-32000 to see what ports are open

```bash
bandit16@bandit:~$ nmap -p 31000-32000 localhost
Starting Nmap 7.80 ( https://nmap.org ) at 2023-12-22 03:19 UTC
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00018s latency).
Not shown: 996 closed ports
PORT      STATE SERVICE
31046/tcp open  unknown
31518/tcp open  unknown
31691/tcp open  unknown
31790/tcp open  unknown
31960/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 0.09 seconds
```

This narrows things down a bit. We need to find out which one uses an SSL connection.

We can use nmap again to see which ports offer an SSL connection

```bash
bandit16@bandit:~$ nmap -p 31000-32000 --script ssl-enum-ciphers localhost
Starting Nmap 7.80 ( https://nmap.org ) at 2023-12-22 03:22 UTC
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00014s latency).
Not shown: 996 closed ports
PORT      STATE SERVICE
31046/tcp open  unknown
31518/tcp open  unknown
| ssl-enum-ciphers:
|   TLSv1.2:
|     ciphers:
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_ARIA_128_GCM_SHA256 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_ARIA_256_GCM_SHA384 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_CAMELLIA_128_CBC_SHA256 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_CAMELLIA_256_CBC_SHA384 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256 (secp256r1) - A
|       TLS_RSA_WITH_AES_128_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_AES_128_CBC_SHA256 (rsa 2048) - A
|       TLS_RSA_WITH_AES_128_CCM (rsa 2048) - A
|       TLS_RSA_WITH_AES_128_CCM_8 (rsa 2048) - A
|       TLS_RSA_WITH_AES_128_GCM_SHA256 (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_CBC_SHA256 (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_CCM (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_CCM_8 (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_GCM_SHA384 (rsa 2048) - A
|       TLS_RSA_WITH_ARIA_128_GCM_SHA256 (rsa 2048) - A
|       TLS_RSA_WITH_ARIA_256_GCM_SHA384 (rsa 2048) - A
|       TLS_RSA_WITH_CAMELLIA_128_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_CAMELLIA_128_CBC_SHA256 (rsa 2048) - A
|       TLS_RSA_WITH_CAMELLIA_256_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_CAMELLIA_256_CBC_SHA256 (rsa 2048) - A
|     compressors:
|       NULL
|     cipher preference: client
|     warnings:
|       Weak certificate signature: SHA1
|_  least strength: A
31691/tcp open  unknown
31790/tcp open  unknown
| ssl-enum-ciphers:
|   TLSv1.2:
|     ciphers:
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_ARIA_128_GCM_SHA256 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_ARIA_256_GCM_SHA384 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_CAMELLIA_128_CBC_SHA256 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_CAMELLIA_256_CBC_SHA384 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256 (secp256r1) - A
|       TLS_RSA_WITH_AES_128_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_AES_128_CBC_SHA256 (rsa 2048) - A
|       TLS_RSA_WITH_AES_128_CCM (rsa 2048) - A
|       TLS_RSA_WITH_AES_128_CCM_8 (rsa 2048) - A
|       TLS_RSA_WITH_AES_128_GCM_SHA256 (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_CBC_SHA256 (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_CCM (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_CCM_8 (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_GCM_SHA384 (rsa 2048) - A
|       TLS_RSA_WITH_ARIA_128_GCM_SHA256 (rsa 2048) - A
|       TLS_RSA_WITH_ARIA_256_GCM_SHA384 (rsa 2048) - A
|       TLS_RSA_WITH_CAMELLIA_128_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_CAMELLIA_128_CBC_SHA256 (rsa 2048) - A
|       TLS_RSA_WITH_CAMELLIA_256_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_CAMELLIA_256_CBC_SHA256 (rsa 2048) - A
|     compressors:
|       NULL
|     cipher preference: client
|     warnings:
|       Weak certificate signature: SHA1
|_  least strength: A
31960/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 2.50 seconds
```

Looks like port 31518 and 31790 offer SSL.
Recall, we need to submit the password for the current level. Let's guess which one to connect to

```bash
bandit16@bandit:~$ cat /etc/bandit_pass/bandit16
JQttfApK4SeyHwDlI9SXGR50qclOAil1
```

```bash
bandit16@bandit:~$ openssl s_client -port 31518
CONNECTED(00000003)
depth=0 CN = localhost
verify error:num=18:self-signed certificate
verify return:1
depth=0 CN = localhost
...
...
...
---
read R BLOCK
JQttfApK4SeyHwDlI9SXGR50qclOAil1
JQttfApK4SeyHwDlI9SXGR50qclOAil1
```

Nope, not this one. Let's try the other port

```bash
bandit16@bandit:~$ openssl s_client -port 31790
CONNECTED(00000003)
depth=0 CN = localhost
verify error:num=18:self-signed certificate
verify return:1
depth=0 CN = localhost
...
...
...
---
read R BLOCK
JQttfApK4SeyHwDlI9SXGR50qclOAil1
Correct!
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAvmOkuifmMg6HL2YPIOjon6iWfbp7c3jx34YkYWqUH57SUdyJ
imZzeyGC0gtZPGujUSxiJSWI/oTqexh+cAMTSMlOJf7+BrJObArnxd9Y7YT2bRPQ
Ja6Lzb558YW3FZl87ORiO+rW4LCDCNd2lUvLE/GL2GWyuKN0K5iCd5TbtJzEkQTu
DSt2mcNn4rhAL+JFr56o4T6z8WWAW18BR6yGrMq7Q/kALHYW3OekePQAzL0VUYbW
JGTi65CxbCnzc/w4+mqQyvmzpWtMAzJTzAzQxNbkR2MBGySxDLrjg0LWN6sK7wNX
x0YVztz/zbIkPjfkU1jHS+9EbVNj+D1XFOJuaQIDAQABAoIBABagpxpM1aoLWfvD
KHcj10nqcoBc4oE11aFYQwik7xfW+24pRNuDE6SFthOar69jp5RlLwD1NhPx3iBl
J9nOM8OJ0VToum43UOS8YxF8WwhXriYGnc1sskbwpXOUDc9uX4+UESzH22P29ovd
d8WErY0gPxun8pbJLmxkAtWNhpMvfe0050vk9TL5wqbu9AlbssgTcCXkMQnPw9nC
YNN6DDP2lbcBrvgT9YCNL6C+ZKufD52yOQ9qOkwFTEQpjtF4uNtJom+asvlpmS8A
vLY9r60wYSvmZhNqBUrj7lyCtXMIu1kkd4w7F77k+DjHoAXyxcUp1DGL51sOmama
+TOWWgECgYEA8JtPxP0GRJ+IQkX262jM3dEIkza8ky5moIwUqYdsx0NxHgRRhORT
8c8hAuRBb2G82so8vUHk/fur85OEfc9TncnCY2crpoqsghifKLxrLgtT+qDpfZnx
SatLdt8GfQ85yA7hnWWJ2MxF3NaeSDm75Lsm+tBbAiyc9P2jGRNtMSkCgYEAypHd
HCctNi/FwjulhttFx/rHYKhLidZDFYeiE/v45bN4yFm8x7R/b0iE7KaszX+Exdvt
SghaTdcG0Knyw1bpJVyusavPzpaJMjdJ6tcFhVAbAjm7enCIvGCSx+X3l5SiWg0A
R57hJglezIiVjv3aGwHwvlZvtszK6zV6oXFAu0ECgYAbjo46T4hyP5tJi93V5HDi
Ttiek7xRVxUl+iU7rWkGAXFpMLFteQEsRr7PJ/lemmEY5eTDAFMLy9FL2m9oQWCg
R8VdwSk8r9FGLS+9aKcV5PI/WEKlwgXinB3OhYimtiG2Cg5JCqIZFHxD6MjEGOiu
L8ktHMPvodBwNsSBULpG0QKBgBAplTfC1HOnWiMGOU3KPwYWt0O6CdTkmJOmL8Ni
blh9elyZ9FsGxsgtRBXRsqXuz7wtsQAgLHxbdLq/ZJQ7YfzOKU4ZxEnabvXnvWkU
YOdjHdSOoKvDQNWu6ucyLRAWFuISeXw9a/9p7ftpxm0TSgyvmfLF2MIAEwyzRqaM
77pBAoGAMmjmIJdjp+Ez8duyn3ieo36yrttF5NSsJLAbxFpdlc1gvtGCWW+9Cq0b
dxviW8+TFVEBl1O4f7HVm6EpTscdDxU+bCXWkfjuRb7Dy9GOtt9JPsX8MBTakzh3
vBgsyi/sN3RqRBcGU40fOoZyfAMT8s1m/uYv52O6IgeuZ/ujbjY=
-----END RSA PRIVATE KEY-----

closed
```

Bingo. Instead of getting a password, we are given an SSH private key file. We can save this key to a file and use it to SSH to the next level. We aren't able to write to bandit16 home directory, so let's make a directory in `/tmp` and copy it there

```bash
bandit16@bandit:~$ mkdir /tmp/bandit16
bandit16@bandit:~$ echo '-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAvmOkuifmMg6HL2YPIOjon6iWfbp7c3jx34YkYWqUH57SUdyJ
imZzeyGC0gtZPGujUSxiJSWI/oTqexh+cAMTSMlOJf7+BrJObArnxd9Y7YT2bRPQ
Ja6Lzb558YW3FZl87ORiO+rW4LCDCNd2lUvLE/GL2GWyuKN0K5iCd5TbtJzEkQTu
DSt2mcNn4rhAL+JFr56o4T6z8WWAW18BR6yGrMq7Q/kALHYW3OekePQAzL0VUYbW
JGTi65CxbCnzc/w4+mqQyvmzpWtMAzJTzAzQxNbkR2MBGySxDLrjg0LWN6sK7wNX
x0YVztz/zbIkPjfkU1jHS+9EbVNj+D1XFOJuaQIDAQABAoIBABagpxpM1aoLWfvD
KHcj10nqcoBc4oE11aFYQwik7xfW+24pRNuDE6SFthOar69jp5RlLwD1NhPx3iBl
J9nOM8OJ0VToum43UOS8YxF8WwhXriYGnc1sskbwpXOUDc9uX4+UESzH22P29ovd
d8WErY0gPxun8pbJLmxkAtWNhpMvfe0050vk9TL5wqbu9AlbssgTcCXkMQnPw9nC
YNN6DDP2lbcBrvgT9YCNL6C+ZKufD52yOQ9qOkwFTEQpjtF4uNtJom+asvlpmS8A
vLY9r60wYSvmZhNqBUrj7lyCtXMIu1kkd4w7F77k+DjHoAXyxcUp1DGL51sOmama
+TOWWgECgYEA8JtPxP0GRJ+IQkX262jM3dEIkza8ky5moIwUqYdsx0NxHgRRhORT
8c8hAuRBb2G82so8vUHk/fur85OEfc9TncnCY2crpoqsghifKLxrLgtT+qDpfZnx
SatLdt8GfQ85yA7hnWWJ2MxF3NaeSDm75Lsm+tBbAiyc9P2jGRNtMSkCgYEAypHd
HCctNi/FwjulhttFx/rHYKhLidZDFYeiE/v45bN4yFm8x7R/b0iE7KaszX+Exdvt
SghaTdcG0Knyw1bpJVyusavPzpaJMjdJ6tcFhVAbAjm7enCIvGCSx+X3l5SiWg0A
R57hJglezIiVjv3aGwHwvlZvtszK6zV6oXFAu0ECgYAbjo46T4hyP5tJi93V5HDi
Ttiek7xRVxUl+iU7rWkGAXFpMLFteQEsRr7PJ/lemmEY5eTDAFMLy9FL2m9oQWCg
R8VdwSk8r9FGLS+9aKcV5PI/WEKlwgXinB3OhYimtiG2Cg5JCqIZFHxD6MjEGOiu
L8ktHMPvodBwNsSBULpG0QKBgBAplTfC1HOnWiMGOU3KPwYWt0O6CdTkmJOmL8Ni
blh9elyZ9FsGxsgtRBXRsqXuz7wtsQAgLHxbdLq/ZJQ7YfzOKU4ZxEnabvXnvWkU
YOdjHdSOoKvDQNWu6ucyLRAWFuISeXw9a/9p7ftpxm0TSgyvmfLF2MIAEwyzRqaM
77pBAoGAMmjmIJdjp+Ez8duyn3ieo36yrttF5NSsJLAbxFpdlc1gvtGCWW+9Cq0b
dxviW8+TFVEBl1O4f7HVm6EpTscdDxU+bCXWkfjuRb7Dy9GOtt9JPsX8MBTakzh3
vBgsyi/sN3RqRBcGU40fOoZyfAMT8s1m/uYv52O6IgeuZ/ujbjY=
-----END RSA PRIVATE KEY-----' > /tmp/bandit16/privatekey
```

In order to use the private key, we must also set the correct permissions on the file, otherwise we will get the following error

```bash
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0664 for '/tmp/bandit16/privatekey' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "/tmp/bandit16/privatekey": bad permissions
bandit17@localhost: Permission denied (publickey).
```

so let's set the correct permissions

```bash
bandit16@bandit:~$ chmod 0600 /tmp/bandit16/privatekey
bandit16@bandit:~$ ssh -i /tmp/bandit16/privatekey bandit17@localhost -p 2220
```

## Bandit Level 16 → Level 17

We are given the following hint

> There are 2 files in the homedirectory: passwords.old and passwords.new. The password for the next level is in passwords.new and is the only line that has been changed between passwords.old and passwords.new
>
> NOTE: if you have solved this level and see ‘Byebye!’ when trying to log into bandit18, this is related to the next level, bandit19

All we have to do is `diff` the files

```bash
bandit17@bandit:~$ diff passwords.old passwords.new
42c42
< p6ggwdNHncnmCNxuAt0KtKVq185ZU7AW
---
> hga5tuuCLF6fFzUpnagiMN8ssu9LFrdg
```

The password is `hga5tuuCLF6fFzUpnagiMN8ssu9LFrdg`.

## Bandit Level 18 → Level 19
