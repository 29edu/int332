# Plugins and Security

Jenkins is extensible through plugins. Almost every integration — Git, Docker, Maven, Slack, AWS — is a plugin. Security controls who can access Jenkins and what they can do.

---

## Plugins

Jenkins has over 1,800 plugins available. Plugins add features like new build triggers, integrations with external tools, UI improvements, and build steps.

---

## Managing Plugins

Go to: **Manage Jenkins → Plugins**

**Available tab** — browse and install new plugins
**Installed tab** — view installed plugins, update or uninstall them
**Updates tab** — plugins with available updates

**Installing a plugin from the UI:**
1. Manage Jenkins → Plugins → Available plugins
2. Search for the plugin name
3. Check the box
4. Click "Install without restart" or "Download now and install after restart"

**Installing from the CLI (Jenkins CLI):**

```bash
java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin git docker-workflow pipeline-github
```

**Essential plugins for a typical setup:**

| Plugin | Purpose |
|---|---|
| Git | Clone repositories from Git |
| GitHub | GitHub integration, webhooks |
| Pipeline | Enables Jenkinsfile pipelines |
| Docker Pipeline | Use Docker inside pipelines |
| Maven Integration | Maven build support |
| Credentials | Manage secrets securely |
| Blue Ocean | Modern pipeline visualization UI |
| Mailer | Email notifications |
| JUnit | Publish test results |
| Workspace Cleanup | Delete workspace after build |

---

## Security Overview

By default, Jenkins may allow anonymous users to do everything. Always configure security before exposing Jenkins to a network.

**Enable security:**
Manage Jenkins → Security → Enable Security (check the box)

---

## Authentication

Jenkins supports multiple authentication methods:

**Jenkins' own user database (default)**
Users are stored in Jenkins itself. Good for small teams.

**LDAP**
Authenticate against a company LDAP/Active Directory server. Install the LDAP plugin.

**GitHub OAuth**
Let users log in with their GitHub account. Install the GitHub Authentication plugin.

**Configure:** Manage Jenkins → Security → Security Realm

---

## Authorization — Users and Roles

Authorization controls what authenticated users can do.

**Matrix-based security:**
Assign permissions to individual users or groups.

Manage Jenkins → Security → Authorization → Matrix-based security

Common permissions to configure:

| Permission | What it controls |
|---|---|
| Overall/Administer | Full admin access |
| Overall/Read | View Jenkins at all |
| Job/Build | Trigger a build |
| Job/Read | View a job |
| Job/Configure | Change job settings |
| Job/Create | Create new jobs |
| Job/Delete | Delete jobs |
| View/Read | See dashboard views |

**Role-based strategy (Role Strategy Plugin):**
Instead of setting permissions per user, define roles (admin, developer, viewer) and assign users to roles.

1. Install "Role-based Authorization Strategy" plugin
2. Manage Jenkins → Security → Authorization → Role-Based Strategy
3. Manage Jenkins → Manage and Assign Roles:
   - Create Global Roles (e.g. `admin`, `developer`, `readonly`)
   - Assign permissions to each role
   - Assign users to roles

**Example role setup:**

```
Role: admin
  Permissions: All

Role: developer
  Permissions: Job/Build, Job/Read, Job/Workspace, View/Read

Role: viewer
  Permissions: Job/Read, View/Read
```

---

## Credentials

Sensitive values like passwords, API tokens, and SSH keys are stored in Jenkins Credentials — not hardcoded in job configs or Jenkinsfiles.

**Add a credential:**
Dashboard → Manage Jenkins → Credentials → System → Global credentials → Add Credentials

**Credential types:**

| Type | Use for |
|---|---|
| Username with password | Docker Hub login, Git over HTTPS |
| SSH Username with private key | Git over SSH, server access |
| Secret text | API tokens, Slack webhooks |
| Secret file | Kubeconfig files, certificates |
| Certificate | PKCS#12 keystores |

**Using credentials in a Jenkinsfile:**

```groovy
pipeline {
  agent any
  stages {
    stage('Push') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'docker-hub-creds',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
        }
      }
    }
  }
}
```

The credential values are masked in the console output.

---

## CSRF Protection

Jenkins has built-in CSRF (Cross-Site Request Forgery) protection enabled by default. Do not disable it unless you have a very specific compatibility reason.

Manage Jenkins → Security → CSRF Protection

---

## Agent-to-Master Security

Control what Jenkins agents are allowed to do on the master. This prevents a compromised agent from affecting the master's configuration.

Manage Jenkins → Security → Agent → Controller Security → Enable Agent → Master Access Control
