---
layout: single
title:  "Hacking Notes"
header:
  overlay_image: /assets/images/banner.jpg
  show_overlay_excerpt: false
---

## Go-to notes websites
```
https://book.hacktricks.wiki/
https://www.thehacker.recipes/
https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master

# reverse shell generator
https://www.revshells.com/
```

## Burp Extensions
```
Active Scan++
Autorize
AutoRepeater
Collaborator Everywhere
Copy As Python-Requests
HTTP Request Smuggler
JS Miner
Param Miner
Retire.js
```

## Nmap
```
# simple and fast (no UDP, max speed), also works with domains
nmap -p- -T5 <ip>
nmap -A -p<ports> <ip>
```

## masscan
```
# port discovery only but ultra speed 
masscan --max-rate 100000 --ports 0-65535 <ip>
masscan --max-rate 100000 --ports 0-65535 -iL <ip_file> 
```

## Domain Enumeration
```
# ASN/ip space enumeration, careful with cloud ranges
https://bgp.he.net/
https://dnschecker.org/all-dns-records-of-domain.php

# find acquired/related companies
https://asrank.caida.org/
Google "<company_name> acquisitions"

# reverse whois
https://www.whoxy.com/

# ad/analytics relationships
https://builtwith.com/

# well... shodan
https://www.shodan.io/
```

## Subdomain Discovery
```
# subdomain scraping
subfinder -v -d <domain> # setup api keys for better results
github-subdomains -d <domain> -t "<github_token>"

# really good, more than just subdomain enum
bbot -t <domain> -p subdomain-enum cloud-enum code-enum web-basic nuclei-budget --allow-deadly

# brute force
amass enum -brute -d twitch.tv
wfuzz -H "Host: FUZZ.<domain>" -c -w "/usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt" --hl=7 <url>

# spidering, finds more than just subdomains
Burp Suite (set a keyword in the scope) --> Target --> Right click target --> Scan --> Crawl

# manual permutation
sub.domain.com --> origin.sub.domain.com
sub.domain.com --> origin-sub.domain.com
www.target.com --> ww2.target.com
```

## Directory Busting
```
# max speed with previously merged rafts wordlist and many extensions
# adapt wordlist and extensions to website stack
feroxbuster -u <url> -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-combined.txt -x asp,aspx,bak,txt,html,json,php,rar,zip,git,env,py,json,conf,yaml -t 50

# to add a cookie, ignore TLS checks, add a custom header and a custom user agent
(...) -b 'session=cookie' -k -H "X-Forwarded-For: 127.0.0.1" -a pentester_brunocaseiro
```

## JavaScript
```
# searches for endpoints, including inline js
xnLinkFinder -i <input>  -v -d 3 -sf <scope>

# unminify or deobfuscate js
https://beautifier.io/
```

## Google Dorks
```
# good starting points
https://taksec.github.io/google-dorks-bug-bounty/
https://dorks.faisalahmed.me/

# fixed vulnerabilities might be bypassable
"<company/domain>" vulnerabilities
```

## GitHub
```
# Jason Haddix google dorks (https://gist.github.com/jhaddix/1fb7ab2409ab579178d2a79959909b33)
./gdorklinks.sh <company>

# some other starting points
"<company/domain>" <service> # i.e "google" ftp
"<company/domain>" <keyword> # i.e password, key, secret, pass, credentials, login, token, ftp, config, pwd, secrutiy_credentials, connectionstring, JDBC, ssh2_auth_password, send_keys, ...
"<company/domain>" NOT keyword # ignores results with "keyword"
user:<username> keyword # keyword can be company name, helps find hidden domains. previous employees do not show under the github organization but can be found from old code
"<company/domain>" language:bash # bash is good to find scripts, but works for other languages

# finds secrets, optionally add --results=verified
trufflehog git <.git_repo_link> # 
trufflehog github --org=<github_org_name>
```

## Cloud
```
# list of services and how to take them over
https://github.com/EdOverflow/can-i-take-over-xyz

# grep for company name in cloud provider's IP ranges
https://kaeferjaeger.gay/?dir=sni-ip-ranges

# s3 bucket search engine
https://grayhatwarfare.com/
```
