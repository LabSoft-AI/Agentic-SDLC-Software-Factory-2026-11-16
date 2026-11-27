# Module 3 Starter: Requirements Agent

Starter agent configuration for Module 3, for Kiro CLI, Pi, or OpenCode.

## Setup

Kiro CLI:

```bash
cp requirements-agent.json /path/to/your-repo/.kiro/agents/
kiro-cli --tui --agent requirements-agent
```

OpenCode:

```bash
mkdir -p /path/to/your-repo/.opencode/agents
cp requirements-agent.md /path/to/your-repo/.opencode/agents/
opencode        # then pick requirements-agent with /agents
```

Pi:

```bash
# Add a requirements-agent preset to .pi/presets.json from the JSON prompt
pi --preset requirements-agent
```

See the [Pi harness guide](../../../docs/pi.md) for the preset format.

## What to Complete

The starter has TODO sections you must fill in:

1. **Story Bundle Template** — Define the complete format for Original + FE + BE + INFRA stories
2. **Pareto Analysis Rules** — Rules for identifying the 20% core stories
3. **INFRA Story Requirements** — Minimum infrastructure stories per user story
4. **Architecture Reference Rules** — How to link stories back to architecture sections

## Usage

```bash
# Kiro CLI
kiro-cli --tui --agent requirements-agent

# OpenCode
opencode
# then pick requirements-agent with /agents

> Read my architecture at architecture/
> List all user stories with Pareto prioritization
> Start with the first core story
```
