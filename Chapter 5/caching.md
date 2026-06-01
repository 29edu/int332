# Caching for Faster Builds

Every time a GitHub Actions job runs, it starts on a fresh virtual machine with nothing installed. Without caching, every run downloads all dependencies from scratch — which can take several minutes for large projects.

Caching saves dependency directories between runs so they are restored instead of re-downloaded.

---

## How Caching Works

1. Before the job runs, GitHub checks if a cache with the given key exists.
2. If a match is found (cache hit), the cached directory is restored.
3. The job runs normally — dependencies are already present.
4. After the job, if no exact match was found earlier, the directory is saved as a new cache entry.

```
First run:
  cache miss → download all dependencies → save cache

Second run:
  cache hit → restore from cache → skip download → faster build
```

---

## actions/cache

The `actions/cache` action is the standard way to cache arbitrary directories.

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.m2/repository        # directory to cache
    key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
    restore-keys: |
      ${{ runner.os }}-maven-
```

**path** — the directory to cache

**key** — a unique string identifying this cache. If the key matches an existing cache, it is restored. A good key includes:
- the OS (`${{ runner.os }}`)
- the dependency file hash (`${{ hashFiles(...) }}`)

The `hashFiles()` function creates a hash of the specified files. When `pom.xml` changes (new dependency added), the hash changes, creating a new cache.

**restore-keys** — fallback keys used when the exact key is not found. The most recent cache whose key starts with the prefix is used.

```
Key: linux-maven-abc123     → exact match? use it
No match →
Restore key: linux-maven-   → use most recent cache with this prefix
```

---

## Maven Caching

```yaml
steps:
  - uses: actions/checkout@v4

  - uses: actions/setup-java@v3
    with:
      distribution: 'temurin'
      java-version: '17'
      cache: maven             # built-in Maven cache support in setup-java

  - run: ./mvnw clean package
```

`actions/setup-java` has built-in caching support. Setting `cache: maven` automatically caches `~/.m2/repository` using `pom.xml` as the cache key. This is the simplest approach.

**Manual Maven cache (for more control):**

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.m2/repository
    key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
    restore-keys: |
      ${{ runner.os }}-maven-
```

---

## Node.js / npm Caching

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'              # built-in npm cache support
```

**Manual npm cache:**

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-npm-
```

---

## Python / pip Caching

```yaml
- uses: actions/setup-python@v5
  with:
    python-version: '3.11'
    cache: 'pip'

- run: pip install -r requirements.txt
```

**Manual pip cache:**

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
    restore-keys: |
      ${{ runner.os }}-pip-
```

---

## Docker Layer Caching

Docker builds can also be cached between runs using GitHub Actions cache as a backend.

```yaml
- uses: docker/setup-buildx-action@v3

- uses: actions/cache@v4
  with:
    path: /tmp/.buildx-cache
    key: ${{ runner.os }}-buildx-${{ github.sha }}
    restore-keys: |
      ${{ runner.os }}-buildx-

- uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    tags: myapp:latest
    cache-from: type=local,src=/tmp/.buildx-cache
    cache-to: type=local,dest=/tmp/.buildx-cache-new,mode=max
```

---

## Cache Limits and Behaviour

- Cache entries expire after 7 days of not being used
- Total cache storage per repository is 10 GB
- When the limit is reached, oldest caches are evicted
- Caches are scoped to the branch — a PR can access the cache from its base branch but not from other PRs

---

## Checking Cache Hit

You can check whether a cache was restored and take different actions:

```yaml
- id: cache-deps
  uses: actions/cache@v4
  with:
    path: ~/.m2/repository
    key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}

- name: Install if cache miss
  if: steps.cache-deps.outputs.cache-hit != 'true'
  run: echo "Cache miss — downloading dependencies"
```
