# Local patches vs upstream nearai/ironclaw

Track non-additive changes (anything outside `homelab/`).

## Active patches

### heartbeat-tool-exec: execute tool calls in the periodic heartbeat loop

- **Files**: `src/agent/heartbeat.rs`, `src/agent/agent_loop.rs`, `src/agent/commands.rs`, `src/main.rs`
- **Reason**: upstream's heartbeat turn renders tool calls but never
  executes them, so autonomous cron/heartbeat turns could not act.
  Adds tool execution to the heartbeat loop (originally shipped
  2026-06-25, `fix(heartbeat): execute tool calls in the periodic
  heartbeat loop`).
- **Upstream PR**: not yet submitted.
- **Last applied**: 2026-07-17 against upstream@81dbdc6d0 (clean merge,
  no reapply needed).

## Retired patches

### `crates/ironclaw_skills/src/registry.rs` — sha256 hex format (retired 2026-07-17)

Upstream used `format!("sha256:{:x}", result)` which stopped compiling
with rustc 1.92 + edition 2024 + generic-array 0.14 (E0277). Our
`homelab/Dockerfile` carried a build-time `sed` swapping it for a
byte-iter hex encode. Upstream now uses `hex::encode` in registry.rs,
so the patch (and its Dockerfile `RUN`) was dropped with the
2026-07-17 upstream merge (`merge: upstream ironclaw @ 81dbdc6d0`).
