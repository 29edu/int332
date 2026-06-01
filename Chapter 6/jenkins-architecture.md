# Jenkins Architecture

Jenkins is an open-source automation server used to build, test, and deploy software. It is one of the most widely used CI/CD tools in enterprise environments.

---

## Master/Agent Model

Jenkins uses a Master/Agent (also called Controller/Agent) architecture to distribute work across multiple machines.

```
            [ Jenkins Master (Controller) ]
                 |         |         |
          [ Agent 1 ] [ Agent 2 ] [ Agent 3 ]
          Linux/Java  Windows    Docker
```

**Jenkins Master (Controller)**
The central server that:
- hosts the Jenkins web UI
- stores all job configurations, build history, and plugin data
- schedules and assigns builds to agents
- collects and displays build results
- does not run builds itself (best practice — keep master lightweight)

**Jenkins Agent (Worker)**
A machine connected to the master that:
- receives build jobs from the master
- runs the actual build steps (compile, test, package)
- reports results back to the master
- can be a physical machine, VM, or Docker container

**Why this model:**
- the master handles coordination; agents handle heavy work
- multiple agents allow builds to run in parallel
- different agents can have different environments (Java 17, Node, Docker)
- agents can be added or removed without touching the master

---

## Installation

**Option 1 — WAR file (any platform with Java):**

```bash
java -jar jenkins.war
```

Access at `http://localhost:8080`

**Option 2 — Debian/Ubuntu:**

```bash
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list

sudo apt update
sudo apt install jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

**Option 3 — Docker:**

```bash
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

- port `8080` — Jenkins web UI
- port `50000` — agent connection port (JNLP)
- volume `jenkins_home` — persists all data

---

## Initial Setup

On first access at `http://localhost:8080`:

1. Unlock Jenkins using the initial admin password:

```bash
# Docker
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

# Linux install
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

2. Install suggested plugins (recommended for beginners)
3. Create the first admin user
4. Set the Jenkins URL

---

## UI Overview

**Dashboard**
The home page listing all jobs, their last build status, and quick links.

```
Dashboard
├── All Jobs
│   ├── my-app (last build: ✓ #42)
│   ├── backend-api (last build: ✗ #15)
│   └── deploy-prod (last build: ✓ #8)
├── Build Queue
└── Build Executor Status
```

**Job/Item**
A Jenkins job defines what to build and how. The main types are Freestyle and Pipeline.

**Build**
A single execution of a job. Each build has a number, logs, and a status (success, failure, unstable, aborted).

**Blue Ocean**
A modern UI for Jenkins pipelines (installed as a plugin). Shows pipeline stages visually.

**Key navigation:**

| URL path | What you see |
|---|---|
| `/` | Dashboard with all jobs |
| `/job/<name>` | Job detail and build history |
| `/job/<name>/<number>` | Specific build details and console output |
| `/manage` | Jenkins system configuration |
| `/pluginManager` | Plugin installation and updates |
| `/credentials` | Stored credentials (passwords, tokens, SSH keys) |

---

## Build Executor

An executor is a slot on a node (master or agent) that can run one build at a time. A node with 3 executors can run 3 builds simultaneously.

Configure executor count: Manage Jenkins → Nodes → click a node → Configure.

Best practice: set the master's executor count to 0 so no builds run on the master itself. All builds go to agents.
