# OpenCode Harness

OpenCode ([opencode.ai](https://opencode.ai/docs/)) is one of the supported
harnesses. An agent is a Markdown file in `.opencode/agents/`. YAML frontmatter
sets the description, mode, model, and permissions. The file body is the system
prompt.

The prompt, tools, and task are the same as in Kiro CLI and Pi. Only the file
format and the permission model change.

## Install

Choose one install method:

```bash
# macOS and Linux (install script)
curl -fsSL https://opencode.ai/v2/install | bash

# macOS (Homebrew)
brew install anomalyco/tap/opencode-v2

# npm
npm install -g @opencode/cli
```

The npm package uses a postinstall script to select the native binary. Bun and
pnpm need an explicit allow-scripts flag. Verify:

```bash
opencode --version
```

## Choose a provider

OpenCode has built-in support for many providers. The options are:

- **Connect in the TUI.** Run `opencode`, then `/connect` and pick a provider.
  Choose a model with `/models`.
- **Connect from the CLI.** Run `opencode auth login` and pick a provider. Use
  `opencode auth login anthropic --method key` to enter an API key directly.
- **Use an environment variable.** Set the provider's supported variable, such
  as `ANTHROPIC_API_KEY`, in the environment that runs the OpenCode server.
- **Use an ambient cloud credential chain.** Amazon Bedrock uses the AWS default
  credential chain, and Google Vertex uses Application Default Credentials.
- **Add a custom provider.** Configure an endpoint under `providers` in
  `opencode.jsonc`.

The course does not require a specific provider. If an agent omits `model`, the
session default is used.

## First run

From your course repository:

```bash
opencode
```

Connect a provider with `/connect` the first time. Then pick an agent with
`/agents`, `Ctrl+X` followed by `A`, or cycle with `Shift+Tab`.

For one-off runs:

```bash
opencode run --agent git-agent "Create an issue for the README"
```

## Agent file format

```markdown
---
description: Git workflow agent
mode: primary
permissions:
  - action: read
    resource: "*"
    effect: allow
  - action: edit
    resource: "*"
    effect: allow
  - action: shell
    resource: "*"
    effect: allow
---

You are a Git workflow automation agent. Use `gh` for all GitHub operations and
follow the issue, branch, commit, and PR conventions from Module 1.
```

Agent files can live in:

- `.opencode/agents/<name>.md` for the project.
- `~/.config/opencode/agents/<name>.md` for every project.
- An `agents` block in `opencode.jsonc` when you prefer JSONC.

The `mode` field has three options:

| Mode | Behavior |
|------|----------|
| `primary` | Runs as the main agent for a session. This is the default. |
| `subagent` | Runs only in a child session through the `subagent` tool. |
| `all` | Runs as either a primary agent or a subagent. |

OpenCode also ships built-in `build`, `plan`, `general`, and `explore` agents.
A custom agent with the same ID overrides the built-in one. Set the starting
agent with `default_agent` in the config.

## What maps from Kiro

| Kiro | OpenCode |
|------|----------|
| `.kiro/agents/<name>.json` | `.opencode/agents/<name>.md` |
| `prompt` (inline or `file://`) | Markdown body. OpenCode has no `file://` prompt, so inline it. |
| `tools` | Built-in tools plus `permissions` rules. Common actions: `read`, `glob`, `grep`, `edit`, `shell`. |
| `resources` | No per-agent auto-load. Use `instructions`, `AGENTS.md`, `references`, or name the files in the prompt. |
| `allowedTools` | `permissions` allow rules. |
| `hooks.postToolUse` | No per-agent hook. Put the rule in the prompt, write a plugin, or use a git pre-commit hook. |
| `mcpServers` | `mcp.servers` in `opencode.jsonc`. Server config is project-wide. |
| `model` | `model: provider/model` in the frontmatter. |

## Module starters

Each module has an OpenCode starter next to the Kiro JSON:

| Module | OpenCode starter |
|--------|------------------|
| 1 Git | `modules/module-1-git/starter/git-agent.md` |
| 2 Architecture | `modules/module-2-architecture/starter/architecture-agent.md` |
| 3 Requirements | `modules/module-3-requirements/starter/requirements-agent.md` |
| 4 CI/CD | `modules/module-4-cicd/starter/cicd-agent.md` |
| 5 TDD/BDD | `modules/module-5-tdd-bdd/starter/tdd-bdd-agent.md` |

Copy a starter into your repository:

```bash
mkdir -p .opencode/agents
cp modules/module-1-git/starter/git-agent.md .opencode/agents/
```

Then start OpenCode and select the agent:

```bash
opencode
> /agents
```

Use the same prompts as the Kiro instructions. The agent, tools, and task are
the same. Only the file format changes.

## Permissions and safety

An agent has an ordered list of permission rules:

```markdown
permissions:
  - action: shell
    resource: "*"
    effect: ask
  - action: shell
    resource: "git status *"
    effect: allow
  - action: shell
    resource: "git push *"
    effect: deny
```

| Field | Meaning |
|-------|---------|
| `action` | Tool or permission action. Wildcards are supported. |
| `resource` | Path, command, URL, agent ID, or other value. |
| `effect` | `allow`, `ask`, or `deny`. |

Rules are evaluated in order and the last match wins. Put a broad rule first
and specific exceptions after it. If an operation touches several resources,
any `deny` denies it, otherwise any `ask` asks.

Every agent starts with a base policy that allows all actions and asks before
reading `.env` files. Global rules apply before agent rules, so an agent rule
can refine them. A saved approval never overrides a configured `deny`.

The `shell` action runs with your user's filesystem, process, and network
authority. Prefer a narrow shell allowlist over patterns that try to recognize
every dangerous command. OpenCode scans compound commands and checks each
subcommand.

## MCP servers

MCP servers are project-wide. Add one with the CLI:

```bash
opencode mcp add context7 --url https://mcp.context7.com/mcp
opencode mcp list
```

Or configure it in `opencode.jsonc`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "servers": {
      "fetch": {
        "type": "local",
        "command": ["uvx", "mcp-server-fetch"]
      }
    }
  }
}
```

Remote servers use OAuth by default. Run `/mcps` to inspect a server, sign in,
or reconnect. Tool permissions use the normalized `<server>_<tool>` name.

## Resources and hooks

OpenCode has no per-agent resource auto-load. The options are:

- **`AGENTS.md`** at the repository root, loaded automatically.
- **`instructions`** in `opencode.jsonc`, with paths, globs, or URLs.
- **`references`** in `opencode.jsonc`, for local directories or Git
  repositories used as named context.
- **The prompt**, which can name the files the agent must read.

OpenCode has no per-agent `postToolUse` hook. The options are:

- Put the exact command in the prompt and require the agent to run it.
- Write an OpenCode plugin that runs after a tool call.
- Use a git pre-commit hook for a deterministic check before a commit.

## Options and differences

- **Agent format.** Markdown for hand-written agents, JSONC for generated or
  shared config.
- **Modes.** `primary`, `subagent`, or `all`. Subagents run in a fresh context
  and can be restricted with the `subagent` permission action.
- **Models.** `provider/model` with an optional `#variant`, or inherit the
  session model.
