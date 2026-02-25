# CNB Custom Deployment

## Overview

CNB supports custom deployment workflows via `tag_deploy` events with configurable environments, approval flows, and deployment pipelines.

## Environment Configuration

Create `.cnb/tag_deploy.yml` in repo root:

```yaml
environments:
  - name: development
    description: Development environment
    env:
      name: development
      tag_name: $CNB_BRANCH
    permissions:                    # Optional access control
      roles: [master, developer]
      users: [user1, user2]

  - name: staging
    description: Staging environment
    env:
      name: staging
      tag_name: $CNB_BRANCH

  - name: production
    description: Production environment
    env:
      name: production
      tag_name: $CNB_BRANCH
    button:                         # Custom action buttons
      - name: Deploy Button
        description: Deploy to production
        event: web_trigger_deploy   # Triggers this CI event
        isDefault: false
        permissions:
          roles: [master]
        env:
          deploy_target: prod
        inputs:                     # User input fields
          version:
            name: Version
            type: input
            required: true
            default: latest
          confirm:
            name: Confirm
            type: select
            options: [yes, no]
            default: "no"
```

## Deployment Pipeline

Configure in `.cnb.yml`:

```yaml
$:
  tag_deploy.development:
    - stages:
        - name: deploy to dev
          script: |
            echo "Deploying $tag_name to development"
            # deployment commands

  tag_deploy.production:
    - stages:
        - name: deploy to prod
          script: |
            echo "Deploying $tag_name to production"
            # deployment commands
```

## Input Types

| Type | Description |
|------|-------------|
| `input` | Single-line text input |
| `textarea` | Multi-line text input |
| `select` | Dropdown (single/multi select) |
| `switch` | Boolean toggle |
| `radio` | Radio button group |

## Approval Flow

Permissions control who can trigger deployments:
- `roles`: Role-based access (not hierarchical - `master` doesn't include `owner`)
- `users`: Username-based access
- Both can be combined (OR logic)

Without permissions config: any user with repo write access + tag_push permission can deploy.

## Button Configuration

Custom buttons can trigger different CI events (`web_trigger_*`) with pre-configured and user-input environment variables. Button `env` and `inputs` are passed as environment variables to the triggered pipeline.

Input groups are supported for organizing complex input forms.
