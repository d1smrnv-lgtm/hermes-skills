---
name: macos-disk-cleanup
description: "Use when freeing macOS disk or finding leftover app files."
version: 1.0.0
author: Nous Research
license: MIT
platforms: [macos]
metadata:
  hermes:
    tags: [macos, disk, cleanup, cache, leftover, trash]
    category: productivity
    related_skills: []
---

# macOS Disk Cleanup / Leftover App Files

## When to Use

- User asks to free disk space on macOS.
- User asks to find leftover files from removed/uninstalled apps.
- User wants a list of junk/cache candidates before deleting.

## Rule: never delete directly — move to ~/.Trash first

Always move candidates to `~/.Trash/` (or `sudo mv` for system paths) and confirm sizes before/after. The user decides what to delete; present a categorized list with sizes first.

## Locations to scan

User-level:
- `~/Library/Application Support/` — app data dirs
- `~/Library/Caches/` — caches
- `~/Library/Containers/` — sandboxed app data
- `~/Library/Preferences/` — plist + dirs
- `~/Library/Logs/`
- `~/Downloads/`, `~/Library/iTunes/iPad Software Updates/` (old IPSW)
- `~/Library/Application Support/com.apple.wallpaper/aerials/videos/` (Aerials)

System-level (need sudo):
- `/Library/Application Support/`, `/Library/Caches/`

## Identifying leftovers

1. List installed apps: `ls /Applications` and `~/Applications`.
2. Compare directory names in the locations above against installed apps (by app name and bundle id).
3. Anything with no matching installed app is a leftover candidate. Include size via `du -sh`.
4. Also flag large caches of still-installed apps and old OS images as optional.

## Moving to Trash — pitfalls

- Name collisions in `~/.Trash/` (e.g. `OpenIDE` exists from both App Support and Caches): rename on move, e.g. `OpenIDE_cache`, `OpenIDE_logs`.
- Some paths are TCC-protected even with sudo (`Operation not permitted`): `~/Library/Application Support/discord`, some `~/Library/Containers/com.adguard.*`. These need Full Disk Access granted to Terminal (System Settings → Privacy & Security → Full Disk Access) before they can be moved; otherwise tell the user to delete manually in Finder.
- Containers under `~/Library/Containers/` are often protected; try sudo, fall back to instructing the user.
- Prefer `mv` over `rm`; verify with a post-move existence check.

## Output format

Categorized markdown table: path + size + note. Group: (1) leftovers from removed apps, (2) large caches/temp, (3) big downloads. State total potential freed space. Do not auto-delete.