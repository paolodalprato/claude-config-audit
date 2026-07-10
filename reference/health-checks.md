# Health Checks — Operational Verification of MCP Servers

A configuration inventory tells you what is declared; health checks tell
you what can actually run. Execute these checks during Phase 2, after the
inventory, when filesystem access is available. Without filesystem access,
mark each server as "unverifiable" and fall back to asking the user.

**Cardinal rule: never launch the MCP server itself to test it.** Starting
a server may spawn long-lived processes, trigger downloads, or consume API
quota. Verify prerequisites (paths, packages, env vars, backing services),
not runtime behavior.

## Checks by install type

Read each server entry (command, args, env) from the config file and apply
the checks matching its install type.

### Local installs (`node <path>/dist/index.js`, `python <path>/server.py`)

- **Script path exists**: the file referenced in args must exist on disk.
  A missing path is the most common breakage after moving or renaming a
  project folder. Status: broken.
- **Runtime resolvable**: the command (`node`, `python`, `uv`) must be
  findable — `Get-Command node` (Windows) / `which node` (macOS/Linux).
  Status if missing: broken.
- **Relative paths in args**: servers that read relative paths at startup
  break when launched from a different working directory. Flag for review.

### npx / uvx managed packages

- **Package still published**: `npm view <package> version` (requires
  network). A renamed, deprecated, or unpublished package fails at every
  session start, silently. Status: broken.
- **Cache presence** (offline alternative): check `~/.npm/_npx` for a
  cached copy. A cached package works offline but may be stale.
- **Version pinning**: unpinned packages (no version, or `@latest`) can
  break without any local change. Flag for user awareness, not removal.

### Docker-based servers

- **Daemon running**: `docker info` succeeds only when the daemon is up.
  On Windows, `Get-Process "com.docker*"` is a cheaper first check.
- **Image present**: `docker image inspect <image>` — a missing image
  means a pull (and its wait) at first use.
- A server whose daemon is down at session start contributes startup
  latency and error noise for that session. Status: degraded.

### Environment variables and credentials

- **Missing or empty values**: an env var declared in the entry with an
  empty value. Status: misconfigured.
- **Placeholder values**: `YOUR_API_KEY`, `xxx`, `changeme`, `<insert…>`.
  Status: misconfigured.
- **Never print values.** Report only the variable name and whether it
  looks set. The redaction rules from SKILL.md apply.
- **Expired credentials are not statically detectable**: for API-based
  servers with plausible keys, ask the user when they last saw the server
  work. Status: unverifiable.

## Backing services

Servers that depend on an external service are healthy only while that
service runs. Common cases:

| Service | Windows | macOS / Linux |
|---------|---------|---------------|
| Docker | `docker info` | `docker info` |
| Ollama | `Get-NetTCPConnection -LocalPort 11434` | `curl -s localhost:11434` |
| GIMP | `Get-Process gimp*` | `pgrep -i gimp` |
| Local databases | port check on the configured port | `lsof -i :<port>` |

A server whose backing service is down is not automatically a removal
candidate: the user may start the service on demand. Status: degraded,
with an explicit note on the dependency.

## Health classification

| Status | Meaning | Typical recommendation |
|--------|---------|------------------------|
| OK | all applicable checks pass | keep |
| Degraded | works only when a backing service is up | keep, document the dependency |
| Broken | command, script path, or package missing | fix the path or remove the entry |
| Misconfigured | missing or placeholder env vars | complete the config or remove |
| Unverifiable | requires network or user knowledge to assess | ask the user |

Feed the resulting status into the MCP inventory table of the report
(see `report-template.md`).

## OS command reference

All checks are read-only. Detect the OS from path separators and adapt.

| Check | Windows (PowerShell) | macOS / Linux |
|-------|----------------------|---------------|
| Command in PATH | `Get-Command <cmd>` | `which <cmd>` |
| File exists | `Test-Path <path>` | `test -f <path>` |
| Port listening | `Get-NetTCPConnection -LocalPort <port>` | `lsof -i :<port>` |
| Process running | `Get-Process <name>` | `pgrep <name>` |
| npm package published | `npm view <pkg> version` | `npm view <pkg> version` |
| Docker image present | `docker image inspect <img>` | `docker image inspect <img>` |

## Ordering and reporting

Run checks in cost order: file existence first (instant), then PATH
lookups, then process/port checks, then network lookups (`npm view`)
last and only when the cheaper checks pass. Group results by status in
the report, and for every broken or misconfigured server include the
exact failing check, so the user can fix instead of removing.
