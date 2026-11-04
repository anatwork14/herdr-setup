# One-time installation and per-workspace bootstrap

## Prerequisites

- Linux host with compatible Herdr TUI, native agent CLIs and authenticated sessions.
- Herdr agent-automation commands accessible to the lead; `HERDR_ENV=1` observed in Herdr-launched agents.
- `gh` optional for cloning this repository. `node`/`npx` needed for the official skill installation example.
- A verified, trusted project directory. Never auto-click folder trust prompts.

```bash
gh repo clone anatwork14/herdr-setup
cd herdr-setup
herdr --version
herdr agent --help
npx skills add herdrdev/herdr --skill herdr -g
```

The skill's installation support varies by agent platform. Confirm each platform actually loads it; installing a skill is **not** sufficient evidence of successful policy discovery.

## Global instruction files

Before any modification, inspect existing files, environment overrides, and the current platform documentation. **Merge; never blindly overwrite.** Create backups or a diff for approval.

- **Codex:** `${CODEX_HOME:-$HOME/.codex}/AGENTS.md`; also check for `AGENTS.override.md`. Tested host had no `CODEX_HOME` and an empty global file. Use canonical policy from [../AGENTS.md](../AGENTS.md), especially the RELAY rule. Do not change Codex sandbox settings.
- **Antigravity:** `~/.gemini/AGENTS.md` was confirmed by a before/after behavior test in the reported installed version. Treat this as version-specific; verify actual rule loading after update.
- **Claude:** `~/.claude/CLAUDE.md` was confirmed via startup diagnostics on the reported host. Avoid relying on a parent project's `AGENTS.md` for arbitrary directories. Make the global instructions conditional on `HERDR_ENV=1`.

Platform instruction wording can be brief, but must **self-contain** the key operating contract: Herdr discovery, roles, Codex socket limitation and relay format, permission boundaries, evidence-first verification, and workspace isolation. A pointer to `~/workspace/AGENTS.md` alone is not portable outside that directory.

### Suggested platform preamble

```text
Herdr coordination policy, version 2026-11-04.
If HERDR_ENV=1, use native Herdr for discovery and control.
Claude is the default lead, Codex a receiving worker, Antigravity a second worker.
On the tested Linux host Codex cannot initiate Herdr Unix-socket calls:
Codex must produce RELAY / Recipient / Task ID / Message type /
Content / Required action / END RELAY in its normal response.
Claude relays and returns answers. Do not loosen Codex sandbox.
Keep project contexts separate; verify results, not idle states.
Never auto-approve folder trust or broader permissions.
```

To avoid drift, maintain this source repository's `AGENTS.md` as canonical and review changes to each installed user-level copy. Installing it into user home files is an **authorized action requiring environment-local execution**, not something that GitHub commits can accomplish.

## Per-workspace setup (repeat on each project)

1. Create/enter the project directory and a Herdr workspace with that directory as working root. Inspect `herdr workspace --help` for the current version's syntax.
2. Review and manually approve that project's trust prompt as appropriate. No blanket trust grant.
3. Inspect live sessions with `herdr agent list`, check actual agent kinds/pane names, and correct aliases only after confirmation.
4. Start or reuse Claude lead, Codex worker and Antigravity worker using the locally supported pane/agent commands. A reviewer slot is optional, on demand.
5. If Herdr reports a duplicate name or occupied pane, **reuse or inspect**, not blindly restart.
6. Check startup policy loading from logs/controlled behavior, not only an agent's guessed source path.
7. Test a read-only message and independent verification per [validation.md](validation.md).
8. Assign a project mission with repository, permissions, expected artifacts, acceptance criteria and review gates.

### Mission handoff template

```text
Mission: <goal>
Workspace/repo: <absolute directory and repository>
Allowed writes: <specific paths/branches>
Forbidden actions: <permission, secrets, production, destructive scope>
Acceptance criteria: <tests + artifacts + review>
Autonomy: plan/delegate/implement/test/review/fix within scope.
Use native Herdr lead-mediated coordination. Codex uses RELAY blocks.
Escalate trust prompts, broader privileges and irreversible actions.
```

## Idempotency and recovery

- Do not create an agent if a suitable named agent is already live.
- Treat pane aliases as ephemeral; re-discover them on every new session.
- Record project task state inside the project's own ledger.
- For interrupted tasks, check artifact and Git status before retrying; do not rerun non-idempotent commands.
- New sessions may need to reload global instructions. A workspace-specific trust prompt may recur.
- Native Herdr agent control does **not** by itself provide a persistent unattended scheduler/restart service.

## Portability cautions

The original tested machine used Herdr 0.9.5, and Codex default sandboxing on Linux. The observed socket path was `$HOME/.config/herdr/herdr.sock`; do **not** hardcode it on another machine. Always inspect `HERDR_SOCKET_PATH` and permissions. Do not publish private absolute paths, tokens, or real task content in a public repo.
