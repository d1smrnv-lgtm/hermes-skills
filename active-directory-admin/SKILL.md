---
name: active-directory-admin
description: "Use when administering AD via RSAT/PowerShell on Windows 10."
---
# Active Directory Administration (RSAT on Windows 10)

## Installed RSAT components
- Rsat.ActiveDirectory.DS-LDS.Tools
- Rsat.GroupPolicy.Management.Tools
- Rsat.Dns.Tools
- Rsat.Dhcp.Tools
- Rsat.CertificateServices.Tools
- Rsat.FileServices.Tools
- Rsat.FailoverCluster.Management.Tools
- Rsat.ServerManager.Tools

## PowerShell modules available
- ActiveDirectory, GroupPolicy, DnsServer, DhcpServer, ServerManager

Import them in a session:
```powershell
Import-Module ActiveDirectory, GroupPolicy, DnsServer, DhcpServer
```

## Common tasks

### Find user
```powershell
Get-ADUser -Filter "SamAccountName -eq 'jdoe'" -Properties *
Get-ADUser -Filter "Name -like '*John*'" | Select Name,SamAccountName,Enabled
```

### Unlock / reset password
```powershell
Unlock-ADAccount -Identity jdoe
Set-ADAccountPassword -Identity jdoe -Reset -NewPassword (ConvertTo-SecureString 'TempP@ss1' -AsPlainText -Force)
Set-ADUser -Identity jdoe -ChangePasswordAtLogon $true
```

### Enable/disable user
```powershell
Enable-ADAccount -Identity jdoe
Disable-ADAccount -Identity jdoe
```

### Groups
```powershell
Get-ADGroup -Filter "Name -like '*Admins*'"
Add-ADGroupMember -Identity 'Domain Admins' -Members jdoe
Remove-ADGroupMember -Identity 'Domain Admins' -Members jdoe -Confirm:$false
Get-ADPrincipalGroupMembership jdoe
```

### GPO
```powershell
Get-GPO -All
New-GPO -Name 'Workstation Baseline'
New-GPLink -Name 'Workstation Baseline' -Target 'OU=Workstations,DC=domain,DC=local'
Get-GPResultantSetOfPolicy -ReportType Html -Path C:\gpo.html
```

### DNS
```powershell
Get-DnsServerResourceRecord -ZoneName 'domain.local' -RRType A
Add-DnsServerResourceRecordA -Name 'host' -ZoneName 'domain.local' -IPv4Address 10.0.0.5
```

### DHCP
```powershell
Get-DhcpServerv4Scope
Get-DhcpServerv4Lease -ScopeId 10.0.0.0
```

## Notes
- Run PowerShell as Administrator for RSAT cmdlets where required.
- Use domain credentials when connecting to remote DCs: `Get-ADUser ... -Server dc01.domain.local -Credential (Get-Credential)`.
- This PC is not joined to the domain; supply -Server/-Credential as needed.
