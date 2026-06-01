# Dependencies in Maven

Maven manages project dependencies automatically. You declare what your project needs in `pom.xml`, and Maven downloads the correct versions from a remote repository and places them on the classpath.

---

## Declaring a Dependency

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.1.0</version>
  </dependency>
</dependencies>
```

Maven looks up this artifact in the central repository, downloads it (and its own dependencies), and adds it to your project's classpath.

---

## Dependency Scope

Scope controls when a dependency is available — during compilation, testing, or runtime — and whether it is included in the final packaged artifact.

**compile (default)**
Available at all times: compile, test, runtime. Included in the final JAR/WAR. Most dependencies use this scope.

```xml
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>
  <version>2.15.0</version>
  <!-- scope defaults to compile -->
</dependency>
```

**test**
Available only during test compilation and execution. Not included in the final artifact. Used for test frameworks.

```xml
<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter</artifactId>
  <version>5.10.0</version>
  <scope>test</scope>
</dependency>
```

**provided**
Available at compile time but not included in the final artifact. The runtime environment (servlet container, application server) provides it.

```xml
<dependency>
  <groupId>jakarta.servlet</groupId>
  <artifactId>jakarta.servlet-api</artifactId>
  <version>6.0.0</version>
  <scope>provided</scope>
</dependency>
```

**runtime**
Not needed at compile time but required at runtime. Example: JDBC drivers — you code against the `java.sql` API but need the driver on the classpath when the app runs.

```xml
<dependency>
  <groupId>org.postgresql</groupId>
  <artifactId>postgresql</artifactId>
  <version>42.6.0</version>
  <scope>runtime</scope>
</dependency>
```

**system**
Like `provided`, but you specify the JAR path on your local filesystem. Avoid this — it ties the build to a specific machine.

**import**
Used only in `<dependencyManagement>` with `type=pom`. Imports a bill of materials (BOM) from another POM.

---

## Scope Summary

| Scope | Compile | Test | Runtime | In artifact |
|---|---|---|---|---|
| compile | yes | yes | yes | yes |
| test | no | yes | no | no |
| provided | yes | yes | no | no |
| runtime | no | yes | yes | yes |

---

## Transitive Dependencies

When you add a dependency, Maven also downloads that dependency's own dependencies automatically. These are called transitive dependencies.

```
Your project
  └── spring-boot-starter-web
        ├── spring-webmvc
        │     └── spring-core
        ├── tomcat-embed-core
        └── jackson-databind
              └── jackson-core
```

You only declare `spring-boot-starter-web`. Maven resolves the entire tree.

To see the full dependency tree:

```bash
mvn dependency:tree
```

---

## Version Conflicts

When two different dependencies both require the same library but at different versions, Maven must pick one. This is a version conflict.

```
Your project
  ├── library-A → requires jackson 2.13.0
  └── library-B → requires jackson 2.15.0
```

**Maven's resolution rule — Nearest Wins**
Maven picks the version that is closest to your project in the dependency tree. If both are at the same depth, the first one declared in `pom.xml` wins.

---

## Resolving Version Conflicts

**Option 1 — Declare the version explicitly**
Adding the dependency directly to your `pom.xml` puts it at depth 1, making it win over any transitive version.

```xml
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>
  <version>2.15.0</version>
</dependency>
```

**Option 2 — Use dependencyManagement**
`dependencyManagement` lets you declare the version once and have it applied across all transitive pulls without physically adding the dependency to the classpath.

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.15.0</version>
    </dependency>
  </dependencies>
</dependencyManagement>
```

Any module declaring `jackson-databind` without a version will get `2.15.0`.

**Option 3 — Exclude a transitive dependency**
If a library pulls in a version you do not want, exclude it:

```xml
<dependency>
  <groupId>com.example</groupId>
  <artifactId>library-A</artifactId>
  <version>1.0.0</version>
  <exclusions>
    <exclusion>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

---

## Importing a BOM

A Bill of Materials (BOM) is a special POM that declares a curated set of compatible dependency versions. Spring Boot provides one.

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-dependencies</artifactId>
      <version>3.1.0</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

After this, all Spring-managed dependencies can be declared without a version and Maven uses the compatible versions from the BOM.
