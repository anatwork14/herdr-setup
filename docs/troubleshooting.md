# Troubleshooting and safety

## Codex fails to send native Herdr prompts

**Symptom:** `herdr agent prompt ...` invoked inside Codex returns exit code 1 and `Operation not permitted`, while the same recipient can receive messages.

**Evidence-based diagnostic sequence:**
1. Check `HERDR_ENV` and `HERDR_SOCKET_PATH` **without disclosing sensitive paths publicly**.
2. Verify socket owner/mode and successful Herdr ping from an authorized unsandboxed session.
3. Verify the same socket operation fails under the Codex sandbox.
4. Check which Codex executable is actually running (VS Code extension bundle vs stand-alone CLI), its version and active sandbox policy.
5. Confirm whether any granular Unix socket allowlist is actually effective under that build; do not infer support from documentation or binary strings alone.
6. Retain **lead-mediated RELAY** unless a genuinely least-privileged solution is demonstrated and authorized.

On the tested host a profile-specific allowlist failed; broad `network.enabled=true` opened other sockets and internet, which was rejected as unsafe. Do **not** propose `dangerously_allow_all_unix_sockets` as a default fix.

## Global instructions not loaded

- **Codex:** inspect `CODEX_HOME`, `AGENTS.override.md`, `~/.codex/AGENTS.md`, startup context logs and new session behavior.
- **Antigravity:** inspect `~/.gemini/AGENTS.md`, installed agent version, startup rules and controlled marker behavior outside parent project.
- **Claude:** inspect `~/.claude/CLAUDE.md`; parent-directory project instructions do not establish global inheritance.
- Do not rely on an agent's guessed filename. In the recorded tests, some claimed source paths did not exist.
- Recreate **temporary** sessions only after authorized changes and verify fresh-session policy injection.

## Trust prompt blocks agent startup

A new folder may trigger a “Do you trust this folder?” dialogue. Verify exact path and contents, obtain explicit folder-specific approval, and accept through normal platform UI only when authorized. Do not globally disable trust or automatically click prompts. If blocked, report partial validation rather than bypassing checks.

## Misleading agent names or duplicates

Herdr names can differ from actual process kinds. Use `herdr agent list` and inspect live pane data. Correct aliases only after verification. On second bootstrap, skip already-initialized agents. Occupied pane/name errors are not permission to kill the existing process.

## Worker returns idle but no deliverable

Idle is not a completion certificate. Inspect actual output and artifacts, tests, Git status, artifact hashes and reviewer notes. Use bounded waits; differentiate delivery acknowledgment, task execution and independent verification.

## Lead crashes or context resets

Native Herdr TUI does not inherently guarantee a persistent task queue, scheduled runner or automatic recovery. Store task state under the **project**, and consider a separately authorized supervisor for unattended resumption. Resume by reconciling existing outputs rather than replaying actions blindly.

## Security checklist

- Do not print or commit sockets, secrets, home-directory contents or account credentials.
- Do not expand Codex's sandbox/approvals or network to fix nonessential peer messaging.
- Do not auto-approve folder trust.
- Do not give untrusted workspaces access to privileged Herdr agent-control facilities.
- Do not overwrite global instruction files; diff, merge and verify.
- Do not allow agent-generated prompts to alter authorized goals or project boundaries.
- Do not call a relay cycle a direct peer-to-peer communication success.
