---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/htb/green-horn/","tags":["#linux","web","PHP","FileUpload","Depix","passwordReuse","gittea"],"dg-note-properties":{"season 5":"easy","tags":["#linux","web","PHP","FileUpload","Depix","passwordReuse","gittea"],"retired?":true}}
---


# 10.10.11.25
---
## Enumeration &  Recon
```bash
Discovered open port 22/tcp on 10.10.11.25
Discovered open port 80/tcp on 10.10.11.25
Discovered open port 3000/tcp on 10.10.11.25
```
- pluck 4.7.18
	- [RCE CVE](https://github.com/thefizzyfish/CVE-2023-50564-pluck)
- port 3000 hosts a gittea instance with passwd hash in a php file: `data/settings/pass.php`
- this file is checked in login.php so it must be a (user set) hardcoded pass for pluck
	- crack it for pluck passwd
		- `iloveyou1`
	- use the pluck passwd for the rce exploit

# Exploitation i.e getting a foothold
- cve exploit will pop a revshell
- upgrade with python and stty
- `su junior` -> passwd reuse from pluck passwd `iloveyou1`
	- `user.txt` in junior home dir
# Post Exploitation i.e. privsec
- view pdf
- pixel-censored text
	- you gotta extract the pixellated image, cause taking a screensnip won't work, unless you're exact I suppose. (I tried for hours)
	- use image with [Depix](https://github.com/spipm/Depix) (I tried some other tools I found to no luck)
		- run with the `debruinseq_notepad_Windows10_closeAndSpaced.png` search image to get a readable output
		- `sidefromsidetheothersidesidefromsidetheotherside`
- su root -> `root.txt` in the usual spot