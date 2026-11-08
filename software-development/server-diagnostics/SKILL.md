---
name: server-diagnostics
description: "Diagnose web-facing servers and enterprise apps remotely."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [diagnostics, troubleshooting, server, enterprise, manageengine, infrastructure]
    related_services: [manage-engine-endpoint-central, endpoint-central]
---

# Server Diagnostics

Remote diagnosis of web-facing servers and enterprise applications without direct console access.

## When to Use

Use when diagnosing a web-facing server or enterprise application where you lack interactive console access:
- Web UI hangs at login / spinner / blank page — service is running but something downstream failed
- All REST calls return a uniform error code (auth module broken, DB unreachable)
- You can reach the host by HTTP(S) but cannot get into the admin panel
- Enterprise software stack appears up (ports open) but specific features fail

**Do NOT use for:** local development debugging, frontend-only issues (missing font, CSS mismatch), or network routing problems between client and server.

## Always-on rules

- **Explain every action before executing on a user's server.** Never type commands blindly into admin sessions.
- **Find the failing component and root cause before attempting recovery.** A service restart can restore availability, but is not a root-cause fix; report it as a mitigation unless evidence identifies and resolves the cause.
- **Verify the same user-visible symptom after any change.** Re-probe the original URL/port and check post-change logs; do not report success from a command exit code alone.
- **Start from outside-in.** Network → ports → HTTP responses → JS/CSS assets → logs. Never jump to logs before confirming the stack actually serves pages.
- **Distinguish agent-side from server-side logs.** Agent logs show endpoint behavior; server-side logs (`wrapper.log`, `catalina.out`, `application-server.log`) show the actual service health. Reading the wrong set wastes time.
- **Treat read-only share access as distinct from remote execution rights.** Before a remediation attempt, confirm the approved remote-control channel and required authorization; if access is denied, stop and request an authorized admin session or explicitly provisioned rights rather than trying alternate credential paths. Never collect passwords in chat or type them into agent tools.
- **Keep credential entry visibly interactive.** Use an OS credential prompt, confirm the user can see it, and never leave a hidden process waiting for a secret; stop the hidden process and relaunch through a visible interactive window if needed.
- **Capture native command streams in remote PowerShell.** A `.bat`/`.cmd` wrapper can emit Java diagnostics on stderr; with `$ErrorActionPreference = 'Stop'`, PowerShell may surface that as a terminating `NativeCommandError` and abort before later recovery or verification steps. Capture stdout/stderr, record `$LASTEXITCODE` immediately, and verify the actual service/HTTP state rather than inferring success or failure from that message alone.
- **When browser tools are unavailable, use Python + urllib or curl equivalents.** They carry less context overhead and survive kernel resets better than driving a headless browser for diagnostics.

## Procedure

### Phase 1: Connectivity and Reachability

Check the server responds and measure latency:

```bash
curl -ks --connect-timeout 5 https://SERVER:PORT/
```

Record response time (< 100ms = fast, > 1s = suspect, timeout = unreachable).
Note the Server header (nginx, Apache, Tomcat, IIS) and SSL certificate details.

If HTTPS port exists, test both HTTP and HTTPS to distinguish reverse-proxy setup from plain backend.

### Phase 2: Web Stack Health

Map the URL structure — where does the root redirect?

```bash
curl -ksIL http://SERVER/  # follow redirects, show headers only
curl -ksIL https://SERVER/client
```

Extract all JS and CSS resource URLs from the HTML:
```python
import re
html = resp.read().decode('utf-8')
scripts = re.findall(r'<script[^>]+src="([^"]+)"', html)
css = re.findall(r'link[^>]+href="([^"]+\.css)["\s]', html)
```

Verify each resource loads individually — a modern SPA will fail silently if even one chunk returns 404.

### Phase 3: API Layer

Probe known API paths for the target application:

```bash
curl -ks https://SERVER/api/status
curl -ks https://SERVER/serverInfo
```

Error codes tell the story:
- `IAM0027` / auth-related = frontend loaded but backend auth chain broken
- `404` on API = route not registered or module didn't start
- `500` = server crashed handling the request
- Empty body + correct headers = upstream timed out or proxy dropped connection

### Phase 4: Service Logs

Locate server-side logs. For enterprise Java apps managed by Apache Commons Daemon Wrapper (Tomcat-like), find `wrapper.log` — it shows:
- Which modules loaded (check names match expected components)
- Which modules failed to start (look for `[FAILED]`)
- Total startup time (long startup = DB migration, index rebuild, or memory pressure)
- JVM warnings (reflective access errors, OutOfMemory pre-filters)

For other stacks:
- PHP/Laravel: `storage/logs/laravel.log`
- Node: `pm2 logs` or stdout capture
- Docker: `docker logs CONTAINER`
- Windows services: Event Viewer → Applications and Services Logs → AppName

### Phase 5: Escalation to Local Access

If remote checks pass but web UI still fails → issue is client-side (browser cache, CORS, mixed content). If server reports errors → fix based on `wrapper.log` output.

When you have local/admin access:
1. Explain the command and its purpose before running
2. Check service status first (`Get-Service *AppName*` on Windows, `systemctl` on Linux); discover the installed service name instead of guessing it
3. Check bound ports (`netstat -ano | findstr :PORT`)
4. Read recent logs across the reverse proxy, application process, and database around the failure time; correlate evidence before choosing a fix
5. Verify database connectivity — most enterprise apps depend on PostgreSQL/MySQL
6. Apply the narrowest evidence-backed recovery, then repeat the original external probe and inspect fresh logs; distinguish a temporary restart recovery from a durable root-cause fix

