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

## Pivot to CloudShell

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

# Level 6
