# Declarative Pipeline

Declarative Pipeline is the modern, recommended way to write Jenkins pipelines. It uses a structured Groovy DSL that is easier to read, write, and validate than Scripted Pipeline.

---

## Jenkinsfile

A `Jenkinsfile` is a text file that defines the pipeline. It lives in the root of the source code repository and is versioned with the code.

```
my-project/
├── src/
├── pom.xml
└── Jenkinsfile        ← pipeline definition
```

Jenkins reads this file when the job runs and executes the defined stages.

---

## Basic Structure

Every Declarative Pipeline starts with `pipeline {}` and must have at least `agent` and `stages`.

```groovy
pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        sh 'echo building'
      }
    }
  }
}
```

**Top-level blocks:**

| Block | Purpose |
|---|---|
| `agent` | Where to run the pipeline |
| `environment` | Define environment variables |
| `options` | Pipeline-level settings |
| `parameters` | User-defined input parameters |
| `triggers` | Automatic build triggers |
| `stages` | The build stages |
| `post` | Actions after all stages finish |

---

## agent

Specifies where the pipeline (or a stage) runs.

**Run on any available agent:**

```groovy
agent any
```

**Run on a specific agent by label:**

```groovy
agent {
  label 'linux-java17'
}
```

**Run inside a Docker container:**

```groovy
agent {
  docker {
    image 'maven:3.9-eclipse-temurin-17'
    args '-v $HOME/.m2:/root/.m2'
  }
}
```

**No global agent — each stage defines its own:**

```groovy
agent none

stages {
  stage('Build') {
    agent { label 'linux' }
    steps { sh 'mvn package' }
  }
  stage('Deploy') {
    agent { label 'deploy-server' }
    steps { sh './deploy.sh' }
  }
}
```

---

## environment

Defines environment variables available to all steps.

```groovy
pipeline {
  agent any

  environment {
    APP_NAME = 'my-app'
    REGISTRY  = 'ghcr.io/myorg'
    IMAGE_TAG = "${env.BUILD_NUMBER}"
  }

  stages {
    stage('Build') {
      steps {
        sh "docker build -t ${REGISTRY}/${APP_NAME}:${IMAGE_TAG} ."
      }
    }
  }
}
```

Environment variables from credentials:

```groovy
environment {
  DOCKER_CREDS = credentials('docker-hub-creds')
}
```

This sets `DOCKER_CREDS_USR` and `DOCKER_CREDS_PSW` automatically.

---

## parameters

Parameters let users provide input when triggering a build manually.

```groovy
pipeline {
  agent any

  parameters {
    string(
      name: 'DEPLOY_ENV',
      defaultValue: 'staging',
      description: 'Target environment'
    )
    choice(
      name: 'LOG_LEVEL',
      choices: ['INFO', 'DEBUG', 'WARN'],
      description: 'Log verbosity'
    )
    booleanParam(
      name: 'SKIP_TESTS',
      defaultValue: false,
      description: 'Skip unit tests'
    )
    password(
      name: 'API_KEY',
      defaultValue: '',
      description: 'Deployment API key'
    )
  }

  stages {
    stage('Deploy') {
      steps {
        sh "deploy.sh --env ${params.DEPLOY_ENV} --log ${params.LOG_LEVEL}"
      }
    }
  }
}
```

Access with `params.PARAMETER_NAME`. On first run, the pipeline may not show parameters — trigger a build once and then "Build with Parameters" appears.

---

## options

Pipeline-wide behaviour settings.

```groovy
options {
  timeout(time: 30, unit: 'MINUTES')     // fail if build takes longer
  buildDiscarder(logRotator(numToKeepStr: '10'))  // keep only last 10 builds
  disableConcurrentBuilds()              // only one build at a time
  retry(3)                               // retry failed pipeline 3 times
  timestamps()                           // add timestamps to console log
  skipDefaultCheckout()                  // do not auto-checkout on agent
}
```

---

## triggers

Define automatic build triggers in the Jenkinsfile.

```groovy
triggers {
  cron('H 2 * * 1-5')           // weekdays at ~2 AM
  pollSCM('H/5 * * * *')        // poll Git every 5 minutes
  upstream(
    upstreamProjects: 'other-job',
    threshold: hudson.model.Result.SUCCESS
  )
}
```

---

## post

Runs actions after the pipeline finishes, regardless of success or failure.

```groovy
post {
  always {
    junit 'target/surefire-reports/**/*.xml'
    cleanWs()                // clean workspace
  }
  success {
    echo 'Build succeeded!'
    mail to: 'team@company.com',
         subject: "Build #${BUILD_NUMBER} passed",
         body: "All good."
  }
  failure {
    echo 'Build failed!'
    mail to: 'team@company.com',
         subject: "Build #${BUILD_NUMBER} FAILED",
         body: "Check the logs at ${BUILD_URL}"
  }
  unstable {
    echo 'Tests have failures'
  }
  changed {
    echo 'Build status changed from previous run'
  }
}
```

**Post conditions:**

| Condition | When it runs |
|---|---|
| `always` | Every time, regardless of result |
| `success` | Only when the pipeline passed |
| `failure` | Only when the pipeline failed |
| `unstable` | Tests passed but some failed (yellow) |
| `changed` | Status different from the previous build |
| `aborted` | Pipeline was manually aborted |

---

## Complete Example

```groovy
pipeline {
  agent any

  environment {
    APP = 'my-service'
    VERSION = "1.0.${env.BUILD_NUMBER}"
  }

  options {
    timeout(time: 20, unit: 'MINUTES')
    buildDiscarder(logRotator(numToKeepStr: '10'))
  }

  parameters {
    choice(name: 'ENV', choices: ['staging', 'production'], description: 'Deploy target')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        sh './mvnw clean package -DskipTests'
      }
    }

    stage('Test') {
      steps {
        sh './mvnw test'
      }
    }

    stage('Deploy') {
      when {
        expression { params.ENV == 'staging' }
      }
      steps {
        sh "./deploy.sh --env ${params.ENV} --version ${VERSION}"
      }
    }
  }

  post {
    always {
      junit 'target/surefire-reports/**/*.xml'
      archiveArtifacts artifacts: 'target/*.jar'
      cleanWs()
    }
    failure {
      echo "Pipeline failed for build ${VERSION}"
    }
  }
}
```
