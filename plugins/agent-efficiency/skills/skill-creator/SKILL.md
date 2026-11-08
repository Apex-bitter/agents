---
name: skill-creator
description: Use when creating, refactoring, or auditing an Agent Skill (SKILL.md), designing progressive-disclosure reference trees, or validating skills against the agentskills.io specification and Codex 8 KB body cap.
---

# Skill Creator & Progressive Disclosure Guide

Author portable, high-signal Agent Skills (`SKILL.md`) that comply with the [agentskills.io specification](https://agentskills.io/specification) and work cleanly across Claude Code, OpenAI Codex CLI, Cursor, OpenCode, Antigravity CLI, GitHub Copilot, and Pi.

## When to Use

- Creating a new `skills/<skill-name>/SKILL.md` package from scratch or from a recurring workflow.
- Refactoring an oversized skill (>8 KB) into a lean `SKILL.md` brain plus on-demand `references/` and `scripts/`.
- Auditing frontmatter triggers and cross-harness portability before publishing.

## 1. Three-Level Progressive Disclosure Architecture

Design every skill so agents load only the tokens they need at each stage:

| Level | Artifact | Token Budget | Purpose |
| :--- | :--- | :--- | :--- |
| **1. Metadata** | YAML frontmatter (`name`, `description`) | < 1,024 chars | Always loaded in system prompt; routes task to skill. |
| **2. Brain** | `SKILL.md` body | **≤ 8 KB** (Codex hard cap) | Core workflow, decision trees, and guardrails loaded on activation. |
| **3. Deep Context** | `references/*.md`, `scripts/*`, `assets/*` | Loaded on demand | Detailed schemas, long code templates, and deterministic helper scripts. |

```text
skills/<skill-name>/
├── SKILL.md                # Frontmatter + core workflow (≤ 8 KB)
├── references/             # Deep docs loaded only when needed
│   └── details.md
└── scripts/                # Deterministic CLI helpers executed via shell
```

## 2. Frontmatter Rules (`agentskills.io` Spec)

```yaml
---
name: my-skill-name
description: Use when [specific user intents, file types, or failure symptoms occur].
---
```

- **`name` invariant**: Lowercase hyphen-case (`^[a-z0-9-]+$`), and **must exactly match the parent directory name** (`skills/my-skill-name/SKILL.md`).
- **`description` trigger**: Must include an explicit activation clause (`Use when ...`, `Use this skill when ...`, or `Trigger when ...`) naming concrete tasks and artifacts.

## 3. Cross-Harness Portability Checklist

1. **Describe actions, not harness-specific tool names**:
   - Write *"Open the configuration file"* or *"Search for pattern X"* instead of naming a single tool's internal CamelCase identifier.
   - Write *"Track progress as you go"* instead of relying on harness-exclusive todo widgets.
   - Write *"Delegate to a subagent"* instead of hardcoding a single spawn API.
2. **Offload deterministic work to `scripts/`**:
   - If a step requires exact parsing, math, or schema validation, ship a small Python or POSIX shell script in `scripts/` rather than asking the LLM to simulate it in tokens.
3. **Keep reference links portable**:
   - Reference bundled files with backticked relative paths (e.g., `` `references/details.md` ``) or valid relative links so static link checkers verify them cleanly.

## 4. Validation Gate

Run both specification and static quality checks before committing a skill:

```bash
# Validate against the agentskills.io specification via GitHub CLI 2.90+
gh skill publish --dry-run

# Score static quality and harness portability in claude-agents
uv run --project plugins/plugin-eval plugin-eval score plugins/<plugin>/skills/<skill-name> --depth quick
```
