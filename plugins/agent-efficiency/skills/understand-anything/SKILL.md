---
name: understand-anything
description: Use PROACTIVELY when bootstrapping architectural documentation, when building semantic knowledge graphs of unfamiliar codebases, when configuring delta scans, or for updating project-doc.md and AGENTS.md with Lum1104/Understand-Anything; automatically invoke during repository onboarding.
---

# Understand Anything (`Lum1104/Understand-Anything`)

Generate and query a semantic knowledge graph of any repository to accelerate architectural onboarding, module boundary mapping, and incremental delta documentation updates without flooding the active context window.

## When to Use & Decision Matrix

Choose between a full semantic synthesis and an incremental delta scan based on repository state and tradeoff comparison:

| Repository State | Scan Mode | Token Budget | Tradeoff & Comparison |
| :--- | :--- | :--- | :--- |
| New repo without `.understand/graph.json` or `project-doc.md` | **Full Graph Synthesis** | ~1,200 tokens | Inspects manifests, entry points, and layers once to establish baseline |
| Routine feature or bugfix commit (`1 - 15` files changed) | **Incremental Delta Scan** | ~250 tokens | Analyzes only `git diff` files + 1-hop neighbors; skips unchanged modules |
| Dependency upgrade (`pyproject.toml`, `package.json`, `Cargo.toml`) | **Manifest Delta Scan** | ~300 tokens | Updates tech stack and runtime tables in `project-doc.md` |
| New top-level service, plugin, or auth middleware | **Structural Drift Scan** | ~450 tokens | Flags architectural drift and proposes `AGENTS.md` map diff for review |

## Input Parameters & Output Format

The `understand-anything` analyzer accepts repository scope input parameters and produces a structured semantic graph output format:

- **Input parameters**:
  - `repo_root` (`string`, required): Path to the target Git repository root.
  - `mode` (`string`, required): Either `"full"` (initial synthesis) or `"delta"` (diff-bounded scan).
  - `base_ref` (`string`, optional): Git commit SHA or branch ref for delta comparison (defaults to `HEAD~1`).
  - `output_files` (`array[string]`, optional): Target doc paths such as `["project-doc.md", "AGENTS.md"]`.
- **Output format**:
  - Produces `.understand/graph.json` (layered module nodes, dependency edges, and baseline commit SHA) and returns a JSON drift report containing `changed_modules`, `architectural_drift_detected` (`boolean`), and `proposed_doc_patches`.

```yaml
# .understand/config.yaml — Semantic scan & drift detection rules
scan:
  baseline_file: ".understand/graph.json"
  doc_targets:
    detailed_architecture: "project-doc.md"
    agent_context_map: "AGENTS.md"
  agents_md_max_lines: 150
drift_triggers:
  - "new_top_level_directory"
  - "manifest_dependency_change"
  - "auth_or_database_schema_change"
```

## Installation & Setup

Install `Understand-Anything` (`https://github.com/Lum1104/Understand-Anything`) and pair it with `context-mode` when available:

```bash
# Install via Claude Code plugin marketplace
/plugin marketplace add Lum1104/Understand-Anything
/plugin install understand-anything

# Verify baseline state in target repository
git status -sb && ls -la .understand/ 2>/dev/null || true
```

## Core Workflow: Baseline Once, Delta Scan Thereafter

### 1. Full Repository Graph Synthesis (First Run)

When onboarding onto a repository with no prior baseline:

1. Extract stack metadata from package manifests (`pyproject.toml`, `package.json`, `go.mod`, `Cargo.toml`) and CI workflows.
2. Map module layers (domain core, application services, HTTP/CLI adapters, data stores) and cross-cutting concerns.
3. Write detailed findings to `project-doc.md` and generate a concise (`< 150` lines) table-of-contents map in `AGENTS.md`:

```json
{
  "tool": "understand_scan",
  "parameters": {
    "repo_root": ".",
    "mode": "full",
    "output_files": ["project-doc.md", "AGENTS.md"]
  }
}
```

### 2. Incremental Delta Scan (Subsequent Runs)

On subsequent runs, compute the changed file set since the recorded baseline SHA and inspect only 1-hop graph neighbors:

```python
import subprocess

def compute_delta_files(base_ref: str = "HEAD~1") -> dict[str, list[str]]:
    raw = subprocess.check_output(
        ["git", "diff", "--name-status", base_ref, "HEAD"],
        text=True,
    ).splitlines()
    added_or_removed = [line.split("\t", 1)[1] for line in raw if line.startswith(("A", "D", "R"))]
    modified = [line.split("\t", 1)[1] for line in raw if line.startswith("M")]
    return {"structural_candidates": added_or_removed, "modified": modified}

print(compute_delta_files())
```

### 3. Guarded Documentation Patching

Apply documentation updates selectively:

```json
{
  "architectural_drift_detected": true,
  "baseline_sha": "6d659bb",
  "structural_changes": [
    "Added plugins/agent-efficiency/ with 6 cross-harness skills"
  ],
  "proposed_doc_patches": [
    {
      "target": "project-doc.md",
      "section": "## Module Architecture",
      "action": "append_module_summary"
    },
    {
      "target": "AGENTS.md",
      "section": "## Map",
      "requires_human_confirmation": true
    }
  ]
}
```

## Examples

### Example 1: Bootstrapping `project-doc.md` and `AGENTS.md` on a Python/FastAPI Repo

1. Run full synthesis (`mode: "full"`).
2. `Understand-Anything` detects FastAPI routers in `src/api/`, SQLAlchemy models in `src/db/`, and pytest fixtures in `tests/`.
3. Writes a 250-line deep reference to `project-doc.md` and a 95-line context map to `AGENTS.md`.

### Example 2: Delta Scan After Adding a New Plugin Directory

1. Run `git diff HEAD~1 --name-status` and detect new files under `plugins/agent-efficiency/`.
2. Classify the change as `new_top_level_directory` (`architectural_drift_detected: true`).
3. Update `project-doc.md` automatically and present the proposed 3-line `AGENTS.md` map addition for human confirmation before writing.

## Troubleshooting

- **Symptom: Delta scan flags every internal commit as architectural drift**
  - **Cause**: File-level modifications inside existing leaf modules are being treated as structural boundary changes.
  - **Fix**: Restrict `AGENTS.md` drift triggers in `.understand/config.yaml` to top-level directory additions, manifest dependency changes, and auth/schema migrations.
- **Symptom: Full scan exceeds context window limits on large monorepos**
  - **Cause**: Raw AST and file listings were printed directly into conversation history.
  - **Fix**: Route file discovery and manifest parsing through `context-mode` (`ctx_batch_execute`) so only the aggregated layer graph is returned.
- **Symptom: `git diff` baseline SHA is missing after a rebase or force-push**
  - **Cause**: The commit recorded in `.understand/graph.json` was rewritten.
  - **Fix**: Fall back to `git merge-base HEAD main` as the `base_ref` parameter and update `commit_sha` in `.understand/graph.json`.

## See Also

- Layer graph JSON schemas and delta classification rules live in `references/details.md`.
- Companion skills in this plugin include `skills/context-mode` for sandboxed execution and `skills/codebase-memory-mcp` for symbol-level call-graph queries.
- Related codebase documentation workflows pair directly with the `scan` skill and `conductor` context artifacts.
