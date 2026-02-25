# CNB Build Skills

[Agent Skill](https://agentskills.io/) for [CNB (Cloud Native Build)](https://cnb.cool) — a cloud-native CI/CD platform.

This skill enables AI agents to create and manage CI/CD pipelines on the CNB platform, covering the full spectrum from basic pipeline configuration to advanced deployment workflows.

## Capabilities

| Category | What's Covered |
|----------|---------------|
| **Pipeline Configuration** | `.cnb.yml` syntax, branch patterns, event triggers, stages, jobs |
| **Job Types** | Script tasks, plugin tasks (Docker images), built-in tasks |
| **Built-in Tasks** | `docker:cache`, `cnb:apply`, `cnb:trigger`, `git:auto-merge`, `git:reviewer`, `git:issue-update`, and 10+ more |
| **Environment Variables** | System defaults, custom declaration, imports, exports, variable substitution |
| **Configuration Reuse** | YAML anchors, `include` (cross-file), `!reference` tag, `imports` |
| **Caching** | File cache (`runner.volumes`), Docker image cache (`docker:cache`) |
| **Deployment** | Custom environments, approval flows, `tag_deploy` events, input forms |
| **Plugin Development** | Creating Docker-based plugins, parameter design, testing, publishing |
| **Migration** | GitHub Actions → CNB concept mapping and migration examples |
| **Advanced** | Scheduled tasks (`crontab`), pipeline visualization, skip pipeline, NPC, remote dev |

## File Structure

```
cnb-build-skills/
├── README.md                          # This file
├── SKILL.md                           # Main skill entry (228 lines)
└── references/
    ├── builtin-tasks.md               # All 16+ built-in tasks with full params & examples
    ├── deployment.md                  # Custom deployment environments & approval flows
    ├── env-vars.md                    # Default environment variables & declaration methods
    ├── migration.md                   # GitHub Actions → CNB migration guide
    └── plugin-dev.md                  # Plugin development workflow
```

The skill follows a **progressive disclosure** pattern: `SKILL.md` contains the core knowledge needed for most tasks, while `references/` files are loaded on-demand for specialized topics.

## Installation

### Quick Install (Recommended)

```bash
# Add skill directly with npx
npx skills add cnb-build-skills

# Or with agent-skills CLI
agent-skills add cnb-build-skills
```

### Using Agent Skills CLI

```bash
# Install globally
agent-skills install cnb-build-skills

# Or install to your project
agent-skills install cnb-build-skills --project
```

### Manual Installation

1. Clone or download this repository
2. Copy to your agent skills directory:
   ```bash
   cp -r cnb-build-skills ~/.agent-skills/
   # Or for project-level installation:
   cp -r cnb-build-skills ./.agent-skills/
   ```
3. Verify installation:
   ```bash
   agent-skills list | grep cnb-build
   ```

### In Your AI Agent Config

Add to your agent configuration (typically `~/.agent/config.yml` or project `.agent.yml`):

```yaml
skills:
  - cnb-build-skills
```

If using a specific version:

```yaml
skills:
  - name: cnb-build-skills
    version: 1.0.0  # Optional
```

## Usage

### Quick Start with Your AI Agent

Once installed, simply ask your AI agent to help with CNB pipelines:

```
"Create a CNB pipeline that builds a Node.js project, runs tests on PR,
and auto-deploys to production on tag push."
```

The AI agent will automatically leverage this skill's knowledge to generate a complete `.cnb.yml` configuration.

### Common Use Cases

#### 1. **Basic CI/CD Pipeline**
```
"Set up a simple CNB pipeline that runs npm install and npm test on every push."
```

#### 2. **Multi-Stage Deployment**
```
"Create a CNB pipeline with:
- Test stage on PR
- Build stage on main branch push
- Deploy to staging on tag push
- Manual approval before production deployment"
```

#### 3. **Docker Build and Push**
```
"Build a Docker image, cache it, and push to registry on tag push."
```

#### 4. **GitHub Actions Migration**
```
"Convert this GitHub Actions workflow to CNB format."
(Paste your workflow.yml)
```

#### 5. **Environment Management**
```
"Set up production and staging environments with approval flows."
```

### Direct Skill Knowledge Access

Ask about specific CNB features:

- **Pipeline structure**: "Explain CNB pipeline configuration"
- **Event triggers**: "What events does CNB support?"
- **Built-in tasks**: "List all available built-in tasks"
- **Environment variables**: "How to set and use env vars in CNB?"
- **Caching**: "Best practices for caching in CNB"
- **Deployment**: "How to set up production deployments?"
- **Plugin development**: "Create a custom CNB plugin"
- **Migration**: "Migrate GitHub Actions to CNB"

### Detailed Reference Guides

This skill includes specialized documentation for advanced scenarios:

| Topic | File | When to Use |
|-------|------|------------|
| **Built-in Tasks** | `references/builtin-tasks.md` | Using docker:cache, git:*, cnb:* tasks |
| **Deployment** | `references/deployment.md` | Setting up environments and approval flows |
| **Environment Variables** | `references/env-vars.md` | Variable declaration, imports, exports |
| **Plugin Development** | `references/plugin-dev.md` | Creating custom Docker-based plugins |
| **GitHub Actions Migration** | `references/migration.md` | Converting from GitHub Actions to CNB |

Your AI agent can access these automatically when needed for your use case.

## Examples

### Example 1: Simple Node.js Pipeline

```yaml
main:
  push:
    - stages:
        - name: install
          script: npm ci
        - name: test
          script: npm test
        - name: build
          script: npm run build
```

### Example 2: With Environment Variables and Caching

```yaml
env:
  NODE_ENV: production

main:
  push:
    - runner:
        docker:
          image: node:20
        volumes:
          - /root/.npm
      stages:
        - name: test
          script: npm test
        - name: build
          script: npm run build

feature/*:
  pull_request:
    - stages:
        - name: validate
          script: npm run lint && npm test
```

### Example 3: With Tag Deployment

```yaml
env:
  DOCKER_REGISTRY: docker.io
  IMAGE_NAME: myapp

main:
  push:
    - stages:
        - name: build
          script: npm run build

  tag_push:
    - stages:
        - name: publish
          type: docker:cache
          options:
            dockerfile: Dockerfile
            by: [package.json, package-lock.json]
        - name: push-image
          image: plugins/docker
          settings:
            repo: $DOCKER_REGISTRY/$IMAGE_NAME
            tags: ${CNB_BRANCH}
            username: $DOCKER_USER
            password: $DOCKER_PASSWORD
```

## Resources

- **Official CNB Site**: https://cnb.cool
- **Agent Skills**: https://agentskills.io/
- **CNB Documentation**: https://cnb.cool/docs
- **GitHub**: This repository

## Support & Issues

Found a problem or want to contribute?

1. Check existing documentation in `references/`
2. Ask your AI agent for help
3. Report issues on GitHub

## License

See [LICENSE](LICENSE) file for details.

## Documentation Source

This skill is built from the official [CNB Cloud Native Build documentation](https://docs.cnb.cool/zh/build/intro.html), covering all 23+ pages of basic and advanced usage guides.

## License

MIT
