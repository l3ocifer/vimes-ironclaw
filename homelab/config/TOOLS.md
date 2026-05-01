# Vimes — tools and environment

## Runtime

- **Framework**: IronClaw (nearai/ironclaw fork at `l3ocifer/vimes-ironclaw`)
- **Image**: `ghcr.io/l3ocifer/vimes-ironclaw:latest`
- **Namespace**: `agents-shared`
- **Schedule**: floats. Soft-prefer thebeast for RAM (audit work
  loads several scan results into memory simultaneously); will run
  on alef or any future worker.
- **State PVC**: `vimes-state` (longhorn-single, 10 GiB — audit run
  artifacts, scan caches, hardstop file)
- **Graph PVCs mounted**:
  - `vimes-graph` RW — own findings, policies, methodology notes
  - `leo-graph` (restricted-write paths only — `pages/world/audit-log.md`,
    `pages/agent-coordination/`)
  - All six siblings' graphs RO — for cross-referencing what they
    *say* they did against what the cluster + Postgres show happened

## Models

Vimes calls models via LiteLLM (`http://litellm.litellm.svc:4000/v1`).
Configured aliases in `openclaw.json`:

| Alias | Use |
|---|---|
| `chat` | default reasoning over scan output |
| `triage` | classifying findings: critical / high / medium / low / FP |
| `long` | long-context audit-log review (full-day journals) |
| `frontier` | reserved for ambiguous findings — small budget, deliberate use |

Vimes is NOT chatty with models. Audit work is mostly local: parse
trivy/kubescape JSON, run policies, generate the report. Models are
used for triage borderline cases and for natural-language report
prose.

## Communication channels

| Channel | Use |
|---|---|
| Matrix `@vimes:leopaska.xyz` | morning report to Leo, peer messages to Frick |
| ntfy `ntfy.leopaska.xyz/vimes-critical` | critical-severity pages (overrides quiet hours) |
| ntfy `ntfy.leopaska.xyz/vimes` | high/medium notifications |
| A2A — peer to all 6 siblings | findings escalation, locked-tower flags to Quirm |
| HTTP API `:3000` | exposes `/audit/run`, `/findings/<date>`, `/policies`, `/locked-tower` |

Vimes does NOT have iMessage, Telegram, Stripe, or BlueBubbles.
Internal-facing only.

## Security tooling (in image)

- `trivy` — container vulnerability scanner (CVEs, misconfigs)
- `kubescape` — k8s posture (NSA/CIS Hardening Guide, MITRE ATT&CK)
- `kubectl` — read-only k8s introspection
- `gh` — GitHub PR review (greps for plaintext secrets, audits
  permissions on workflow files)
- `1password-cli` — credential lookup for self-only secrets
- `psql` — direct sibling DB introspection via `*_ro` roles
- Custom policies in `vimes-graph/pages/policies/` — yaml/rego/python

## Postgres

| Database | Access | Purpose |
|---|---|---|
| `ironclaw_vimes` (owner: `vimes`) | RW | own session DB, findings registry, audit run history |
| `ironclaw_frick`, `openclaw_frack`, `hermes_sancho`, `openfang_vetinari`, `hermes_quirm`, `openclaw_puck` | RO via `vimes_ro` role | sibling action audit (compare journals to actual writes) |
| `audit` (owner: `audit-system`) | RW | central audit log written by all agents — Vimes reads + appends |

## Kubernetes access

ServiceAccount `vimes-ops` in `agents-shared`. Cluster-wide
**read-only audit** via ClusterRole `vimes-cluster-audit`. See
`k8s/rbac.yaml` for the full grant — secrets are listable (names
only) but not gettable (no values). Vimes correlates names against
SealedSecret manifests in `l3ocifer/homelab` for what the values
*should* be.

## Findings layout (in `vimes-graph`)

```
vimes-graph/
├── pages/
│   ├── policies/                    ← rego/python/yaml policies
│   │   ├── secret-hygiene.rego
│   │   ├── rbac-least-privilege.rego
│   │   ├── network-policy-coverage.py
│   │   └── image-cve-thresholds.yaml
│   ├── findings/                    ← per-finding markdown, dated
│   │   └── YYYY-MM-DD/
│   │       ├── critical/
│   │       ├── high/
│   │       ├── medium/
│   │       └── low/
│   ├── runs/                        ← raw scan output (trivy, kubescape)
│   ├── reports/                     ← daily + weekly markdown reports
│   ├── locked-tower/                ← Quirm prototypes Vimes has flagged
│   └── methodology-reviews/         ← weekly self-audit
└── journals/                        ← daily activity log
```

## Skills (planned, in `vimes-graph/pages/skills/`)

- `nightly-scan.sh` — orchestrates trivy + kubescape + custom policies
- `triage-findings.py` — applies severity heuristics + LLM borderlines
- `cross-reference.py` — sibling journal vs actual k8s/DB writes
- `secret-inventory.py` — sealed-secret + cluster + git triangulation
- `policy-add.sh` — scaffold a new policy file with test cases
- `false-positive-review.py` — weekly FP rate + drift detection

## IronClaw config

Configured in `homelab/config/openclaw.json`. Vimes' enabled toolsets:

- `read` (filesystem RO across mounted graphs)
- `exec` (sandboxed — only `kubectl get/list`, `trivy`, `kubescape`, `psql -c`)
- `web_search` (CVE database lookups)
- No `write`, no `edit` outside own state.
- No browser tool — audit doesn't need browsing.

## Required env vars

Provided by `vimes-secrets` SealedSecret in `agents-shared`:

| Var | Use |
|---|---|
| `LITELLM_API_KEY` | virtual key tagged `agent:vimes` |
| `DATABASE_URL` | `postgres://vimes@homelab-pg-rw...` |
| `VIMES_RO_PASSWORD` | psql for sibling DB introspection |
| `MATRIX_HOMESERVER` + `MATRIX_ACCESS_TOKEN` | `@vimes:leopaska.xyz` |
| `NTFY_TOKEN` | findings notifications |
| `OFP_SHARED_SECRET` | A2A mutual auth |
| `OP_SERVICE_ACCOUNT_TOKEN` | 1Password (own credentials only) |
| `HEALTHCHECKS_UUID` | per-agent UUID for hc-ping.com heartbeats |
| `GITHUB_TOKEN` | gh CLI for PR audit (read-only) |
