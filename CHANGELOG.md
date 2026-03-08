# Changelog

## [0.2.0] - 2026-03-07

### Added

- Extension manifest (`gemini-extension.json`) with inline MCP server config
- `GEMINI.md` context file with dual-mode documentation
- Skills:
  - `/symbi-init` -- scaffold a governed agent project
  - `/symbi-policy` -- create/edit Cedar authorization policies
  - `/symbi-verify` -- SchemaPin MCP tool verification
  - `/symbi-audit` -- query cryptographic audit logs
  - `/symbi-dsl` -- parse/validate DSL agent definitions
  - `/symbi-agent-sdk` -- Gemini CLI + ORGA governance boilerplate
- Commands (TOML format):
  - `/symbi:status` -- runtime health check
  - `/symbi:init` -- quick project scaffold
  - `/symbi:verify` -- verify MCP tools
- Hooks:
  - `policy-log.sh` -- PreToolUse advisory policy logging (dual-mode aware)
  - `audit-log.sh` -- PostToolUse audit logging (dual-mode aware)
  - `install-check.sh` -- symbi binary verification (dual-mode aware)
  - `mcp-wrapper.sh` -- MCP transport switching (stdio/HTTP)
- Native Gemini CLI policies:
  - `symbi-governance.toml` -- MCP tool and shell command warnings
  - `tool-safety.toml` -- dangerous pattern blocking (rm -rf, force push)
- Agents (preview):
  - `symbi-governor` -- governance-aware coding agent
  - `symbi-dev` -- DSL development specialist
- Dual-mode architecture: Mode A (standalone) and Mode B (ORGA-managed)
- Examples: standalone, cli-executor, agent-sdk
- Documentation: README, ROADMAP, CHANGELOG
- Install script for symbi binary
