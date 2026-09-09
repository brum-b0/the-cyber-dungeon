---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/htb/sightless/","tags":["#linux","sshtunnel","unshadow","chrome-remote-debug-inspect","password_cracking","passwordReuse"],"dg-note-properties":{"season 5?":"easy","tags":["#linux","sshtunnel","unshadow","chrome-remote-debug-inspect","password_cracking","passwordReuse"],"retired?":true}}
---

# 10.10.11.32
---
# Enumeration &  Recon

## ports
```bash hl:1,3
21
22
80
```

sqlpad app linked at `sqlpad.sightless.htb`, shows version `6.10.0`
- cve [here](https://huntr.com/bounties/46630727-d923-4444-a421-537ecd63e7fb)

## creds
`michael:insaneclownposse`


# Exploitation i.e getting a foothold

- we are already root, so this is prob a container
	- we can see a `docker-entrypoint` at the root
- there's a sqlite db
	- open it up grab some hashed creds from it
		- admin -> crack the hash (hashcat mode 3200:`bcrypt $2*$, Blowfish (Unix)`)
		- michael is a user on this container so I assume he would be the admin
		- ssh as michael for password reuse test and it works
		- could also probably check the shadow file and unshadow-crack it

`user.txt` is in the michael's home dir

# Post Exploitation i.e. privsec

- port forward froxlor running locally at port 8080
- see chrome remote debugging process, you can do remote inspect
- inspect it, get creds.
	- go to `chome://inspect`
	- click configure
	- add the ports until one works like `localhost:45454`, as there are a few processes running with different ports
		- it will show up as a remote target if it works
	- you will be able to inspect the user, who is logging in to froxlor, and you can grab their submitted creds by viewing the post request on the network tab
- log in to froxlor
	- you will need to set your localhost in `/etc/hosts` to `admin.sightless.htb` to be able to log in
- on the PHP/PHP-FPM versions page, we can modify a php command that runs when it restarts
- php-fpm start command can be whatever you want, command execution as a feature
	- go to settings/PHP-FPM and toggle the enable switch to restart it
- do what you like, I like `chmod +s /bin/bash` -> so I can then run `/bin/bash -p` to get an elevated shell
	- you can also catch a reverse shell, or any other privesc technique you like


`root.txt` is in the usual place