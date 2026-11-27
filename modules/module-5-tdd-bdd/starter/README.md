# Module 5 Starter: TDD/BDD Agent

Starter agent configuration for Module 5, for Kiro CLI, Pi, or OpenCode.

This is the most complex agent in the course. In Kiro it uses a **file URI
prompt** (`file://./tdd-bdd-prompt.md`) to keep the prompt in a separate file,
a best practice from the
[Kiro CLI configuration reference](https://kiro.dev/docs/cli/custom-agents/configuration-reference/).
OpenCode has no `file://` prompt, so the starter inlines the same prompt into
`tdd-bdd-agent.md`.

## Setup

Kiro CLI:

```bash
cp tdd-bdd-agent.json /path/to/your-repo/.kiro/agents/
cp tdd-bdd-prompt.md /path/to/your-repo/.kiro/agents/
kiro-cli --agent tdd-bdd-agent
```

OpenCode:

```bash
mkdir -p /path/to/your-repo/.opencode/agents
cp tdd-bdd-agent.md /path/to/your-repo/.opencode/agents/
opencode        # then pick tdd-bdd-agent with /agents
```

Pi:

```bash
# Add a tdd-bdd-agent preset to .pi/presets.json, inline tdd-bdd-prompt.md
pi --preset tdd-bdd-agent
```

See the [Pi harness guide](../../../docs/pi.md) for the preset format.

## What to Complete

The prompt file (`tdd-bdd-prompt.md`) has TODO sections you must fill in:

1. **Test Naming Convention** — Rules for naming test functions with Story/Scenario IDs
2. **GIVEN-WHEN-THEN Test Template** — The pytest template every test must follow
3. **Green Bar Pattern Rules** — When to use Fake It, Triangulate, Obvious Implementation
4. **Refactoring Checklist** — What to check during the refactoring step
5. **Commit Message Format** — How to format TDD commits with story references
6. **postToolUse Hook** — Replace the placeholder hook in `tdd-bdd-agent.json` with a real hook that runs pytest after every file write. OpenCode has no per-agent hook, so in `tdd-bdd-agent.md` add the exact pytest command to the prompt instead.

## Agent Configuration Features

In Kiro, this agent uses advanced features:

- **`file://` prompt** — Prompt lives in a separate `.md` file for readability
- **`resources`** — Auto-loads user stories from `docs/user-stories/`
- **`allowedTools`** — `read` is auto-approved for speed
- **`hooks.postToolUse`** — Runs after every file write (configure to run tests)

See the [Kiro CLI configuration reference](https://kiro.dev/docs/cli/custom-agents/configuration-reference/)
for all available options.

In OpenCode, the same ideas map to frontmatter and the prompt:

- The prompt is the Markdown body.
- `resources` have no auto-load, so the prompt names the story files to read.
- `permissions` grant the tools.
- `hooks.postToolUse` has no equivalent; put the test command in the prompt or
  add an OpenCode plugin / pre-commit hook.

See the [OpenCode harness guide](../../../docs/opencode.md) for the mapping.

## Usage

```bash
# Kiro CLI
kiro-cli --agent tdd-bdd-agent

# OpenCode
opencode
# then pick tdd-bdd-agent with /agents

> Read my user stories at docs/user-stories/
> Start with the first BE scenario of the first core story
> Follow strict RED-GREEN-REFACTOR — one test at a time
```
