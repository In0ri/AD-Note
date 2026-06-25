xfreerdp /v:10.129.32.65 /u:htb-student /p:Academy_student_AD!

remmina -c rdp://htb-student@10.129.32.65
rdesktop -u htb-student -p 'Academy_student_AD!' 10.129.32.65
ssh htb-student@10.129.32.79

## Psexec
- psexec.py inlanefreight.local/htb-student:'Academy_student_AD!'@10.129.19.207
- Add `-c option` to spesific binary to excute on remote host
- evil-winrm -i 10.129.32.79  -u htb-student -p Academy_student_AD!


## Enter-PSSession 
- Target must enable Powershell Remoting before
`Enable-PSRemoting -Force`
```
$cred = Get-Credential
Enter-PSSession -ComputerName 192.168.1.10 -Credential $cred

Enter-PSSession -ComputerName PC01 -Credential $cred
```