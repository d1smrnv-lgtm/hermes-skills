---
name: macos-sysadmin-tooling
description: Use when installing sysadmin/security tools on macOS.
version: 1.0.0
author: Hermes
license: MIT
metadata:
  hermes:
    tags: [macos, homebrew, sysadmin, security]
    related_skills: []
---
# macOS Sysadmin Tooling Setup

## When to use
- User asks to install or configure system-administration, networking, or basic security tooling on a macOS host.

## Procedure
1. Confirm scope before installing. Ask which areas are needed (SSH/Ansible, containers, cloud CLIs, networking/diagnostics, security) and whether only base security tools (nmap, sqlmap, hydra) or a full pentest set.
2. Check Homebrew availability: `brew --version`. If missing, install Homebrew first.
3. Install in grouped batches (SSH/admin, networking, security) rather than one giant command; each group is easier to retry.
4. Run `brew install ...` with a generous timeout (600s). Homebrew batches with python@3.x / ansible can exceed the default 180s.
5. Verify each tool with `command -v` and a version flag after install.

## Tool name pitfalls (macOS/Homebrew)
- John the Ripper is `john-jumbo`, not `john`.
- Wireshark CLI is `tshark` (package `wireshark`); `tcpdump` is already provided by macOS.
- `jq` may be shadowed by `/usr/bin/jq`; Homebrew's jq is still installed but may not win PATH.
- `sshpass`, `autossh`, `mosh` are separate formulae; install them explicitly.

## Local web dashboards on macOS
- Port 5000 is occupied by the AirPlay Receiver (Control Center) on macOS. Use 5050 (or another free port) for local Flask/HTTP servers, and update any desktop shortcut accordingly.
- When writing files with the write_file tool, the tool may append a trailing artifact line (`""", "path": ...`) after content that ends in triple quotes. After writing, verify with `python3 -m py_compile` and strip trailing garbage with a small Python snippet if needed.
- Prefer light, modern UI for dashboards: white cards on a light gray background, no pure black background, no emoji in labels.

## User preferences
- Default to base security set only: nmap, sqlmap, hydra, john-jumbo, hashcat. Do not install full pentest/metasploit unless explicitly requested.
- Install for the macOS host via Homebrew; do not generate remote-server scripts unless asked.

## Verification
```bash
for c in ansible sshpass mosh autossh tmux wget httpie jq yq nmap mtr iperf3 nc whois dig hydra sqlmap john hashcat; do
  command -v "$c" >/dev/null 2>&1 && echo "OK  $c" || echo "MISSING $c"
done
```