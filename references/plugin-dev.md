# CNB Plugin Development

## Overview

A CNB plugin is a Docker image. Plugin parameters are passed as environment variables with `PLUGIN_` prefix (uppercase).

## Development Workflow

### 1. Design Parameters

```yaml
# Usage in .cnb.yml:
- name: hello world
  image: cnbcool/hello-world
  settings:
    text: hello world
```

This passes `PLUGIN_TEXT="hello world"` to the container.

### 2. Parameter Types

| Type | Conversion |
|------|-----------|
| String | As-is |
| Number | String representation |
| Boolean | `"true"` or `"false"` |
| Array | Comma-separated: `hello,world` |
| Object | JSON string: `{"key":"value"}` |

### 3. Write Entrypoint Script

```bash
#!/bin/sh
echo "$PLUGIN_TEXT"
```

### 4. Create Dockerfile

```dockerfile
FROM alpine
ADD entrypoint.sh /bin/
RUN chmod +x /bin/entrypoint.sh
ENTRYPOINT /bin/entrypoint.sh
```

### 5. Build and Test

```bash
# Build
docker build -t cnbcool/hello-world .

# Test locally
docker run --rm \
  -e PLUGIN_TEXT="hello world" \
  cnbcool/hello-world

# Test with workspace files
docker run --rm \
  -e PLUGIN_TEXT="hello world" \
  -v $(pwd):$(pwd) \
  -w $(pwd) \
  cnbcool/hello-world
```

### 6. Export Variables

Use `exports` in `.cnb.yml` to capture plugin output:

```yaml
- name: my-plugin
  image: my-plugin-image
  exports:
    result: MY_RESULT
```

### 7. Publish

Push to CNB artifact registry or Docker Hub:

```bash
docker push cnbcool/hello-world
```

## Key Notes

- Plugin `settings` are converted to `PLUGIN_*` env vars (uppercase key, `PLUGIN_` prefix)
- Custom env vars from `imports`/`env` are NOT passed to plugins; use `settings` with variable substitution
- CNB system env vars ARE passed to plugins
- Plugin workspace is mapped to the build directory
- `args` parameter appends to ENTRYPOINT (like `docker run image <args>`)
- Use `settingsFrom` to load settings from JSON/YAML files

## Plugin vs Script Task

| Aspect | Script Task | Plugin Task |
|--------|-------------|-------------|
| `image` | Runtime environment | The plugin itself |
| `script` | Commands to run | Not used |
| `settings` | Not used | Plugin parameters |
| Env vars | All env vars available | Only `PLUGIN_*` + system vars |
| Reusability | Low | High (shareable Docker image) |
