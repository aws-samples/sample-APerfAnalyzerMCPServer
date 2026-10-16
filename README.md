# aperf-analyzer-mcp: Deploy Package

An MCP server that analyzes [aperf](https://github.com/aws/aperf) performance captures. It
gives your AI assistant run summaries and comparisons, hotspots, regression detection,
EC2 right-sizing and Graviton readiness backed by benchmark data, and customer-ready HTML
reports.

This package holds only what is needed to run the server: the wheel, a hash-pinned
`requirements.txt`, and MCP client config templates.

## Requirements

- Python 3.10 or newer
- aperf capture directories (or `.tar.gz` archives) on the same machine
- An MCP client: Claude Code, Kiro, Claude Desktop, or any stdio MCP client

## Install

Get the files, either way works:

- clone https://github.com/aws-samples/sample-APerfAnalyzerMCPServer, or
- download `aperf-analyzer-mcp-<version>.tar.gz` from that repository's Releases page and
  extract it (`tar -xzf aperf-analyzer-mcp-<version>.tar.gz`).

Then, from that directory:

```bash
python3 -m venv .venv
source .venv/bin/activate

# Exact, hash-verified dependency versions, then the server itself.
pip install --require-hashes -r requirements.txt
pip install --no-deps ./aperf_analyzer_mcp-*.whl
```

Check that it loads:

```bash
python -c "from src.server import TOOLS; print(len(TOOLS), 'tools')"
```

## Configure

All configuration is environment variables, set in your MCP client's config (below):

| Variable | Default | Purpose |
|---|---|---|
| `APERF_DATA_DIR` | `./aperf_data` | Directory holding your aperf runs. |
| `APERF_MAX_PROJECTED_UTILIZATION_PCT` | `50` | Right-sizing CPU ceiling. A candidate instance is only proposed if the workload's observed peak, projected onto it, stays under this percentage. Lower is more conservative. |
| `APERF_APPROVED_PROJECT_DIRS` | unset | Optional. Colon-separated absolute paths where `improve_app` may run your test suite. Unset means it never runs tests. |
| `PLAYBOOK_DISABLE` | unset | Optional. Comma-separated playbook rule IDs to suppress. |

## Connect your MCP client

Replace `/path/to/...` with absolute paths: the directory you installed into above, and your
aperf data directory.

**Claude Code**
```bash
claude mcp add aperf-analyzer \
  -e APERF_DATA_DIR=/path/to/aperf/runs \
  -e APERF_MAX_PROJECTED_UTILIZATION_PCT=50 \
  -- /path/to/install-dir/.venv/bin/python -m src.server
```

**Kiro** (`~/.kiro/settings/mcp.json`) and **Claude Desktop**
(`~/.config/claude/claude_desktop_config.json`): merge `mcp-config-kiro.json` or
`mcp-config-claude-desktop.json` from this package into the client's config, then set
`command` and the `env` paths.

## Tools

| Tool | What it does |
|---|---|
| `list_runs` | List the aperf runs in `APERF_DATA_DIR` |
| `get_run_summary` | Performance summary for one run |
| `compare_runs` / `detect_regressions` | Side-by-side comparison and regression verdicts |
| `get_hotspots` / `get_profile_summary` | Top hotspots and profiling data |
| `get_system_health` / `diagnose_config_errors` | Health check and common misconfigurations |
| `get_language_advisory` | Language/runtime-specific advice (Java, Python, Go, ...) |
| `analyze_findings` / `sysctl_ai_fallback` | aperf's own findings with prescriptive fixes |
| `get_right_sizing` / `graviton_readiness` | EC2 right-sizing and Graviton (ARM64) path |
| `analyze_app_context` / `improve_app` | Dependency analysis and code-level improvement plan |
| `generate_html_report` | Customer-ready HTML report for a run |
| `pricing_status` | Bundled EC2 pricing freshness, with an optional refresh |

## Security notes

Full security model: `SECURITY.md` in this package.

- The server trusts its caller completely and has no authentication. Run it only as a stdio
  subprocess of your own MCP client; never expose it over a network transport.
- It speaks MCP over stdio only and opens no network listener.
- It makes one outbound request, and only when `pricing_status` is called with
  `confirm=true`: a fetch of the public AWS pricing index. It needs no credentials.
- It reads aperf data from `APERF_DATA_DIR`. Archives are extracted with path-traversal
  and size limits.
- `improve_app` runs tests only inside directories listed in `APERF_APPROVED_PROJECT_DIRS`,
  using a fixed `pytest` command with no shell.
- Right-sizing output is a starting point. Validate any instance change under your real
  workload before committing to it.

## License

MIT-0. See `LICENSE`.
