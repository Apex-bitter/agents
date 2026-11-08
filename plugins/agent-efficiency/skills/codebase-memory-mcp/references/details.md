# Codebase Memory MCP Reference Details (`DeusData/codebase-memory-mcp`)

## Graph Schema & Edge Types

`codebase-memory-mcp` builds a persistent property graph over 158 programming languages using Tree-sitter grammars and static symbol resolution.

| Node Kind | Key Properties | Typical Edge Relationships |
| :--- | :--- | :--- |
| `Symbol` | `id`, `name`, `kind`, `file`, `start_line`, `end_line`, `signature` | `CALLS`, `CALLED_BY`, `IMPLEMENTS`, `OVERRIDES`, `REFERENCES` |
| `Module` | `path`, `language`, `package`, `exports` | `IMPORTS`, `IMPORTED_BY`, `CONTAINS_SYMBOL` |
| `Route` | `method`, `path`, `framework`, `handler_symbol_id` | `HANDLED_BY`, `MIDDLEWARE_CHAIN` |
| `TestCase` | `name`, `file`, `framework`, `target_symbols` | `COVERS_SYMBOL`, `TRANSITIVELY_TESTS` |

## MCP Server Configuration

Register `codebase-memory-mcp` in your harness configuration file:

```json
{
  "mcpServers": {
    "codebase-memory": {
      "command": "codebase-memory-mcp",
      "args": ["--workspace", ".", "--ui=true", "--port=9749"],
      "env": {
        "CBM_INDEX_PATH": ".cbm/graph.db",
        "CBM_WATCH": "true"
      }
    }
  }
}
```

## Fan-In Gravity & Refactoring Risk Thresholds

Use fan-in and cyclic dependency metrics to classify symbol modification risk before editing shared code:

| Fan-In Count | Risk Tier | Required Safeguard |
| :--- | :--- | :--- |
| `0 - 3` callers | Low | Direct edit + unit test execution |
| `4 - 15` callers | Medium | Inspect all incoming call sites via `cbm_get_callers` |
| `16+` callers | High (Hotspot) | Preserve backward-compatible signature or add adapter shim |
| Cyclic group `> 1` | Architectural | Extract shared interface module to break import cycle first |
