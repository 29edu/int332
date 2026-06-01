# Why Build Tools Exist

A build tool automates the steps needed to turn source code into a working, deployable application. Without a build tool, developers must perform these steps manually every time.

---

## What Happens Without a Build Tool

For even a small Java project, building manually means:

```bash
# Compile every .java file
javac -cp lib/spring.jar:lib/jackson.jar src/main/java/com/example/*.java -d out/

# Run tests manually
java -cp out/:lib/junit.jar org.junit.runner.JUnitCore com.example.AppTest

# Create a JAR by hand
jar cf app.jar -C out/ .

# Copy dependencies manually
cp lib/*.jar dist/
```

This is tedious, error-prone, and impossible to reproduce reliably across different machines or team members.

---

## Problems Solved by Automated Builds

**Reproducibility**
The build produces the same output on every machine — developer laptop, CI server, production — regardless of the environment.

**Dependency management**
The build tool downloads the exact versions of libraries your project needs from a central repository. No more manually downloading JAR files and dropping them in a folder.

```
Before: download spring-5.3.jar manually, add to lib/ folder, repeat for every dependency
After: declare <dependency> in pom.xml, Maven downloads and manages it
```

**Consistent build steps**
Every developer on the team runs the same sequence of steps in the same order. No more "it works on my machine."

**Automation of repetitive tasks**
Compile, test, package, deploy — all triggered with one command. The build tool runs them in the correct order.

**Integration with CI/CD**
Build tools integrate directly with CI/CD pipelines. A push to a repository automatically triggers a build, runs tests, and produces an artifact.

**Version management**
The project version is declared in one place. Changing it updates all generated artifacts consistently.

---

## Common Java Build Tools

| Tool | Notes |
|---|---|
| Maven | Convention over configuration, XML-based, widely adopted in enterprise Java |
| Gradle | Flexible, Groovy/Kotlin DSL, used heavily in Android and modern projects |
| Ant | Older, fully manual task definitions, largely replaced by Maven and Gradle |

---

## What Maven Does

Maven is a build automation and project management tool for Java. Given a `pom.xml` file describing your project, Maven can:

- download all declared dependencies automatically
- compile source code
- run unit and integration tests
- package the application into a JAR or WAR file
- install the artifact to a local repository
- deploy the artifact to a remote repository or server

All of this with one command:

```bash
mvn package
```

Maven works by following conventions. A standard project structure means Maven already knows where to find source files, test files, and resources without being told explicitly.
