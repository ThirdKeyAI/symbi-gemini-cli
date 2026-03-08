# symbi-gemini-cli

<p align="center">
  <img src="symbi-gemini-cli.png" alt="symbi-gemini-cli" width="300">
</p>

A Gemini CLI extension that brings [Symbiont](https://symbiont.dev)'s zero-trust AI agent governance to your development workflow. Enforce Cedar authorization policies, verify MCP tool integrity with SchemaPin, maintain cryptographic audit trails, and manage governed agents -- all from within Gemini CLI.

## Prerequisites

- [Gemini CLI](https://github.com/google-gemini/gemini-cli) installed
- `symbi` binary on PATH (optional -- extension degrades gracefully without it)
- `jq` for JSON parsing in hook scripts (`apt install jq` / `brew install jq`)

Install `symbi`:
```bash
# From source
cargo install symbi

# Or via Docker
docker pull ghcr.io/thirdkeyai/symbi:latest
```

Or run the included install script:
```bash
./install.sh
```

## Installation

```bash
gemini extensions install https://github.com/thirdkeyai/symbi-gemini-cli
```

**For development:**
```bash
git clone https://github.com/thirdkeyai/symbi-gemini-cli
cd symbi-gemini-cli
gemini extensions link .
```

## Quick Start

1. Install the extension (see above)
2. Run `/symbi:init` to scaffold a governed project
3. Define agents in `agents/*.dsl`
4. Create Cedar policies in `policies/*.cedar`
5. Use `/symbi:status` to verify everything is connected

## Skills

Skills require `experimental.skills` to be enabled in Gemini CLI settings.

| Skill | Description |
|-------|-------------|
| `/symbi-init` | Scaffold a governed agent project with starter files |
| `/symbi-policy` | Create, edit, or validate Cedar authorization policies |
| `/symbi-verify` | Verify MCP tool schemas using SchemaPin |
| `/symbi-audit` | Query and analyze cryptographic audit logs |
| `/symbi-dsl` | Parse, validate, and create Symbiont DSL agent definitions |
| `/symbi-agent-sdk` | Generate boilerplate for Gemini CLI + ORGA governance |

## Commands

| Command | Description |
|---------|-------------|
| `/symbi:status` | Check health of the Symbiont runtime and installed components |
| `/symbi:init` | Quick project scaffold (command fallback when skills aren't enabled) |
| `/symbi:verify` | Verify MCP tool schemas |

## Agents (Preview)

Agents are a preview feature in Gemini CLI.

| Agent | Description |
|-------|-------------|
| `symbi-governor` | Governance-aware coding agent. Enforces policies and maintains audit trails. |
| `symbi-dev` | DSL development specialist for writing agents and Cedar policies. |

## Hooks

The extension installs hooks that run automatically:

- **PreToolUse** (`policy-log.sh`): Advisory logging of state-modifying tool calls (does not block)
- **PostToolUse** (`audit-log.sh`): Logs tool usage to `.symbiont/audit/tool-usage.jsonl`

Hooks apply to `write_file`, `replace`, `run_shell_command`, and all `symbi__*` MCP tools.

## Native Policies

Gemini CLI's built-in policy engine provides governance even without the symbi binary:

- **symbi-governance.toml**: Warns on unverified MCP tool usage and shell commands
- **tool-safety.toml**: Blocks dangerous patterns (`rm -rf`, `git push --force`)

This is a Gemini CLI-exclusive feature -- Claude Code has no native policy engine.

## MCP Tools

When `symbi` is on PATH, the extension connects to the Symbiont MCP server exposing:

- `symbi__invoke_agent` -- Run a governed agent with a prompt
- `symbi__list_agents` -- List available agents from `agents/*.dsl`
- `symbi__parse_dsl` -- Parse and validate DSL files
- `symbi__get_agent_dsl` -- Read an agent's DSL definition
- `symbi__get_agents_md` -- Get the project's AGENTS.md content
- `symbi__verify_schema` -- Verify a tool schema with SchemaPin

## Dual-Mode Architecture

The extension supports two integration patterns:

### Mode A -- Standalone (Extension-First)

Developer installs the extension directly into Gemini CLI. The extension spawns its own `symbi mcp` server, provides policy checking via hooks and native policies, and logs to local audit files.

```
Developer -> Gemini CLI + symbi extension -> symbi mcp (stdio)
```

Best for: individual developers adding governance awareness to their workflow.

### Mode B -- ORGA-Managed (Runtime-First)

Symbiont's CliExecutor spawns Gemini CLI as a governed subprocess. The extension detects `SYMBIONT_MANAGED=true` and defers to the outer ORGA Gate for hard enforcement that cannot be bypassed.

```
Symbiont Runtime (ORGA Loop)
  -> CliExecutor (sandbox + budget enforcement)
    -> Gemini CLI (with symbi extension)
      -> Extension connects back to parent MCP server
```

Best for: automated pipelines, dark factory deployments, enterprise governance.

See `examples/` for complete setups of each mode.

## Examples

| Directory | Description |
|-----------|-------------|
| `examples/standalone/` | Mode A setup for individual developers |
| `examples/cli-executor/` | Mode B setup with DSL + Cedar policy for ORGA-wrapped Gemini CLI |
| `examples/agent-sdk/` | Headless agent wrapper pattern for programmatic use |

## Gemini CLI-Exclusive Features

Capabilities available in Gemini CLI that the Claude Code plugin doesn't have:

- **Native policy engine**: `policies/*.toml` files provide governance without the symbi binary
- **`excludeTools` in manifest**: Declaratively block dangerous tools at the extension level
- **Pattern-specific tool exclusion**: e.g., `run_shell_command(rm -rf)` blocking
- **Secure settings storage**: `"sensitive": true` for API keys in system keychain

## Configuration

Project-level configuration lives in `symbiont.toml` (created by `/symbi:init`).

## File Conventions

| Path | Purpose |
|------|---------|
| `agents/*.dsl` | Agent DSL definitions |
| `policies/*.cedar` | Cedar authorization policies |
| `symbiont.toml` | Symbiont runtime configuration |
| `AGENTS.md` | Agent manifest |
| `.symbiont/audit/` | Audit log output |

## Comparison with Claude Code Plugin

This extension delivers the same Symbiont governance capabilities as [symbi-claude-code](https://github.com/thirdkeyai/symbi-claude-code), adapted for Gemini CLI's extension format. Key differences:

| Aspect | Claude Code | Gemini CLI |
|--------|-------------|------------|
| Commands | Markdown files | TOML files |
| MCP tool prefix | `mcp__symbi__` | `symbi__` |
| Native policies | No | Yes (`policies/*.toml`) |
| Tool restriction | Allow list | Deny list (`excludeTools`) |
| Context file | `CLAUDE.md` | `GEMINI.md` |

## License

Apache 2.0 -- see [LICENSE](LICENSE).

## Links

- [Symbiont Documentation](https://docs.symbiont.dev)
- [ThirdKey AI](https://thirdkey.ai)
- [Claude Code Plugin](https://github.com/thirdkeyai/symbi-claude-code)
- [Implementation Roadmap](ROADMAP.md)
