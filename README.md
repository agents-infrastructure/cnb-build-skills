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

## Usage

### As an Agent Skill

Install this skill in any [Agent Skills](https://agentskills.io/)-compatible AI agent. The agent will automatically use it when working with CNB pipeline configurations.

### Quick Example

Ask your AI agent:

> "Create a CNB pipeline that builds a Node.js project, runs tests on PR, and auto-deploys to production on tag push."

The agent will generate a complete `.cnb.yml` configuration leveraging the knowledge in this skill.

## Documentation Source

This skill is built from the official [CNB Cloud Native Build documentation](https://docs.cnb.cool/zh/build/intro.html), covering all 23+ pages of basic and advanced usage guides.

## License

MIT
