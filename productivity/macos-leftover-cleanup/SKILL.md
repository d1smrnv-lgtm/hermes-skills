---
name: macos-leftover-cleanup
version: 1.0.0
description: Use when cleaning leftover files of uninstalled macOS apps.
author: user
license: MIT
metadata:
  hermes:
    tags: [macos, cleanup, leftover, uninstall]
    related_skills: []
---

# macOS Leftover Cleanup

## When to Use

- User asks to clean residual/leftover files of deleted/uninstalled macOS programs.
- User wants a categorized list of leftover files across ~/Library and /Library.
- User wants to selectively delete leftover app data.

## Procedure

1. Gather installed apps and bundle IDs from `/Applications`, `/System/Applications`, `~/Applications`, `/System/Library/CoreServices` (read `Contents/Info.plist` → `CFBundleIdentifier`).
2. Scan these locations for entries not matching any installed app name/bundle ID and not `com.apple.*` / `group.com.apple.*`:
   - `~/Library/Application Support`, `~/Library/Preferences`, `~/Library/Caches`, `~/Library/Containers`, `~/Library/Group Containers`
   - `/Library/Application Support`, `/Library/Preferences`
   - `~/Library/LaunchAgents`, `/Library/LaunchAgents`, `/Library/LaunchDaemons`
3. Skip entries that no longer exist (already deleted). Exclude native Apple apps and system components.
4. Present as categorized list with FULL DISK PATHS (not just names), grouped by location, with per-item and per-category delete buttons. Optimistically remove item from UI on click.
5. Delete on request: `rm -rf` for user-owned; `sudo rm -rf` for root-owned `/Library` paths.

## Pitfalls
- `~/Library/Containers` and `~/Library/Group Containers` entries contain `.com.apple.containermanagerd.metadata.plist` protected by SIP — `rm`/`sudo rm` fails with "Operation not permitted". Must be deleted manually in Finder (or via Full Disk Access).
- Some files in `~/Library/Application Support` (e.g. `Microsoft Edge`) are TCC-protected — same limitation.
- `/Library/*` entries are root-owned → use sudo.
- Always show full paths so the user knows exactly what's being removed.
- Re-scan before presenting to avoid listing already-deleted items.
