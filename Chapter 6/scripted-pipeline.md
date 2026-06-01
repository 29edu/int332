# Scripted Pipeline

Scripted Pipeline is the older of the two Jenkins pipeline syntaxes. It is written in plain Groovy and wrapped in a `node {}` block. It gives complete programmatic control over the pipeline at the cost of more complexity.

---

## Basic Structure

```groovy
node {
  stage('Checkout') {
    checkout scm
  }

  stage('Build') {
    sh 'mvn clean package'
  }

  stage('Test') {
    sh 'mvn test'
  }
}
```

All code runs inside `node {}`, which acquires an executor on the master or an agent. Stages group steps but are optional — they are used for visualization in the UI.

---

## Declarative vs Scripted

| | Declarative | Scripted |
|---|---|---|
| Syntax | Structured DSL inside `pipeline {}` | Full Groovy inside `node {}` |
| Validation | Jenkins validates structure before running | Errors only appear at runtime |
| Readability | Easier to read | More flexible, less readable |
| Error handling | `post {}` block | `try/catch/finally` |
| Conditionals | `when {}` block | `if/else` in Groovy |
| Recommended for | Most use cases | Complex logic, dynamic pipelines |

---

## Variables and Expressions

Full Groovy is available, so all standard programming constructs work.

```groovy
node {
  def appName = 'my-app'
  def version = "1.0.${env.BUILD_NUMBER}"
  def deployTarget = env.BRANCH_NAME == 'main' ? 'production' : 'staging'

  stage('Info') {
    echo "Building ${appName} version ${version}"
    echo "Deploy target: ${deployTarget}"
  }
}
```

---

## Conditionals

```groovy
node {
  stage('Build') {
    sh 'mvn clean package'
  }

  stage('Deploy') {
    if (env.BRANCH_NAME == 'main') {
      sh './deploy.sh production'
    } else if (env.BRANCH_NAME == 'develop') {
      sh './deploy.sh staging'
    } else {
      echo "Branch ${env.BRANCH_NAME} — skipping deploy"
    }
  }
}
```

---

## Loops

```groovy
node {
  def services = ['user-service', 'order-service', 'payment-service']

  stage('Build all') {
    for (service in services) {
      sh "cd ${service} && mvn clean package"
    }
  }
}
```

---

## Error Handling with try/catch/finally

```groovy
node {
  try {
    stage('Checkout') {
      checkout scm
    }

    stage('Build') {
      sh 'mvn clean package'
    }

    stage('Test') {
      sh 'mvn test'
    }

    currentBuild.result = 'SUCCESS'

  } catch (Exception e) {
    currentBuild.result = 'FAILURE'
    echo "Pipeline failed: ${e.message}"
    throw e

  } finally {
    stage('Cleanup') {
      cleanWs()
      junit allowEmptyResults: true, testResults: 'target/surefire-reports/**/*.xml'
    }
  }
}
```

`finally` always runs, equivalent to `post { always {} }` in Declarative.

---

## Running on a Specific Agent

```groovy
node('linux-agent') {
  stage('Build') {
    sh 'mvn clean package'
  }
}
```

Or run different stages on different agents:

```groovy
stage('Build') {
  node('linux') {
    sh 'mvn clean package'
  }
}

stage('Deploy') {
  node('deploy-server') {
    sh './deploy.sh'
  }
}
```

---

## Parallel Execution

```groovy
node {
  stage('Test') {
    parallel(
      'Unit Tests': {
        sh 'mvn test'
      },
      'Integration Tests': {
        sh 'mvn verify -Pintegration'
      },
      'Security Scan': {
        sh 'mvn dependency-check:check'
      }
    )
  }
}
```

All three test suites run at the same time on the same node (or separate nodes if each block has its own `node {}`).

---

## Using Credentials

```groovy
node {
  withCredentials([
    usernamePassword(
      credentialsId: 'docker-creds',
      usernameVariable: 'DOCKER_USER',
      passwordVariable: 'DOCKER_PASS'
    )
  ]) {
    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
  }
}
```

---

## When to Use Scripted Pipeline

Use Scripted Pipeline when:
- the pipeline logic is highly dynamic — stage names, agents, or steps change based on runtime conditions
- you need complex data structures (maps, lists, closures) that are awkward in Declarative
- you are extending or migrating an existing Scripted pipeline

For new pipelines, Declarative is almost always the better choice. Declarative syntax supports `script {}` blocks for cases where you need Groovy logic inside a Declarative pipeline.

---

## Mixing Declarative with Groovy (script block)

You can embed Groovy code inside a Declarative pipeline using `script {}`:

```groovy
pipeline {
  agent any
  stages {
    stage('Dynamic logic') {
      steps {
        script {
          def services = ['svc-a', 'svc-b', 'svc-c']
          for (svc in services) {
            sh "deploy.sh ${svc}"
          }
        }
      }
    }
  }
}
```

This keeps the Declarative structure while allowing full Groovy where needed.
