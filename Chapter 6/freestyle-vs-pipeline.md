# Freestyle vs Pipeline Jobs

Jenkins has two main job types: Freestyle and Pipeline. They represent two different ways to define and run builds.

---

## Freestyle Jobs

A Freestyle job is the classic Jenkins job type. The entire configuration is done through the web UI — no code files needed.

**How to create:**
Dashboard → New Item → Freestyle project

**Configuration sections:**

- **General** — job description, discard old builds, parameters
- **Source Code Management** — Git repository URL, branch, credentials
- **Build Triggers** — when to run (cron, webhook, after another job)
- **Build Environment** — clean workspace, inject environment variables
- **Build Steps** — shell commands, Maven goals, Gradle tasks, batch commands
- **Post-build Actions** — archive artifacts, publish test results, email notifications, trigger other jobs

**Example Freestyle job build step:**

```bash
mvn clean package -DskipTests
```

**Pros:**
- quick to set up
- no scripting knowledge needed
- good for simple, single-step builds

**Cons:**
- configuration lives in Jenkins UI only — not versioned with code
- hard to maintain as the pipeline grows complex
- difficult to reuse logic across jobs
- limited control flow (no `if`, loops, parallel stages)
- cannot be reviewed or tested like code

---

## Pipeline Jobs

A Pipeline job defines the build as code — written in Groovy and stored in a `Jenkinsfile`. The file lives in the source code repository alongside the application.

**How to create:**
Dashboard → New Item → Pipeline

**Two pipeline syntaxes:**
- **Declarative** — structured, opinionated, easier to read
- **Scripted** — full Groovy, more flexible but more complex

**Why Pipeline is preferred:**

- the `Jenkinsfile` is versioned in Git — changes are tracked and reviewable
- the same pipeline definition is used for all branches
- supports complex control flow: parallel stages, conditions, loops
- stages are visualized in Blue Ocean
- can be tested and validated before running
- sharable across projects via shared libraries

---

## Side-by-Side Comparison

| | Freestyle | Pipeline |
|---|---|---|
| Configuration | Web UI only | Jenkinsfile in Git |
| Versioning | Not versioned | Versioned with code |
| Scripting | Not required | Groovy DSL |
| Complex logic | Very limited | Full control flow |
| Parallel stages | Not supported | Supported |
| Reusability | Low | High (shared libraries) |
| Visualization | Basic | Rich (Blue Ocean) |
| Best for | Simple one-off tasks | All real CI/CD pipelines |

---

## When to Use Each

**Use Freestyle when:**
- you need a quick one-off task with no complex logic
- the team has no Groovy knowledge and the pipeline is truly simple
- running a single command or script that does not need stages

**Use Pipeline when:**
- you want the build definition in version control
- the pipeline has multiple stages (build, test, deploy)
- you need conditions, loops, or parallel execution
- the pipeline will grow over time
- multiple jobs share similar logic

In practice, Freestyle jobs are rarely used for real CI/CD today. Pipeline jobs with a `Jenkinsfile` are the standard.

---

## Converting a Freestyle Job to a Pipeline

A Freestyle job with these steps:

1. Git clone
2. `mvn clean package`
3. Archive `target/*.jar`

Becomes this Declarative Pipeline:

```groovy
pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        sh 'mvn clean package'
      }
    }

    stage('Archive') {
      steps {
        archiveArtifacts artifacts: 'target/*.jar'
      }
    }
  }
}
```
