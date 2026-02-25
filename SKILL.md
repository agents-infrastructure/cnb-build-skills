---
name: cnb-build
description: Create and manage CI/CD pipelines on CNB (Cloud Native Build, cnb.cool) platform. Use when writing .cnb.yml configuration files, setting up build pipelines, configuring triggers, using built-in tasks, creating plugins, managing environments, deploying applications, or migrating from GitHub Actions to CNB.
---

# CNB Cloud Native Build

CNB (cnb.cool) is a cloud-native CI/CD platform. Pipelines are configured via `.cnb.yml` in the repository root.

## Quick Start

Minimal `.cnb.yml` example:

```yaml
main:
  push:
    - stages:
        - name: hello
          script: echo "hello world"
```

This triggers on push to `main` branch, running a single script job.

## Configuration Structure

```yaml
<branch-pattern>:          # Branch matching (glob supported for non-crontab)
  <event>:                 # Trigger event type
    [pipeline-name]:       # Optional pipeline name (object form = named)
      runner:              # Runner configuration
        docker:
          image: <image>   # Docker image for build environment
        volumes:           # Cache volumes
          - <path>
        tags:              # Node selector tags
          - <tag>
      env: {}              # Pipeline-level environment variables
      imports: []          # Import env vars from files
      services: []         # Services: docker, China-mirror, npc, vscode
      stages: []           # Serial execution stages
      endStages: []        # Always-run stages (after stages, even on failure)
      failStages: []       # Run only on failure stages
```

## Events Reference

| Event | Description |
|-------|-------------|
| `push` | Git push to branch |
| `pull_request` | PR created/updated (runs on source branch) |
| `pull_request.target` | PR event (runs on target branch code) |
| `pull_request.mergeable` | PR passed review + CI checks |
| `pull_request.merged` | PR merged |
| `pull_request.update` | PR metadata updated |
| `pull_request.approved` / `.changes_requested` | PR review events |
| `commit.add` | New commits added |
| `branch.create` / `branch.delete` | Branch lifecycle |
| `tag_push` | Tag pushed |
| `tag_deploy` / `tag_deploy.<env>` | Tag deployment |
| `api_trigger` / `api_trigger_<name>` | API-triggered |
| `web_trigger` / `web_trigger_<name>` | Web UI-triggered |
| `crontab: <cron-expr>` | Scheduled (POSIX cron, min 5min interval, Asia/Shanghai) |
| `vscode` | Remote dev environment |
| `issue.*` / `issue_comment.*` | Issue and comment events |

Branch patterns support glob: `main`, `feature/*`, `release/**`, `v*`. Use `$` for all-branch matching.

## Job Types

### Script Job
```yaml
- name: build
  image: node:20          # Optional runtime image
  script: npm run build   # Shell script (array joined with &&)
```

Shorthand: `- echo hello` equals `{name: "echo hello", script: "echo hello"}`.

### Plugin Job (Docker image as task)
```yaml
- name: publish
  image: plugins/npm
  settings:               # Plugin params (passed as PLUGIN_* env vars)
    username: $NPM_USER
    registry: https://registry.npmjs.org
  settingsFrom: ./secrets.json  # Load settings from file
```

### Built-in Task
```yaml
- name: cache
  type: docker:cache      # Built-in task type
  options:                # Task-specific options
    dockerfile: cache.dockerfile
    by: [package.json, package-lock.json]
  exports:
    name: CACHE_IMAGE
```

For complete built-in tasks reference, see `references/builtin-tasks.md`.

## Key Job Properties

| Property | Type | Description |
|----------|------|-------------|
| `if` | String/Array | Shell condition (exit 0 = run) |
| `ifModify` | String/Array | Run only if files changed (glob) |
| `ifNewBranch` | Boolean | Run only on new branches |
| `env` | Object | Job-scoped env vars |
| `imports` | String/Array | Import env vars from files |
| `exports` | Object | Export results as env vars |
| `timeout` | Number/String | Job timeout (e.g., `100s`, `1h`, max 12h) |
| `allowFailure` | Boolean | Don't fail pipeline on job failure |
| `retry` | Number | Retry count (backoff: 1s,2s,4s...) |
| `lock` | Boolean/Object | Distributed lock (`key`, `expires`, `wait`, `timeout`) |

## Environment Variables

Priority (high to low): Job env > Stage env > Pipeline env > imports > System defaults.

Key system variables: `CNB_REPO_SLUG`, `CNB_BRANCH`, `CNB_COMMIT`, `CNB_EVENT`, `CNB_TOKEN`, `CNB_BUILD_USER`.

For full default env vars list, see `references/env-vars.md`.

### Variable Substitution
Supported in: `image`, `settings`, `args`, `imports`, `optionsFrom`, `settingsFrom`.
Syntax: `$VAR` or `${VAR}`.

### Exporting Variables
```yaml
- name: get version
  script: echo "##[set-output version=1.0.0]"
  exports:
    version: MY_VERSION
- name: use it
  script: echo $MY_VERSION
```

## Services

```yaml
services:
  - docker                # Enable Docker-in-Docker
  - China-mirror          # China mirror acceleration
  - npc                   # Network Proxy Client
  - vscode                # Remote development
```

## Configuration Reuse

### YAML Anchors (single file)
```yaml
.pipeline: &pipeline
  stages:
    - name: test
      script: npm test
main:
  push:
    - <<: *pipeline
  pull_request:
    - <<: *pipeline
```

### Include (cross-file)
```yaml
include:
  - ./templates/common.yml
  - https://cnb.cool/<repo>/-/blob/main/template.yml
  - path: optional.yml
    ignoreError: true
  - config:
      main:
        push:
          - stages:
              - name: inline
                script: echo hello
```

Merge rules: arrays append, objects merge (local overrides included). Max 50 files. No cross-file YAML anchors.

### !reference Tag (cross-file value reference)
```yaml
# After include merging, reference values by path:
script: !reference [.templates, build-script]
# Reference entire pipeline:
main:
  push: !reference [.common-pipeline]
```

Max 10 nesting levels. Use `.` prefix for reference-only top-level keys.

## Caching Strategies

1. **File cache** via `runner.volumes`: `- /root/.npm` (node-local OverlayFS cache)
2. **Docker cache** via `docker:cache` built-in task (cross-node, image-based)

## Advanced Features

- **Built-in tasks** (docker:cache, cnb:apply/trigger, git:auto-merge, git:reviewer, etc.): See `references/builtin-tasks.md`
- **Custom deployment** (tag_deploy, environments, approval): See `references/deployment.md`
- **Plugin development** (creating Docker-based plugins): See `references/plugin-dev.md`
- **Pipeline visualization**: Use `##[group]name` / `##[endgroup]` in scripts
- **Skip pipeline**: `[ci skip]` in commit message, or `git push -o ci.skip`
- **Scheduled tasks**: `"crontab: <expr>"` as event key (POSIX cron, Asia/Shanghai)

## File Reference & Security

Four reference types: `include`, `imports`, `optionsFrom`, `settingsFrom`.

Cross-repo file access control fields (in referenced file):
- `allow_slugs`: Allowed repo patterns
- `allow_events`: Allowed event types
- `allow_branches`: Allowed branch patterns
- `allow_images`: Allowed plugin image patterns

Without these fields, the pipeline trigger user must be a Developer+ role in the file's repo.

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success, continue |
| `78` | Success, but stop pipeline |
| Other | Failure, stop pipeline |

## GitHub Actions Migration

Key mapping: `on` → event keys, `jobs` → `stages`, `steps` → `jobs`, `uses` → `image`, `with` → `settings`, `run` → `script`. For details see `references/migration.md`.