- **Permissions.** Ordered rules with `allow`, `ask`, and `deny`; the last
  match wins.
- **MCP.** Project-wide `mcp.servers`, with local and remote transports.
- **Plugins.** The extension point for hooks and other deterministic behavior.
- **Skills and commands.** Native `.opencode/skills/` and custom slash commands.
- **Instructions and references.** Additional context files or directories,
  loaded for every agent.

## Troubleshooting

- **An agent does not appear.** Confirm the file is in `.opencode/agents/` or
  `~/.config/opencode/agents/`, the frontmatter parses, and the mode allows the
  way you start it.
- **Provider requests fail.** Run `/connect` or `opencode auth login`, then
  check the model with `/models`. For an environment variable, confirm it is
  set in the process that runs the OpenCode server.
- **A permission rule does not apply.** Remember that rules are ordered and the
  last match wins. Move the specific rule after the broad one.
- **A shell command is blocked by the scanner.** Simplify the command or add a
  narrow allow rule. Dynamic evaluation such as `eval` cannot be checked
  against a deny rule.
- **An MCP server does not connect.** Run `opencode mcp list`, then `/mcps` for
  the error. Sign in with `opencode mcp auth <server>` when it needs OAuth.

## References

- [OpenCode documentation](https://opencode.ai/docs/)
- [OpenCode agents](https://opencode.ai/docs/agents/)
- [OpenCode configuration](https://opencode.ai/docs/config/)
- [OpenCode permissions](https://opencode.ai/docs/permissions/)
- [OpenCode MCP servers](https://opencode.ai/docs/mcp-servers/)
- [OpenCode CLI providers](https://opencode.ai/docs/cli/providers/)