## Common Patterns

| Symptom | Likely Cause |
|---------|-------------|
| Root redirects to `/login` or `/client`, page hangs at spinner | Auth module failed, DB unreachable, or IAM middleware error |
| Static assets (JS/CSS) return 404 | Build mismatch, deployment incomplete, or URL rewrite rules wrong |
| All REST calls return same error code | Routing layer works but specific module hasn't initialized |
| Response time spikes (> 2s) | Database query, index rebuild, or memory pressure causing GC pauses |
| Server header shows nginx but app returns JSON errors | Reverse proxy configured but app's internal port exposed behind it |

## ManageEngine ServiceDesk Plus Specifics

### Error Log Investigation

ServiceDesk Plus stores system errors accessible via the web UI at:
```
https://<host>:<port>/ErrorLogDetails.do?errorId=<id>
```
This page requires authentication. If credential-based login fails, use SAML SSO:
1. Navigate to the error log URL (redirects to login page)
2. Click "Войти с помощью единого входа SAML" (Login via SAML SSO)
3. Complete ADFS/IdP authentication
4. The error log page loads with full details

The page contains a `textarea#systemLogMsg` with the full error message (not visible in `innerText` — extract from HTML).

### Active Directory Import Errors

Common error pattern: `AD import failed in domain - <DOMAIN>. Failure reason - The Given Servers are Not operational. Full sync`

Diagnosis steps:
1. Extract the domain name from the error message
2. Verify DNS resolution: `nslookup <DOMAIN>` and `nslookup _ldap._tcp.dc._msdcs.<DOMAIN>`
3. If DNS fails → the domain name is wrong or DNS is misconfigured
4. Check network connectivity to AD servers on ports 389 (LDAP), 636 (LDAPS), 3268 (GC)
5. Verify AD credentials in ServiceDesk Plus admin settings (Admin → Active Directory)
6. Check ServiceDesk Plus server logs for detailed connection errors

The "Full sync" suffix indicates a complete synchronization attempt (not incremental).

**Pitfall — LDAPS port 636 often closed:** The error "The Given Servers are Not operational" frequently occurs when ServiceDesk Plus is configured to use LDAPS (SSL on port 636) but the domain controller does not have LDAPS enabled. Verify port 636 is open (`bash -c 'echo > /dev/tcp/<dc-host>/636'`); if closed, either enable LDAPS on the DC (install a certificate for AD DS) or switch ServiceDesk Plus AD settings to plain LDAP (port 389).

**Pitfall — domain name vs. server hostname:** The `<DOMAIN>` in the error message is the AD domain name (e.g., "AT"), not necessarily a resolvable hostname. The actual domain controller hostname (e.g., `s01at.at.local`) is configured separately in ServiceDesk Plus AD settings. Always check the admin settings for the configured server name, not just the domain name from the error.

**Pitfall — all domains failing simultaneously:** If multiple domains (e.g., 6 domains) all fail with "The Given Servers are Not operational" at the same time, the issue is systemic — not per-domain. Check: (1) network connectivity from the ServiceDesk server to ALL DCs (not just from your workstation), (2) AD service account credentials (password may have expired), (3) local firewall on the ServiceDesk server blocking outbound LDAP. A single domain failing while others work indicates a per-domain configuration issue.

**Pitfall — error code 0x8007203a:** This is the Windows LDAP error code behind "The Given Servers are Not operational". It means the LDAP client could not establish a connection to the DC. Common causes: (a) DC hostname doesn't resolve from the ServiceDesk server, (b) firewall blocks port 389/636, (c) AD service account locked/expired, (d) DC is actually down.

**Log file location:** ServiceDesk Plus server logs are often on a separate log share (e.g., `\\\\S08\\logs\\`). Look for `serverout*.txt` files. Search for "Not operational" or "given_servers_not_opertional" to find the relevant entries. The log shows DomainId and DomainName for each failing domain, plus the full Java stack trace.

**SAML SSO authentication flow (ADFS):** When credential-based login fails, use SAML SSO:
1. Navigate to the error log URL (redirects to login page)
2. Click "Войти с помощью единого входа SAML" (Login via SAML SSO)
3. On the ADFS login page, fill `#userNameInput` (format: `DOMAIN\username`) and `#passwordInput`
4. Submit the form (`document.querySelector('form').submit()`)
5. Browser vault can fill the password field; fill the username manually
6. After successful auth, the error log page loads with full details

The page contains a `textarea#systemLogMsg` with the full error message (not visible in `innerText` — extract from HTML via `document.documentElement.outerHTML`).

ManageEngine products (Endpoint Central / UEMS / DesktopCentral) use:
- **Apache Commons Daemon Wrapper** → `wrapper.log` in `<install>/bin/` or share path
- **Ports vary by build and generated configuration.** Inspect the active `server.xml` and web-server config before probing: one observed Endpoint Central layout used nginx TLS on 8383 proxying to Tomcat HTTP at `127.0.0.1:8000`, while 8019 was an AJP connector; do not assume 8019 is the web UI backend.
- **Agent logs** at `<shared>/logs/` show endpoint behavior, NOT server health
- **Server logs** in `<shared>/logs2/` or `<install>/logs/` contain actual startup diagnostics
- **IAM0027** = authentication module error — usually means DB connection or config issue
- **Services named**: `DeskStopClientManagerPlus` or similar with `-Service` suffix