---
trigger: always_on
description: Step-by-step guide for setting up a bare machine (physical or VM) to run Buildkite CI jobs.
---

# Preparing a Machine as a Buildkite Agent

Step-by-step guide for setting up a bare machine (physical or VM) to run Buildkite CI jobs.

## Prerequisites

- A Linux machine (Ubuntu 20.04+ or Amazon Linux 2/2023) with root/sudo access
- Network access to the internet (for package installs and Buildkite registration)
- A Buildkite agent token (from your org's Buildkite dashboard under Agents → Reveal Agent Token)

## 1. Install Docker

```bash
# Ubuntu / Debian
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin

# Amazon Linux 2023
sudo dnf install -y docker
sudo systemctl enable --now docker
```

Verify:

```bash
docker --version
sudo systemctl is-active docker
```

## 2. Install AWS CLI

```bash
# Universal installer (works on any Linux x86_64)
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
rm -rf aws awscliv2.zip
```

Verify:

```bash
aws --version
```

Configure credentials as needed for ECR pulls or S3 access (instance profile, env vars, or `aws configure`).

## 3. Install the Buildkite Agent

```bash
# Ubuntu / Debian — `apt-key` is gone on Ubuntu 24.04, so use a signed-by keyring
curl -fsSL "https://keys.openpgp.org/vks/v1/by-fingerprint/32A37959C2FA5C3C99EFBC32A79206696452D198" \
  | sudo gpg --dearmor -o /usr/share/keyrings/buildkite-agent-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/buildkite-agent-archive-keyring.gpg] https://apt.buildkite.com/buildkite-agent stable main" \
  | sudo tee /etc/apt/sources.list.d/buildkite-agent.list > /dev/null
sudo apt-get update
sudo apt-get install -y buildkite-agent

# Amazon Linux / RHEL
sudo sh -c 'echo -e "[buildkite-agent]\nname = Buildkite Pty Ltd\nbaseurl = https://yum.buildkite.com/buildkite-agent/stable/x86_64/\nenabled=1\ngpgcheck=0\npriority=1" > /etc/yum.repos.d/buildkite-agent.repo'
sudo yum install -y buildkite-agent
```

`stable` currently installs the 4.x agent. Check what the other machines on the
same queue run (`dpkg -l buildkite-agent`) and match them — pin with
`apt-get install -y buildkite-agent=<version>` from `apt-cache madison
buildkite-agent` if they are still on 3.x.

### Grant buildkite-agent access to Docker and AWS

```bash
# Add buildkite-agent to the docker group so it can run containers
sudo usermod -aG docker buildkite-agent

# Verify group membership (may need a new shell or reboot)
sudo -u buildkite-agent docker ps
```

If you use instance-level AWS credentials (IAM role), the buildkite-agent user inherits them automatically. For explicit credentials, set `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` in the agent's environment hook at `/etc/buildkite-agent/hooks/environment`.

> **Do not start the agent yet.** Finish all remaining setup steps first (docker/containerd data roots, GPU drivers, etc.) so the agent doesn't pick up jobs on a half-configured machine.

## 4. Move Docker and containerd Data Roots (Required on GPU Machines)

By default Docker stores images/containers under `/var/lib/docker` and containerd under `/var/lib/containerd`. On machines with a small root partition and a large secondary mount (NVMe, tmpfs, etc.), move both to the larger volume.

> **This is effectively required for GPU CI machines.** vLLM CI images are
> ~34 GB each and a busy machine accumulates several, which fills a typical
> 200-250 GB root disk. When that happens the agents stay connected and keep
> accepting jobs, but every job fails at initialization with
> `no space left on device`. Move the data roots **before** starting the agent,
> and put the agent's `build-path` on the same large volume.

Use the provided script:

```bash
sudo ./scripts/move-docker-containerd.sh /path/to/target
# e.g. /dev/shm for RAM-backed ephemeral storage
# e.g. /mnt/fast-nvme for a mounted NVMe drive
```

The script will:
- Set Docker's `data-root` in `/etc/docker/daemon.json` (creating the file if it's missing)
- Set containerd's `root` in `/etc/containerd/config.toml` (same)
- Move the Buildkite agent's `build-path` to the same volume, if buildkite-agent is already installed
- Install systemd drop-ins so the target directories are recreated on boot
- Restart both services and run a smoke test

It works on a fresh machine — no prerequisites beyond Docker/containerd
themselves (it uses `jq` if present, otherwise falls back to `python3`).
Existing images are not migrated; the new roots start empty and images re-pull
on demand.

> The script rewrites `build-path` in the agent config that exists **at the
> time it runs**. If you install a config from the templates afterwards (step 7
> / RUNBOOK step 4), that template's `build-path` replaces it — re-apply the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vllm-project/ci-infra](https://github.com/vllm-project/ci-infra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
