# Module 4 Starter: CI/CD Agent

Starter agent configuration for Module 4, for Kiro CLI, Pi, or OpenCode.

## Setup

Kiro CLI:

```bash
cp cicd-agent.json /path/to/your-repo/.kiro/agents/
kiro-cli --tui --agent cicd-agent
```

OpenCode:

```bash
mkdir -p /path/to/your-repo/.opencode/agents
cp cicd-agent.md /path/to/your-repo/.opencode/agents/
opencode        # then pick cicd-agent with /agents
```

Pi:

```bash
# Add a cicd-agent preset to .pi/presets.json from the JSON prompt
pi --preset cicd-agent
```

See the [Pi harness guide](../../../docs/pi.md) for the preset format.

## What to Complete

The starter has TODO sections you must fill in:

1. **Dockerfile Template** — Add a Python Dockerfile template (`python:3.12-slim` base, pip install, pytest CMD)
2. **GitHub Actions Workflow Template** — Add the complete `ci.yml` template with triggers, permissions, and Docker build/run steps
3. **Path Filtering Rules** — Define which file changes should trigger the pipeline. Hint: think about where your implementation lives (`src/`), where your tests live (`tests/`), and the files that define the build environment (`Dockerfile`, `requirements.txt`, `.github/workflows/ci.yml`)
4. **Docker Build Verification Step** — Add a step that verifies the Docker image builds before running tests

## Usage

```bash
# Kiro CLI
kiro-cli --tui --agent cicd-agent

# OpenCode
opencode
# then pick cicd-agent with /agents

> Read my kata project structure
> Generate a Dockerfile for building and testing
> Generate a GitHub Actions CI workflow
```
