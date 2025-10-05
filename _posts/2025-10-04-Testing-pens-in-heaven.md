---
layout: single
title:  "Testing pens in heaven"
header:
  overlay_image: /assets/images/banner.jpg
  show_overlay_excerpt: false
---

<b><i>bruno from the future, don't forget to add an index here when you're done</i></b>



# Pentesting cloud environments

It seems there aren't many resources on cloud pentesting, so I decided to take matters into my own hands. I won't create any new learning material, instead I'll aggregate the best free resources I can find. Any CTFs or practical exercises I complete will be documented, as well as any notes I take on theoretical material. This is more or less the process I take when learning a new topic, but this time I'll be making it public. I'll be focusing on the cloud provider AWS.

I'll update this page continuously and share it when I'm satisfied with it so it can be useful to someone out there. At the moment this post is still in development and I have not gone through all of this material, I may add or remove stuff in the future and cannot vouch for their quality yet.

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

* Cheatsheets and references
  * <a href="https://docs.aws.amazon.com/cli/latest/reference/#cli-aws">AWS CLI Documentation</a>
  * <a href="https://cloud.hacktricks.wiki/en/index.html">HackTricks Cloud</a>
  * <a href="https://github.com/kh4sh3i/cloud-penetration-testing">kh4sh3i's Cloud pentesting cheatsheet</a>
  * <a href="https://github.com/carnal0wnage/weirdAAL">WeirdAAAL: AWS Attack Library</a>

* Tools
  * <a href="https://github.com/NetSPI/aws_consoler">AWS Consoler: Convert CLI credentials into console access</a>
  * <a href="https://github.com/zer1t0/awsenum">awsenum: Enumerate AWS permissions and resources</a>
  * <a href="https://github.com/andresriancho/enumerate-iam">Enumerate IAM: Enumerate permissions associated with a credential set</a>
  * <a href="https://github.com/prowler-cloud/prowler">Prowler: Perform automatic security assessment</a>
  * <a href="https://github.com/nccgroup/ScoutSuite">Scout Suite: Highlights risk areas from gathered configuration data</a>


<br>

# Introductory material
## Introduction to Cloud Pentesting
* The most common cloud security problems are excessive permissions and several kinds of misconfigurations such as ignoring default settings, exposing sensitive data or simply misunderstanding how things work
* A major problem is publicly exposing S3 buckets, there are "search engines" for publicly accessible data in the cloud such as <a href="https://buckets.grayhatwarfare.com/">GrayHat Warfare</a>

<br>

* Try to interact with any S3 buckets you find, for example `https://megabank-supportstorage.s3.amazonaws.com/index-3-1-1144x912.jpg`
  * List files with `aws s3 ls s3://megabank-supportstorage --recursive`
  * Download files (recursively) with `aws s3 sync s3://megabank-supportstorage/pentest_report/ --recursive`

<br>

* **Compute Instance Metadata** is a cloud service which provides administrative endpoints, it's usually bound to the internal IP `169.254.169.254`
* **IMDSv2** will require token authentication to call these endpoints, but not the default IMDSv1
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
Similar to other types of penetration testing, the scoping process should include questions such as
* How many non-standard AWS IAM policies exist?
* Which services are used?
* How many IAM policies are assigned?
* How many accounts exist?

<br>

Keep the **Shared Responsibility Model** in mind: security of the cloud is AWS's responsiblity, security in the cloud is the customer's responsibility

Remember AWS's <a href="https://aws.amazon.com/security/penetration-testing/">policy for penetration testing</a>

<br>

Start by **identifying the attack surface**, determining which services are:
* Used by the application
* Externally exposed
* Managed by AWS or by the customer

Also **enumerate** and **fingerprint** the cloud infrastructure for used components and third-party software. Just like with a web or infra penetration test






# CTFs and Hands-on material
## flaws.cloud

## flaws2.cloud

## pwnedlabs.io

## HTB Fortress: AWS

## CloudGoat 2.0

## AWSGoat
  


