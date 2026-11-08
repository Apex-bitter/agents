---
name: context-mode
description: Use when tool outputs, build logs, test suites, or browser snapshots exceed a few kilobytes and risk bloating the context window, or when configuring mksglu/context-mode sandbox execution and SQLite FTS5/BM25 compaction recovery.
---

# Context Mode (`mksglu/context-mode`)

Keep large tool outputs, logs, and DOM snapshots out of the active context window by executing noisy commands inside a local sandbox and querying a per-project SQLite FTS5/BM25 index (`~/.context-mode/context.db`).

## When to Use

- Running commands that emit multi-kilobyte outputs (`pytest`, `npm test`, `git log -p`, build logs, API payloads, or Playwright snapshots).
- Recovering session continuity and prior tool results after context compaction.
- Applying the **Thinking in Code** pattern: writing a compact filter script in the sandbox so only the final extracted answer enters the context window.

## Installation & Harness Setup

Install the binary and register the plugin/MCP server (`https://github.com/mksglu/context-mode`):

```bash
# Install the local CLI and MCP server binary
npm install -g context-mode

# Claude Code marketplace install
/plugin marketplace add mksglu/context-mode
/plugin install context-mode@context-mode

# Antigravity CLI install
agy plugin install https://github.com/mksglu/context-mode
```

## Core Workflow: Sandbox First, Query Second

Never dump raw multi-kilobyte outputs into the conversation. Follow this three-step loop:

### 1. Route Noisy Execution Through the Sandbox

When `context-mode` MCP tools are available (`ctx_execute`, `ctx_execute_file`, `ctx_batch_execute`), run high-volume commands or file inspections through them instead of printing raw stdout:

```json
{
  "tool": "ctx_execute",
  "command": "pytest -v --tb=short",
  "intent": "Identify failing test names and assertion lines only"
}
```

Full stdout/stderr is stored in `~/.context-mode/context.db` for the current project while the reply returns only a compact summary and index handle.

### 2. Search Indexed Output with FTS5 / BM25

Instead of re-running a slow command or scrolling through thousands of lines, query the indexed output with `ctx_search`:

```json
{
  "tool": "ctx_search",
  "query": "AssertionError OR FAILED",
  "limit": 10
}
```

### 3. Thinking in Code (Scripted Reduction)

When inspecting large JSON payloads, CSV exports, lockfiles, or logs, write a one-liner or short script that computes the exact slice needed and prints under 50 lines:

```bash
# Instead of dumping a 200 KB JSON file into context:
python3 -c '
import json, sys
data = json.load(open("report.json"))
failures = [r["id"] for r in data["results"] if r["status"] == "failed"]
print(f"Failed ({len(failures)}):", failures[:20])
'
```

## Compaction Recovery Checklist

When a session compacts or resumes after a long run:

1. Query `ctx_search` for recent decisions, failing test names, or file paths before re-running expensive builds.
2. Bound every fallback shell command with `head`, `tail`, `--limit`, or `--jq` when `context-mode` is not installed.
3. Keep replies under 1 KB per tool turn whenever possible to extend usable session duration from ~30 minutes to 3+ hours.
