---
name: codebase-memory-mcp
description: Use PROACTIVELY when navigating unfamiliar repositories, when debugging cross-file call chains, when testing impacted modules, or for refactoring high-fan-in hotspots with DeusData/codebase-memory-mcp; automatically invoke before reading multiple files sequentially.
---

# Codebase Memory MCP (`DeusData/codebase-memory-mcp`)

Query a persistent local knowledge graph of AST symbols, static call chains, service routes, and test coverage edges across 158 programming languages through Model Context Protocol (MCP) tools instead of reading dozens of files sequentially.

## When to Use & Decision Matrix

Select the right graph query operation based on your engineering task and comparison tradeoff:

| Engineering Task | Graph Query Operation | Token Cost | Comparison vs Sequential File Reads |
| :--- | :--- | :--- | :--- |
| Locate symbol definition & signature | `cbm_find_symbol` | ~120 tokens | Returns exact file, line span, and type signature in 1 call |
| Check blast radius before signature change | `cbm_get_callers` (fan-in) | ~200 tokens | Lists every caller across the repo; prevents missed call sites |
| Trace request flow from route to DB | `cbm_trace_call_path` | ~250 tokens | Computes shortest static call chain with line-level evidence |
| Identify circular package imports | `cbm_detect_cycles` | ~180 tokens | Finds strongly connected module components automatically |
| Run only tests impacted by a diff | `cbm_impacted_tests` | ~140 tokens | Traverses reverse call edges to select exact test entry points |

## Input Parameters & Output Format

The `codebase-memory-mcp` server accepts typed graph query input parameters and returns structured JSON output format payloads:

- **Input parameters (`cbm_find_symbol` / `cbm_get_callers` / `cbm_get_callees`)**:
  - `symbol` (`string`, required): Symbol name or qualified identifier (e.g., `AuthService.verify_token`).
  - `kind` (`string`, optional): Filter by node kind (`function`, `class`, `interface`, `method`, `route`).
  - `depth` (`integer`, optional): Traversal hop count from `1` to `5` (defaults to `1`).
- **Input parameters (`cbm_trace_call_path`)**:
  - `source_symbol` (`string`, required): Entry-point symbol or HTTP route handler.
  - `target_symbol` (`string`, required): Downstream sink symbol or database method.
- **Output format**:
  - Returns a JSON object containing `nodes` (symbol metadata, file path, `start_line`, `end_line`), `edges` (`CALLS`, `IMPLEMENTS`, `COVERS_SYMBOL`), `fan_in_count`, and `impacted_tests`.

```yaml
# .cbm/config.yaml — Workspace graph indexing configuration
workspace:
  index_path: ".cbm/graph.db"
  watch_filesystem: true
  exclude_globs:
    - "**/node_modules/**"
    - "**/.venv/**"
    - "**/dist/**"
analysis:
  max_call_depth: 5
  hotspot_fan_in_threshold: 15
```

## Setup & Indexing Workflow

Build or launch `codebase-memory-mcp` (`https://github.com/DeusData/codebase-memory-mcp`) and index the workspace:

```bash
# Clone and build with the CBM Lens visual graph UI
git clone https://github.com/DeusData/codebase-memory-mcp.git
cd codebase-memory-mcp && ./scripts/build.sh --with-ui

# Start the MCP server and local CBM Lens UI on port 9749
codebase-memory-mcp --workspace . --ui=true --port=9749
```

## Core Workflow: Graph First, Targeted Slice Second

Follow this four-step discipline to navigate codebases with minimal token consumption:

### 1. Symbol & Fan-In Lookup Before Editing

Query the symbol node and its incoming callers before modifying any shared function or class signature:

```json
{
  "tool": "cbm_get_callers",
  "parameters": {
    "symbol": "validate_webhook_signature",
    "kind": "function",
    "depth": 2
  }
}
```

### 2. Static Call-Path Tracing Across Layers

Trace how an HTTP route or CLI command reaches a downstream storage or external API layer:

```json
{
  "tool": "cbm_trace_call_path",
  "parameters": {
    "source_symbol": "POST /api/v1/orders",
    "target_symbol": "OrderRepository.insert_order",
    "max_hops": 6
  }
}
```

### 3. Change-Aware Test Selection

Compute the reverse reachability cone from modified symbols to test cases so you can run impacted unit tests immediately:

```python
import json
import subprocess

# Parse impacted test paths returned by codebase-memory-mcp
cbm_output = '{"impacted_tests": ["tests/test_auth.py::test_token_expiry", "tests/test_orders.py::test_create_order"]}'
tests = json.loads(cbm_output)["impacted_tests"]
if tests:
    subprocess.run(["pytest", "-q", *tests], check=True)
```

## Examples

### Example 1: Auditing a High-Fan-In Hotspot Before Refactoring

1. Query `cbm_get_callers` for `parse_config` with `depth: 1`.
2. Observe `fan_in_count: 24` across 11 modules (`Risk Tier: High Hotspot`).
3. Keep the existing `parse_config(path: str)` signature intact and introduce an optional keyword parameter `strict: bool = False` so all 24 existing callers continue working without edits.

### Example 2: Isolating a Circular Import Between Two Packages

1. Invoke `cbm_detect_cycles` on `src/services` and `src/models`:
   ```json
   {
     "tool": "cbm_detect_cycles",
     "parameters": {
       "scope": "src/",
       "min_cycle_length": 2
     }
   }
   ```
2. Inspect the returned cycle edge (`src/services/billing.py:14 -> src/models/invoice.py:9 -> src/services/billing.py:3`).
3. Extract shared protocol types into `src/models/types.py` and re-run `cbm_detect_cycles` to confirm zero remaining cycles.

## Troubleshooting

- **Symptom: Newly created functions or routes do not appear in `cbm_find_symbol` results**
  - **Cause**: File watcher daemon is disabled or incremental Tree-sitter re-parse has not flushed.
  - **Fix**: Trigger an explicit incremental re-index via `cbm_reindex_paths` or pass `--watch=true` on server startup.
- **Symptom: `cbm_trace_call_path` returns an empty path for dynamic dependency injection**
  - **Cause**: Runtime container bindings or reflection-based calls bypass static direct call edges.
  - **Fix**: Query `cbm_find_symbol` with `kind: "interface"` and expand `IMPLEMENTS` edges to bridge interface-to-implementation hops.
- **Symptom: Indexing takes longer than 30 seconds on monorepos**
  - **Cause**: Generated build artifacts (`dist/`, `.next/`, `target/`, `.venv/`) are being parsed.
  - **Fix**: Add generated directories to `exclude_globs` in `.cbm/config.yaml`.

## See Also

- Node/edge schema tables, MCP configuration snippets, and fan-in risk thresholds live in `references/details.md`.
- Companion skills in this plugin include `skills/context-mode` for sandboxed test output compression and `skills/understand-anything` for semantic architecture delta scans.
- Related repository mapping workflows pair with `scan` for generating `AGENTS.md` context maps.
