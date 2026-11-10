---
name: macos-active-directory-powershell
description: "Use when working AD or PowerShell from macOS."
---

# macOS: Active Directory & PowerShell

## Goal
Work with AD and PowerShell from a macOS host without Windows RSAT.

## Tooling
- PowerShell Core: `brew install powershell` (large ~hundreds MB; may exceed default 180s timeout — run with timeout=600 or in background). Binary `/opt/homebrew/bin/pwsh`.
- LDAP CLI: macOS ships `/usr/bin/ldapsearch`. Extra: `brew install openldap`.
- Python AD libs: `pip3 install ldap3 impacket`.
- Optional helpers (pip): `netexec`, `certipy-ad`.

## Quick commands
- Run PS script: `pwsh -File script.ps1`
- Inline: `pwsh -Command 'Get-Date'`
- LDAP search: `ldapsearch -x -H ldap://dc.example.com -D 'user@domain' -w 'pass' -b 'DC=example,DC=com' '(objectClass=user)'`
- Python ldap3:
  ```python
  from ldap3 import Server, Connection
  s = Server('dc.example.com')
  c = Connection(s, user='DOMAIN\\user', password='pass', auto_bind=True)
  c.search('DC=example,DC=com', '(objectClass=user)', attributes=['sAMAccountName'])
  print(c.entries)
  ```
- Impacket: `GetNPUsers.py`, `GetUserSPNs.py`, `secretsdump.py`.

## Notes
- RSAT ActiveDirectory module is Windows-only; on macOS use LDAP or Impacket.
- Store creds in env vars, never in scripts.
