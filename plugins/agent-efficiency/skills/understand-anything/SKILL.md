---
name: understand-anything
description: Use when bootstrapping architectural documentation, building a semantic knowledge graph of a repository, or running delta scans with Lum1104/Understand-Anything to update project-doc.md and AGENTS.md.
---

# Understand Anything (`Lum1104/Understand-Anything`)

Generate and query a semantic knowledge graph of any codebase to accelerate onboarding, architectural mapping, and incremental delta documentation updates.

## When to Use

- Onboarding onto a new or undocumented repository to synthesize `AGENTS.md` and architecture blueprints.
- Running incremental **delta scans** after commits to detect new frameworks, directory patterns, or auth/data-layer shifts.
- Pairing with the `scan` skill and `context-mode` so full-repo analysis never floods the active context window.

## Installation

```bash
# Claude Code plugin marketplace
/plugin marketplace add Lum1104/Understand-Anything
/plugin install understand-anything
```

## Core Workflows

### 1. Full Repository Graph Synthesis (First Run)

When no prior architecture baseline exists:

1. Run semantic graph extraction across the repository's entry points, package manifests (`pyproject.toml`, `package.json`, `Cargo.toml`, `go.mod`), and module boundaries.
2. If `context-mode` is active, route raw graph output through `ctx_batch_execute` / `ctx_execute_file` so only synthesized summaries enter the conversation.
3. Emit a structured architecture map covering:
   - **Tech Stack & Runtime Versions**
   - **Layered Module Boundaries & Dependency Direction**
   - **Data Models, ORM Schemas, and State Stores**
   - **Cross-Cutting Concerns** (authentication, error handling, logging, validation)
   - **Test Coverage & Critical Untested Paths**

### 2. Incremental Delta Scan (Subsequent Runs)

Avoid re-analyzing the entire codebase after routine commits:

1. Compute the changed file set:
   ```bash
   git diff HEAD~1 --name-only
   ```
2. Re-analyze only the modified files and their immediate 1-hop graph neighbors.
3. Flag **architectural drift** only when one of the following changes occurs:
   - A new top-level module or service directory is introduced.
   - A core dependency is added, removed, or swapped.
   - Authentication, session management, or database migration patterns change.
4. Patch only the affected sections of `AGENTS.md` or project documentation after human confirmation—never overwrite hand-curated rules wholesale.
