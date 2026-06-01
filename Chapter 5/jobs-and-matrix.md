# Jobs and Matrix Strategies

Jobs are the main execution units in a GitHub Actions workflow. Matrix strategies let you run the same job multiple times across different combinations of variables.

---

## Jobs

A job is a set of steps that run on the same runner in sequence. All jobs in a workflow run in parallel by default.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "building"

  lint:
    runs-on: ubuntu-latest
    steps:
      - run: echo "linting"
```

`build` and `lint` run at the same time on separate runners.

---

## Job Dependencies with needs

Use `needs:` to make jobs run in sequence.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "build"

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "test"

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - run: echo "deploy"
```

```
build → test → deploy
```

A job with multiple dependencies:

```yaml
jobs:
  deploy:
    needs: [unit-test, integration-test, build]
```

`deploy` only runs after all three jobs pass.

---

## Passing Data Between Jobs with Outputs

Jobs run on separate machines and do not share files. Use `outputs:` to pass values between jobs.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.get-version.outputs.version }}
    steps:
      - id: get-version
        run: echo "version=1.0.${{ github.run_number }}" >> $GITHUB_OUTPUT

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying version ${{ needs.build.outputs.version }}"
```

---

## Sharing Files Between Jobs with Artifacts

To share built files between jobs, upload them from one job and download them in another.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: ./mvnw clean package
      - uses: actions/upload-artifact@v4
        with:
          name: app-jar
          path: target/*.jar

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: app-jar
      - run: ls *.jar
```

---

## Matrix Strategy

A matrix runs the same job multiple times with different variable values. This is used to test across multiple OS versions, language versions, or environments simultaneously.

**Basic matrix — test on multiple Node.js versions:**

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm install && npm test
```

This creates three parallel jobs: one for Node 18, one for 20, one for 22.

**Multi-dimensional matrix — test across OS and language version:**

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        java: [17, 21]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v3
        with:
          java-version: ${{ matrix.java }}
          distribution: 'temurin'
      - run: ./mvnw test
```

This produces 6 jobs: 3 OS × 2 Java versions.

**Excluding specific combinations:**

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
    java: [17, 21]
    exclude:
      - os: windows-latest
        java: 21
```

Removes the `windows + java 21` combination.

**Adding extra variables to specific combinations:**

```yaml
strategy:
  matrix:
    java: [17, 21]
    include:
      - java: 21
        experimental: true
```

---

## Fail Fast

By default, if one matrix job fails, GitHub cancels all remaining matrix jobs. To disable this:

```yaml
strategy:
  fail-fast: false
  matrix:
    node-version: [18, 20, 22]
```

Useful when you want to see all failures across versions, not just the first one.

---

## Concurrency Control

Prevent multiple workflow runs from deploying at the same time:

```yaml
concurrency:
  group: deploy-${{ github.ref }}
  cancel-in-progress: true
```

If a new push triggers the workflow while one is already running for the same branch, the running one is cancelled and the new one starts.
