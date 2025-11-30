---
layout: single
title:  "Hacking Notes"
header:
  overlay_image: /assets/images/banner.jpg
  show_overlay_excerpt: false
---

By no means an exhaustive list. Just a list of things I run very often and thought it would be useful to always have handy. Mostly for enumeration

## My go-to notes websites
```
https://book.hacktricks.wiki/
https://www.thehacker.recipes/
https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master
```

## Reverse Shell generator
```
https://www.revshells.com/
```

## Google Dorks
```
https://taksec.github.io/google-dorks-bug-bounty/ # quick google dork cheatsheet
```

## Nmap
```
# simple, good for CTFs (no UDP, max speed)
nmap -p- -T5 <ip>
nmap -A -p<ports> <ip>
```

## Directory Busting
```
# max speed, but takes a long time - previously merged rafts wordlist and many extensions (adapt to website's stack)
feroxbuster -u <url> -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-combined.txt --scan-dir-listings -x asp,aspx,bak,txt,html,json,php,rar,zip,git,env,py,json,conf,yaml -t 50

# to add a cookie, ignore TLS checks, add a custom header and a custom user agent
(...) -b 'session=cookie' -k -H "X-Forwarded-For: 127.0.0.1" -a pentester_brunocaseiro
```

## Subdomain Discovery
```
# brute force with huge wordlist, ignoring a specific response length (change if needed)
wfuzz -H "Host: FUZZ.fries.htb" -c -w "/usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt" --hl=7 http://fries.htb


```

## Nmap
```
aaaa
```

## Nmap
```
aaaa
```

## Nmap
```
aaaa
```

## Nmap
```
aaaa
```


