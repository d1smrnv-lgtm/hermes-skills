---
name: server-diagnostics
description: "Diagnose web-facing servers and enterprise apps remotely."
---

# ManageEngine Log Path Distinction

ManageEngine products (Endpoint Central / UEMS / DesktopCentral / Application Control Plus) have two entirely separate log trees. Reading from the wrong one gives misleading diagnostics.

## Agent Logs — Endpoint Behavior

Path: `<shared_root>/logs/` (e.g., `\\S17\logs\` on Windows servers)

These logs come from the **UEMS_Agent** installed on every managed endpoint. They report what endpoints are doing:

| File | What it shows |
|------|--------------|
| `user4.log`, `agent-*.log` | Per-user agent operations (USB events, app control, DLP scans) |
| `CRU_Reporter.log` | Crash reporting status per endpoint |
| `EdlpUtilProcess.log` | Endpoint Data Loss Prevention process state |
| `EDRDCLogger.log` | Endpoint detection & response data collection |
| `Internet_Connection.log` | Whether endpoints can reach the server |
| `uemsServiceEvents.log` | Periodic service health checks from agents |

These do **NOT** show whether the server's web interface works. An endpoint can log perfectly while the server auth module crashes.

## Server Logs — Service Health

Path: `<install_dir>\bin\` or `<shared_root>\logs2\` (varies by deployment)

These logs come from the **JVM-based server process**:

| File | What it shows |
|------|-------------- |
| `wrapper.log` | Startup sequence — which modules loaded/started, JVM warnings, startup duration. This is THE primary diagnostic file. |
| `application-server.log` | Tomcat-level request handling, servlet errors, connection pool issues. |
| `build-history.txt` | Version/build info from last deployment. |
| `asyncproclog*.log` | Background task processing queue status. |
| `quarantinequeuelog*.txt` | Patch/quarantine processing queues. |

### Reading wrapper.log

Key sections to scan:
1. **"Creating Services"** — modules being instantiated. A missing module here means it won't load.
2. **"Starting Services"** — check for `[FAILED]` status instead of `[STARTED]`.
3. **"Server started in :: [<N> ms]"** — normal startup is 60-180 seconds. >5 minutes suggests DB migration or GC pressure.
4. **WARNING lines** — reflective access warnings are normal; OutOfMemory pre-filters are not.
5. **Error codes after startup** — look for `Exception`, `SQLException`, or `Connection refused` near the bottom.

## Quick Decision Tree

```
Web UI fails?
├── Can curl https://SERVER:PORT/ respond?
│   ├── Yes → Stack is running. Check wrapper.log for FAILED modules.
│   └── No → Network issue or service stopped. Check services.msc.
├── Page loads but hangs at login?
│   ├── JS/CSS assets all load? (check with Python urllib)
│   │   ├── Yes → API/auth failure. wrapper.log for IAM/AuthenticationService status.
│   │   └── No → Missing assets. Re-deploy or check URL rewrite rules.
└── Specific feature broken?
    ├── Feature-specific log in <shared>/logs2/<feature>.log
    └── General error → application-server.log tail
```

## Common Error Codes

| Code | Meaning |
|------|--------|
| `IAM0027` | Invalid URL to authentication layer — IAM module crashed or DB unreachable |
| `SQLException` | Database connection failed — check PostgreSQL port 5432 |
| `OutOfMemory` | JVM heap exhausted — increase MaxJavaHeapSize in wrapper.conf |
| Port `8019` closed | Internal Tomcat listener down — check if `DesktopCentral` service is running |
| Port `8383` responds but 405 | Reverse proxy configured but SSL/TLS misconfigured |
