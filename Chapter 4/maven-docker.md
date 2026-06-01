# Maven and Docker Integration

Maven can build Docker images and push them to registries as part of the standard Maven build lifecycle. This removes the need to write separate shell scripts or CI steps for Docker operations.

---

## Why Integrate Maven with Docker

Without integration, building and deploying a Spring Boot application requires separate steps:

```bash
mvn clean package          # step 1: build the JAR
docker build -t myapp .    # step 2: build the image (separate Dockerfile)
docker push myapp          # step 3: push to registry
```

With Maven-Docker integration, all of this can happen in one command:

```bash
mvn clean package dockerfile:build dockerfile:push
```

Or as part of the standard lifecycle:

```bash
mvn deploy
```

---

## dockerfile-maven-plugin (Spotify)

The `dockerfile-maven-plugin` by Spotify is a widely used plugin that integrates Docker image building directly into the Maven lifecycle.

**pom.xml configuration:**

```xml
<build>
  <plugins>
    <plugin>
      <groupId>com.spotify</groupId>
      <artifactId>dockerfile-maven-plugin</artifactId>
      <version>1.4.13</version>
      <executions>
        <execution>
          <id>build-image</id>
          <phase>package</phase>
          <goals>
            <goal>build</goal>
          </goals>
        </execution>
        <execution>
          <id>push-image</id>
          <phase>deploy</phase>
          <goals>
            <goal>push</goal>
          </goals>
        </execution>
      </executions>
      <configuration>
        <repository>myregistry.io/myapp</repository>
        <tag>${project.version}</tag>
        <buildArgs>
          <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
        </buildArgs>
      </configuration>
    </plugin>
  </plugins>
</build>
```

**Dockerfile in project root:**

```dockerfile
FROM eclipse-temurin:17-jre-alpine
ARG JAR_FILE
COPY ${JAR_FILE} app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

The `JAR_FILE` build argument is passed from Maven to Docker, pointing to the built JAR.

**Build the image during package:**

```bash
mvn clean package
# compiles, tests, packages JAR, then builds Docker image
```

**Push to registry during deploy:**

```bash
mvn deploy
# runs everything above and also pushes the image
```

---

## jib-maven-plugin (Google)

Jib is a plugin from Google that builds Docker images without requiring Docker to be installed or a Dockerfile to be written. It builds directly from the Maven project.

**pom.xml configuration:**

```xml
<plugin>
  <groupId>com.google.cloud.tools</groupId>
  <artifactId>jib-maven-plugin</artifactId>
  <version>3.3.2</version>
  <configuration>
    <from>
      <image>eclipse-temurin:17-jre-alpine</image>
    </from>
    <to>
      <image>myregistry.io/myapp</image>
      <tags>
        <tag>${project.version}</tag>
        <tag>latest</tag>
      </tags>
    </to>
    <container>
      <mainClass>com.example.App</mainClass>
      <ports>
        <port>8080</port>
      </ports>
    </container>
  </configuration>
</plugin>
```

**Build and push directly to registry (no Docker daemon needed):**

```bash
mvn jib:build
```

**Build to local Docker daemon:**

```bash
mvn jib:dockerBuild
```

**Why Jib:**
- no Dockerfile needed
- no Docker installed on the build machine
- faster builds — Jib layers dependencies and application code separately, so unchanged layers are not rebuilt or re-pushed
- deterministic image output — same inputs always produce the same image

---

## Dockerizing a Maven-Based Spring Boot Application

### Step 1 — pom.xml with Spring Boot Plugin

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
  </plugins>
</build>
```

### Step 2 — Multi-Stage Dockerfile

```dockerfile
# Stage 1: Build
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline -B
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Run
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Why multi-stage:**
- the build stage uses a heavy Maven + JDK image
- the run stage uses a lightweight JRE-only image
- Maven, source code, and intermediate files are NOT included in the final image
- final image is much smaller and safer

### Step 3 — Build the Image

```bash
docker build -t myapp:1.0.0 .
```

### Step 4 — Run Locally

```bash
docker run -p 8080:8080 myapp:1.0.0
```

---

## Pushing Artifacts to Registries

### Pushing Docker Image to Docker Hub

```bash
docker login
docker tag myapp:1.0.0 username/myapp:1.0.0
docker push username/myapp:1.0.0
```

### Pushing Docker Image to a Private Registry

```bash
docker login myregistry.io
docker tag myapp:1.0.0 myregistry.io/myapp:1.0.0
docker push myregistry.io/myapp:1.0.0
```

### Pushing Maven Artifacts to Nexus or Artifactory

Configure the remote repository in `pom.xml`:

```xml
<distributionManagement>
  <repository>
    <id>nexus-releases</id>
    <url>http://nexus.company.com/repository/maven-releases/</url>
  </repository>
  <snapshotRepository>
    <id>nexus-snapshots</id>
    <url>http://nexus.company.com/repository/maven-snapshots/</url>
  </snapshotRepository>
</distributionManagement>
```

Add credentials in `~/.m2/settings.xml` (never in pom.xml):

```xml
<servers>
  <server>
    <id>nexus-releases</id>
    <username>deploy-user</username>
    <password>deploypass</password>
  </server>
</servers>
```

Deploy:

```bash
mvn deploy
# uploads JAR, POM, and sources to Nexus
```

---

## Full CI/CD Example with Maven + Docker

A typical pipeline using GitHub Actions:

```yaml
name: Build and Push

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Build with Maven
        run: ./mvnw clean package -DskipTests

      - name: Build Docker image
        run: docker build -t myregistry.io/myapp:${{ github.sha }} .

      - name: Login to registry
        run: echo "${{ secrets.REGISTRY_PASSWORD }}" | docker login myregistry.io -u ${{ secrets.REGISTRY_USER }} --password-stdin

      - name: Push Docker image
        run: docker push myregistry.io/myapp:${{ github.sha }}
```

The pipeline uses `mvnw` (Maven Wrapper) so no Maven installation is needed on the CI runner.
