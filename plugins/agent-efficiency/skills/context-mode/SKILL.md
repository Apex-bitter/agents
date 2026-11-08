---
name: context-mode
description: Use PROACTIVELY when running noisy CLI commands, when testing large suites, when debugging build logs, or for configuring mksglu/context-mode sandbox execution and SQLite FTS5/BM25 compaction recovery; automatically invoke before multi-kilobyte outputs enter the context window.
---

# Context Mode (`mksglu/context-mode`)

Keep multi-kilobyte tool outputs, build traces, API responses, and DOM snapshots out of the active context window by executing high-volume commands inside a local sandbox and querying a per-project SQLite FTS5/BM25 index (`~/.context-mode/context.db`).

## When to Use & Decision Matrix

Compare execution paths before running any shell command or file inspection that could emit more than 1 KB of text:

| Scenario | Recommended Tool / Pattern | Inline Token Cost | Tradeoff & Comparison |
| :--- | :--- | :--- | :--- |
| Full test suite (`pytest`, `vitest`, `cargo test`) | `ctx_execute` with failure intent | ~150 tokens | Stores full stdout in SQLite; returns only failing assertion summaries |
| Multi-step diagnostics (git + lint + typecheck) | `ctx_batch_execute` | ~250 tokens | Runs commands in one turn; avoids sequential round-trip latency |
| Inspecting `> 50 KB` JSON, CSV, or lockfiles | `ctx_execute_file` or Python reduction | ~100 tokens | Extracts exact keys in sandbox without loading raw file into context |
| Searching prior command logs or session state | `ctx_search` (FTS5 / BM25) | ~120 tokens | Instant keyword/phrase recall without re-running slow builds |
| Quick `< 20` line status check (`git status -sb`) | Direct shell with `--limit` / `head` | ~80 tokens | Lowest overhead when output is guaranteed to be tiny |

## Input Parameters & Output Format

The `context-mode` MCP server accepts structured input parameters and returns compact, indexed output format payloads:

- **Input parameters (`ctx_execute` / `ctx_batch_execute`)**:
  - `command` (`string`, required): Shell command string to execute inside the isolated project sandbox.
  - `intent` (`string`, required): Natural-language filter goal describing which lines or metrics to extract.
  - `timeout_ms` (`integer`, optional): Execution timeout in milliseconds (defaults to `60000`).
- **Input parameters (`ctx_search`)**:
  - `query` (`string`, required): SQLite FTS5 query string supporting `AND`, `OR`, `NEAR`, prefix `*`, and column filters.
  - `limit` (`integer`, optional): Maximum ranked BM25 snippets to return (defaults to `10`).
- **Output format**:
  - Returns a JSON object containing `chunk_id`, `exit_code`, `byte_count`, `summary` (under `1,024` bytes), and `top_matches` (ranked snippets with line ranges).

```yaml
# ~/.context-mode/config.yaml — Sandbox & BM25 retention policy
sandbox:
  max_inline_bytes: 1024
  default_timeout_ms: 60000
  strip_ansi_codes: true
storage:
  db_path: "~/.context-mode/context.db"
  fts_tokenizer: "porter unicode61"
  retention_days: 14
```

## Installation & Harness Setup

Install the binary and register the MCP server (`https://github.com/mksglu/context-mode`):

```bash
# Install the local CLI and MCP server binary globally
npm install -g context-mode

# Register in Claude Code via marketplace
/plugin marketplace add mksglu/context-mode
/plugin install context-mode@context-mode

# Register in Antigravity CLI
agy plugin install https://github.com/mksglu/context-mode
```

## Core Workflow: Sandbox First, Query Second

Follow this three-stage loop so raw multi-kilobyte logs never pollute conversation memory:

### 1. Route Noisy Execution Through the Sandbox

Execute test suites, compilers, or verbose CLI queries via `ctx_execute` with a specific `intent`:

```json
{
  "tool": "ctx_execute",
  "parameters": {
    "command": "pytest -v --tb=short",
    "intent": "Return only failing test identifiers, file paths, and AssertionError lines",
    "timeout_ms": 45000
  }
}
```

### 2. Query Indexed Output with SQLite FTS5 / BM25

Instead of re-running a build or reading thousands of log lines, search `~/.context-mode/context.db` using `ctx_search`:

```json
{
  "tool": "ctx_search",
  "parameters": {
    "query": "source_command:pytest (AssertionError OR TypeError)",
    "limit": 8
  }
}
```

### 3. Thinking in Code (Scripted Reduction)

When analyzing large JSON reports, API dumps, or CSV exports without the MCP server active, run a compact Python filter script so only the computed answer prints to stdout:

```python
import json
from pathlib import Path

payload = json.loads(Path("coverage-summary.json").read_text(encoding="utf-8"))
low_coverage = [
    {"file": path, "pct": metrics["lines"]["pct"]}
    for path, metrics in payload.items()
    if path != "total" and metrics["lines"]["pct"] < 80.0
]
low_coverage.sort(key=lambda item: item["pct"])
print(json.dumps({"count": len(low_coverage), "lowest": low_coverage[:10]}, indent=2))
```

## Examples

### Example 1: Inspecting a 5 MB GitHub Actions Log

1. Fetch and index the log inside the sandbox using `ctx_execute`:
   ```bash
   # Fallback bounded CLI pattern when running directly in shell
   gh run view 123456789 --log-failed | tail -n 40
   ```
2. Query the exact failure root cause via `ctx_search`:
   ```json
   {
     "tool": "ctx_search",
     "parameters": {
       "query": "NEAR(ERROR exit, 6) OR \"ModuleNotFoundError\"",
       "limit": 5
     }
   }
   ```

### Example 2: Post-Compaction Session Recovery

When a coding session resumes after context compaction, recover prior verification results in one query instead of re-running the test suite:

1. Call `ctx_search` with `query: "FAILED OR passed OR commit"` and `limit: 5`.
2. Read the returned `chunk_id` summary to verify which test files were already green before compaction occurred.

## Troubleshooting

- **Symptom: `ctx_search` returns zero matches for known output tokens**
  - **Cause**: Special punctuation (such as `-`, `:`, or `/`) in raw file paths was interpreted as an FTS5 column operator.
  - **Fix**: Wrap file paths or hyphenated identifiers in escaped double quotes (`"\"src/api/auth.py\""`).
- **Symptom: Sandbox command times out on interactive prompts**
  - **Cause**: CLI tools waiting on stdin hang non-interactive sandbox workers.
  - **Fix**: Pass non-interactive flags (`--yes`, `--no-input`, `CI=true`, `PAGER=cat`) in the `command` parameter.
- **Symptom: SQLite database lock contention (`SQLITE_BUSY`) during parallel runs**
  - **Cause**: Multiple subagents writing to `~/.context-mode/context.db` simultaneously without WAL mode enabled.
  - **Fix**: Run `sqlite3 ~/.context-mode/context.db "PRAGMA journal_mode=WAL;"` once to enable concurrent readers and writers.

## See Also

- Deep FTS5 query syntax, schema definitions, and multi-harness MCP config live in `references/details.md`.
- Companion skills in this plugin include `skills/codebase-memory-mcp` for structural AST/call-graph queries and `skills/understand-anything` for semantic delta scans.
- Related memory and session-continuity workflows pair cleanly with `superself` and `deja-vu`.
