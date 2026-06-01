# Chapter 6: CI/CD with Jenkins

---

## Jenkins Foundations

- [Jenkins Architecture](jenkins-architecture.md) — Master/Agent model, installation (WAR, apt, Docker), UI overview, executors
- [Plugins and Security](plugins-and-security.md) — plugin management, essential plugins, authentication, users, roles, credentials, CSRF

---

## Jenkins Pipelines

- [Freestyle vs Pipeline Jobs](freestyle-vs-pipeline.md) — comparison, when to use each, converting Freestyle to Pipeline
- [Declarative Pipeline](declarative-pipeline.md) — Jenkinsfile structure, agent, environment, parameters, options, triggers, post actions
- [Scripted Pipeline](scripted-pipeline.md) — Groovy node blocks, conditionals, loops, try/catch, parallel execution, when to use scripted
- [Multi-Branch Pipeline](multibranch-pipeline.md) — auto-discovery of branches, branch-specific logic, scan triggers, organisation folders

---

## Pipeline Stages

- [Pipeline Stages](pipeline-stages.md) — Checkout, Build, Test, Package, Post actions, archiving artifacts, stash/unstash

---

## Docker and Maven Integration

- [Docker and Jenkins](docker-jenkins.md) — building images, Docker inside agents, Docker Pipeline DSL, pushing to Docker Hub and GHCR, GitHub integration
- [Jenkins and Maven](jenkins-maven.md) — installing Maven in Jenkins, global tool config, Maven wrapper, Docker-based builds, JaCoCo coverage, test reports

---

## Agents and Deployment

- [Jenkins Agents](jenkins-agents.md) — SSH agents, JNLP agents, container-based agents, Kubernetes agents, shared pipeline libraries, SFTP deploy
- [Deployments, Backup, and Best Practices](deployments-and-backup.md) — pollSCM vs webhooks, SSH deploy, ECS, Kubernetes, backup/restore, pipeline best practices
