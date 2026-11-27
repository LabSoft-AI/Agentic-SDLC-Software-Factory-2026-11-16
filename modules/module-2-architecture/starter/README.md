# Architecture Agent Starter

Starter agent configuration for Module 2, for Kiro CLI, Pi, or OpenCode.

## Usage

From your kata repo root.

Kiro CLI:

```bash
mkdir -p .kiro/agents
cp modules/module-2-architecture/starter/architecture-agent.json .kiro/agents/
kiro-cli --tui --agent architecture-agent
```

OpenCode:

```bash
mkdir -p .opencode/agents
cp modules/module-2-architecture/starter/architecture-agent.md .opencode/agents/
opencode        # then pick architecture-agent with /agents
```

Pi:

```bash
# Add an architecture-agent preset to .pi/presets.json from the JSON prompt
pi --preset architecture-agent
```

See the [Pi harness guide](../../../docs/pi.md) for the preset format.

## What You Need to Complete

The starter config has the arc42 workflow and output format. You must add:

1. **C4 PlantUML templates** — Context, Container, and Component diagram
   examples using `!include` from the C4-PlantUML library
2. **ADR template** — Architecture Decision Record format with Status,
   Context, Decision, Rationale, Consequences
3. **Architectural principles** — the principles your agent should follow
   (e.g. serverless, EDA, DDD, scales-to-zero, single-table DynamoDB)

Refer to the Module 2 theory (README.md) for all patterns and formats.
