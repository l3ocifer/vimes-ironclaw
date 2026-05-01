# Local patches vs upstream nearai/ironclaw

Track non-additive changes (anything outside `homelab/`).

## Active patches

### `crates/ironclaw_skills/src/registry.rs` — sha256 hex format

Upstream uses `format!("sha256:{:x}", result)` where `result =
Sha256::finalize()`. With rustc 1.92 + edition 2024 + generic-array 0.14,
the `LowerHex` trait is no longer auto-implemented on the
`GenericArray<u8, ...>` output, producing the upstream-tracked compile
error E0277.

Our `homelab/Dockerfile` applies a build-time `sed` patch that swaps
the format to a byte-iter hex encode (`result.iter().map(|b|
format!("{:02x}", b)).collect::<String>()`), which uses `LowerHex` on
individual `u8`s and always works.

**Drop this when:** upstream `nearai/ironclaw` fixes registry.rs to use
`hex::encode` (or equivalent). Then the upstream-sync workflow's auto-PR
will surface the change and `homelab/Dockerfile` can drop the `sed`.
