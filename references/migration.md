# Migrating from GitHub Actions to CNB

## Concept Mapping

| GitHub Actions | CNB |
|---------------|-----|
| `.github/workflows/*.yml` | `.cnb.yml` (single file) |
| `on:` | Event keys (`push`, `pull_request`, etc.) |
| `jobs:` | `stages:` (serial) or stage `jobs:` (parallel) |
| `steps:` | `jobs:` within a stage |
| `uses: action/name` | `image: plugin-image` |
| `with:` | `settings:` |
| `run:` | `script:` |
| `env:` | `env:` (same concept) |
| `secrets.*` | `imports:` from secret repo files |
| `if:` | `if:` (shell script, exit 0 = true) |
| `needs:` | Sequential stages (implicit ordering) |
| `matrix:` | Parallel jobs within a stage |
| `services:` | `services:` (docker, etc.) |
| `artifacts` | `runner.volumes` or `docker:cache` |
| `cron schedule` | `"crontab: <expr>"` event key |

## Migration Examples

### Basic Build

**GitHub Actions:**
```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm test
```

**CNB:**
```yaml
main:
  push:
    - stages:
        - name: build and test
          image: node:20
          script:
            - npm ci
            - npm test
  pull_request:
    - stages:
        - name: build and test
          image: node:20
          script:
            - npm ci
            - npm test
```

Note: CNB auto-clones the repo; no checkout step needed.

### Docker Build & Push

**GitHub Actions:**
```yaml
jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USER }}
          password: ${{ secrets.DOCKER_PASS }}
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: myimage:latest
```

**CNB:**
```yaml
main:
  push:
    - services:
        - docker
      imports: https://cnb.cool/<secret-repo>/-/blob/main/docker-creds.yml
      stages:
        - name: build and push
          script: |
            docker login -u $DOCKER_USER -p "$DOCKER_PASS"
            docker build -t myimage:latest .
            docker push myimage:latest
```

### Scheduled Tasks

**GitHub Actions:**
```yaml
on:
  schedule:
    - cron: '0 2 * * *'
```

**CNB:**
```yaml
main:
  "crontab: 0 2 * * *":
    - stages:
        - name: nightly
          script: echo "nightly build"
```

### Matrix Strategy (Parallel Jobs)

**GitHub Actions:**
```yaml
jobs:
  test:
    strategy:
      matrix:
        node: [16, 18, 20]
    steps:
      - run: npm test
```

**CNB:**
```yaml
main:
  push:
    - stages:
        - name: test matrix
          jobs:
            node-16:
              image: node:16
              script: npm test
            node-18:
              image: node:18
              script: npm test
            node-20:
              image: node:20
              script: npm test
```

## Key Differences

1. **Single config file**: CNB uses one `.cnb.yml` vs multiple workflow files
2. **Auto checkout**: No need for `actions/checkout`
3. **Image-based**: Use Docker images directly instead of `setup-*` actions
4. **Branch-first**: Config is organized by branch pattern, then event
5. **Secrets**: Use `imports` from secret repos with access control
6. **Parallel/Serial**: Stages are serial; jobs within a stage can be parallel (object form)
7. **Caching**: Use `runner.volumes` or `docker:cache` instead of `actions/cache`
