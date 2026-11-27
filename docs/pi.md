# Pi Harness

Pi ([pi.dev](https://pi.dev)) is one of the supported harnesses. It is a
minimal, extensible harness with no named agents. A small **preset extension**
adds them, so a Pi agent config is a JSON preset plus the prompt it loads.

Pi gives you two options:

- **Public preset extension.** The smallest setup. A preset sets a model, a
  thinking level, a tool list, and an inline prompt. Start it with
  `pi --preset <name>` or `/preset`.
- **Custom extension.** Pi's extension API can add instruction files, resource
  globs, context commands, per-preset MCP servers, subagents, and a permission
  gate. This is more work, but it is the same harness engineering the course
  teaches.

The prompt, tools, and task are the same as in Kiro CLI and OpenCode. Only the
file format and the extension features change.

## Install

Pi runs on Node 22.19 or newer. The course's other tools run on Node 20+, so
install Node 24 alongside it with `nvm`:

```bash
nvm install 24
nvm use 24
npm install -g --ignore-scripts @earendil-works/pi-coding-agent@1.0.4
pi --version   # 1.0.4
```

`--ignore-scripts` is intentional: Pi does not need post-install scripts. You
can also use the installer, which pins its dependencies:

```bash
curl -fsSL https://pi.dev/install.sh | sh
```

The course targets Pi 1.0.4. Do not run `pi update` without rerunning your
harness checks.

## Choose a provider

Pi supports several provider options:

- **Amazon Bedrock.** Use the AWS default credential chain. Run
  `aws sso login --profile <profile>`, export `AWS_PROFILE` and `AWS_REGION`,
  then set `provider` to `amazon-bedrock` and `model` to a model ID available in
  your account and region.
- **Subscription or API key.** Run `/login` inside Pi and pick Anthropic,
  OpenAI, Google, or another supported provider. The stored credential is used
  by the session.
- **Custom provider.** Add an endpoint under `models.json` when you need a
  gateway or a private model.

The course does not require a specific provider. If a preset omits `provider`
and `model`, Pi keeps the session default.

## First run

From your course repository:

```bash
nvm use 24
pi
```

The first time, Pi asks whether to trust the project. Run `/trust` once so Pi
loads `.pi/` (settings, extensions, skills, and prompts). In print, JSON, and
RPC modes the trust prompt does not appear, so run an interactive `/trust`
first or the project resources are skipped.

## Add preset support

The smallest setup is Pi's public preset extension. It ships with the npm
package:

```bash
mkdir -p .pi/extensions
cp "$(npm root -g)/@earendil-works/pi-coding-agent/examples/extensions/preset.ts" \
  .pi/extensions/preset.ts
```

If the installed package does not include the example, download it from the
pinned Pi release:

```bash
curl -fsSL \
  https://raw.githubusercontent.com/earendil-works/pi/v1.0.4/packages/coding-agent/examples/extensions/preset.ts \
  -o .pi/extensions/preset.ts
```

The public example supports `provider`, `model`, `thinkingLevel`, `tools`, and
`instructions`. For instruction files, resources, context commands, MCP servers,
subagents, or a permission gate, write your own extension with Pi's extension
API. Start from the examples in the Pi package.

Optional: set the default provider and model in `.pi/settings.json`:

```json
{
  "defaultProvider": "amazon-bedrock",
  "defaultModel": "<model-id>"
}
```

## Define an agent

Presets live in `.pi/presets.json` at the repository root. Each key is a preset
name. The public example uses an inline prompt:

```json
{
  "git-agent": {
    "tools": ["read", "grep", "find", "ls", "bash", "edit", "write"],
    "instructions": "You are a Git workflow automation agent. Use `gh` for all GitHub operations and follow the issue, branch, commit, and PR conventions from Module 1."
  }
}
```

Put the full prompt from the module's Kiro starter in `instructions` (the
`prompt` field of `modules/module-1-git/starter/git-agent.json`). JSON strings
use `\n` for line breaks. Add `provider` and `model` only when you want to pin
them.

A custom extension can use a richer schema. For example, it can read the prompt
from a file and expose resources and tool rules:

```json
{
  "git-agent": {
    "tools": ["read", "grep", "find", "ls", "bash", "edit", "write"],
    "instructionsFiles": [".pi/agents/git-agent.md"],
    "resourceGlobs": ["docs/**/*.md"],
    "policy": {
      "allowTools": ["read", "grep", "find", "ls", "bash"]
    }
  }
}
```

The field names are yours to define. This shape is a useful default: the
extension reads `instructionsFiles` into the system prompt, resolves
`resourceGlobs`, and checks `policy` before each tool call.

Start the agent:

```bash
pi --preset git-agent        # start in the preset
# or, inside a session:
/preset git-agent            # switch preset
/preset                      # pick from a list
/session                     # tokens, context, and session stats
```

The preset's model is the default. A `--model <id>` on the command line or
`/model` inside the session wins over the preset.

## What maps from Kiro

| Kiro | Pi public preset | Pi custom extension |
|------|------------------|---------------------|
| `.kiro/agents/<name>.json` | `.pi/presets.json` entry | Same |
| `prompt` (inline) | `instructions` string | `instructionsFiles` |
| `prompt: "file://..."` | Inline the file into `instructions` | `instructionsFiles` |
| `tools` | `tools` | `tools` |
| `resources` | No auto-load; name the files in `instructions` | `resourceGlobs` |
| `allowedTools` | No gate; `tools` limits what the agent can use | A `policy` gate |
| `hooks.postToolUse` | No per-agent hook; put the command in the prompt or a pre-commit hook | Run a check after tool calls |
| `mcpServers` | Project `.pi/mcp.json` | Per-preset `mcpServers` |

The Pi tool names for the course starters are:

| Kiro tool | Pi tool |
|-----------|---------|
| `fs_read` | `read`, `grep`, `find`, `ls` |
| `fs_write` | `edit`, `write` |
| `shell` | `bash` |

## Module starters

The course ships Kiro JSON and OpenCode Markdown starters. For Pi, use the same
prompt and rules from the Kiro starter and turn them into a preset:

1. Take the `prompt` value from `starter/<name>-agent.json`.
2. Add a `<name>-agent` entry to `.pi/presets.json` with the mapped tools.
3. Use `instructions` with the public extension, or save the prompt as
   `.pi/agents/<name>-agent.md` and use `instructionsFiles` with a custom
   extension.

Module 5 needs two adjustments:

- The prompt lives in `tdd-bdd-prompt.md`. Inline its contents into
  `instructions`, or keep it as an instruction file with a custom extension.
- `resources` and the `postToolUse` hook have no direct equivalent in the
  public extension. Tell the agent to read `docs/user-stories/**/*.md`, and put
  the exact pytest command in the prompt or a git pre-commit hook.

This is the same trade-off as the [OpenCode harness](opencode.md) and the
[Kiro harness](kiro.md): the prompt, tools, and task stay the same; only the
file format and the harness features change.

## Permissions and safety

The public preset extension only sets the model, tools, and prompt. Pi does not
ask for approval before every tool call and does not sandbox tool execution.
Run it with the permissions of a dedicated user or inside a container, and
review the diffs it produces. See [Pi security](https://pi.dev/docs/latest/security)
for the trust model and isolation options.

A custom extension can add a gate in front of every tool call:

- **allow** runs without prompting.
- **ask** shows `Allow? [Yes/No]` and blocks the call when you decline or when
  there is no interactive UI.
- **deny** blocks the call immediately.

A useful baseline is to allow read-only tools and the preset's read-only
`git`/`gh` commands, ask for writes and chained commands, and deny destructive
git (`--force`, `reset --hard`, `--no-verify`, `SKIP=`). The gate is a speed
bump, not a sandbox: a wrapper such as `bash -c`, `env`, or `python3 -c` can
still run something indirectly when the agent may run arbitrary commands.

## What a custom extension can add

Pi's extension API supports these options:

- **Context commands** that run before every prompt and add their output to the
  context.
- **Reference files** that are embedded in the system prompt or listed for the
  agent to read on demand.
- **Subagents** that run a preset in an isolated context and ask for approvals
  in the parent session.
- **A commit command** that delegates the commit flow to a Git preset.
- **MCP per preset**, so switching preset registers only that preset's servers.
- **A permission gate** with allow, ask, and deny rules.

Pi also supports skills natively and MCP servers from `.pi/mcp.json`. Add a
server with:

```bash
pi mcp add -l fetch -- uvx mcp-server-fetch
```

## Differences to keep in mind

- Pi needs Node 22.19 or newer, while the rest of the course runs on Node 20+.
- Named agents require a preset extension; plain Pi has no `--preset` flag.
- Pi has no language-server support, so Kiro's LSP is not available.
- Pi has no `web_search`; add a fetch MCP server if you need to read web pages,
  or use the `aws` CLI through `bash`.
- The public preset extension has no permission gate and no resource auto-load.
- Pi has no per-agent `postToolUse` hook. Use a prompt rule, a git pre-commit
  hook, or an extension for a deterministic sensor.

## Troubleshooting

- **`pi` fails with `enableCompileCache` or another `node:module` error.** You
  are on Node 20. Run `nvm use 24` and try again.
- **Provider errors or an expired token.** Re-run `/login`, or refresh your AWS
  credentials with `aws sso login --profile <profile>` and confirm
  `AWS_PROFILE` and `AWS_REGION` are exported.
- **Project resources are skipped.** Run an interactive `/trust` once before
  using print, JSON, or RPC mode.
- **`Unknown preset`.** Check that `.pi/presets.json` exists at the repository
  root, the name matches a key, and the preset extension loaded.
- **An MCP server fails to start.** Run `/mcp` to see the server's stderr and
  fix the command in `.pi/mcp.json`.

## References

- [Pi documentation](https://pi.dev/docs/latest)
- [Pi extensions](https://pi.dev/docs/latest/extensions)
- [Pi preset example](https://github.com/earendil-works/pi/blob/v1.0.4/packages/coding-agent/examples/extensions/preset.ts)
- [Pi security](https://pi.dev/docs/latest/security)
- [Pi MCP servers](https://pi.dev/docs/latest/mcp)
- [Kiro CLI custom agents](https://kiro.dev/docs/custom-agents/configuration-reference/)
