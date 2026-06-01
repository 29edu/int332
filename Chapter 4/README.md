# Chapter 4: Maven Build Automation

---

## Build Fundamentals

- [Why Build Tools Exist](build-tools.md) — problems with manual builds, what Maven solves, common Java build tools
- [Project Object Model (POM)](pom.md) — pom.xml structure, GAV coordinates, directory layout, properties
- [Build Lifecycle Phases](build-lifecycle.md) — validate, compile, test, package, verify, install, deploy

---

## Project Structure and Dependencies

- [Parent POM](parent-pom.md) — inheriting from Spring Boot parent, custom parent, multi-module projects, dependencyManagement
- [Dependencies](dependencies.md) — dependency scope, transitive dependencies, version conflicts, exclusions, BOM import

---

## Plugins and Tools

- [Maven Plugins and Execution](maven-plugins.md) — compiler plugin, Surefire (unit testing), Shade (uber JAR), Maven Wrapper (mvnw)

---

## Docker Integration

- [Maven and Docker Integration](maven-docker.md) — dockerfile-maven-plugin, Jib plugin, multi-stage Dockerfile, pushing to registries, CI/CD pipeline
