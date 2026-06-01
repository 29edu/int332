# Workflow Automation with GitHub Actions

GitHub Actions is a CI/CD platform built directly into GitHub. It lets you automate tasks — building, testing, and deploying code — in response to events that happen in your repository.

---

## What is Workflow Automation

Workflow automation means replacing manual, repetitive steps with scripts that run automatically when something happens in your repository.

Without automation, a typical release process looks like:

```
developer pushes code
  → manually run tests locally
  → manually build the JAR
  → manually build a Docker image
  → manually push to the registry
  → manually SSH into the server
  → manually restart the service
```

With GitHub Actions, the same process runs automatically every time code is pushed:

```
developer pushes code
  → GitHub detects the push
  → workflow runs automatically:
      → tests run
      → JAR is built
      → Docker image is built and pushed
      → server is updated
```

---

## What are Events

An event is something that happens in your GitHub repository that can trigger a workflow. GitHub Actions supports dozens of events.

Common events:

| Event | When it fires |
|---|---|
| `push` | When commits are pushed to a branch |
| `pull_request` | When a PR is opened, updated, or merged |
| `schedule` | On a cron schedule (e.g. nightly builds) |
| `workflow_dispatch` | Manually triggered from the GitHub UI |
| `release` | When a GitHub release is created or published |
| `issues` | When an issue is opened, closed, or labeled |

---

## Workflow Directory Structure

Workflows are YAML files stored inside your repository under:

```
.github/
└── workflows/
    ├── ci.yml           ← runs on every push
    ├── release.yml      ← runs on release creation
    └── nightly.yml      ← runs on a schedule
```

GitHub automatically detects any `.yml` file inside `.github/workflows/` and registers it as a workflow.

Each file defines one workflow. A repository can have multiple workflows running independently.

---

## Anatomy of a Workflow File

```yaml
name: CI Pipeline               # display name in GitHub UI

on:                             # what triggers this workflow
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:                           # what to do
  build:
    runs-on: ubuntu-latest      # which runner to use

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Build with Maven
        run: ./mvnw clean package
```

---

## How GitHub Actions Fits Into CI/CD

**CI (Continuous Integration):** automatically build and test code on every push or pull request, catching bugs before they reach the main branch.

**CD (Continuous Delivery/Deployment):** automatically deploy tested code to staging or production environments.

```
Code push
  ↓
GitHub Actions triggers
  ↓
[ CI: build + test ]
  ↓ (if passing)
[ CD: build image + deploy ]
  ↓
Application updated in production
```

GitHub Actions handles both CI and CD in the same YAML file, in the same repository, with no external CI server needed.
