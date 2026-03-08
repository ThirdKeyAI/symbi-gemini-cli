# ROADMAP -- symbi-gemini-cli Extension Implementation Plan

## Project Overview

Build a Gemini CLI extension that brings Symbiont's trust stack (ORGA, Cedar policies, SchemaPin, sandboxing) to Gemini CLI users. The extension exposes Symbiont agents as MCP tools, enforces policies via hooks and Gemini CLI's native policy engine, and provides skills for agent development and governance.

**Repo**: `thirdkeyai/symbi-gemini-cli`
**License**: Apache 2.0

---

## Directory Structure

```
symbi-gemini-cli/
├── gemini-extension.json            # Extension manifest (required, at root)
├── GEMINI.md                        # Context/playbook for the model
├── hooks/
│   └── hooks.json                   # Hook config for policy enforcement
├── scripts/
│   ├── policy-log.sh                # PreToolUse hook: advisory policy logging
│   ├── audit-log.sh                 # PostToolUse hook: audit trail
│   ├── install-check.sh             # Verify symbi is installed
│   └── mcp-wrapper.sh              # MCP transport switching (stdio/HTTP)
├── policies/
│   ├── symbi-governance.toml        # Native Gemini CLI policy rules
│   └── tool-safety.toml             # Tool restriction policies
├── agents/
│   ├── symbi-governor.md            # Governance-aware agent
│   └── symbi-dev.md                 # DSL development agent
├── skills/
│   ├── symbi-init/
│   │   └── SKILL.md                 # Scaffold a governed agent project
│   ├── symbi-policy/
│   │   └── SKILL.md                 # Create/edit Cedar policies
│   ├── symbi-verify/
│   │   └── SKILL.md                 # SchemaPin verify MCP tools
│   ├── symbi-audit/
│   │   └── SKILL.md                 # Query audit logs
│   ├── symbi-dsl/
│   │   └── SKILL.md                 # Parse/validate DSL files
│   └── symbi-agent-sdk/
│       └── SKILL.md                 # Gemini CLI + ORGA governance boilerplate
├── commands/
│   └── symbi/
│       ├── status.toml              # /symbi:status -- runtime health check
│       ├── init.toml                # /symbi:init -- quick project scaffold
│       └── verify.toml              # /symbi:verify -- verify MCP tools
├── examples/
│   ├── standalone/                  # Mode A: extension-only setup
│   ├── cli-executor/                # Mode B: ORGA-wrapped Gemini CLI
│   └── agent-sdk/                   # Headless agent pattern
├── README.md
├── LICENSE                          # Apache 2.0
├── CHANGELOG.md
├── ROADMAP.md                       # This file
└── install.sh                       # Optional: install symbi binary
```

---

## Gemini CLI Extension Conventions

Key conventions for Gemini CLI extensions:

| Aspect | Convention |
|--------|-----------|
| Manifest | `gemini-extension.json` at repo root |
| Context file | `GEMINI.md` (set via `contextFileName` in manifest) |
| MCP config | Inline `mcpServers` field in manifest |
| Commands | `commands/<namespace>/<name>.toml` with `[command]` and `[prompt]` sections |
| Path variable | `${extensionPath}` in hook configs |
| Skills | `skills/*/SKILL.md` (requires `experimental.skills`) |
| Agents | `agents/*.md` (preview feature) |
| Hooks | `hooks/hooks.json`, shell scripts receiving JSON on stdin |
| Native policies | `policies/*.toml` with `[[rules]]` blocks |
| Tool restriction | `excludeTools` deny list in manifest |
| Tool names | `read_file`, `write_file`, `run_shell_command`, `replace`, `glob`, etc. |
| MCP tool prefix | `<server>__<tool>` (e.g., `symbi__invoke_agent`) |
| Install | `gemini extensions install <url>` |
| Dev mode | `gemini extensions link ./path` |

---

## Implementation Tasks

### Phase 1: Manifest and MCP Server Integration

**Goal**: Get the extension installable and the MCP server connected.

#### Task 1.1: Create extension manifest

Create `gemini-extension.json` with:
- Extension name, version, description
- `contextFileName` pointing to `GEMINI.md`
- Inline `mcpServers` config for `symbi mcp`

Notes:
- MCP servers are defined inline in the manifest, not in a separate file
- `${extensionPath}` is available but not needed for the MCP command since `symbi` is expected on PATH
- No `excludeTools` initially -- all symbi MCP tools should be available
- The `name` field must be lowercase with dashes only

#### Task 1.2: Create GEMINI.md context file

Gemini CLI loads this automatically when the extension is active. Contains:
- Key concepts (ORGA, Cedar, SchemaPin, AgentPin, DSL)
- Available MCP tools with parameter documentation
- File conventions
- Dual-mode operation explanation
- Governance behavior instructions

---

### Phase 2: Skills

**Goal**: Provide agent skills that showcase Symbiont's capabilities.

