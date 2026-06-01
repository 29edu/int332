# Docker and Jenkins Integration

Jenkins and Docker work together in two main ways: Jenkins can build Docker images as part of a pipeline, and Jenkins agents can themselves run inside Docker containers.

---

## Required Plugins

Install these from Manage Jenkins → Plugins:

- **Docker Pipeline** — enables `docker.build()`, `docker.image()`, and `docker.withRegistry()` in pipelines
- **Docker** — connects Jenkins to a Docker host

---

## Building Docker Images in a Pipeline

**Using shell commands (simplest approach):**

```groovy
pipeline {
  agent any

  environment {
    IMAGE = "myapp:${env.BUILD_NUMBER}"
  }

  stages {
    stage('Build Image') {
      steps {
        sh "docker build -t ${IMAGE} ."
      }
    }
  }
}
```

**Using the Docker Pipeline DSL:**

```groovy
pipeline {
  agent any

  stages {
    stage('Build Image') {
      steps {
        script {
          def image = docker.build("myapp:${env.BUILD_NUMBER}")
        }
      }
    }
  }
}
```

---

## Docker Inside Jenkins Agents

Jenkins agents can run build steps inside Docker containers. This means you do not need to pre-install build tools (Java, Node, Maven) on the agent — Docker pulls the image and runs the step inside it.

**Run all stages inside a Docker container:**

```groovy
pipeline {
  agent {
    docker {
      image 'maven:3.9-eclipse-temurin-17'
      args '-v $HOME/.m2:/root/.m2'   // mount Maven cache
    }
  }

  stages {
    stage('Build') {
      steps {
        sh 'mvn clean package'
      }
    }
  }
}
```

Jenkins pulls `maven:3.9-eclipse-temurin-17`, runs all pipeline steps inside it, and removes the container when done.

**Different container per stage:**

```groovy
pipeline {
  agent none

  stages {
    stage('Build') {
      agent {
        docker { image 'maven:3.9-eclipse-temurin-17' }
      }
      steps {
        sh 'mvn clean package'
        stash name: 'jar', includes: 'target/*.jar'
      }
    }

    stage('Test') {
      agent {
        docker { image 'maven:3.9-eclipse-temurin-17' }
      }
      steps {
        sh 'mvn test'
      }
    }

    stage('Build Image') {
      agent any    // needs Docker daemon, not Maven container
      steps {
        unstash 'jar'
        sh 'docker build -t myapp .'
      }
    }
  }
}
```

---

## Using Docker Plugins (Docker Pipeline DSL)

The Docker Pipeline plugin provides a Groovy DSL for working with images and registries.

**Build an image:**

```groovy
def img = docker.build("myapp:${env.BUILD_NUMBER}", "--no-cache .")
```

**Run a container and execute commands inside it:**

```groovy
docker.image('postgres:15').withRun('-e POSTGRES_PASSWORD=test') { db ->
  docker.image('maven:3.9').inside("--link ${db.id}:db") {
    sh 'mvn verify'
  }
}
```

This starts a PostgreSQL container, then runs Maven tests inside a Maven container that is linked to the database — useful for integration tests.

---

## Publishing to Docker Hub

**Step 1 — Add a credential in Jenkins:**
Manage Jenkins → Credentials → Add
- Type: Username with password
- Username: Docker Hub username
- Password: Docker Hub access token
- ID: `docker-hub-creds`

**Step 2 — Pipeline:**

```groovy
pipeline {
  agent any

  environment {
    DOCKER_IMAGE = "myuser/myapp:${env.BUILD_NUMBER}"
  }

  stages {
    stage('Build & Push') {
      steps {
        script {
          docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-creds') {
            def img = docker.build(DOCKER_IMAGE)
            img.push()
            img.push('latest')
          }
        }
      }
    }
  }
}
```

`docker.withRegistry()` logs in, pushes the image, and logs out automatically.

---

## Publishing to GitHub Container Registry (GHCR)

**Step 1 — Add a credential:**
- Type: Username with password
- Username: GitHub username
- Password: GitHub Personal Access Token (PAT) with `write:packages` scope
- ID: `ghcr-creds`

**Step 2 — Pipeline:**

```groovy
pipeline {
  agent any

  environment {
    REGISTRY   = 'ghcr.io'
    IMAGE_NAME = 'ghcr.io/myorg/myapp'
    IMAGE_TAG  = "${env.BUILD_NUMBER}"
  }

  stages {
    stage('Build & Push to GHCR') {
      steps {
        script {
          docker.withRegistry("https://${REGISTRY}", 'ghcr-creds') {
            def img = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
            img.push()
            img.push('latest')
          }
        }
      }
    }
  }
}
```

---

## Jenkins and GitHub Integration

**GitHub plugin** enables:
- triggering builds on push via webhooks
- reporting build status back to GitHub (green/red check on commits and PRs)
- scanning repositories for Multi-Branch Pipeline jobs

**Reporting build status to GitHub:**

```groovy
pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        githubNotify status: 'PENDING', description: 'Build started'
        sh './mvnw clean package'
      }
    }
  }

  post {
    success {
      githubNotify status: 'SUCCESS', description: 'Build passed'
    }
    failure {
      githubNotify status: 'FAILURE', description: 'Build failed'
    }
  }
}
```

With the GitHub Branch Source plugin and proper credential setup, Jenkins automatically reports pipeline status to the GitHub PR check.

---

## Docker Agent with Docker Socket

To run Docker commands inside a Jenkins agent container, mount the Docker socket:

```groovy
agent {
  docker {
    image 'docker:24-dind'
    args '-v /var/run/docker.sock:/var/run/docker.sock'
  }
}
```

This gives the container access to the host Docker daemon. Use with caution — it gives the container root-level access to the host.
