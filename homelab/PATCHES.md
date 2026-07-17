# Local patches vs upstream nearai/ironclaw

Track non-additive changes (anything outside `homelab/`).

## Active patches

_(none today — Vimes ships pure-vanilla IronClaw, customized only via
configs in `homelab/config/`)_

## Retired patches

### `crates/ironclaw_skills/src/registry.rs` — sha256 hex format (retired 2026-07-17)

Upstream used `format!("sha256:{:x}", result)` which stopped compiling
with rustc 1.92 + edition 2024 + generic-array 0.14 (E0277). Our
`homelab/Dockerfile` carried a build-time `sed` swapping it for a
byte-iter hex encode. Upstream now uses `hex::encode` in registry.rs,
so the patch (and its Dockerfile `RUN`) was dropped with the
2026-07-17 upstream merge (`merge: upstream ironclaw @ 81dbdc6d0`).
