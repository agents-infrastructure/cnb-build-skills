# CNB Default Environment Variables

## Repository Information

| Variable | Description |
|----------|-------------|
| `CNB_REPO_SLUG` | Full repo path (e.g., `group/repo`) |
| `CNB_REPO_ID` | Repo ID |
| `CNB_REPO_URL` | Repo URL |
| `CNB_REPO_OWNER` | Repo owner username |

## Branch & Commit

| Variable | Description |
|----------|-------------|
| `CNB_BRANCH` | Current branch name (or tag name for tag events) |
| `CNB_COMMIT` | Full commit SHA |
| `CNB_COMMIT_SHORT` | Short commit SHA |
| `CNB_IS_NEW_BRANCH` | `true` if branch is newly created |
| `CNB_LATEST_COMMIT_MESSAGE` | Latest commit message |

## Build Information

| Variable | Description |
|----------|-------------|
| `CNB_BUILD_ID` | Build ID |
| `CNB_BUILD_USER` | Build trigger user |
| `CNB_PIPELINE_ID` | Pipeline ID |
| `CNB_PIPELINE_NAME` | Pipeline name |
| `CNB_EVENT` | Trigger event name |
| `CNB_EVENT_URL` | Event URL |
| `CNB_TOKEN` | Temporary access token (auto-created/destroyed) |

## Pull Request

| Variable | Description |
|----------|-------------|
| `CNB_PULL_REQUEST_ID` | PR ID |
| `CNB_PULL_REQUEST_IID` | PR internal ID |
| `CNB_PULL_REQUEST_TITLE` | PR title |
| `CNB_PULL_REQUEST_MERGE_SHA` | PR merge commit SHA |
| `CNB_TARGET_BRANCH` | PR target branch |
| `CNB_SOURCE_BRANCH` | PR source branch |

## Tag

| Variable | Description |
|----------|-------------|
| `CNB_TAG` | Tag name (for tag events) |

## Environment Variable Declaration

### Pipeline Level
```yaml
main:
  push:
    - env:
        MY_VAR: value
      stages: [...]
```

### Stage Level
```yaml
stages:
  - name: stage1
    env:
      STAGE_VAR: value
    jobs: [...]
```

### Job Level
```yaml
- name: job1
  env:
    JOB_VAR: value
  script: echo $JOB_VAR
```

### Import from Files
```yaml
imports:
  - ./local-env.yml
  - https://cnb.cool/<repo>/-/blob/main/env.yml
```

Supported formats: `.json`, `.yml`, `.yaml`, text (`key=value`).

### Export from Jobs
```yaml
- name: produce
  script: echo "##[set-output key=value]"
  exports:
    key: ENV_KEY
- name: consume
  script: echo $ENV_KEY
```

Sensitive values: Use `##[set-secret]value` to mask in logs.

### Variable Substitution
Supported in: `image`, `settings`, `args`, `imports`, `optionsFrom`, `settingsFrom`.
Syntax: `$VAR` or `${VAR}`.
Not supported in: `script` (use shell syntax directly), `env` values.
