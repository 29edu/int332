# Pipeline Stages

A Jenkins pipeline is organised into stages that represent the key phases of building and delivering software. Each stage is visible in the pipeline visualization and has a clear, single responsibility.

---

## Standard Pipeline Stages

```
Checkout → Build → Test → Package → [Deploy] → Post Actions
```

---

## Checkout

Fetches the source code from the Git repository onto the agent's workspace.

**Using `checkout scm` (recommended for Pipeline jobs):**

```groovy
stage('Checkout') {
  steps {
    checkout scm
  }
}
```

`checkout scm` uses the source configuration defined in the Jenkins job (the repository URL and branch). It automatically works for both regular Pipeline jobs and Multi-Branch Pipeline jobs.

**Explicit Git checkout:**

```groovy
stage('Checkout') {
  steps {
    git branch: 'main',
        credentialsId: 'github-ssh-key',
        url: 'git@github.com:myorg/myapp.git'
  }
}
```

**Checkout with shallow clone (faster for large repos):**

```groovy
checkout([
  $class: 'GitSCM',
  branches: [[name: '*/main']],
  extensions: [[$class: 'CloneOption', depth: 1, shallow: true]],
  userRemoteConfigs: [[url: 'https://github.com/myorg/myapp.git']]
])
```

---

## Build

Compiles the source code.

**Maven:**

```groovy
stage('Build') {
  steps {
    sh './mvnw clean compile -DskipTests'
  }
}
```

**Node.js:**

```groovy
stage('Build') {
  steps {
    sh 'npm install'
    sh 'npm run build'
  }
}
```

**Docker image build:**

```groovy
stage('Build Image') {
  steps {
    sh "docker build -t myapp:${env.BUILD_NUMBER} ."
  }
}
```

---

## Test

Runs unit and integration tests. Jenkins collects the JUnit XML reports to show pass/fail details in the UI.

**Maven unit tests:**

```groovy
stage('Test') {
  steps {
    sh './mvnw test'
  }
  post {
    always {
      junit 'target/surefire-reports/**/*.xml'
    }
  }
}
```

**Node.js tests:**

```groovy
stage('Test') {
  steps {
    sh 'npm test -- --reporters=junit --reporter-options="outputFile=test-results.xml"'
  }
  post {
    always {
      junit 'test-results.xml'
    }
  }
}
```

**Parallel test stages:**

```groovy
stage('Test') {
  parallel {
    stage('Unit Tests') {
      steps {
        sh './mvnw test'
      }
    }
    stage('Integration Tests') {
      steps {
        sh './mvnw verify -Pintegration-tests'
      }
    }
  }
  post {
    always {
      junit 'target/**/TEST-*.xml'
    }
  }
}
```

---

## Package

Creates the deployable artifact — a JAR, Docker image, or ZIP.

**Maven package:**

```groovy
stage('Package') {
  steps {
    sh './mvnw package -DskipTests'
  }
}
```

**Build and tag Docker image:**

```groovy
stage('Package') {
  steps {
    sh """
      docker build -t myapp:${env.BUILD_NUMBER} .
      docker tag myapp:${env.BUILD_NUMBER} myregistry.io/myapp:${env.BUILD_NUMBER}
      docker tag myapp:${env.BUILD_NUMBER} myregistry.io/myapp:latest
    """
  }
}
```

---

## Post Actions

Post actions run after all stages complete. They handle reporting, cleanup, and notifications.

```groovy
post {
  always {
    // runs regardless of result
    junit allowEmptyResults: true, testResults: 'target/surefire-reports/**/*.xml'
    cleanWs()       // delete workspace
  }
  success {
    echo "Build ${env.BUILD_NUMBER} passed"
  }
  failure {
    mail to: 'dev-team@company.com',
         subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
         body: "Build failed. See details at ${env.BUILD_URL}"
  }
  unstable {
    echo 'Some tests failed — build marked as unstable'
  }
}
```

---

## Managing Artifacts

Artifacts are files produced by a build (JARs, reports, logs) that you want to keep or pass to other stages.

**Archiving artifacts — saves files attached to the build:**

```groovy
stage('Archive') {
  steps {
    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
  }
}
```

Archived files are accessible from the Jenkins build page. `fingerprint: true` records a checksum to track where the artifact was used.

**Stash/unstash — passes files between stages on the same or different agent:**

```groovy
stage('Build') {
  steps {
    sh './mvnw package'
    stash name: 'jar-file', includes: 'target/*.jar'
  }
}

stage('Deploy') {
  agent { label 'deploy-agent' }
  steps {
    unstash 'jar-file'
    sh 'scp target/*.jar server:/opt/app/'
  }
}
```

`stash` stores files temporarily for the duration of the pipeline. `archiveArtifacts` stores files permanently with the build record.

---

## Complete Four-Stage Pipeline

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
        sh './mvnw clean compile'
      }
    }

    stage('Test') {
      steps {
        sh './mvnw test'
      }
    }

    stage('Package') {
      steps {
        sh './mvnw package -DskipTests'
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      }
    }
  }

  post {
    always {
      junit 'target/surefire-reports/**/*.xml'
      cleanWs()
    }
    success {
      echo "Build ${env.BUILD_NUMBER} completed successfully"
    }
    failure {
      echo "Build ${env.BUILD_NUMBER} failed"
    }
  }
}
```
