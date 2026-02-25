# CNB Built-in Tasks Reference

## Table of Contents
- [docker:cache](#dockercache)
- [cnb:await / cnb:resolve](#cnbawait--cnbresolve)
- [cnb:apply](#cnbapply)
- [cnb:trigger](#cnbtrigger)
- [cnb:read-file](#cnbread-file)
- [cnb:destroy-token](#cnbdestroy-token)
- [vscode:go](#vscodego)
- [git:auto-merge](#gitauto-merge)
- [git:reviewer](#gitreviewer)
- [git:issue-update](#gitissue-update)
- [git:release](#gitrelease)
- [git:pr-commit-message-preset](#gitpr-commit-message-preset)
- [git:pr-update](#gitpr-update)
- [testing:coverage](#testingcoverage)
- [artifact:remove-tag](#artifactremove-tag)
- [tapd:status-update / tapd:comment](#tapdstatus-update--tapdcomment)

---

## docker:cache

Build a Docker image as dependency cache for future builds.

```yaml
- name: build cache
  type: docker:cache
  options:
    dockerfile: cache.dockerfile    # Required: Dockerfile path
    by:                             # Files needed during build
      - package.json
      - package-lock.json
    versionBy:                      # Files for version calculation (default: by)
      - package-lock.json
    buildArgs:                      # Docker build args
      NODE_ENV: production
    target: build-stage             # Multi-stage build target
    sync: false                     # Wait for push completion
    ignoreBuildArgsInVersion: false  # Ignore buildArgs in version hash
  exports:
    name: CACHE_IMAGE_NAME          # Export cache image name
```

Version formula: `sha1(Dockerfile + versionBy + buildArgs + target + arch)`.

Usage pattern:
```yaml
- name: build cache
  type: docker:cache
  options:
    dockerfile: cache.dockerfile
    by: [package.json, package-lock.json]
  exports:
    name: DOCKER_CACHE_IMAGE_NAME
- name: use cache
  image: $DOCKER_CACHE_IMAGE_NAME
  commands:
    - cp -r "$NODE_PATH" ./node_modules
- name: build
  script: npm run build
```

Example `cache.dockerfile`:
```dockerfile
FROM node:14
WORKDIR /space
COPY . .
RUN npm ci
ENV NODE_PATH=/space/node_modules
```

## cnb:await / cnb:resolve

Cross-pipeline synchronization mechanism. `await` waits for `resolve` signal.

```yaml
# Pipeline A
- name: wait for B
  type: cnb:await
  options:
    key: my-signal        # Required: pairing ID
  exports:
    data_field: MY_VAR    # Access resolve data

# Pipeline B
- name: notify A
  type: cnb:resolve
  options:
    key: my-signal        # Required: same pairing ID
    data:                 # Optional: pass data to await
      data_field: value
```

Constraints:
- Only works across pipelines triggered by the same event
- One key can only be resolved once, but awaited multiple times
- Cannot be used in `endStages` or `failStages`
- Deadlock detection handles circular waits and missing resolves

## cnb:apply

Trigger a custom event pipeline in the **current** repository.

```yaml
- name: trigger sub-pipeline
  type: cnb:apply
  options:
    event: api_trigger_test       # Must be api_trigger or api_trigger_*
    configFrom: ./test/.cnb.yml   # Optional: config file path
    config: |                     # Optional: inline config (highest priority)
      main:
        api_trigger_test:
          - stages:
              - name: test
                script: echo test
    sync: false                   # Wait for completion
    continueOnBuildError: false   # Continue if triggered build fails
    title: "Custom Title"         # Custom pipeline title
```

Config priority: `config` > `configFrom` > repo `.cnb.yml`.

Output: `{ sn, buildLogUrl, message, buildSuccess (sync only), lastJobExports }`.

Auto-injected env vars: `APPLY_TRIGGER_BUILD_ID`, `APPLY_TRIGGER_PIPELINE_ID`, `APPLY_TRIGGER_REPO_SLUG`, etc.

## cnb:trigger

Trigger a custom event pipeline in a **different** repository.

```yaml
- name: trigger other repo
  type: cnb:trigger
  options:
    token: $TOKEN                 # Personal access token (default: CNB_TOKEN)
    slug: group/repo              # Required: target repo path
    event: api_trigger_test       # Required: must be api_trigger or api_trigger_*
    branch: main                  # Target branch (default: main branch)
    sha: abc123                   # Specific commit (default: branch HEAD)
    env:                          # Pass env vars to target
      MY_VAR: value
    sync: false                   # Wait for completion
    continueOnBuildError: false   # Continue if triggered build fails
    title: "Custom Title"
```

Output: `{ sn, buildLogUrl, message, buildSuccess (sync only), lastJobExports }`.

## cnb:read-file

Read a local file and export contents as environment variables.

```yaml
- name: read config
  type: cnb:read-file
  options:
    filePath: config.json         # Required: local file path (.json/.yml/.yaml/text)
  exports:
    myVar: ENV_MY_VAR
    deep.myVar: ENV_DEEP_VAR     # Nested key access
    $envKey.myVar: ENV_DYNAMIC   # Dynamic key via env var
```

## cnb:destroy-token

Destroy `CNB_TOKEN` early to reduce security risk.

```yaml
- name: destroy token
  type: cnb:destroy-token
```

## vscode:go

Control when remote development environment becomes available. Without this task, IDE entry appears after code-server starts. With it, IDE entry appears after this task completes.

```yaml
# In vscode event:
- name: setup
  script: npm install
- name: vscode go
  type: vscode:go
- name: background tasks
  script: npm run dev
```

## git:auto-merge

Auto-merge PR. Typically used in `pull_request.mergeable` event.

```yaml
- name: automerge
  type: git:auto-merge
  options:
    mergeType: auto               # merge | squash | rebase | auto (default)
    mergeCommitMessage: $CNB_PULL_REQUEST_TITLE  # Custom commit message
    mergeCommitFooter: "--story=123\n--story=456" # Footer lines
    removeSourceBranch: false     # Delete source branch after merge
    ignoreAssignee: false         # Merge even if PR has assignee
  exports:
    reviewedBy: REVIEWED_BY
    reviewers: REVIEWERS
```

`auto` mode: multi-author → merge, single-author → squash.

## git:reviewer

Add/remove PR reviewers or assignees.

```yaml
- name: add reviewers
  type: git:reviewer
  options:
    type: add-reviewer            # add-reviewer | add-reviewer-from-repo-members |
                                  # add-reviewer-from-group-members |
                                  # add-reviewer-from-outside-collaborator-members |
                                  # remove-reviewer | add-assignee | remove-assignee
    reviewers: "user1,user2"      # Usernames (comma or semicolon separated)
    count: 2                      # Number to add (random selection)
    exclude: "bot-user"           # Exclude users
    reviewersConfig: config.json  # File-based reviewer mapping
    role: reviewer                # reviewer | assignee
    skip_on_wip: false            # Skip if PR title starts with WIP
```

`reviewersConfig` format (JSON):
```json
{
  "./src": "reviewer1,reviewer2",
  ".cnb.yml": "reviewer3"
}
```

Events: `pull_request`, `pull_request.target`, `pull_request.update`.

## git:issue-update

Update Issue state, labels, and assignees.

```yaml
- name: update issues
  type: git:issue-update
  options:
    state: close                  # open | close
    label:
      add: "released"
      remove: "in-progress"
    assignee:
      add: ["user1"]
      remove: ["user2"]
    when:                         # Filter: only process issues matching these labels
      label: [feature, bug]
    lint:                         # Validate: fail if issues don't match
      label: [feature]
    fromText: "text with #123"   # Parse issue IDs from text
    fromFile: CHANGELOG.md        # Parse issue IDs from file
    prefix: [close, closed]       # Only process issues with these prefixes
    defaultColor: "#ff0000"       # Default color for new labels
```

Issue ID formats: `#123` (current repo), `group/repo#123` (cross-repo).

## git:release

Create a repository release.

```yaml
- name: release
  type: git:release
  options:
    tag: v1.0.0
    title: "Release v1.0.0"
    body: "Release notes..."
```

## git:pr-commit-message-preset

Pre-set the commit message for PR merge.

## git:pr-update

Update PR title and labels.

## testing:coverage

Report test coverage results.

```yaml
- name: coverage
  type: testing:coverage
  options:
    # Coverage report options
```

## artifact:remove-tag

Delete a CNB artifact tag.

## tapd:status-update / tapd:comment

Update TAPD linked items (story/task/bug) status or add comments.
