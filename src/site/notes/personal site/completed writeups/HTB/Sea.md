---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/htb/sea/","tags":["#linux","web","XSS","CommandInjection","PHP","revshell","password_cracking","SUIDBinary","csrf"]}
---

# 10.10.11.28
---
# Enumeration &  Recon
```bash hl:2,4,9
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Sea - Home
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
Seems like a php web app, session cookie httponly flag not set, xss steal maybe?

### feroxbuster
```bash
feroxbuster -u http://10.10.11.28 --wordlist ~/Documents/src/gh-tools/SecLists/Discovery/Web-Content/raft-large-directories.txt -x php txt md -g
```
```bash
200      GET      118l      226w     2731c http://10.10.11.28/contact.php
200      GET       15l       50w      318c http://10.10.11.28/themes/bike/README.md
200      GET        1l        1w        6c http://10.10.11.28/themes/bike/version
200      GET       21l      168w     1067c http://10.10.11.28/themes/bike/LICENSE

```

- `readme.md`
	- shows us WonderCMS
- `version`
	- gives us 3.2.0
		- exploit [here](https://github.com/thefizzyfish/CVE-2023-41425-wonderCMS_RCE)
		- CVE writeup [here](https://medium.com/@uu660111/cve-2023-41425-technical-analysis-viktor-v%C3%A4xby-d23fbde9b6e8)
- `Contact.php`
	- contact form with a website field, I wonder if they click on it
	- put a http link to our listening server at port 8000
	- and we get a callback
```bash
[09:45:20 PM]❯ python3 -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.10.11.28 - - [03/Nov/2024 21:48:27] "GET / HTTP/1.1" 200 -
```

# Exploitation i.e getting a foothold
So it seems that we need to get whoever is authenticated on the back end checking out links to click a link that downloads and installs a zip file as a WonderCMS php plugin
so:
- csrf them to download and install the shell.php.zip
- access the shell.php installed as a plugin

trying to do it manually was acting up for me, so I just used the exploit linked above. 

```bash
> nc -nvlp 4451
Ncat: Version 7.92 ( https://nmap.org/ncat )
Ncat: Listening on :::4451
Ncat: Listening on 0.0.0.0:4451
Ncat: Connection from 10.10.11.28.
Ncat: Connection from 10.10.11.28:44904.
bash: cannot set terminal process group (1159): Inappropriate ioctl for device
bash: no job control in this shell
www-data@sea:/var/www/sea/themes/shell$
```
# Post Exploitation i.e. privsec
- shell is `www-data` so we need to pivot
- amay is a machine user
- amay's passwd hash is in `database.js`
- crack it with `john` or `hashcat`
- ssh as amay
	- user flag is in amay's home dir
	  
---
- `netstat -tulpn` shows a local webapp running on 8080
- forward with an ssh tunnel
- web app has command injection, just add a command to the end of the log being loaded eg: `access_log; touch /home/amay/test.txt` and see that the command injection is running as root since the test.txt is owned by root
- inject `chmod +s /bin/bash` then run `/bin/bash -p` as amay for a root shell
- root flag is in the usual spot





I promise I will take more screenshots again in future writeups, just kinda getting these out quickly at the moment.

