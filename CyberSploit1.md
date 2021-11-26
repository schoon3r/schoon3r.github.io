# CyberSploit1

- Scanned the machine using NMAP, since the target is defined, I just used the -A option
- Found port 80 and 22 are open
- Went to browser using target IP, checked the hood and found a username (great!)
- enumerated the website using gobuster and ran this script `gobuster dir -u http://192.168.218.92 -w /usr/share/wordlists/dirb/common.txt`
- found a robots.txt file, appended it in the browser and got Flag1 (wohoo!)
- now I have a username and possibly this Flag1 is the password... time to break out metasploit (yeah i know, such a noob)
- in metasploit, search `ssh_login` and used this option. Set rhost and username and password
- voila!
- this time went to search for shell_to_meterpreter and used this option
- set session to 1 (given when i logged in)
- transferred to meterpreter session 2
- enumerated the folder and found `local.txt` which included the Flag. (not sure why there is Flag2.txt hahaha!)
- enumerated the machine a bit more `(uname -a lsb_release -a)` and found it ran on Ubuntu 12.04.5 LTS
- googled for exploit and there it is vuln to privesc
- download the exploit, ran an http server to transfer file into target machine and download the file into target using wget
- whoami reveals i am root (huzzah!)
- a bit of probing into directory and found final flag
