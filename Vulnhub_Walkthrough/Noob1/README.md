# VulnHub Walkthrough: Noob 1 (Boot2Root CTF)

**Noob: 1** is a beginner-friendly Linux boot2root machine from VulnHub designed to test foundational enumeration, encoding, steganography, and basic Linux privilege escalation. Below is a step-by-step walkthrough of how I compromised the machine.

## Table of Contents

1. [Network Discovery & Enumeration](#1-network-discovery--enumeration)
2. [FTP Enumeration & Credential Discovery](#2-ftp-enumeration--credential-discovery)
3. [Web Application & Archive Extraction](#3-web-application--archive-extraction)
4. [Steganography Analysis](#4-steganography-analysis)
5. [Cracking the Second Image & Decoding the Cipher](#5-cracking-the-second-image--decoding-the-cipher)
6. [Initial Foothold via SSH](#6-initial-foothold-via-ssh)
7. [Privilege Escalation to Root](#7-privilege-escalation-to-root)

---

## 1. Network Discovery & Enumeration

### Host Discovery

First, I switched to root on my Kali machine and ran `arp-scan` to identify the target machine's IP address on the local network:

![arp-scan](images/image1.png)

The scan revealed the target IP address: **192.168.182.137**

### Port Scanning with Nmap

Next, I ran a full-port Nmap scan with default scripts and version detection to map out all running services:

![nmap scan](images/image2.png)

**Key findings:**

- **Port 21 (FTP):** vsftpd 3.0.3 with **anonymous login allowed**. It already lists two files: `cred.txt` and `welcome`.
- **Port 80 (HTTP):** Apache httpd 2.4.29 (Ubuntu) with a page titled **Login**.
- **Port 55077 (SSH):** OpenSSH 7.6p1 on a non-standard high port.

## 2. FTP Enumeration & Credential Discovery

Since anonymous FTP login was enabled, I connected to the FTP server with the username `Anonymous` and downloaded both files:

![FTP session](images/image3.png)

```bash
ftp 192.168.182.137
# Name: Anonymous
# Password: Anonymous (any value works)
ls
get cred.txt
get welcome
exit
```

Checking the contents of `cred.txt` revealed a Base64-encoded string:

```bash
cat cred.txt
```

![cred.txt contents](images/image4.png)

Decoding the Base64 string gave a set of credentials:

- **Username:** `champ`
- **Password:** `password`

![Base64 decode](images/image5.png)

## 3. Web Application & Archive Extraction

Navigating to `http://192.168.182.137` presented a login portal. Using the credentials `champ:password`, I logged into the web application.

From the portal, I downloaded an archive named `downloads.rar` (via the **About Us** page) and extracted it using `unrar`:

![downloading the archive](images/image6.png)

```bash
unrar e downloads.rar
ls
```

![extracted files](images/image7.png)

## 4. Steganography Analysis

Two image files (`funny.bmp` and `funny.jpg`) in a CTF are a classic hint for steganography. I used `steghide` to check both images for embedded data (empty passphrase):

```bash
steghide info funny.bmp
steghide info funny.jpg
```

![steghide info](images/image8.png)

While `funny.bmp` returned nothing, `funny.jpg` contained an embedded file named **`hint.py`**.

I extracted `hint.py` from `funny.jpg` (no passphrase needed) and read its contents:

```bash
steghide extract -sf funny.jpg
cat hint.py
```

![steghide extract](images/image9.png)

![hint.py contents](images/image10.png)

## 5. Cracking the Second Image & Decoding the Cipher

The third file extracted from `downloads.rar` earlier was simply named **`sudo`**. Using `sudo` as the passphrase for `funny.bmp` unlocked a hidden file:

![steghide extract funny.bmp](images/image11.png)

Following the clue from `hint.py` ("try to rotate some words too"), I ran the string through a **ROT13** decoder (CyberChef or `tr` in the terminal):

![ROT13 decode](images/image12.png)

Or from the terminal:

```bash
echo "jgs:guvf bar vf n fvzcyr bar" | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

This gave a new set of credentials:

- **Username:** `wtf`
- **Password:** `this one is a simple one`

## 6. Initial Foothold via SSH

From the initial Nmap scan, we knew SSH was running on the non-standard port **55077**. I used the decoded credentials to log in:

```bash
ssh wtf@192.168.182.137 -p 55077
```

![SSH login as wtf](images/image13.png)

After entering the password (`this one is a simple one`), I landed a shell as the user **wtf**.

## 7. Privilege Escalation to Root

Once inside, the first thing I checked was the user's sudo privileges:

```bash
sudo -l
sudo su
cd /root
ls -la
cat root.txt
```

![sudo -l and root shell](images/image14.png)

![root flag](images/image15.png)

---

## Takeaways

That wraps up **Noob: 1**! It's a fun, quick beginner box that reinforces:

- Check every open port, especially non-standard SSH ports.
- Inspect every extracted file for clues.
- Always run `sudo -l` as soon as you get a foothold.

*Thanks for reading! Feel free to open an issue with any questions or feedback.*
