# Chapter 5: Continuous Integration with GitHub Actions

---

## Understanding GitHub Actions

- [Workflow Automation](workflow-automation.md) — what CI/CD automation is, events, directory structure, how GitHub Actions fits in
- [Key Components](workflow-components.md) — workflows, jobs, steps, actions, runners and how they connect
- [Workflow Triggers](workflow-triggers.md) — push, pull_request, schedule, workflow_dispatch, release, combining triggers

---

## Building Workflows

- [Jobs and Matrix Strategies](jobs-and-matrix.md) — job dependencies, parallel jobs, matrix builds, artifacts between jobs, concurrency
- [Steps and Shell Commands](steps-and-commands.md) — run commands, environment variables, secrets, conditional steps, step outputs
- [Using Marketplace Actions](marketplace-actions.md) — core actions, language setup (Java, Node, Python), Docker actions, notifications, pinning versions

---

## Performance and Advanced Patterns

- [Caching for Faster Builds](caching.md) — actions/cache, Maven/npm/pip caching, Docker layer caching, cache keys
- [Multi-Job Workflows](multi-job-workflows.md) — sequential and parallel jobs, artifact sharing, conditional jobs, environment protection rules

---

## Runners

- [Runners](runners.md) — GitHub-hosted runners, self-hosted runner setup, labels, runner groups, security best practices

---

## Docker Integration

- [Docker and GitHub Actions](docker-github-actions.md) — building images in CI, pushing to Docker Hub, pushing to GHCR, multi-platform builds, deploying to servers and cloud (AWS ECS, Azure, VPS)
