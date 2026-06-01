# Multi-Branch Pipeline

A Multi-Branch Pipeline job automatically discovers branches and pull requests in a repository, creates a pipeline for each one, and runs the `Jenkinsfile` found in that branch.

---

## Why Multi-Branch

Without multi-branch:
- you create one pipeline job per branch manually
- when a developer creates a new branch, they also have to create a new Jenkins job
- when the branch is deleted, the job stays behind unless removed manually

With multi-branch:
- Jenkins scans the repository and automatically creates a pipeline for every branch that has a `Jenkinsfile`
- new branches get pipelines automatically
- deleted branches get their pipelines removed automatically

---

## Creating a Multi-Branch Pipeline

1. Dashboard → New Item
2. Enter a name → select **Multibranch Pipeline** → OK
3. Under **Branch Sources**, add a source:
   - **GitHub** (requires GitHub plugin and credentials)
   - **Git** (any Git URL)
4. Set the repository URL and credentials
5. Under **Build Configuration**, set the script path to `Jenkinsfile` (default)
6. Save → Jenkins scans the repository and creates pipelines

---

## How it Works

```
Repository branches:
  main         → has Jenkinsfile → pipeline created and run
  develop      → has Jenkinsfile → pipeline created and run
  feature/auth → has Jenkinsfile → pipeline created and run
  hotfix/bug   → no Jenkinsfile  → ignored

Jenkins job tree:
  my-app (Multibranch Pipeline)
    ├── main
    ├── develop
    └── feature/auth
```

Each branch pipeline is independent. A failing build on `feature/auth` does not affect `main`.

---

## Branch-Specific Behaviour in Jenkinsfile

Because `env.BRANCH_NAME` is set, you can write branch-specific logic in the same `Jenkinsfile`:

```groovy
pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        sh './mvnw clean package'
      }
    }

    stage('Test') {
      steps {
        sh './mvnw test'
      }
    }

    stage('Deploy to Staging') {
      when {
        branch 'develop'
      }
      steps {
        sh './deploy.sh staging'
      }
    }

    stage('Deploy to Production') {
      when {
        branch 'main'
      }
      steps {
        input message: 'Approve production deploy?', ok: 'Deploy'
        sh './deploy.sh production'
      }
    }

    stage('PR Validation') {
      when {
        changeRequest()             // runs only on pull requests
      }
      steps {
        sh './mvnw verify'
      }
    }
  }
}
```

**when conditions for branches:**

| Condition | Matches |
|---|---|
| `branch 'main'` | Exact branch name |
| `branch 'feature/*'` | Wildcard match |
| `changeRequest()` | Pull requests only |
| `not { branch 'main' }` | All branches except main |

---

## Scan Triggers

Jenkins can automatically scan the repository for new or deleted branches:

- **Periodically** — set an interval (e.g. every 5 minutes) in the Multibranch Pipeline configuration
- **Webhook** — configure GitHub/GitLab to send a webhook to Jenkins on branch creation

Configure: Job → Configure → Scan Multibranch Pipeline Triggers

---

## Orphaned Item Strategy

When a branch is deleted from the repository, Jenkins can:
- delete the pipeline job immediately
- keep it for a configured period (e.g. 7 days) before removing it

Configure: Job → Configure → Orphaned Item Strategy

---

## Environment Variables Set by Multi-Branch

Jenkins automatically sets these when running a multi-branch pipeline:

| Variable | Value |
|---|---|
| `BRANCH_NAME` | The branch name (e.g. `main`, `feature/login`) |
| `CHANGE_ID` | PR number (only on pull requests) |
| `CHANGE_TARGET` | Target branch of the PR |
| `CHANGE_AUTHOR` | GitHub username who opened the PR |

---

## Organisation Folders (GitHub Organisations)

One level above multi-branch is the Organisation Folder. It scans an entire GitHub organisation and creates Multi-Branch Pipeline jobs for every repository that has a `Jenkinsfile`.

Requires the **GitHub Branch Source** plugin.

```
GitHub Organisation: my-company
  ├── repo-A (has Jenkinsfile) → Multibranch Pipeline job
  │     ├── main
  │     └── develop
  ├── repo-B (has Jenkinsfile) → Multibranch Pipeline job
  │     └── main
  └── repo-C (no Jenkinsfile) → ignored
```

New repositories are picked up automatically on the next scan.
