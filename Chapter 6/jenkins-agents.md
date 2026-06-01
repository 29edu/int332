# Jenkins Agents

Agents (also called nodes or workers) are the machines that run Jenkins build jobs. The master delegates work to agents and collects results. Agents can be permanent machines, containers, or cloud instances.

---

## Adding an Agent to Jenkins

Manage Jenkins → Nodes → New Node → Permanent Agent

**Key fields:**

- **Name** — display name for the agent
- **Number of executors** — how many concurrent builds this agent can run
- **Remote root directory** — path on the agent where Jenkins stores workspace files (e.g. `/home/jenkins/workspace`)
- **Labels** — tags used by pipelines to target this agent (e.g. `linux`, `java17`, `docker`)
- **Launch method** — how Jenkins connects to this agent

---

## SSH Agents

The most common agent type for Linux/macOS servers. Jenkins connects to the agent over SSH and runs build commands.

**Requirements on the agent machine:**
- SSH server running
- Java installed
- A user account for Jenkins

**Setup:**

1. Generate an SSH key pair on the Jenkins master:

```bash
ssh-keygen -t ed25519 -C "jenkins-agent"
```

2. Add the public key to `~/.ssh/authorized_keys` on the agent machine.

3. Add the private key to Jenkins credentials:
   - Manage Jenkins → Credentials → Add → SSH Username with private key
   - ID: `agent-ssh-key`
   - Username: `jenkins`
   - Private Key: paste the private key content

4. Add the node:
   - Manage Jenkins → Nodes → New Node
   - Launch method: **Launch agents via SSH**
   - Host: agent's IP address
   - Credentials: select `agent-ssh-key`

**Use in pipeline:**

```groovy
pipeline {
  agent { label 'linux-agent' }

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

## JNLP/TCP Agents (Inbound Agents)

The agent initiates the connection to the master. Used when the master cannot SSH into the agent (firewall, NAT).

**Setup:**

1. Create the node in Jenkins (same as above but launch method: **Launch agent by connecting it to the controller**)
2. Jenkins generates a command to run on the agent machine:

```bash
java -jar agent.jar \
  -url http://jenkins-master:8080 \
  -secret <generated-secret> \
  -name my-agent \
  -workDir /home/jenkins
```

3. Run this command on the agent machine. The agent connects back to the master.

**Run as a service (Linux):**

```bash
# Create systemd service
sudo nano /etc/systemd/system/jenkins-agent.service
```

```ini
[Unit]
Description=Jenkins Agent

[Service]
User=jenkins
ExecStart=/usr/bin/java -jar /home/jenkins/agent.jar \
  -url http://master:8080 \
  -secret <secret> \
  -name my-agent \
  -workDir /home/jenkins
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable jenkins-agent
sudo systemctl start jenkins-agent
```

---

## Container-Based Agents (Docker)

Jenkins can spin up a Docker container as a temporary agent for a single build and destroy it when done. This is a clean, reproducible, and scalable approach.

**Requires:** Docker Pipeline plugin + Docker running on the Jenkins master or on a Docker host.

**Define agent as a Docker container per pipeline:**

```groovy
pipeline {
  agent {
    docker {
      image 'maven:3.9-eclipse-temurin-17'
      label 'docker-host'
      args  '-v $HOME/.m2:/root/.m2'
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

Jenkins pulls the image, starts a container, runs all pipeline steps inside it, and removes the container when the pipeline ends. No tools need to be installed on the host agent.

**Kubernetes agents (dynamic cloud agents):**

With the Kubernetes plugin, Jenkins creates a pod in a Kubernetes cluster for each build and destroys it when done.

```groovy
pipeline {
  agent {
    kubernetes {
      yaml """
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: maven
            image: maven:3.9-eclipse-temurin-17
            command: ['sleep', '9999']
          - name: docker
            image: docker:24-dind
            securityContext:
              privileged: true
      """
    }
  }

  stages {
    stage('Build') {
      steps {
        container('maven') {
          sh 'mvn clean package'
        }
      }
    }
    stage('Docker Build') {
      steps {
        container('docker') {
          sh 'docker build -t myapp .'
        }
      }
    }
  }
}
```

---

## Pipeline Shared Libraries

A Shared Library lets you extract common pipeline code into a separate repository and reuse it across many Jenkinsfiles.

**Directory structure of the shared library repository:**

```
jenkins-shared-library/
├── vars/
│   ├── buildMaven.groovy       ← global variable/function
│   └── deployApp.groovy
└── src/
    └── org/example/
        └── Utils.groovy        ← Groovy class
```

**vars/buildMaven.groovy:**

```groovy
def call(String javaVersion = '17') {
  pipeline {
    agent {
      docker { image "maven:3.9-eclipse-temurin-${javaVersion}" }
    }
    stages {
      stage('Build') {
        steps { sh 'mvn clean package' }
      }
      stage('Test') {
        steps { sh 'mvn test' }
      }
    }
    post {
      always { junit 'target/surefire-reports/**/*.xml' }
    }
  }
}
```

**Register the library in Jenkins:**
Manage Jenkins → System → Global Pipeline Libraries → Add Library
- Name: `my-shared-lib`
- Default version: `main`
- Retrieval method: Modern SCM → Git → URL of the library repo

**Use in a Jenkinsfile:**

```groovy
@Library('my-shared-lib') _

buildMaven('17')
```

The entire pipeline is now one line. Updating the shared library updates all pipelines that use it.

---

## SFTP / File Transfer to Servers

For deploying files to a server using SFTP (requires SSH Agent or Publish Over SSH plugin):

```groovy
stage('Deploy') {
  steps {
    sshagent(['server-ssh-key']) {
      sh """
        scp -o StrictHostKeyChecking=no \
          target/myapp.jar \
          user@server:/opt/myapp/myapp.jar

        ssh user@server 'systemctl restart myapp'
      """
    }
  }
}
```

`sshagent` loads the SSH key from Jenkins credentials and makes it available to `ssh` and `scp` commands.
