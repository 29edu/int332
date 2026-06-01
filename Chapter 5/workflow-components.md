# Key Components of GitHub Actions

A GitHub Actions workflow is made up of five core components: workflows, jobs, steps, actions, and runners. Understanding how they relate to each other is essential to writing and debugging pipelines.

---

## Workflows

A workflow is the top-level automation unit. It is a YAML file stored in `.github/workflows/`. A workflow defines:

- what triggers it (the `on:` section)
- what jobs to run
- the overall pipeline behaviour

A repository can have multiple workflows. They run independently.

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]

jobs:
  build:
    ...
```

---

## Jobs

A job is a collection of steps that run together on the same runner (virtual machine). Jobs inside one workflow run in parallel by default. You can make them sequential using `needs:`.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "building"

  test:
    runs-on: ubuntu-latest
    needs: build        # waits for build to finish first
    steps:
      - run: echo "testing"

  deploy:
    runs-on: ubuntu-latest
    needs: test         # waits for test to finish
    steps:
      - run: echo "deploying"
```

Each job gets a fresh virtual machine. Files created in one job are not available in another unless explicitly shared using artifacts.

---

## Steps

Steps are the individual tasks inside a job. They run sequentially, one after another. If a step fails, the job stops by default.

A step can either:
- run a shell command with `run:`
- use a reusable action with `uses:`

```yaml
steps:
  - name: Checkout repository
    uses: actions/checkout@v4

  - name: Set up Node.js
    uses: actions/setup-node@v4
    with:
      node-version: '20'

  - name: Install dependencies
    run: npm install

  - name: Run tests
    run: npm test

  - name: Build
    run: npm run build
```

---

## Actions

An action is a reusable unit of work packaged for use in a step. Instead of writing the same shell commands over and over, you use a pre-built action from the GitHub Marketplace or from your own repository.

Actions are referenced with `uses:`:

```yaml
uses: actions/checkout@v4
uses: actions/setup-java@v3
uses: docker/login-action@v3
uses: actions/upload-artifact@v4
```

**Action versions:**
Always pin to a specific version tag (`@v4`, `@v3`) rather than `@main` or `@latest`. This prevents breaking changes in an action from unexpectedly breaking your pipeline.

**Types of actions:**

| Type | Description |
|---|---|
| JavaScript action | Runs Node.js code directly on the runner |
| Docker container action | Runs code inside a Docker container |
| Composite action | A sequence of steps bundled together |

---

## Runners

A runner is the virtual machine or physical server that executes a job. When a job runs, GitHub assigns it to a runner, which sets up the environment and executes each step.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest   # GitHub-hosted Ubuntu runner
```

**GitHub-hosted runners** are managed by GitHub — you pay per minute of use on private repositories. Public repositories get free runners.

**Self-hosted runners** are machines you manage yourself, registered with GitHub to run your workflows.

---

## How They All Connect

```
Workflow (.github/workflows/ci.yml)
  └── triggered by: push to main
      └── Job: build (runs-on: ubuntu-latest)
            ├── Step 1: uses: actions/checkout@v4
            ├── Step 2: uses: actions/setup-java@v3
            └── Step 3: run: ./mvnw clean package
```

- the **workflow** defines the full pipeline
- the **job** defines what machine to use and groups related steps
- each **step** either runs a command or calls an **action**
- the **runner** is the machine that actually executes all of this
