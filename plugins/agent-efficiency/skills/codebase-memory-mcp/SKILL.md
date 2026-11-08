---
name: codebase-memory-mcp
description: Use when navigating unfamiliar repositories, tracing static call graphs, finding high-fan-in hotspots, mapping HTTP/RPC service routes, or selecting change-aware tests via DeusData/codebase-memory-mcp instead of reading dozens of files.
---

# Codebase Memory MCP (`DeusData/codebase-memory-mcp`)

Query a persistent local knowledge graph of code symbols, call chains, documentation, and service routes across 158 programming languages through Model Context Protocol (MCP) tools instead of brute-force file reads.

## When to Use

- Tracing callers and callees of a function, class, or interface across a repository.
- Locating architectural entry points, service routes, and cyclic dependencies.
- Identifying high-fan-in hotspot symbols before refactoring shared modules.
- Running change-aware test selection to verify only the tests impacted by a diff.

## Setup & Indexing

Build or install `codebase-memory-mcp` (`https://github.com/DeusData/codebase-memory-mcp`) and index the workspace:

```bash
# Clone and build with CBM Lens UI
git clone https://github.com/DeusData/codebase-memory-mcp.git
cd codebase-memory-mcp && ./scripts/build.sh --with-ui

# Or run the MCP server with the local visual graph on port 9749
codebase-memory-mcp --ui=true --port=9749
```

## High-Efficiency Query Patterns

Replace multi-file search-and-read loops with targeted graph queries:

### 1. Symbol & Relationship Lookup (Instead of Blind Grep)

1. Query the symbol index by name or kind (`function`, `class`, `interface`, `route`) to get exact file paths, line spans, and signatures.
2. Expand incoming edges (**callers / fan-in**) before changing a function signature or return type.
3. Expand outgoing edges (**callees / fan-out**) to understand side effects without opening intermediate pass-through files.

### 2. Static Call-Path Tracing

When debugging a request flow from an API controller down to a database query:

1. Specify the **source symbol** (e.g., route handler or CLI entry point) and **target symbol** (e.g., repository method or external client).
2. Request the static call path with source evidence statements.
3. Open only the specific line ranges along the returned path.

### 3. Hotspot & Cycle Inspection Before Refactoring

- **Hotspots (Fan-In Gravity)**: Check measured fan-in and complexity signals before modifying foundational utilities. High fan-in symbols require strict backward compatibility or staged deprecation.
- **Cycle Isolation**: Query component dependency groups to detect and break circular imports before splitting packages.

### 4. Change-Aware Test Selection

After editing a set of files or symbols:

1. Query reverse reachability from the modified symbols to test entry points.
2. Execute the impacted unit and integration tests first for sub-second feedback before running the full test suite.
