# Workflow Triggers

The `on:` key in a workflow file defines what events trigger the workflow. GitHub Actions supports a wide range of triggers, from code events to time-based schedules.

---

## push

Triggers when commits are pushed to a branch.

**Trigger on any push:**

```yaml
on:
  push:
```

**Trigger only on pushes to main:**

```yaml
on:
  push:
    branches:
      - main
```

**Trigger on multiple branches:**

```yaml
on:
  push:
    branches:
      - main
      - develop
      - 'release/**'   # wildcard: any branch starting with release/
```

**Trigger only when specific files change:**

```yaml
on:
  push:
    paths:
      - 'src/**'
      - 'pom.xml'
```

This is useful for monorepos — only rebuild the backend when backend files change.

**Ignore certain branches or paths:**

```yaml
on:
  push:
    branches-ignore:
      - 'docs/**'
    paths-ignore:
      - '**.md'
      - 'docs/**'
```

---

## pull_request

Triggers when a pull request is opened, updated, or closed.

```yaml
on:
  pull_request:
    branches:
      - main
```

This is the most common CI trigger — run tests on every PR before it can be merged.

**Activity types:**

```yaml
on:
  pull_request:
    types:
      - opened
      - synchronize    # new commits pushed to the PR
      - reopened
```

Default types are `opened`, `synchronize`, and `reopened`.

---

## schedule

Triggers on a cron schedule. Useful for nightly builds, weekly dependency checks, or scheduled reports.

```yaml
on:
  schedule:
    - cron: '0 2 * * *'    # every day at 2:00 AM UTC
```

**Cron syntax:**

```
┌─── minute (0-59)
│  ┌─── hour (0-23)
│  │  ┌─── day of month (1-31)
│  │  │  ┌─── month (1-12)
│  │  │  │  ┌─── day of week (0-6, 0=Sunday)
│  │  │  │  │
*  *  *  *  *
```

Common examples:

```yaml
- cron: '0 0 * * *'      # daily at midnight UTC
- cron: '0 8 * * 1-5'   # weekdays at 8 AM UTC
- cron: '0 0 * * 0'     # weekly on Sunday at midnight
- cron: '*/15 * * * *'  # every 15 minutes
```

Scheduled workflows always run on the default branch.

---

## workflow_dispatch (Manual Trigger)

Allows a workflow to be triggered manually from the GitHub UI or via the API.

```yaml
on:
  workflow_dispatch:
```

**With input parameters:**

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
      debug:
        description: 'Enable debug logging'
        required: false
        type: boolean
        default: false
```

Access inputs in steps:

```yaml
steps:
  - name: Deploy
    run: |
      echo "Deploying to ${{ github.event.inputs.environment }}"
      echo "Debug mode: ${{ github.event.inputs.debug }}"
```

To trigger from the UI: go to the repository → Actions tab → select the workflow → click "Run workflow".

---

## release

Triggers when a GitHub release is created or published.

```yaml
on:
  release:
    types:
      - published
```

Commonly used to build and push a Docker image tagged with the release version.

---

## Multiple Triggers

You can combine multiple triggers in one workflow:

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * 0'
  workflow_dispatch:
```

This workflow runs on pushes, PRs, weekly, and when manually triggered.
