# Herdr setup: native multi-agent coordination

Reproducible operating guide for **Claude Code (lead), Codex (worker), and Antigravity/agy (worker)** using **native Herdr TUI**. This repository records the **2026-11-04 tested configuration**; it is a playbook, not a turnkey daemon or proof that every Herdr release supports the same CLI flags.

**Status:** CROSS-WORKSPACE READY — **lead-mediated mode**. Codex receives Herdr prompts but, under the tested Linux sandbox, cannot initiate calls to the Herdr Unix socket. Claude relays messages on its behalf. This is deliberate; do not weaken the Codex sandbox to chase direct peer-to-peer messaging.

## Quick start

```bash
gh repo clone anatwork14/herdr-setup
cd herdr-setup

# Check prerequisites and CLI syntax on THIS host:
herdr --version
herdr --help
herdr agent --help
herdr pane --help

# Install the official Herdr skill globally for supported agent CLIs:
npx skills add herdrdev/herdr --skill herdr -g

# Start/attach to Herdr using its installed TUI interface:
herdr
```

Install the agent CLIs independently and authenticate using their official setup flows. Commands, arguments, and available agent kinds can differ by Herdr version. **Do not copy example pane IDs or assume a launch command succeeded; inspect actual agents first.**

### Configure global agent instructions (one time per user/machine)

Use the canonical [agent policy](AGENTS.md) as the *source of truth*. Merge the policy into the appropriate platform user-level instruction files **without overwriting existing content**:

| Agent | Verified path in tested host | Notes |
| --- | --- | --- |
| Codex | `~/.codex/AGENTS.md` | Check `CODEX_HOME` and `AGENTS.override.md` first |
| Antigravity | `~/.gemini/AGENTS.md` | Verify this path against installed Antigravity version |
| Claude Code | `~/.claude/CLAUDE.md` | Global instructions; do not rely only on parent `AGENTS.md` |

The templates and installation safeguards are in [docs/bootstrap.md](docs/bootstrap.md). New sessions must be started after installing/updating policies. **Global instructions are not global live memory or task sharing.** Keep project data inside the corresponding workspace.

### New workspace

1. Start or select a Herdr workspace whose working directory is the **specific project**.
2. Approve the IDE/agent's folder-trust prompt only after verifying the path and contents; never disable trust checks globally.
3. Discover agents with `herdr agent list`; check actual kinds, pane IDs, names and status.
4. Reuse appropriately scoped agents or start missing ones using the installed CLI help. Rename misleading aliases only after checking identities.
5. Set Claude as lead, Codex as a receiving implementation worker, Antigravity as secondary worker. Reviewer is started **on demand**.
6. Give Claude a project mission with scope, acceptance criteria, permissions, and repository location. Claude coordinates workers and returns verifiable artifacts.
7. Validate a harmless mediated relay cycle before relying on a new setup.

**Illustrative commands** (confirm exact flags with `--help`):

```bash
herdr agent list
herdr agent prompt codex-worker "Report the line count of README.md and include a RELAY request for agy-worker"
herdr agent read codex-worker --source recent-unwrapped --lines 120
# Claude sends a separate prompt to agy-worker, then returns the result to Codex.
```

Do **not** run `herdr agent prompt` from sandboxed Codex on this host: it fails with `Operation not permitted`. Codex emits a structured `RELAY` block in its own response instead.

## Docs

- [AGENTS.md](AGENTS.md) — canonical agent operating contract and autonomous lead loop
- [docs/bootstrap.md](docs/bootstrap.md) — global policy installation and workspace bootstrap
- [docs/architecture.md](docs/architecture.md) — communication topology, relay protocol and isolation
- [docs/validation.md](docs/validation.md) — test matrix, evidence standards and observed results
- [docs/troubleshooting.md](docs/troubleshooting.md) — Codex Unix-socket sandbox, trust prompts and diagnostic workflow

## Proven vs not proven

**Observed in the reported 2026-11-04 environment:** all three platform policy inheritance paths in fresh sessions, a read-only Codex → Claude → Antigravity → Claude → Codex relay, independent line-count verification, idempotent bootstrap for tested sessions, and isolation from other temporary workspace data. Original agents remained untouched.

**Not proven:** fully direct peer-to-peer communication; fault-tolerant daemon operation across crashes; full autonomous code implementation/review/repair at production scale; general compatibility across every OS, model or agent version. Any new host should rerun the checks in [docs/validation.md](docs/validation.md).

## Security posture

- Keep Codex sandbox/default restrictions; **never enable** `dangerously_allow_all_unix_sockets` or broad `network.enabled=true` solely for Herdr messaging.
- Native Herdr socket access can control other agents; treat it as privileged. Do not casually expose it to untrusted workspaces.
- Ask for approval for new trust boundaries, destructive operations, secrets and permission expansion. No auto-approval of folder-trust prompts.
- Keep per-project Git worktrees/ownership and separate logs and evidence.
- Treat the observed paths and aliases as **examples**, not portable literals.

Official references: [Herdr agent automation](https://herdr.dev/tui/docs/agent-automation/), [Herdr agent skill](https://herdr.dev/tui/docs/agent-skill/), [Herdr socket API](https://herdr.dev/tui/docs/socket-api/).
