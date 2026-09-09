---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/htb/monitors-three/","tags":["#linux","web","PHP","sqli","vhost","duplicati","authbypass","sshtunnel","hashcat","fileBackupAndRestore"],"dg-note-properties":{"season ?":"medium","tags":["#linux","web","PHP","sqli","vhost","duplicati","authbypass","sshtunnel","hashcat","fileBackupAndRestore"],"retired?":true}}
---


# 10.10.11.30
---
# Enumeration &  Recon
```bash hl:2,6,11
PORT     STATE    SERVICE VERSION
22/tcp   open     ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 86:f8:7d:6f:42:91:bb:89:72:91:af:72:f3:01:ff:5b (ECDSA)
|_  256 50:f9:ed:8e:73:64:9e:aa:f6:08:95:14:f0:a6:0d:57 (ED25519)
80/tcp   open     http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://monitorsthree.htb/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0 (Ubuntu)
8084/tcp filtered websnp
```

### Vhosts:
```bash hl:23

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.0.2
________________________________________________

 :: Method           : GET
 :: URL              : http://monitorsthree.htb
 :: Header           : Host: FUZZ.monitorsthree.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
 :: Filter           : Response words: 3598
________________________________________________

cacti                   [Status: 302, Size: 0, Words: 1, Lines: 1]
```

#### cacti vhost
- version `1.2.26`
- CVE-2024-25641
![Pasted image 20241128130832.png](/img/user/img/Pasted%20image%2020241128130832.png)
- generic error message for failed creds, so no valid account determination
- seems to be stripping sqli syntax as well

#### root vhost
![Pasted image 20241128133104.png](/img/user/img/Pasted%20image%2020241128133104.png)
- injection point in `forgot_password`
	- also has valid account determination
- user entity/table has cardinality 9:
- `' union select 1,2,3,4,5,6,7,8,9; -- -`
	![Pasted image 20241128133501.png](/img/user/img/Pasted%20image%2020241128133501.png)
- but it's not giving me my numbers back or any, so I think it's boolean based blind -> sqlmap guesses time but sqlmap is gonna think it's time-based as always.
- we can adjust our technique to look for that with `--technique=B` with max risk and level cause why not.
- let it run while keeping an eye on it
	- 2 dbs:
		- information_schema (granted)
		- monitorsthree_db
			- 6 tables:
				- users entity (9 fields, 4 records)
					- id
					- username
					- email
					- password
					- name
					- position
					- dob
					- start_date
					- salary
				- invoices (probably don't care about these)
				- customers
				- changelog
				- tasks
				- invoice_tasks

## Creds
`admin:greencacti2001 `

`marcus:12345678910`

crack the bcrypts with hashcat mode 3200
# Exploitation i.e getting a foothold
- found a [rce advisory](https://github.com/cacti/cacti/security/advisories/GHSA-7cmj-g5qc-pj88) for importing a package which is likely just another php shell in a zip file
	- - https://github.com/5ma1l/CVE-2024-25641 it just werks
		- needs creds tho
		- grab creds from sql injection on root vhost forgot_password.php
- take creds, craft package payload and upload
- the PHP file will be written into the `resource` directory, accessible [here](http://cacti.monitorsthree.htb/cacti/resource/test.php)


```bash
Ncat: Version 7.92 ( https://nmap.org/ncat )
Ncat: Listening on :::4451
Ncat: Listening on 0.0.0.0:4451
Ncat: Connection from 10.10.11.30.
Ncat: Connection from 10.10.11.30:54244.
Linux monitorsthree 5.15.0-118-generic #128-Ubuntu SMP Fri Jul 5 09:28:59 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
 22:25:26 up 12:23,  0 users,  load average: 0.05, 0.04, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```
# Post Exploitation i.e. privesc
- `cacti/include/config.php` has db creds
- db cacti has table user_auth with entries for admin and marcus
- crack their hashes
- login as marcus, maybe root?
- but that mono on port 8084 has to be part of the privesc so prob can only pivot to marcus... (maybe not)
	- user.txt in marcus home dir
- forward port 8200 that is listening locally, it is duplicati
- basic search says duplicati stuff is in `/var`, `/etc/` or `/opt`, which makes sense
- `/opt/duplicati` has sqlite dbs, grab them
- open them locally with sqlite3
- `>select * from Option;` in `duplicati-server.sqlite3`:
```sqlite
-2||server-passphrase|Wb6e855L3sN9LTaCuwPXuautswTIQbekmMAr7BrK2Ho=
-2||server-passphrase-salt|xTfykWV1dATpFZvPhClEJLJzYA5A4L74hX7FK8XmY0I=
-2||server-passphrase-trayicon|56340ef0-a4aa-47d3-b39e-46ae7b47c9b2
-2||server-passphrase-trayicon-hash|P+RIgoWSSOVOZ7QcMlu5D61R2raeb8rQYxE7v6H92VU=
```

- follow this: https://medium.com/@STarXT/duplicati-bypassing-login-authentication-with-server-passphrase-024d6991e9ee
- duplicati running `Duplicati - 2.0.8.1_beta_2024-05-07`
- make a new backup job
	- name it whatever, no encryption
	- manually type path somewhere you want to store the backups, I chose `/tmp`, but it wasn't actually going to there so I looked at the backup of cacti's sqlite3 db and looked at the `File` table
		- it seems that it's appending a '`source/`' dir to the filepath so I just did the same and it worked
			- so `/source/tmp/myfolder`
	- do the same for the sourcing data, i.e. what you want to backup, so `/source/root/`
	- uncheck automatically run backups
	- save the backup job
	- once the backup job is saved, run it from the home page
	- once the job is ran, restore the job you made
	- make sure it's the restore point you want (there should only be one)
	- choose your restore location, e.g. `/source/tmp/myfolder/restored/`
	- you should now be able to browse the dir for anything
	- no `.ssh/id_rsa` and an empty `authorized_keys`
	- only world readable root flag, but I really want a root shell.
		- kinda tired of the box at this point so maybe see what other people figured out.
### Actually getting a shell

> [!info]
> I didn't figure this out at the time, so this is taken from others.

- you can restore marcus' authorized keys file to root's `.ssh` dir (why didn't I think of this...)
	- this will allow you to login as root with marcus' key 
