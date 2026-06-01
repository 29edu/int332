# Parent POM

A Parent POM is a `pom.xml` that other POMs inherit from. It centralises shared configuration — versions, plugins, properties, dependency management — so child modules do not repeat it.

---

## Why Use a Parent POM

In a multi-module project, without a parent POM every module must declare the same Spring version, the same compiler settings, and the same plugin configuration independently. When the Spring version changes, you must update every module.

With a parent POM:
- shared configuration lives in one place
- version updates happen in one file
- new modules inherit everything automatically

---

## Inheriting from Spring Boot's Parent

The most common use of parent POM in Spring Boot projects is inheriting from `spring-boot-starter-parent`. This gives your project sensible defaults for Java version, encoding, dependency versions, and plugin configuration.

```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>3.1.0</version>
  <relativePath/>
</parent>

<groupId>com.example</groupId>
<artifactId>my-app</artifactId>
<version>1.0.0</version>
```

With this parent, you no longer need to specify versions for Spring dependencies — the parent manages them:

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <!-- no <version> needed, inherited from parent -->
  </dependency>
</dependencies>
```

---

## Creating Your Own Parent POM

For a company or team sharing common config across multiple projects, you create a custom parent POM.

**Parent `pom.xml` (`packaging` must be `pom`):**

```xml
<project>
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.example</groupId>
  <artifactId>company-parent</artifactId>
  <version>1.0.0</version>
  <packaging>pom</packaging>

  <properties>
    <java.version>17</java.version>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <spring.version>3.1.0</spring.version>
    <jackson.version>2.15.0</jackson.version>
  </properties>

  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>${spring.version}</version>
      </dependency>
      <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>${jackson.version}</version>
      </dependency>
    </dependencies>
  </dependencyManagement>

  <build>
    <pluginManagement>
      <plugins>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.11.0</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>

</project>
```

**Child `pom.xml`:**

```xml
<project>
  <modelVersion>4.0.0</modelVersion>

  <parent>
    <groupId>com.example</groupId>
    <artifactId>company-parent</artifactId>
    <version>1.0.0</version>
  </parent>

  <artifactId>user-service</artifactId>
  <version>2.0.0</version>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <!-- version comes from parent's dependencyManagement -->
    </dependency>
  </dependencies>

</project>
```

---

## Multi-Module Project Structure

A parent POM can also aggregate multiple child modules into one build.

```
company-project/
├── pom.xml                ← parent POM (lists modules)
├── user-service/
│   └── pom.xml            ← child module
├── order-service/
│   └── pom.xml            ← child module
└── common-lib/
    └── pom.xml            ← child module (shared library)
```

**Parent `pom.xml` with modules:**

```xml
<packaging>pom</packaging>

<modules>
  <module>common-lib</module>
  <module>user-service</module>
  <module>order-service</module>
</modules>
```

Running `mvn install` from the parent directory builds all modules in the correct order.

---

## dependencyManagement vs dependencies

| | `dependencyManagement` | `dependencies` |
|---|---|---|
| Where | Parent POM | Any POM |
| Effect | Declares versions for child POMs to inherit | Actually adds the dependency to the classpath |
| Child behaviour | Child must still declare the dependency (without version) | Child inherits the actual dependency |

Use `dependencyManagement` to control versions centrally without forcing every child to use every dependency.
