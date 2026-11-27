# Kiro CLI Harness

Kiro CLI ([kiro.dev](https://kiro.dev/docs/cli/)) is one of the supported
harnesses. An agent is a JSON or Markdown file in `.kiro/agents/`. The file sets
the system prompt, the tools, the resources, the MCP servers, the hooks, and,
in current Kiro CLI, the permission rules.

The course starters use the CLI 2.x JSON shape. Current Kiro CLI can run that
shape in its V2 harness, or migrate it to the V3 shape with `/upgrade-agent`.
Both options use the same prompt, tools, and task.

## Install

Choose one install method:

```bash
# macOS (Homebrew)
brew install kiro-cli

# macOS and Linux (install script)
curl -fsSL https://cli.kiro.dev/install | bash

# Windows 11 (PowerShell)
irm 'https://cli.kiro.dev/install.ps1' | iex
```

Verify:

```bash
kiro-cli --version
```

## Sign in

An interactive Kiro CLI session uses a Kiro account. Headless CI runs can use
API key authentication instead. Run Kiro and follow the browser sign-in prompt:

```bash
kiro-cli
```

The sign-in options are:

- **AWS Builder ID** for a personal account.
- **Google or GitHub** social login.
- **IAM Identity Center** for an organization account.

Useful commands:

```bash
kiro-cli login          # sign in again
kiro-cli whoami         # show the current account
kiro-cli logout         # clear the local credentials
```

## First run

From your course repository:

```bash
kiro-cli
```

Kiro CLI 3.x starts the V3 harness by default and asks you to trust the
workspace the first time. Trust the folder to load its `.kiro/agents`, hooks,
MCP servers, and steering files. A V2 session (`--v2`) does not show that
screen.

## Agent format

A course agent is a JSON file in `.kiro/agents/`:

```json
{
  "name": "git-agent",
  "description": "Git workflow agent",
  "prompt": "You are a Git workflow automation agent. Use `gh` for all GitHub operations.",
  "tools": ["fs_read", "fs_write", "shell"],
  "allowedTools": ["fs_read"],
  "resources": ["file://docs/**/*.md"],
  "mcpServers": {
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch"]
    }
  },
  "hooks": {
    "postToolUse": [
      {
        "matcher": "fs_write",
        "command": "echo 'Run the tests after every change'"
      }
    ]
  }
}
```

The main fields:

| Field | Purpose |
|-------|---------|
| `name` | Agent name. It defaults to the file name. |
| `description` | What the agent does. |
| `prompt` | Inline system prompt, or a `file://` URI. |
| `tools` | Tools the agent may call. `*` includes all tools. |
| `allowedTools` | CLI 2.x pre-approval list. V3 uses `permissions.rules`. |
| `resources` | Files loaded at startup. V3 also supports `skill://` and `knowledgeBase`. |
| `mcpServers` | MCP servers for this agent. |
| `hooks` | Commands run on agent events. |
| `model` | Optional model pin. Without it, the session model is used. |

`prompt` supports a separate file. Relative URIs resolve from the agent config
directory:

```json
{
  "prompt": "file://./tdd-bdd-prompt.md"
}
```

A tool name in the course starters uses the CLI 2.x names. In the V3 harness the
same capabilities are called:

| CLI 2.x | V3 |
|---------|----|
| `fs_read` | `read` |
| `fs_write` | `write` |
| `shell` | `shell` |

V3 also accepts category selectors (`read`, `write`, `shell`, `web`,
`subagent`), MCP selectors (`@server`, `@server/tool`), `@builtin`, and `*`.

### V3 Markdown agents

Current Kiro CLI also accepts a Markdown agent with YAML frontmatter. The body
is the system prompt:

```markdown
---
name: git-agent
description: Git workflow agent
tools: ["read", "write", "shell"]
permissions:
  rules:
    - capability: shell
      match: ["git *", "gh *"]
      effect: allow
    - capability: fs_write
      match: ["docs/**", "src/**"]
      effect: allow
---

You are a Git workflow automation agent. Use `gh` for all GitHub operations.
```

Use the Markdown form when the prompt is long or you want the config and the
prompt in one readable file.

## V2 and V3 compatibility

Kiro CLI 3.x starts V3 by default. A legacy JSON profile that contains
`allowedTools` or `toolsSettings` can be skipped by V3 until it is migrated.
Pick one path and use it for every module:

- **Run the course on V2:** `kiro-cli --v2 --agent git-agent`. The JSON
  starters work as written.
- **Stay on V3 and migrate:** run `/upgrade-agent` in an interactive session,
  review the conversion, then start the agent normally. V3 converts supported
  `allowedTools` and `toolsSettings` entries into `permissions.rules`.

The course modules show `kiro-cli --tui --agent <name>`. On current Kiro CLI,
`kiro-cli --agent <name>` is enough; `--tui` is from an older release.

## Start an agent

```bash
kiro-cli --agent git-agent       # start a chat with the agent
kiro-cli --v2 --agent git-agent  # legacy JSON profile on V3 installs
```

Inside a session:

```bash
/agent list          # list agents
/agent swap git-agent
/agent create my-agent
/agent edit my-agent
/model               # switch the session model
/tools               # inspect the available tools
/context             # show context and token usage
/permissions         # inspect the permission policy
```

From the shell:

```bash
kiro-cli agent list
kiro-cli agent create my-agent
kiro-cli agent validate --path .kiro/agents/my-agent.json
```

## Course starters

Each module has a Kiro JSON starter:

| Module | Starter | Extra file |
|--------|---------|------------|
| 1 Git | `modules/module-1-git/starter/git-agent.json` | — |
| 2 Architecture | `modules/module-2-architecture/starter/architecture-agent.json` | — |
| 3 Requirements | `modules/module-3-requirements/starter/requirements-agent.json` | — |
| 4 CI/CD | `modules/module-4-cicd/starter/cicd-agent.json` | — |
| 5 TDD/BDD | `modules/module-5-tdd-bdd/starter/tdd-bdd-agent.json` | `tdd-bdd-prompt.md` |

Copy a starter into your repository:

```bash
mkdir -p .kiro/agents
cp modules/module-1-git/starter/git-agent.json .kiro/agents/
kiro-cli --agent git-agent
```

Module 5 uses a file URI prompt, so copy both files:

```bash
cp modules/module-5-tdd-bdd/starter/tdd-bdd-agent.json .kiro/agents/
cp modules/module-5-tdd-bdd/starter/tdd-bdd-prompt.md .kiro/agents/
kiro-cli --agent tdd-bdd-agent
```

## Permissions and safety

Kiro asks before an action unless a rule allows it. The policy sources are:

- **Default policy.** Workspace reads and common read-only commands are
  allowed. Everything else asks.
- **Agent rules.** `allowedTools` in CLI 2.x, or `permissions.rules` in V3.
- **User and workspace rules.** `~/.kiro/settings/permissions.yaml` and the
  per-workspace file under `~/.kiro/workspace-roots/`.
- **Workspace trust.** An untrusted workspace does not load its agents, hooks,
  MCP servers, or steering files, and asks before shell and MCP calls.

V3 permission rules have three effects:

| Effect | Result |
|--------|--------|
| `allow` | Run without prompting. |
| `ask` | Prompt every time. |
| `deny` | Block. A deny in any scope wins over an allow. |

Example V3 rules:

```json
{
  "permissions": {
    "rules": [
      { "capability": "shell", "match": ["git *", "gh *"], "effect": "allow" },
      { "capability": "fs_write", "match": ["src/**", "tests/**"], "effect": "allow" },
      { "capability": "shell", "match": ["rm -rf *", "sudo *"], "effect": "deny" }
    ]
  }
}
```

CLI 2.x uses `toolsSettings` for the same idea:

```json
{
  "toolsSettings": {
    "write": {
      "allowedPaths": ["src/**", "tests/**"]
    }
  }
}
```

Use `--trust-all-tools` only in a disposable CI or container environment. It
skips the approval prompts for the session.

Hooks are another safety option. A `PreToolUse` hook can audit or block a
sensitive command, and a `PostToolUse` hook can run the test suite after a
write. Keep hook commands exit-0 and correctly quoted so they do not warn on
every turn.

## Options and differences

- **Harness version.** V2 runs the legacy JSON starters as written. V3 adds
  Markdown agents, `permissions.rules`, and `excludedTools`.
- **Agent file.** JSON is compact and generated. Markdown keeps a long prompt
  readable next to the config.
- **Prompt.** Inline for short prompts. `file://` for long prompts and Module
  5's TDD prompt.
- **Resources.** `file://` loads files into context at startup. `skill://`
  loads skill metadata and the body on demand. `knowledgeBase` indexes a
  directory.
- **Hooks.** V3 uses an array of `SessionStart`, `PreToolUse`, and `PostToolUse`
  entries. CLI 2.x uses the object form with `agentSpawn`, `preToolUse`, and
  `postToolUse`. Standalone `.kiro/hooks/*.json` works across agents.
- **MCP.** Put servers in the agent's `mcpServers` or in `.kiro/settings/mcp.json`
  with `includeMcpJson: true`. `kiro-cli mcp add` edits the MCP config.
- **Models.** Use the session default, pin one with the agent's `model` field,
  or switch with `/model`.
- **Subagents.** V3 agents can start other agents as subagents. The parent
  policy still applies.
- **Steering and AGENTS.md.** Custom agents inherit steering files, skills, and
  `AGENTS.md` by default. Disable that with
  `kiro-cli settings chat.disableInheritingDefaultResources true`.

## Troubleshooting

- **`kiro-cli: command not found`.** Install it with one of the commands above
  and make sure the install directory is on `PATH`.
- **Sign-in fails or expires.** Run `kiro-cli login` and check the account with
  `kiro-cli whoami`.
- **An agent does not appear.** Confirm the file is in `.kiro/agents/`, the
  JSON parses, and the workspace is trusted. Run
  `kiro-cli agent validate --path .kiro/agents/<name>.json`.
- **A legacy agent is skipped on V3.** Run that session with `--v2`, or migrate
  the profile with `/upgrade-agent`.
- **A `file://` prompt is not found.** The path resolves from the agent config
  directory. For an agent in `.kiro/agents/`, use
  `file://./tdd-bdd-prompt.md` and place the prompt next to the JSON.
- **A hook warns on every turn.** Make the command exit 0 and quote it
  correctly.
- **An MCP server fails to start.** Run `kiro-cli mcp status --name <server>`
  or check `/mcp` for the server's stderr.

## References

- [Kiro CLI documentation](https://kiro.dev/docs/cli/)
- [Custom agents](https://kiro.dev/docs/custom-agents/)
- [Agent configuration reference](https://kiro.dev/docs/custom-agents/configuration-reference/)
- [Permissions](https://kiro.dev/docs/permissions/)
- [Hooks](https://kiro.dev/docs/hooks/)
- [MCP](https://kiro.dev/docs/mcp/)
- [CLI commands](https://kiro.dev/docs/reference/cli-commands/)
