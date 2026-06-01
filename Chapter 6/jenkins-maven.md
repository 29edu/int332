# Jenkins and Maven

Jenkins has built-in support for Maven through the Maven Integration plugin. Maven can be installed globally in Jenkins and referenced by name in any pipeline or Freestyle job.

---

## Installing Maven in Jenkins

**Step 1 — Install the Maven Integration plugin:**
Manage Jenkins → Plugins → Available → search "Maven Integration" → Install

**Step 2 — Add Maven in Global Tool Configuration:**
Manage Jenkins → Tools → Maven installations → Add Maven

- Name: `maven-3.9` (any name you choose — you reference this in jobs)
- Install automatically: checked
- Version: select `3.9.4` (or the version you need)

Jenkins downloads Maven on demand and installs it on any agent that needs it.

You can add multiple Maven installations with different versions and reference them by name in pipelines.

---

## Global Tool Configuration

Global Tool Configuration (Manage Jenkins → Tools) lets you define installations of:

- JDK versions
- Maven versions
- Gradle versions
- Node.js versions
- Git

Each tool has a name and either a local path (already installed on the agent) or an automatic installer.

**Adding a JDK:**
- Name: `jdk-17`
- Install automatically → from Adoptium → version 17

**Adding Maven:**
- Name: `maven-3.9`
- Install automatically → from Apache → version 3.9.4

---

## Using Maven in Declarative Pipelines

**Reference the globally configured Maven installation:**

```groovy
pipeline {
  agent any

  tools {
    maven 'maven-3.9'      // name from Global Tool Configuration
    jdk   'jdk-17'
  }

  stages {
    stage('Build') {
      steps {
        sh 'mvn clean package -DskipTests'
      }
    }

    stage('Test') {
      steps {
        sh 'mvn test'
      }
    }
  }
}
```

The `tools {}` block ensures Maven and the JDK are available on the agent before any step runs.

**Using the Maven wrapper (no tool config needed):**

```groovy
pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        sh './mvnw clean package'
      }
    }
  }
}
```

The Maven wrapper (`mvnw`) is committed with the project and downloads the correct Maven version automatically. This is the simplest approach and requires no Jenkins tool configuration.

---

## Running Maven Inside a Docker Container

No Maven installation needed on the agent at all:

```groovy
pipeline {
  agent {
    docker {
      image 'maven:3.9-eclipse-temurin-17'
      args '-v $HOME/.m2:/root/.m2'   // cache ~/.m2 across builds
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

---

## Code Coverage with JaCoCo

JaCoCo measures test code coverage. The JaCoCo plugin in Jenkins reads the coverage report and shows trends in the build.

**pom.xml — add JaCoCo plugin:**

```xml
<plugin>
  <groupId>org.jacoco</groupId>
  <artifactId>jacoco-maven-plugin</artifactId>
  <version>0.8.10</version>
  <executions>
    <execution>
      <goals>
        <goal>prepare-agent</goal>
      </goals>
    </execution>
    <execution>
      <id>report</id>
      <phase>test</phase>
      <goals>
        <goal>report</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

**Jenkinsfile — publish coverage report:**

```groovy
pipeline {
  agent any

  tools {
    maven 'maven-3.9'
  }

  stages {
    stage('Test') {
      steps {
        sh 'mvn clean verify'
      }
    }
  }

  post {
    always {
      junit 'target/surefire-reports/**/*.xml'

      jacoco(
        execPattern: 'target/jacoco.exec',
        classPattern: 'target/classes',
        sourcePattern: 'src/main/java',
        minimumLineCoverage: '70',       // fail if below 70%
        minimumBranchCoverage: '60'
      )
    }
  }
}
```

---

## Publishing Test Reports

Jenkins reads JUnit XML test results and displays:
- total tests run
- tests passed / failed / skipped
- test trends across builds
- which tests failed and their stack traces

```groovy
post {
  always {
    junit 'target/surefire-reports/**/*.xml'
  }
}
```

The JUnit report is attached to every build. Jenkins shows a trend graph on the job's main page.

**Surefire report HTML (full HTML test report):**

```groovy
publishHTML([
  allowMissing: false,
  alwaysLinkToLastBuild: true,
  keepAll: true,
  reportDir: 'target/site/surefire-report',
  reportFiles: 'surefire-report.html',
  reportName: 'Surefire Report'
])
```

Requires the HTML Publisher plugin.

---

## Complete Maven Pipeline with Reports

```groovy
pipeline {
  agent any

  tools {
    maven 'maven-3.9'
    jdk   'jdk-17'
  }

  options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
    timeout(time: 20, unit: 'MINUTES')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        sh 'mvn clean compile'
      }
    }

    stage('Test') {
      steps {
        sh 'mvn test'
      }
    }

    stage('Package') {
      steps {
        sh 'mvn package -DskipTests'
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      }
    }

    stage('Coverage') {
      steps {
        sh 'mvn verify'
      }
    }
  }

  post {
    always {
      junit 'target/surefire-reports/**/*.xml'
      jacoco execPattern: 'target/jacoco.exec'
      cleanWs()
    }
  }
}
```
