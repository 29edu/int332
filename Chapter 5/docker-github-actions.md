# Docker and GitHub Actions

GitHub Actions integrates tightly with Docker. You can build images, run tests inside containers, push to registries, and trigger deployments — all within a single workflow.

---

## Building Docker Images in CI

The standard way to build Docker images in GitHub Actions is with `docker/build-push-action`, which uses BuildKit for faster, more efficient builds.

**Basic image build (no push):**

```yaml
name: Build Docker Image

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: false
          tags: myapp:latest
```

`docker/setup-buildx-action` enables BuildKit, which supports multi-platform builds and better layer caching.

---

## Pushing to Docker Hub

**Step 1 — Add secrets to your repository:**
- `DOCKER_USERNAME` — your Docker Hub username
- `DOCKER_PASSWORD` — your Docker Hub password or access token (preferred over password)

**Workflow:**

```yaml
name: Build and Push to Docker Hub

on:
  push:
    branches: [main]

jobs:
  build-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ${{ secrets.DOCKER_USERNAME }}/myapp:latest
            ${{ secrets.DOCKER_USERNAME }}/myapp:${{ github.sha }}
```

The image is pushed with two tags: `latest` and the full commit SHA for traceability.

---

## Pushing to GitHub Container Registry (GHCR)

GHCR is GitHub's own container registry. Images are hosted at `ghcr.io` and linked to your GitHub account or organisation. No separate account is needed.

**Authentication uses `GITHUB_TOKEN`** — a token automatically provided to every workflow with no manual setup.

```yaml
name: Build and Push to GHCR

on:
  push:
    branches: [main]

jobs:
  build-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write         # required to push to GHCR

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository_owner }}/myapp:latest
```

The `permissions:` block grants the workflow write access to GitHub Packages (GHCR). Without it, the push will fail with a 403 error.

---

## Auto-Tagging with docker/metadata-action

For real releases, you want tags like `v1.2.3` from git tags, branch names, and `latest` automatically. `docker/metadata-action` generates these for you.

```yaml
- name: Docker metadata
  id: meta
  uses: docker/metadata-action@v5
  with:
    images: ghcr.io/${{ github.repository_owner }}/myapp
    tags: |
      type=ref,event=branch          # branch name: main
      type=ref,event=pr              # pr number: pr-42
      type=semver,pattern={{version}}  # git tag: v1.2.3 → 1.2.3
      type=semver,pattern={{major}}.{{minor}}  # 1.2
      type=sha                       # short commit SHA

- name: Build and push
  uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    tags: ${{ steps.meta.outputs.tags }}
    labels: ${{ steps.meta.outputs.labels }}
```

When you push a git tag like `v1.2.3`, this produces images tagged as `1.2.3`, `1.2`, and the commit SHA — all in one step.

---

## Multi-Platform Builds

Build images for multiple CPU architectures (Intel and ARM) in one step:

```yaml
- name: Set up QEMU
  uses: docker/setup-qemu-action@v3

- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3

- name: Build and push multi-platform
  uses: docker/build-push-action@v5
  with:
    context: .
    platforms: linux/amd64,linux/arm64
    push: true
    tags: ghcr.io/myorg/myapp:latest
```

QEMU enables emulation of non-native architectures on the runner.

---

## Deployments to Servers and Clouds

### Deploy to a VPS/Server via SSH

After pushing the image, SSH into the server and pull the new image:

```yaml
- name: Deploy to server
  uses: appleboy/ssh-action@v1.0.0
  with:
    host: ${{ secrets.SERVER_HOST }}
    username: ${{ secrets.SERVER_USER }}
    key: ${{ secrets.SSH_PRIVATE_KEY }}
    script: |
      docker pull ghcr.io/myorg/myapp:latest
      docker stop myapp || true
      docker rm myapp || true
      docker run -d \
        --name myapp \
        --restart always \
        -p 8080:8080 \
        ghcr.io/myorg/myapp:latest
```

### Deploy to AWS ECS

```yaml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: us-east-1

- name: Login to Amazon ECR
  uses: aws-actions/amazon-ecr-login@v2

- name: Build and push to ECR
  uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    tags: ${{ steps.login-ecr.outputs.registry }}/myapp:latest

- name: Deploy to ECS
  uses: aws-actions/amazon-ecs-deploy-task-definition@v1
  with:
    task-definition: task-definition.json
    service: myapp-service
    cluster: myapp-cluster
    wait-for-service-stability: true
```

### Deploy to Azure Container Apps

```yaml
- uses: azure/login@v1
  with:
    creds: ${{ secrets.AZURE_CREDENTIALS }}

- uses: azure/container-apps-deploy-action@v1
  with:
    appSourcePath: ${{ github.workspace }}
    acrName: myregistry
    containerAppName: myapp
    resourceGroup: my-resource-group
```

### Deploy with Docker Compose on a server

```yaml
- name: Deploy with Docker Compose
  uses: appleboy/ssh-action@v1.0.0
  with:
    host: ${{ secrets.SERVER_HOST }}
    username: ${{ secrets.SERVER_USER }}
    key: ${{ secrets.SSH_PRIVATE_KEY }}
    script: |
      cd /opt/myapp
      echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
      docker compose pull
      docker compose up -d --remove-orphans
      docker image prune -f
```

---

## Full Spring Boot CI/CD Pipeline

A complete end-to-end example: build JAR → run tests → build Docker image → push to GHCR → deploy to server.

```yaml
name: CI/CD

on:
  push:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven
      - run: ./mvnw clean verify
      - uses: actions/upload-artifact@v4
        with:
          name: jar
          path: target/*.jar

  docker-push:
    needs: build-and-test
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read
    steps:
      - uses: actions/checkout@v4
      - uses: actions/download-artifact@v4
        with:
          name: jar
          path: target/
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository_owner }}/myapp:${{ github.sha }}

  deploy:
    needs: docker-push
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            docker pull ghcr.io/${{ github.repository_owner }}/myapp:${{ github.sha }}
            docker stop myapp || true
            docker rm myapp || true
            docker run -d --name myapp --restart always -p 8080:8080 \
              ghcr.io/${{ github.repository_owner }}/myapp:${{ github.sha }}
```
