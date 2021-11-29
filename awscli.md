---
layout: default
title: AWS CLI
description: Cheat Sheet
---

# General API

Configure Default Profile

```
aws configure
```

Configure Named Profile

```
aws configure --profile NAME
```

Grab Access Identity

```
aws sts get-caller-identity
```

Meta-data (append to container uri)

```
http://169.254.169.254/latest/meta-data/
```

User Meta Data

```
http://169.254.169.254/latest/user-data/
```

User credential folder

```
http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

<br><br>

# s3 Buckets

Default bucket names

```
http://flaws.cloud.s3.amazonaws.com/

http://[bucket_name].s3.amazonaws.com/

http://s3.amazonaws.com/[bucket_name]/
```

List public bucket content without sign-in

```
aws s3 ls s3://flaws.cloud/ --no-sign-request
```

Copy files into a bucket

```
aws s3 cp local.txt s3://some-bucket/remote.txt --acl authenticated-read
```

Download the whole bucket to your KALI

```
aws s3 sync s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/ ~/home/Flaws
```

<br><br>

# EC2

Get snapshot information

```
aws ec2 describe-snapshots --owner-id 975426262029
```

Create a volume from a snapshot information

```
aws ec2 create-volume --availability-zone us-west-2a --region us-west-2  --snapshot-id  snap-0b49342abd1bdcb89
```

<br><br>

# ECR

List public AMI images

```
aws ecr list-images --repository-name level2 --registry-id 653711331788
```

See ECR image content

```
aws ecr batch-get-image --repository-name level2 --registry-id 653711331788 --image-ids imageTag=latest | jq '.images[].imageManifest | fromjson'
```

See ECR image content with a specific digest

```
aws ecr get-download-url-for-layer --repository-name level2 --registry-id 653711331788 --layer-digest "sha256:edfaad38ac10904ee76c81e343abf88f22e6cfc7413ab5a8e4aeffc6a7d9087a"
```

<br><br>

[back to HOME](./)
