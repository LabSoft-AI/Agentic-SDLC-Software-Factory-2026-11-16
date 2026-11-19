# Git Agent Starter

Starter agent configuration for Module 1, for Kiro CLI or OpenCode.

## Usage

Copy to your course repository.

Kiro CLI:

```bash
mkdir -p .kiro/agents
cp modules/module-1-git/starter/git-agent.json .kiro/agents/
kiro-cli --tui --agent git-agent
```

OpenCode:

```bash
mkdir -p .opencode/agents
cp modules/module-1-git/starter/git-agent.md .opencode/agents/
opencode        # then pick git-agent with /agents
```

## What You Need to Complete

The starter config only includes the GitHub Issue template. You must add:

1. **Commit convention** — format, types, scope rules
2. **Branch naming convention** — prefixes and naming rules
3. **Pull Request template** — format and required sections

Refer to the Module 1 theory for all conventions.
