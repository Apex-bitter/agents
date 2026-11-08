# Context Mode Reference Details (`mksglu/context-mode`)

## SQLite FTS5 & BM25 Indexing Architecture

`context-mode` stores per-project command outputs, file extracts, and session snapshots inside `~/.context-mode/context.db` using SQLite FTS5 virtual tables ranked with BM25 relevance weights.

### Schema Overview

```sql
CREATE VIRTUAL TABLE IF NOT EXISTS context_chunks USING fts5(
  project_id UNINDEXED,
  session_id UNINDEXED,
  source_command,
  heading_path,
  content,
  created_at UNINDEXED,
  tokenize = 'porter unicode61'
);
```

### Advanced FTS5 Query Syntax

| Operator | Example Query | Behavior |
| :--- | :--- | :--- |
| Boolean `OR` | `AssertionError OR TypeError` | Matches chunks containing either token |
| Phrase match | `"connection refused"` | Matches exact consecutive token sequence |
| Prefix match | ` migrate* ` | Matches `migrate`, `migration`, `migrated` |
| Column filter | `source_command:pytest FAILED` | Restricts match to chunks from `pytest` runs |
| `NEAR` proximity | `NEAR(timeout retry, 5)` | Matches tokens within 5 words of each other |

## Multi-Harness MCP Server Configuration

Add `context-mode` to your harness MCP configuration when the marketplace plugin installer is unavailable:

```json
{
  "mcpServers": {
    "context-mode": {
      "command": "npx",
      "args": ["-y", "context-mode", "mcp"],
      "env": {
        "CONTEXT_MODE_DB_PATH": "~/.context-mode/context.db",
        "CONTEXT_MODE_MAX_INLINE_BYTES": "1024"
      }
    }
  }
}
```

## Batch Execution Recipes

Use `ctx_batch_execute` to run multiple diagnostic commands in a single round-trip while keeping all verbose output indexed:

```json
{
  "tool": "ctx_batch_execute",
  "commands": [
    {
      "id": "git-state",
      "command": "git status -sb && git log -n 5 --oneline"
    },
    {
      "id": "test-suite",
      "command": "pytest -q --tb=short"
    },
    {
      "id": "typecheck",
      "command": "uv run ty check"
    }
  ],
  "intent": "Verify working tree state, failing tests, and type errors"
}
```