Note: Gemini CLI skills are gated behind `experimental.skills`. The GEMINI.md context and commands provide fallback functionality when skills aren't enabled.

#### Task 2.1: /symbi-init skill

Create `skills/symbi-init/SKILL.md`:
1. Check for existing `symbiont.toml`
2. Create `agents/` and `policies/` directories
3. Generate `symbiont.toml` with sensible defaults
4. Create a starter agent at `agents/assistant.dsl`
5. Create a starter Cedar policy at `policies/default.cedar`
6. Generate `AGENTS.md` manifest
7. Report what was created

#### Task 2.2: /symbi-policy skill

Create `skills/symbi-policy/SKILL.md`:
- Cedar policy management -- creation, editing, validation
- Common policy patterns (role-based, resource-scoped, approval-gated, etc.)

#### Task 2.3: /symbi-verify skill

Create `skills/symbi-verify/SKILL.md`:
- SchemaPin verification workflow
- MCP tool name format: `symbi__verify_schema` (server alias + double underscore)

#### Task 2.4: /symbi-audit skill

Create `skills/symbi-audit/SKILL.md`:
- Audit log querying and analysis
- Security event investigation

#### Task 2.5: /symbi-dsl skill

Create `skills/symbi-dsl/SKILL.md`:
- DSL parsing, validation, and creation
- `allowed-tools` references use Gemini CLI MCP tool naming: `symbi__parse_dsl`, `symbi__get_agent_dsl`, `symbi__list_agents`

#### Task 2.6: /symbi-agent-sdk skill

Create `skills/symbi-agent-sdk/SKILL.md`:
- Generate boilerplate for Gemini CLI agents governed by ORGA
- Document `gemini_cli` executor type in DSL
- Document CliExecutor environment variables

---

### Phase 3: Commands (TOML format)

**Goal**: Provide slash commands using Gemini CLI's TOML format.

#### Task 3.1: /symbi:status command

Create `commands/symbi/status.toml`:
- Check symbi binary, MCP server, project config, agents, policies
- Provide install instructions if symbi is missing

#### Task 3.2: /symbi:init command

