## Enumerating ACLs with PowerView
- In their simplest form, ACLs are lists that define a) who has access to which asset/resource and b) the level of access they are provisioned. The settings themselves in an ACL are called Access Control Entries (ACEs)
- There are four specific ACEs to highlight the power of ACL attacks:
  - ForceChangePassword - gives us the right to reset a user's password without first knowing their password (should be used cautiously and typically best to consult our client before resetting passwords).
  - GenericWrite - gives us the right to write to any non-protected attribute on an object. If we have this access over a user, we could assign them an SPN and perform a Kerberoasting attack (which relies on the target account having a weak password set). Over a group means we could add ourselves or another security principal to a given group. Finally, if we have this access over a computer object, we could perform a resource-based constrained delegation attack which is outside the scope of this module.
  - AddSelf - shows security groups that a user can add themselves to.
  - GenericAll - this grants us full control over a target object. Again, depending on if this is granted over a user or group, we could modify group membership, force change a password, or perform a targeted Kerberoasting attack. If we have this access over a computer object and the Local Administrator Password Solution (LAPS) is in use in the environment, we can read the LAPS password and gain local admin access to the machine which may aid us in lateral movement or privilege escalation in the domain if we can obtain privileged controls or gain some sort of privileged access.

### Using Get-DomainObjectACL
- To forcus on an user, which we compomised later. We will find every any interesting ACL rights that we could take advantage of from this user by Get-DomainObjectACL function
`$sid = Convert-NameToSid wley`
`Get-DomainObjectACL -Identity * | ? {$_.SecurityIdentifier -eq $sid}`
- Searching ObjectAceType on google or excute Get-ADObject func to get more abount ACL object
```
$guid= "00299570-246d-11d0-a768-00aa006e0529"
Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * |Select Name,DisplayName,DistinguishedName,rightsGuid| ?{$_.rightsGuid -eq $guid} | fl
```
### Alternative methods is using Get-Acl and Get-ADUser cmdlets instate of Powerview
- Creating a List of Domain Users
  ```
  Get-ADUser -Filter * | Select-Object -ExpandProperty SamAccountName > ad_users.txt
  foreach($line in [System.IO.File]::ReadLines("C:\Users\htb-student\Desktop\ad_users.txt")) {get-acl  "AD:\$(Get-ADUser $line)" | Select-Object Path -ExpandProperty Access | Where-Object {$_.IdentityReference -match 'INLANEFREIGHT\\wley'}}
  ```


