# Architecture and communication

## Supported topology (tested)

```text
                   human mission / authorization
                               |
                         Claude lead
                      (Herdr CLI control)
                       /            \
                native prompt    native prompt
                    /                 \
             Codex worker      Antigravity worker
             (receives)        (can send/receive)
                    \                 /
                    worker outputs / relay blocks
                               |
                  Claude verifies and routes
                               |
                    reviewer (on demand)
```

This is **lead-mediated autonomy**, not universal peer-to-peer networking or shared conversation memory. Herdr's native CLI is the control plane; the lead reads outputs and passes addressed messages.

## Why Codex relays

On the reported Linux host:
- Herdr server: 0.9.5, socket at `~/.config/herdr/herdr.sock` (mode 600, same user).
- Socket ping succeeded **outside** Codex sandbox.
- The same ping under Codex sandbox failed with `PermissionError: [Errno 1] Operation not permitted`.
- A temporary `[permissions.<profile>.network.unix_sockets]` allowlist alone did **not** enable access.
- A configuration with `network.enabled=true` opened broader networking and other sockets, contrary to least privilege; proxy-constrained variant still blocked Herdr.
- VS Code extension's bundled Codex binary may differ from a separate installed CLI binary. Root cause is strongly supported as sandbox behavior; exact implementation limitation is not definitively established.

**Accepted design:** do not disable sandboxing, request all-socket access, or require Codex to call `herdr agent prompt`. Codex reports addressed relay requests in text; Claude sends via native Herdr and returns the reply. Peer communication is treated as routed, not direct.

## Message contract

```text
RELAY
Recipient: agy-worker
Task ID: X1-unique
Message type: VERIFY
Content: Count the lines of assigned test data.txt; answer LINES=<integer>.
Required action: Send the result to the Claude lead with Task ID.
END RELAY
```

Claude must:
1. Verify recipient is a live authorized agent in the correct workspace.
2. Ensure requested action lies within the mission scope.
3. Relay the exact substantive request with a unique correlation ID.
4. Capture recipient acknowledgment/results and pass them to Codex.
5. Record task state and independent evidence, e.g. `wc -l`.
6. Mark completion only when acceptance checks pass.

For long-running tasks include `message_id`, reply correlation, sender, timestamps and a ledger. Do not treat a text relay as an authorization to write files or execute unsafe actions.

## Workspace boundaries

- Global instruction policy: user-level, reused by new **sessions**.
- Herdr panes, processes, names and folder-trust grants: specific to workspace/session.
- Source code, Git worktrees, tasks, results and artifacts: per **project**.
- Agent review: independent where required. No self-certification.
- Explicit grant required for new permission scopes, secrets, cross-workspace file access or destructive operations.

## Autonomy model

Lead loop: discover → plan → delegate → execute → verify → review → repair → complete. It runs while the lead is alive and has a mission. Idle agents should not invent work. Cross-session recovery requires durable project-local state and, for unattended restarts, an **additional** supervisor; native Herdr control alone is not such a supervisor.