Create `commands/symbi/init.toml`:
- Quick project scaffold (fallback when skills aren't enabled)
- Create symbiont.toml, starter agent, default Cedar policy, AGENTS.md

#### Task 3.3: /symbi:verify command

Create `commands/symbi/verify.toml`:
- SchemaPin verification via command
- Accepts `{{args}}` for specifying which server to verify

---

### Phase 4: Hooks

**Goal**: Enforce policies and maintain audit trails via Gemini CLI's hook system.

#### Task 4.1: Create hook scripts

All scripts support dual-mode operation:
- Mode A (standalone): Extension handles its own policy checks and audit logging
- Mode B (SYMBIONT_MANAGED): Defers to outer ORGA Gate, skips redundant logging

Scripts:
- `scripts/policy-log.sh` -- PreToolUse advisory policy logging
- `scripts/audit-log.sh` -- PostToolUse audit logging to `.symbiont/audit/`
- `scripts/install-check.sh` -- Verify symbi binary availability
- `scripts/mcp-wrapper.sh` -- MCP transport switching (stdio vs HTTP)

Tool name matchers use Gemini CLI names:
- Read-only (skipped): `read_file`, `list_directory`, `glob`, `search_file_content`, `google_web_search`, `web_fetch`
- State-modifying (checked): `write_file`, `replace`, `run_shell_command`

#### Task 4.2: Hook configuration

Create `hooks/hooks.json`:
- Uses `${extensionPath}` for script paths
- Matchers: `write_file|replace|run_shell_command|symbi__*`
- MCP tool prefix pattern: `symbi__*`

---

### Phase 5: Native Gemini CLI Policies

**Goal**: Leverage Gemini CLI's built-in policy engine for governance rules that work even without the symbi binary.

#### Task 5.1: Governance policy

Create `policies/symbi-governance.toml`:
- Warn on unverified MCP tool usage
- Warn on shell command execution (for audit awareness)

#### Task 5.2: Tool safety policy

Create `policies/tool-safety.toml`:
- Block recursive force deletion (`rm -rf`)
- Block force push to branches (`git push --force`)

---

### Phase 6: Agents (Preview Feature)

**Goal**: Define sub-agents for governance-aware development.

Agents in Gemini CLI are a preview feature. Agent definitions are system prompts with behavioral instructions -- no `allowed-tools` or `model` frontmatter. Tool restrictions are handled at the extension level via `excludeTools` or via policies.

#### Task 6.1: Governance agent

Create `agents/symbi-governor.md`:
- Policy awareness, tool verification, audit trail, least privilege
- Escalation rules for policy violations

#### Task 6.2: DSL development agent

Create `agents/symbi-dev.md`:
- DSL syntax, Cedar policies, trust configuration, testing
- Best practices enforcement

---

### Phase 7: Documentation, Examples, and Distribution

#### Task 7.1: README.md

Comprehensive README covering:
- Extension pitch and prerequisites
- Installation via `gemini extensions install`
- Development setup via `gemini extensions link`
- Quick start guide
- Available components (skills, commands, hooks, policies, agents)
- Note that skills require `experimental.skills`
- Dual-mode architecture (Mode A standalone, Mode B ORGA-managed)
- Gemini CLI-exclusive features
- Configuration and file conventions

#### Task 7.2: Examples

Create `examples/` directory with:
- `examples/standalone/` -- Extension-only setup for individual developers
- `examples/cli-executor/` -- DSL + config for running Gemini CLI wrapped in ORGA
- `examples/agent-sdk/` -- Headless agent pattern for programmatic use

#### Task 7.3: CHANGELOG.md

Initialize with v0.2.0 entry listing all components.

#### Task 7.4: LICENSE

Apache 2.0 with copyright: Jascha Wanger / ThirdKey AI.

#### Task 7.5: install.sh

Convenience script to install `symbi` binary via cargo or Docker.

---

## Implementation Order

1. **Phase 1**: Extension manifest (`gemini-extension.json`), context file (`GEMINI.md`)
2. **Phase 2**: Skills (6 SKILL.md files)
3. **Phase 3**: Commands (3 TOML files)
4. **Phase 4**: Hook scripts and `hooks/hooks.json`
5. **Phase 5**: Native Gemini CLI policies (Gemini-exclusive feature)
6. **Phase 6**: Agent definitions
7. **Phase 7**: README, examples, CHANGELOG, LICENSE, install.sh

## Testing Checklist

- [ ] Extension installs: `gemini extensions install ./` from repo root
- [ ] Extension links for dev: `gemini extensions link .`
- [ ] `/extensions list` shows symbi extension as enabled
- [ ] `/mcp list` shows symbi server connected (requires symbi binary on PATH)
- [ ] `/symbi:status` command works
- [ ] `/symbi:init` command scaffolds project
- [ ] `/symbi:verify` command triggers verification
- [ ] Skills activate when enabled via `experimental.skills` setting
- [ ] Hooks fire on `write_file`, `replace`, `run_shell_command` (check `.symbiont/audit/`)
- [ ] Native policies in `policies/` are loaded (check with `/extensions list` detail)
- [ ] All scripts are executable (`chmod +x scripts/*.sh`)
- [ ] Extension works without symbi binary (graceful degradation -- hooks warn, skills still guide)
- [ ] Hook scripts detect `SYMBIONT_MANAGED=true` and adjust behavior
- [ ] MCP wrapper script switches transport based on `SYMBIONT_MCP_URL`
- [ ] Example DSL files parse without errors

## Dependencies on Symbiont Monorepo

This extension depends on the `symbi` binary being available. The MCP server is built from the Symbiont monorepo and exposes these tools:

- `invoke_agent` (InvokeAgentParams: agent, prompt, system_prompt?)
- `list_agents` (no params)
- `parse_dsl` (ParseDslParams: file?, content?)
- `get_agent_dsl` (GetAgentDslParams: agent)
- `get_agents_md` (no params)
- `verify_schema` (VerifySchemaParams: schema, public_key_url)

If the symbi binary is not installed, the extension degrades gracefully -- skills and agents still provide guidance, native policies still enforce, hooks still warn, but MCP tools are unavailable.

## CliExecutor Bridge

When Symbiont's CliExecutor wraps Gemini CLI:

```bash
# CliExecutor spawns Gemini CLI with the extension loaded
SYMBIONT_MANAGED=true SYMBIONT_MCP_URL=http://localhost:PORT \
  gemini --prompt "Implement feature #42"
```

- Hook scripts detect `SYMBIONT_MANAGED` and defer to outer ORGA Gate
- Audit logs write to the parent runtime's journal instead of local JSONL
- MCP wrapper connects to parent runtime instead of spawning new server

## Gemini CLI-Exclusive Capabilities

Features available in Gemini CLI that other platforms don't support:

1. **Native policy engine**: `policies/*.toml` files provide governance even without the symbi binary installed
2. **`excludeTools` in manifest**: Declaratively block dangerous tools at the extension level without hook scripts
3. **Pattern-specific tool exclusion**: e.g., `run_shell_command(rm -rf)` blocking directly in the manifest
4. **Settings with secure storage**: Extension settings can use `"sensitive": true` to store API keys in the system keychain, useful for SchemaPin key management
5. **Planning directory**: Extensions can set a `directory` for planning artifacts -- could map to Symbiont's durable journal output

## Future Enhancements

- SchemaPin PreToolUse enforcement via native policies
- Cedar policy simulation via `symbi` binary integration
- `excludeTools` manifest entries for known-dangerous patterns
- Secure settings for SchemaPin key management
- Planning directory integration with Symbiont journal
- Gallery submission to `geminicli.com/extensions`
