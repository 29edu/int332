# Deployments, Backup, and Best Practices

---

## Triggering Builds

### pollSCM

Jenkins polls the Git repository on a schedule. If new commits are found, it triggers a build.

```groovy
triggers {
  pollSCM('H/5 * * * *')    // check every 5 minutes
}
```

`H` is a hash of the job name used to spread the load — all jobs do not poll at exactly the same second.

**Downside:** there is a delay between the push and the build (up to 5 minutes with the above config). Also wastes resources — Jenkins polls even when nothing has changed.

### Webhooks (Preferred)

GitHub/GitLab sends an HTTP POST to Jenkins immediately when a push or PR event happens. No polling, no delay.

**Setup:**

1. In Jenkins: Manage Jenkins → System → GitHub → Add GitHub Server → set the URL
2. In the pipeline, enable the GitHub hook trigger:

```groovy
triggers {
  githubPush()
}
```

3. In GitHub: Repository → Settings → Webhooks → Add webhook
   - Payload URL: `http://your-jenkins:8080/github-webhook/`
   - Content type: `application/json`
   - Events: Just the push event (or pull requests too)

With Multi-Branch Pipeline and the GitHub Branch Source plugin, webhooks are configured automatically.

---

## Deploying to Servers

### SSH deploy

```groovy
stage('Deploy to Server') {
  steps {
    sshagent(['prod-server-key']) {
      sh '''
        ssh -o StrictHostKeyChecking=no deploy@prod-server "
          docker pull ghcr.io/myorg/myapp:${BUILD_NUMBER}
          docker stop myapp || true
          docker rm myapp || true
          docker run -d --name myapp --restart always -p 8080:8080 \
            ghcr.io/myorg/myapp:${BUILD_NUMBER}
        "
      '''
    }
  }
}
```

### Deploy with Docker Compose on a remote server

```groovy
stage('Deploy') {
  steps {
    sshagent(['prod-server-key']) {
      sh """
        scp docker-compose.yml deploy@prod-server:/opt/myapp/
        ssh deploy@prod-server '
          cd /opt/myapp
          export IMAGE_TAG=${BUILD_NUMBER}
          docker compose pull
          docker compose up -d --remove-orphans
        '
      """
    }
  }
}
```

### Deploy to AWS ECS

```groovy
stage('Deploy to ECS') {
  steps {
    withCredentials([[
      $class: 'AmazonWebServicesCredentialsBinding',
      credentialsId: 'aws-creds'
    ]]) {
      sh """
        aws ecs update-service \
          --cluster my-cluster \
          --service my-service \
          --force-new-deployment \
          --region us-east-1
      """
    }
  }
}
```

### Deploy to Kubernetes

```groovy
stage('Deploy to Kubernetes') {
  steps {
    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
      sh """
        kubectl set image deployment/myapp \
          myapp=ghcr.io/myorg/myapp:${BUILD_NUMBER} \
          --kubeconfig=$KUBECONFIG
        kubectl rollout status deployment/myapp \
          --kubeconfig=$KUBECONFIG
      """
    }
  }
}
```

---

## Backup and Restore

Jenkins stores all its configuration, job definitions, build history, credentials, and plugin data in the `JENKINS_HOME` directory.

**Default JENKINS_HOME locations:**

- Linux: `/var/lib/jenkins`
- Docker: `/var/jenkins_home` (inside the container)
- Windows: `C:\ProgramData\Jenkins\.jenkins`

### What to Back Up

| Path | Contents |
|---|---|
| `JENKINS_HOME/config.xml` | Main Jenkins configuration |
| `JENKINS_HOME/jobs/` | All job configurations and build history |
| `JENKINS_HOME/credentials.xml` | Stored credentials (encrypted) |
| `JENKINS_HOME/secrets/` | Encryption keys — critical, back up securely |
| `JENKINS_HOME/plugins/` | Installed plugins |
| `JENKINS_HOME/nodes/` | Agent configurations |

**Do not need to back up:**
- `JENKINS_HOME/workspace/` — can be recreated by running a build
- `JENKINS_HOME/war/` — Jenkins application itself (reinstall from the package)

### Backup with ThinBackup Plugin

1. Install the ThinBackup plugin
2. Manage Jenkins → ThinBackup → Settings:
   - Backup directory: `/backup/jenkins`
   - Schedule full backups: `0 2 * * 0` (weekly)
   - Schedule differential backups: `0 2 * * 1-6` (daily)
3. To trigger manually: Manage Jenkins → ThinBackup → Backup Now

### Manual Backup (Shell / Cron)

```bash
#!/bin/bash
JENKINS_HOME=/var/lib/jenkins
BACKUP_DIR=/backup/jenkins
DATE=$(date +%Y%m%d-%H%M)

tar -czf ${BACKUP_DIR}/jenkins-${DATE}.tar.gz \
  --exclude=${JENKINS_HOME}/workspace \
  --exclude=${JENKINS_HOME}/war \
  ${JENKINS_HOME}

# Keep only the last 7 backups
ls -t ${BACKUP_DIR}/jenkins-*.tar.gz | tail -n +8 | xargs rm -f
```

Add to crontab:

```bash
0 2 * * * /opt/scripts/backup-jenkins.sh
```

### Restore

```bash
sudo systemctl stop jenkins

sudo tar -xzf /backup/jenkins/jenkins-20240601-0200.tar.gz -C /

sudo systemctl start jenkins
```

---

## Pipeline Best Practices

**Keep Jenkinsfile in the repository**
The pipeline definition lives with the code. Changes are tracked, reviewed, and rolled back with the same tools as the application code.

**Use Declarative syntax**
Easier to read and validate. Use `script {}` blocks for Groovy logic when needed.

**Fail fast**
Put quick stages (lint, compile) before slow ones (integration tests, deploy). The pipeline gives feedback sooner and wastes less time on failures.

```
Checkout → Compile → Unit Test → Integration Test → Package → Deploy
  (fast)     (fast)    (medium)       (slow)         (medium)  (manual)
```

**Keep stages focused**
Each stage should do one thing. "Build and Test and Package" is harder to diagnose than three separate stages.

**Use credentials properly**
Never hardcode passwords, tokens, or API keys in the Jenkinsfile. Always use Jenkins Credentials and `withCredentials {}`.

**Archive test results**
Always publish JUnit results with `junit`. This gives trend data and makes it easy to see which tests started failing.

**Clean the workspace**
Use `cleanWs()` in `post { always {} }` to prevent old build files from affecting the next build.

**Set timeouts**
Prevent runaway builds from blocking agents indefinitely:

```groovy
options {
  timeout(time: 30, unit: 'MINUTES')
}
```

**Limit parallel stages**
Running many parallel stages speeds up the build but increases agent load. Find the right balance for your infrastructure.

**Use shared libraries for common logic**
If multiple pipelines share the same stages (checkout, Maven build, Docker push), extract that into a shared library and maintain it in one place.

**Use `input` step for production deploys**
Require manual approval before deploying to production:

```groovy
stage('Deploy to Production') {
  steps {
    input message: 'Deploy to production?', ok: 'Deploy'
    sh './deploy.sh production'
  }
}
```

**Pin plugin versions**
In production Jenkins, do not auto-update plugins. Test updates in a staging Jenkins instance first and then apply them to production during a maintenance window.
