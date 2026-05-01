# Vimes — IronClaw security & audit agent

This is **Leo's fork-equivalent of [nearai/ironclaw](https://github.com/nearai/ironclaw)**,
extended to run as `Vimes` — the seventh-of-seven agents in [Leo's
homelab](https://github.com/l3ocifer/homelab), named for **Sam Vimes**
of Ankh-Morpork's Watch. Vimes audits the rest of the fleet:
"the one who watches the watchers."

> Note: GitHub limits forks of a single upstream repo to one per
> account, and `l3ocifer/frick-ironclaw` already holds that fork
> relationship. This repo was created as a regular repo, with the
> upstream cloned in and `upstream` configured as an explicit remote.
> Functionally identical for sync purposes — `git fetch upstream &&
> git merge upstream/main` works the same.

## Layout

```
vimes-ironclaw/                      ← repo root
├── (upstream ironclaw source)       ← from nearai/ironclaw
│   ├── src/, migrations/, wit/, ...
│   ├── Cargo.toml, Cargo.lock
│   └── ...
└── homelab/                          ← everything we add
    ├── Dockerfile                    ← Rust build + audit toolchain
    │                                   (trivy, kubescape, gh CLI, etc.)
    ├── k8s/                          ← kustomize tree
    ├── config/                       ← SOUL.md, TOOLS.md, openclaw.json
    ├── shared/                       ← submodule → l3ocifer/homelab
    ├── .github/workflows/
    ├── PATCHES.md, CHANGELOG.md, README.md
```

## Persona, in 30 seconds

Tired but unshakeable. Reads everything in the cluster; modifies
nothing. Cross-references what each sibling claims (their journals)
against what the cluster + Postgres actually show happened. Flags
findings by severity. Locks Quirm prototypes that shouldn't ship.
Audits Vimes himself, weekly, by handing his methodology to Quirm.

See `config/SOUL.md` for the full persona.

## Required env vars

Provided by `vimes-secrets` SealedSecret in `agents-shared` namespace.
See `config/openclaw.json` for the full reference.

## Build locally

```bash
git clone https://github.com/l3ocifer/vimes-ironclaw
cd vimes-ironclaw
git remote add upstream https://github.com/nearai/ironclaw.git
docker build -f homelab/Dockerfile \
  -t ghcr.io/l3ocifer/vimes-ironclaw:dev .
```

## Sync from upstream

```bash
git fetch upstream
git merge upstream/main          # or use the weekly auto-PR from CI
```

## License

- IronClaw upstream: Apache-2.0 (see `../LICENSE`).
- Homelab additions in `homelab/`: same.
- Persona text in `config/SOUL.md` is Leo Paska's IP.
