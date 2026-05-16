# vimes-ironclaw/homelab/Dockerfile
#
# Builds the IronClaw image for **Vimes**, Leo's security & audit
# agent — "the one who watches the watchers". The build context is
# the repository root (this fork of nearai/ironclaw); upstream Rust
# source is at the repo root, plus any patches we've layered on top.
#
# Mirrors upstream's Dockerfile (kept aligned via upstream-sync.yml)
# and adds the runtime tooling Vimes needs in-cluster.

# syntax=docker/dockerfile:1

# ─── Stage 1: build ──────────────────────────────────────────────────
FROM rust:1.95-slim-bookworm AS builder

RUN apt-get update \
 && apt-get install -y --no-install-recommends \
      pkg-config libssl-dev cmake gcc g++ ca-certificates curl \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY Cargo.toml Cargo.lock ./

COPY src/           src/
COPY crates/        crates/
COPY migrations/    migrations/
COPY registry/      registry/
COPY wit/           wit/
COPY skills/        skills/
COPY channels-src/  channels-src/
COPY tools-src/     tools-src/
COPY tests/         tests/
COPY profiles/      profiles/
COPY build.rs       ./
COPY providers.json ./

# Patch upstream nearai/ironclaw `crates/ironclaw_skills/src/registry.rs`:
# `format!("sha256:{:x}", result)` doesn't compile with rustc 1.92 +
# generic-array 0.14 + edition 2024 (LowerHex no longer auto-impls on the
# GenericArray output of sha2::Sha256::finalize). Replace with byte-wise
# hex which only needs LowerHex on individual u8s. Tracked in
# homelab/PATCHES.md. Drop this RUN once upstream fixes registry.rs.
RUN /bin/sed -i 's#format!("sha256:{:x}", result)#format!("sha256:{}", result.iter().map(|b| format!("{:02x}", b)).collect::<String>())#' \
      crates/ironclaw_skills/src/registry.rs \
 && /bin/grep -F 'result.iter().map(' crates/ironclaw_skills/src/registry.rs

ARG LTO=true
ARG CODEGEN_UNITS=1
ENV CARGO_PROFILE_RELEASE_LTO=${LTO} \
    CARGO_PROFILE_RELEASE_CODEGEN_UNITS=${CODEGEN_UNITS}

RUN cargo build --release --bin ironclaw \
 && strip target/release/ironclaw

# ─── Stage 2: runtime ────────────────────────────────────────────────
FROM debian:bookworm-slim

RUN apt-get update \
 && apt-get install -y --no-install-recommends \
      ca-certificates curl wget tini \
      libssl3 \
      python3 python3-pip \
      jq ripgrep git unzip \
      postgresql-client \
      gnupg lsb-release \
      # Security tooling Vimes uses for audit work
      openssl gnupg2 \
      tcpdump nmap-common \
 && rm -rf /var/lib/apt/lists/*

ARG KUBECTL_VERSION=v1.34.0
RUN curl -fsSL "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl" \
      -o /usr/local/bin/kubectl \
 && chmod +x /usr/local/bin/kubectl

# Bitwarden CLI (`bw`) — credential lookups against the cluster's
# Vaultwarden (https://warden.leopaska.xyz). Vimes runs read-only:
# `bw get item` / `bw list items` for audit and rotation checks; never
# `bw edit`. Persistent Vimes creds arrive via SealedSecrets → envFrom.
ARG BW_VERSION=2026.4.1
RUN ARCH="$(dpkg --print-architecture)" \
 && case "$ARCH" in \
      amd64) BW_ARCH=linux ;; \
      arm64) BW_ARCH=linux-arm64 ;; \
      *) echo "unsupported arch: $ARCH" >&2; exit 1 ;; \
    esac \
 && curl -fsSL "https://github.com/bitwarden/clients/releases/download/cli-v${BW_VERSION}/bw-oss-${BW_ARCH}-${BW_VERSION}.zip" \
        -o /tmp/bw.zip \
 && unzip -q /tmp/bw.zip -d /usr/local/bin \
 && chmod +x /usr/local/bin/bw \
 && rm -f /tmp/bw.zip

# trivy — container vulnerability scanner (Vimes uses it nightly)
RUN curl -sSfL https://aquasecurity.github.io/trivy-repo/deb/public.key \
      | gpg --dearmor -o /usr/share/keyrings/trivy.gpg \
 && echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb bookworm main" \
      > /etc/apt/sources.list.d/trivy.list \
 && apt-get update \
 && apt-get install -y --no-install-recommends trivy \
 && rm -rf /var/lib/apt/lists/*

# kubescape — k8s posture
RUN curl -sSfL https://raw.githubusercontent.com/kubescape/kubescape/master/install.sh \
      | /bin/bash -s -- -v latest

# gh CLI
RUN curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg \
      -o /usr/share/keyrings/githubcli-archive-keyring.gpg \
 && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
      > /etc/apt/sources.list.d/github-cli.list \
 && apt-get update \
 && apt-get install -y --no-install-recommends gh \
 && rm -rf /var/lib/apt/lists/*

COPY --from=builder /app/target/release/ironclaw /usr/local/bin/ironclaw
COPY --from=builder /app/migrations               /app/migrations
COPY --from=builder /app/skills                   /usr/local/bin/skills

RUN useradd -m -u 1000 -s /bin/bash ironclaw \
 && mkdir -p /data /etc/ironclaw/persona \
 && chown -R ironclaw:ironclaw /data /etc/ironclaw

USER ironclaw
WORKDIR /home/ironclaw

EXPOSE 3000 9100

HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
  CMD curl -fsS http://localhost:3000/health || exit 1

ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["/usr/local/bin/ironclaw", "--bind", "0.0.0.0:3000"]
