# Using Marketplace Actions

The GitHub Marketplace has thousands of pre-built actions that handle common tasks. Instead of writing shell scripts for setup, authentication, notifications, and deployments, you use an action written and maintained by the community or GitHub itself.

---

## What is an Action

An action is a reusable step packaged as a GitHub repository. You reference it in your workflow with `uses:` and pass inputs with `with:`.

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0
```

---

## Core GitHub Actions

These are official actions maintained by GitHub:

**actions/checkout** — checks out your repository code onto the runner

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0        # 0 = full history, default 1 = only latest commit
    ref: main             # specific branch, tag, or SHA to checkout
```

**actions/upload-artifact** — saves files from a job for later use or download

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: target/surefire-reports/
    retention-days: 7
```

**actions/download-artifact** — retrieves files uploaded by another job

```yaml
- uses: actions/download-artifact@v4
  with:
    name: test-results
    path: ./reports
```

**actions/cache** — caches dependencies between workflow runs

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.m2
    key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
```

---

## Language-Specific Setup Actions

**Java — actions/setup-java**

```yaml
- uses: actions/setup-java@v3
  with:
    distribution: 'temurin'    # Eclipse Temurin (AdoptOpenJDK)
    java-version: '17'
    cache: maven               # auto-caches ~/.m2
```

Distributions available: `temurin`, `zulu`, `corretto`, `microsoft`, `liberica`

**Node.js — actions/setup-node**

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'               # auto-caches node_modules
```

**Python — actions/setup-python**

```yaml
- uses: actions/setup-python@v5
  with:
    python-version: '3.11'
    cache: 'pip'
```

**Go — actions/setup-go**

```yaml
- uses: actions/setup-go@v5
  with:
    go-version: '1.21'
    cache: true
```

**.NET — actions/setup-dotnet**

```yaml
- uses: actions/setup-dotnet@v3
  with:
    dotnet-version: '8.0'
```

---

## Docker Actions

**docker/login-action** — logs in to a container registry

```yaml
- uses: docker/login-action@v3
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}
```

**docker/build-push-action** — builds and pushes Docker images

```yaml
- uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    tags: ghcr.io/myorg/myapp:latest
```

**docker/metadata-action** — generates Docker image tags and labels automatically

```yaml
- uses: docker/metadata-action@v5
  id: meta
  with:
    images: ghcr.io/myorg/myapp
    tags: |
      type=ref,event=branch
      type=semver,pattern={{version}}
```

---

## Notification Actions

**Slack notification:**

```yaml
- uses: slackapi/slack-github-action@v1.26.0
  with:
    payload: '{"text":"Build ${{ job.status }} on ${{ github.repository }}"}'
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

---

## Deployment Actions

**Deploy to AWS ECS:**

```yaml
- uses: aws-actions/amazon-ecs-deploy-task-definition@v1
  with:
    task-definition: task-def.json
    service: my-service
    cluster: my-cluster
```

**Deploy to Azure Web App:**

```yaml
- uses: azure/webapps-deploy@v3
  with:
    app-name: my-webapp
    publish-profile: ${{ secrets.AZURE_PUBLISH_PROFILE }}
    images: myregistry.azurecr.io/myapp:latest
```

---

## Finding Actions

- GitHub Marketplace: marketplace.github.com/actions
- In the workflow editor on GitHub, click "Browse Marketplace" on the right panel
- Search the action name on GitHub — most are public repositories

**Pinning action versions:**

Always pin to a specific version to avoid unexpected breaking changes:

```yaml
uses: actions/checkout@v4         # good — pinned to major version
uses: actions/checkout@v4.1.1     # better — pinned to exact version
uses: actions/checkout@abc123def  # best for security — pinned to commit SHA
```

Never use:

```yaml
uses: actions/checkout@main       # dangerous — changes any time main changes
```
