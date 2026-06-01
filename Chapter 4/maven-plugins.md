# Maven Plugins and Execution

Maven itself does very little on its own. Almost all of the work in a Maven build — compiling, testing, packaging — is done by plugins. A plugin is a collection of goals, and each goal does one specific task.

---

## How Plugins Work

A plugin goal is bound to a lifecycle phase. When Maven runs a phase, it runs all goals bound to that phase.

```
compile phase  →  maven-compiler-plugin:compile
test phase     →  maven-surefire-plugin:test
package phase  →  maven-jar-plugin:jar
```

You can also run a plugin goal directly without going through a lifecycle:

```bash
mvn compiler:compile
mvn surefire:test
mvn dependency:tree
```

---

## Compiler Plugin

The Maven Compiler Plugin compiles Java source code. It is bound to the `compile` and `test-compile` phases automatically.

**Default configuration:**

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.11.0</version>
      <configuration>
        <source>17</source>
        <target>17</target>
        <encoding>UTF-8</encoding>
      </configuration>
    </plugin>
  </plugins>
</build>
```

**Shorthand via properties (preferred with Spring Boot parent):**

```xml
<properties>
  <maven.compiler.source>17</maven.compiler.source>
  <maven.compiler.target>17</maven.compiler.target>
</properties>
```

**Enabling annotation processing (for Lombok, MapStruct, etc.):**

```xml
<configuration>
  <annotationProcessorPaths>
    <path>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.18.28</version>
    </path>
  </annotationProcessorPaths>
</configuration>
```

---

## Surefire Plugin (Unit Testing)

The Maven Surefire Plugin runs unit tests during the `test` phase. It finds test classes automatically and generates reports.

**Default behaviour:**
Surefire runs any class whose name matches:
- `**/*Test.java`
- `**/Test*.java`
- `**/*Tests.java`
- `**/*TestCase.java`

**Basic configuration:**

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <version>3.1.2</version>
</plugin>
```

**Running with JUnit 5:**

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <version>3.1.2</version>
  <configuration>
    <includes>
      <include>**/*Test.java</include>
    </includes>
  </configuration>
</plugin>

<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter</artifactId>
  <version>5.10.0</version>
  <scope>test</scope>
</dependency>
```

**Skipping tests:**

```bash
mvn package -DskipTests
```

**Running a single test class:**

```bash
mvn test -Dtest=UserServiceTest
```

**Running a single test method:**

```bash
mvn test -Dtest=UserServiceTest#shouldReturnUser
```

**Test reports** are written to `target/surefire-reports/` as XML and plain text files.

---

## Shade Plugin (Uber JAR)

The Maven Shade Plugin packages the application and all its dependencies into a single executable JAR file — called an uber JAR or fat JAR. This makes the JAR self-contained and runnable without a separate classpath.

```bash
java -jar target/my-app-1.0.0.jar
```

**Configuration:**

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-shade-plugin</artifactId>
  <version>3.5.0</version>
  <executions>
    <execution>
      <phase>package</phase>
      <goals>
        <goal>shade</goal>
      </goals>
      <configuration>
        <transformers>
          <transformer implementation=
            "org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
            <mainClass>com.example.App</mainClass>
          </transformer>
        </transformers>
      </configuration>
    </execution>
  </executions>
</plugin>
```

Running `mvn package` now produces two JARs:
- `my-app-1.0.0.jar` — the uber JAR with all dependencies inside
- `original-my-app-1.0.0.jar` — the original thin JAR

**When to use Shade vs Spring Boot Plugin:**
- Use the Shade plugin for plain Java applications without a framework
- Use `spring-boot-maven-plugin` for Spring Boot applications — it produces an executable JAR in a similar way but with Spring Boot's launcher

---

## Maven Wrapper (mvnw)

The Maven Wrapper bundles a specific Maven version with your project. Anyone who clones the repository can build it immediately without installing Maven globally.

**Generate the wrapper:**

```bash
mvn wrapper:wrapper
```

Or specify a Maven version:

```bash
mvn wrapper:wrapper -Dmaven=3.9.4
```

This creates:

```
project/
├── mvnw          ← shell script for Linux/macOS
├── mvnw.cmd      ← batch script for Windows
└── .mvn/
    └── wrapper/
        └── maven-wrapper.properties  ← stores Maven version to download
```

**Using the wrapper:**

```bash
# Linux/macOS
./mvnw clean package

# Windows
mvnw.cmd clean package
```

The wrapper downloads the declared Maven version on first use and caches it locally. Subsequent runs use the cached version.

**Why commit the wrapper:**
- CI/CD pipelines can build without pre-installing Maven
- every developer uses exactly the same Maven version
- no "works on my machine" issues caused by different Maven versions

**maven-wrapper.properties:**

```properties
distributionUrl=https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/3.9.4/apache-maven-3.9.4-bin.zip
wrapperUrl=https://repo.maven.apache.org/maven2/org/apache/maven/wrapper/maven-wrapper/3.2.0/maven-wrapper-3.2.0.jar
```
