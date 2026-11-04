# Validation and acceptance suite

This is an evidence-based checklist, not an executable test harness. Treat the recorded outcome as a **single host/session observation on 2026-11-04**, not universal compatibility.

## Stage A: discovery and communication

Verify real agent kind, pane ID and alias (aliases may be wrong). The observed initial aliases were swapped, then corrected without restarting processes.

| Direction | 2026-11-04 outcome | Explanation |
| --- | --- | --- |
| Claude → Codex | PASS | Native prompt received |
| Claude → Antigravity | PASS | Native prompt received |
| Codex → Antigravity | FAIL, accepted limitation | Sandbox socket permission denied |
| Antigravity → Codex | PASS | Native prompt received |
| Codex → Claude | FAIL, accepted limitation | Sandbox socket permission denied |
| Antigravity → Claude | PASS | Native prompt received |

**Do not declare six-of-six direct communication PASS.** The supported acceptance criterion is the separate **mediated** relay, not direct Codex socket operation.

## Stage B: mediated return cycle

Reported read-only runs:
- M1: Codex counted `README.md` (46 lines) and emitted a structured request to Antigravity; Claude relayed; Antigravity independently returned 46; Claude passed the result back and independently checked `wc -l`.
- X1: In the temporary workspace, Codex produced a correct `RELAY` block without being prompted with the format. Antigravity answered `LINES=7` for `data.txt`; Claude independently verified the file had seven lines; Codex acknowledged `MATCH`.

**Required new-host test:** assign a harmless, workspace-local read-only task, require a Codex `RELAY` message, relay to an authorized second agent, return the result and verify it locally. Do not include secrets or use real research datasets.

## Stage C: global policy inheritance in a new workspace

| Platform | Reported result | Evidence |
| --- | --- | --- |
| Codex | PASS | Session log showed global `AGENTS.md` injected before first prompt; policy recognized in separate working directory |
| Antigravity | PASS | Distinct marker `Policy version 2026-11-04` appeared outside parent workspace; controlled before/after experiment |
| Claude | PASS | Fresh-session startup diagnostics identified `~/.claude/CLAUDE.md` as user-level instructions |

**Important:** both Codex and Antigravity previously gave nonexistent source paths when asked to name where policy was loaded. Do not trust source-path self-reports; use startup instrumentation/logs or controlled marker tests.

## Stage D: isolation, idempotency and trust

- A new workspace did not have knowledge of the other temporary workspace's test data (reported PASS). This is a limited behavioral check, not a full information-flow security proof.
- Bootstrap twice using a skip-if-exists check yielded no duplicate agents; raw launch against occupied pane rejected the name. Original process IDs remained unchanged (reported PASS).
- Antigravity asked to trust the newly created folder; approval was only for the inspected **temporary** path. New directories can continue to request approval; do not auto-approve.
- No permanent Codex sandbox change was applied.
- Temporary workspaces, test agents and scratch files were cleaned up after validation; the original three agents remained untouched.

## Checklist before marking status

- [ ] Platform-global instructions loaded in fresh sessions from a directory outside the original parent workspace.
- [ ] All agents discoverable with accurate types and names.
- [ ] Lead can delegate and inspect a result.
- [ ] Codex produces RELAY (without direct socket call).
- [ ] Lead routes to another agent and returns answer.
- [ ] Independent artifact verification matches.
- [ ] Workspace-specific trust boundaries respected.
- [ ] No duplicate agents on second bootstrap.
- [ ] Existing sessions and other projects untouched.
- [ ] No unauthorized sandbox, network or permissions changes.

Record command output, versions, exact scope and failure modes. **Cross-workspace ready** does not imply autonomous software-development review loops have been fully exercised. Validate a small coding task + tests + independent reviewer + repair separately before claiming that.
