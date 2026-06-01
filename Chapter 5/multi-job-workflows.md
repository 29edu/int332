# Multi-Job Workflows

A multi-job workflow splits the pipeline into separate stages. Each job runs on its own runner, can run in parallel or sequence, and has a clear single responsibility.

---

## Why Use Multiple Jobs

A single job with all steps is simple but has limits:

- if tests fail, the deploy step is skipped — but you still wait for the whole job to set up
- you cannot run different OS builds in parallel
- it is hard to see which stage failed at a glance

With multiple jobs:

- stages are clearly separated (build, test, deploy)
- independent jobs run in parallel, saving time
- the dependency chain is explicit with `needs:`
- each job can use a different OS or environment

---

## Basic Multi-Job Pipeline

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven
      - run: ./mvnw clean package -DskipTests
      - uses: actions/upload-artifact@v4
        with:
          name: app-jar
          path: target/*.jar

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven
      - run: ./mvnw test
      - uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: target/surefire-reports/

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: app-jar
      - run: echo "Deploying app..."
```

Flow:

```
build → test → deploy (only on main)
```

---

## Parallel Jobs

Jobs that do not depend on each other run at the same time.

```yaml
jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: ./mvnw test

  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: ./mvnw checkstyle:check

  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: ./mvnw dependency-check:check

  deploy:
    needs: [unit-tests, lint, security-scan]
    runs-on: ubuntu-latest
    steps:
      - run: echo "All checks passed, deploying"
```

```
unit-tests ─┐
lint        ─┼→ deploy
security    ─┘
```

All three checks run simultaneously. Deploy only runs after all three pass.

---

## Conditional Job Execution

**Run a job only on specific branches:**

```yaml
deploy-staging:
  needs: test
  if: github.ref == 'refs/heads/develop'
  runs-on: ubuntu-latest
  steps:
    - run: ./deploy.sh staging

deploy-production:
  needs: test
  if: github.ref == 'refs/heads/main'
  runs-on: ubuntu-latest
  steps:
    - run: ./deploy.sh production
```

**Run a job only when triggered manually:**

```yaml
rollback:
  if: github.event_name == 'workflow_dispatch'
  runs-on: ubuntu-latest
  steps:
    - run: ./rollback.sh
```

**Always run a cleanup job even if others fail:**

```yaml
cleanup:
  needs: [build, test, deploy]
  if: always()
  runs-on: ubuntu-latest
  steps:
    - run: ./cleanup.sh
```

---

## Environment Protection Rules

Jobs that deploy to production can require manual approval through GitHub Environments.

```yaml
deploy:
  needs: test
  runs-on: ubuntu-latest
  environment: production     # links to a protected GitHub Environment
  steps:
    - run: ./deploy-prod.sh
```

In the GitHub repository settings, the `production` environment can be configured to:
- require one or more reviewers to approve before the job runs
- restrict which branches can deploy to it
- set environment-specific secrets

This prevents accidental deploys to production — a reviewer must click "Approve" in the GitHub UI before the deploy job starts.

---

## Complete CI/CD Multi-Job Example

```yaml
name: Full CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.value }}
    steps:
      - uses: actions/checkout@v4
      - id: version
        run: echo "value=1.0.${{ github.run_number }}" >> $GITHUB_OUTPUT
      - uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven
      - run: ./mvnw clean package -DskipTests
      - uses: actions/upload-artifact@v4
        with:
          name: jar-${{ github.sha }}
          path: target/*.jar

  unit-test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven
      - run: ./mvnw test

  docker:
    needs: unit-test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - uses: actions/download-artifact@v4
        with:
          name: jar-${{ github.sha }}
          path: target/
      - uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      - uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: myuser/myapp:${{ needs.build.outputs.version }}

  deploy:
    needs: docker
    runs-on: ubuntu-latest
    environment: production
    if: github.ref == 'refs/heads/main'
    steps:
      - run: echo "Deploying version ${{ needs.build.outputs.version }}"
```
