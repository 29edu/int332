# Runners

A runner is the machine that executes a GitHub Actions job. Every job needs a runner specified with `runs-on:`. GitHub provides hosted runners, or you can register your own self-hosted runners.

---

## GitHub-Hosted Runners

GitHub-hosted runners are virtual machines fully managed by GitHub. They are provisioned fresh for every job and torn down when the job completes.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
```

**Available GitHub-hosted runners:**

| Label | OS | Notes |
|---|---|---|
| `ubuntu-latest` | Ubuntu 22.04 | Most common, fast |
| `ubuntu-22.04` | Ubuntu 22.04 | Pinned version |
| `ubuntu-20.04` | Ubuntu 20.04 | Older LTS |
| `windows-latest` | Windows Server 2022 | |
| `windows-2022` | Windows Server 2022 | Pinned |
| `macos-latest` | macOS 14 (Apple Silicon) | |
| `macos-14` | macOS 14 | Pinned |
| `macos-13` | macOS 13 (Intel) | |

**What comes pre-installed on ubuntu-latest:**
- Docker, Docker Compose
- Java (multiple versions), Maven, Gradle
- Node.js, npm, Python, Go, Ruby
- Git, curl, wget, jq
- AWS CLI, Azure CLI, GCloud CLI

**Pricing:**
- Public repositories: free with no limits
- Private repositories: billed per minute (Linux is cheapest, macOS is most expensive)

---

## Self-Hosted Runners

A self-hosted runner is a machine you own and register with GitHub. GitHub sends jobs to it the same way it sends jobs to hosted runners.

**Why use self-hosted runners:**

- access to internal network resources (private databases, internal registries)
- specific hardware requirements (GPUs, high memory, custom hardware)
- cost savings on large-scale private repository builds
- compliance requirements that prohibit running on third-party infrastructure
- persistent state between jobs (tools pre-installed, no cold start)

**Supported platforms:**
Linux, Windows, macOS, ARM, Docker containers.

---

## Setting Up a Self-Hosted Runner

**Step 1 — Go to repository Settings → Actions → Runners → New self-hosted runner**

GitHub shows a download and configure script for your chosen OS.

**Step 2 — On your machine (Linux example):**

```bash
mkdir actions-runner && cd actions-runner

# Download the runner
curl -o actions-runner-linux-x64-2.311.0.tar.gz -L \
  https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-x64-2.311.0.tar.gz

# Extract
tar xzf ./actions-runner-linux-x64-2.311.0.tar.gz

# Configure — GitHub generates this command with your token
./config.sh --url https://github.com/your-org/your-repo --token YOUR_TOKEN

# Start the runner
./run.sh
```

**Step 3 — Install as a service (so it starts on reboot):**

```bash
sudo ./svc.sh install
sudo ./svc.sh start
```

**Step 4 — Use in your workflow:**

```yaml
jobs:
  build:
    runs-on: self-hosted
```

Or with labels to target a specific runner:

```yaml
runs-on: [self-hosted, linux, x64, gpu]
```

---

## Runner Labels

Labels identify runners and let workflows target specific machines.

Default labels on a self-hosted runner:
- `self-hosted`
- OS: `linux`, `windows`, or `macos`
- Architecture: `x64`, `arm64`, `arm`

Custom labels can be added during `./config.sh`:

```bash
./config.sh --url ... --token ... --labels production,high-memory,gpu
```

Target by label in a workflow:

```yaml
runs-on: [self-hosted, production, gpu]
```

---

## Runner Groups

Runner groups let you organise self-hosted runners and control which repositories can use them. Only available on GitHub Enterprise.

- create groups (e.g. `production-runners`, `staging-runners`)
- assign runners to groups
- restrict which repositories or organisations can use each group

---

## Runner Security and Management

**GitHub-hosted runner security:**
- each job gets a fresh VM — no state leaks between jobs or repositories
- the VM is deleted after the job completes
- no other jobs can access its filesystem or environment

**Self-hosted runner risks:**

Self-hosted runners are a security responsibility. Key risks:

1. **Code execution on your infrastructure**
   Any workflow that runs on a self-hosted runner executes code on your machine. If a pull request from a fork triggers the workflow, that untrusted code runs on your hardware.

2. **Environment persistence**
   Unlike GitHub-hosted runners, self-hosted runners are not wiped between jobs. A previous job may leave behind files, credentials, or environment variables.

**Security best practices for self-hosted runners:**

- never use self-hosted runners for public repositories — fork PRs can execute arbitrary code
- for private repositories, restrict which workflows can use self-hosted runners (Settings → Actions → Runner groups)
- run runners in a sandboxed environment: Docker containers, VMs, or ephemeral cloud instances
- use least-privilege — the runner user should have only the permissions it needs
- rotate runner tokens regularly
- monitor runner logs for unusual activity

**Ephemeral self-hosted runners:**
For higher security, configure runners that register, run one job, and then deregister — similar to GitHub-hosted runners. This prevents state accumulation.

```bash
./config.sh --url ... --token ... --ephemeral
```

---

## Comparing Runner Types

| | GitHub-Hosted | Self-Hosted |
|---|---|---|
| Setup | None | Manual registration |
| Maintenance | GitHub handles it | You maintain the machine |
| Cost | Free (public), billed (private) | Infrastructure cost only |
| Fresh environment | Every job | Persistent unless ephemeral |
| Network access | Public internet only | Can access private networks |
| Hardware | Fixed sizes | Any hardware you have |
| Security | Very high | Your responsibility |
