# Build Lifecycle Phases

Maven organises the build process into a set of ordered phases called the build lifecycle. Each phase represents a stage in building and distributing an artifact. Running a phase automatically runs all phases that come before it.

---

## The Default Lifecycle

The default lifecycle is the main one used for building and deploying projects. It has 23 phases in total, but these are the ones you will use most:

```
validate
    ↓
compile
    ↓
test
    ↓
package
    ↓
verify
    ↓
install
    ↓
deploy
```

---

## Phase by Phase

**validate**
Validates that the project is correct and all necessary information is available. Checks that `pom.xml` is well-formed and required fields are present.

```bash
mvn validate
```

**compile**
Compiles the source code in `src/main/java`. Output goes to `target/classes`.

```bash
mvn compile
```

**test**
Compiles test code in `src/test/java` and runs unit tests. Output goes to `target/test-classes`. Build fails if any test fails.

```bash
mvn test
```

**package**
Takes the compiled code and packages it into a distributable format — a JAR, WAR, or ZIP depending on `<packaging>` in `pom.xml`. Output goes to `target/`.

```bash
mvn package
# produces target/my-app-1.0.0.jar
```

**verify**
Runs any checks to verify the package is valid and meets quality criteria. Integration tests typically run here (using the Failsafe plugin). Ensures the package is ready to be installed.

```bash
mvn verify
```

**install**
Installs the packaged artifact into the local Maven repository (`~/.m2/repository`). Other projects on the same machine can then use it as a dependency.

```bash
mvn install
# artifact is now at ~/.m2/repository/com/example/my-app/1.0.0/my-app-1.0.0.jar
```

**deploy**
Copies the final artifact to a remote repository (such as Nexus or Artifactory) so other team members or CI/CD systems can access it.

```bash
mvn deploy
```

---

## Running Phases

Running a phase runs it and all phases before it:

```bash
mvn package
# runs: validate → compile → test → package
```

```bash
mvn install
# runs: validate → compile → test → package → verify → install
```

---

## Skipping Tests

You can skip test execution during a build:

```bash
mvn package -DskipTests
```

Or skip compiling tests entirely (faster but less safe):

```bash
mvn package -Dmaven.test.skip=true
```

---

## The Clean Lifecycle

Separate from the default lifecycle, `clean` deletes the `target/` directory.

```bash
mvn clean
```

Always run `clean` before a fresh build to remove stale compiled files:

```bash
mvn clean package
mvn clean install
```

---

## The Site Lifecycle

Generates project documentation and reports (Javadoc, test reports, code coverage):

```bash
mvn site
```

---

## Lifecycle Summary

| Phase | What it does | Output location |
|---|---|---|
| validate | Checks POM is valid | — |
| compile | Compiles source code | `target/classes/` |
| test | Runs unit tests | `target/surefire-reports/` |
| package | Packages into JAR/WAR | `target/*.jar` |
| verify | Runs integration tests | `target/failsafe-reports/` |
| install | Installs to local repo | `~/.m2/repository/` |
| deploy | Uploads to remote repo | Remote Nexus/Artifactory |
