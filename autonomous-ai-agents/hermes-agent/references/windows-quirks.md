# Windows-Specific Quirks

Hermes runs natively on Windows (PowerShell, cmd, Windows Terminal, git-bash
mintty, VS Code integrated terminal). Most of it just works, but a handful
of differences between Win32 and POSIX have bitten us — document new ones
here as you hit them so the next person (or the next session) doesn't
rediscover them from scratch.

### Input / Keybindings

**Alt+Enter doesn't insert a newline** — Windows Terminal (and mintty) grab it
for fullscreen before prompt_toolkit sees it. Use **Ctrl+Enter** instead (the
CLI binds it to newline on Windows; raw Ctrl+J does the same, harmlessly).
To inspect how your terminal reports a keystroke, run
`python scripts/keystroke_diagnostic.py` from the repo root.

### Config / Files

**HTTP 400 "No models provided" on first run** — `config.yaml` was saved with
a UTF-8 BOM (Notepad does this). Re-save as UTF-8 without BOM;
`hermes config edit` writes correctly.

### `execute_code` / Sandbox

**WinError 10106** from the sandbox child process — it can't create an
`AF_INET` socket. Root cause is usually Hermes's env scrubber dropping
`SYSTEMROOT`/`WINDIR`/`COMSPEC` (Python's `socket` needs `SYSTEMROOT` to find
`mswsock.dll`), not a broken Winsock LSP. The `_WINDOWS_ESSENTIAL_ENV_VARS`
allowlist in `tools/code_execution_env.py` covers it; if you still hit it,
echo `os.environ` inside an `execute_code` block to confirm `SYSTEMROOT` is set.

### Testing on Windows

Prepare the checkout through PM first. With its Python 3.14, build an independent
test environment at a fresh path:

```powershell
python -m pm.build_env --source . --out .venv --group dev --group test
```

The output must not exist. Before regeneration, stop its processes and explicitly
remove only that disposable environment. Do not install into the application
payload or modify a selected dependency generation.

Run the canonical runner through Git Bash:

```bash
scripts/run_tests.sh tests/foo/test_bar.py -v --tb=short
```

The runner discovers `.venv/Scripts/python.exe`, clears credentials and
`PYTHONPATH`, and isolates each test file. For an external test environment,
set `HERMES_PYTHON` to its Python executable. PM shell activation alone does
not supply pytest after the runner clears `PYTHONPATH`.

### Path / Filesystem

**Line endings.** Git may warn `LF will be replaced by CRLF`. Cosmetic — the
repo's `.gitattributes` normalizes. Don't let editors auto-convert committed
POSIX-newline files to CRLF.

**Forward slashes work almost everywhere.** `C:/Users/...` is accepted by
every Hermes tool and most Windows APIs. Prefer forward slashes in code
and logs — avoids shell-escaping backslashes in bash.

### Terminal / Working Directory

**Stale cwd after moving/deleting the working directory.** The terminal tool
persists its shell cwd across calls. If that directory is moved or deleted
(e.g. you `mv` a clone out of scratch), every subsequent command fails with
`cd: ... No such file or directory` (exit 126) before it runs — even with
`workdir` set, the shell still tries to cd into the dead path first. Recovery:
run a command via `execute_code` (Python, cwd-independent) to do the work, or
open a fresh terminal session; do not keep retrying shell commands in the
dead cwd. Prefer doing filesystem moves with `execute_code`/`shutil` when the
target directory is the current cwd.

**Background processes and output capture.** A foreground command that exceeds
its timeout is killed, so anything that must stay alive (device-flow auth,
long builds) must run with `background=true`. To read its output, redirect to
a file under `$TMPDIR`/scratch and `cat`/read it in a follow-up call — do
not rely on capturing background stdout directly. Example:

```bash
# start, keep alive
bash -c 'gh auth login --web --git-protocol https > "$HOME/AppData/Local/hermes/cache/scratch/gh_auth.log" 2>&1'
# later
cat "$HOME/AppData/Local/hermes/cache/scratch/gh_auth.log"
```

