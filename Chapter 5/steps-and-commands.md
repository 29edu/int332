# Steps and Shell Commands

Steps are the individual units of work inside a job. They run one after another in the order they are defined. Each step either runs a shell command or uses a pre-built action.

---

## Step Structure

```yaml
steps:
  - name: Step display name      # optional, shown in GitHub UI logs
    id: my-step                  # optional, used to reference this step's outputs
    uses: actions/checkout@v4    # use a pre-built action
    # OR
    run: echo "hello"            # run a shell command
    with:                        # inputs for actions
      key: value
    env:                         # environment variables for this step
      MY_VAR: hello
    if: ${{ condition }}         # conditional execution
    continue-on-error: true      # don't fail the job if this step fails
```

---

## run — Shell Commands

The `run:` key executes shell commands on the runner.

**Single line:**

```yaml
- run: npm install
```

**Multiple lines using the pipe character:**

```yaml
- name: Build and test
  run: |
    echo "Starting build"
    npm install
    npm run build
    npm test
```

**Specifying the shell explicitly:**

```yaml
- name: Run bash script
  run: echo "Running in bash"
  shell: bash

- name: Run PowerShell
  run: Write-Host "Running in PowerShell"
  shell: pwsh

- name: Run Python
  run: print("hello from python")
  shell: python
```

On Linux/macOS runners, the default shell is `bash`. On Windows runners, the default is `pwsh` (PowerShell).

---

## Environment Variables

**Set for a single step:**

```yaml
- name: Run with env
  run: echo "App is $APP_NAME version $APP_VERSION"
  env:
    APP_NAME: myapp
    APP_VERSION: 1.0.0
```

**Set for the whole job:**

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    env:
      NODE_ENV: production
      APP_PORT: 3000
    steps:
      - run: echo "Port is $APP_PORT"
```

**Set for the whole workflow:**

```yaml
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: myapp

jobs:
  build:
    steps:
      - run: echo "Pushing to $REGISTRY/$IMAGE_NAME"
```

---

## Accessing GitHub Context Variables

GitHub provides built-in variables through contexts:

```yaml
steps:
  - run: |
      echo "Repository: ${{ github.repository }}"
      echo "Branch: ${{ github.ref_name }}"
      echo "Commit SHA: ${{ github.sha }}"
      echo "Actor: ${{ github.actor }}"
      echo "Run number: ${{ github.run_number }}"
      echo "Event: ${{ github.event_name }}"
```

Common context variables:

| Variable | Value |
|---|---|
| `github.repository` | `owner/repo-name` |
| `github.ref_name` | branch or tag name (e.g. `main`) |
| `github.sha` | full commit SHA |
| `github.actor` | username who triggered the workflow |
| `github.run_number` | number that increments with each run |
| `github.event_name` | `push`, `pull_request`, etc. |

---

## Secrets

Sensitive values like passwords and tokens are stored as GitHub repository secrets and accessed via `secrets`:

```yaml
steps:
  - name: Login to Docker Hub
    run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
```

Secrets are masked in logs — GitHub replaces their values with `***`.

To add a secret: repository Settings → Secrets and variables → Actions → New repository secret.

---

## Conditional Steps

Run a step only when a condition is true using `if:`:

```yaml
- name: Deploy to production
  if: github.ref == 'refs/heads/main'
  run: ./deploy.sh

- name: Notify on failure
  if: failure()
  run: curl -X POST ${{ secrets.SLACK_WEBHOOK }} -d '{"text":"Build failed"}'

- name: Run only on manual trigger
  if: github.event_name == 'workflow_dispatch'
  run: echo "manually triggered"
```

Built-in status functions:

| Function | When true |
|---|---|
| `success()` | All previous steps passed |
| `failure()` | Any previous step failed |
| `cancelled()` | Workflow was cancelled |
| `always()` | Always runs regardless of status |

---

## Step Outputs

Steps can produce output values that other steps in the same job can read:

```yaml
steps:
  - name: Get version
    id: version
    run: echo "value=1.0.${{ github.run_number }}" >> $GITHUB_OUTPUT

  - name: Use version
    run: echo "Version is ${{ steps.version.outputs.value }}"
```

`$GITHUB_OUTPUT` is a special file provided by the runner. Writing `key=value` to it makes the value available through `steps.<id>.outputs.<key>`.
