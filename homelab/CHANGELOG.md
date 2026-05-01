# Changelog

Vimes-IronClaw releases.

## Unreleased

### Added

- Initial homelab/ overlay scaffolding (mirrors frick-ironclaw layout
  — same upstream framework, different role).
- Dockerfile builds IronClaw from local fork + adds audit toolchain:
  `trivy`, `kubescape`, `gh`, `bw` (Bitwarden CLI against Vaultwarden,
  read-only), `kubectl`, `psql`.
- k8s manifests for `agents-shared` namespace + floating
  (soft-prefer thebeast for RAM) + Longhorn-backed (longhorn-single
  state, longhorn-rwx graphs, all 7 sibling graphs mounted RO).
- ClusterRole `vimes-cluster-audit`: read-only across the cluster,
  including secret *names* (no values). Cross-references with
  SealedSecret manifests in `l3ocifer/homelab` for expected values.
- config/SOUL.md (Sam-Vimes persona, security & audit, watches the
  watchers).
- config/TOOLS.md (audit toolchain, RO Postgres + cluster, findings
  layout, methodology).
- config/openclaw.json (LiteLLM-routed, allowlisted exec, A2A peers
  for all 6 siblings, 4 cron tasks: nightly scan, morning report,
  mid-day spot-check, weekly methodology review).
- GitHub Actions: build.yml, upstream-sync.yml (Sun 03:15 UTC),
  shared-docs-bump.yml.
- Submodule of l3ocifer/homelab at homelab/shared/.

### New repo

- Created fresh as `l3ocifer/vimes-ironclaw` (cannot be a GitHub fork
  because `l3ocifer/frick-ironclaw` already forks the same upstream;
  instead, we cloned upstream and rewired remotes — see README.md
  "Sync from upstream" for the workflow).
