---
name: skill-creator
description: Use PROACTIVELY when creating new Agent Skills (SKILL.md), when refactoring oversized skill files into progressive-disclosure reference trees, when debugging harness portability warnings, or for validating skills against the agentskills.io specification and Codex 8 KB cap; automatically invoke when authoring or auditing skills.
---

# Skill Creator & Progressive Disclosure Guide

Author portable, high-signal Agent Skills (`SKILL.md`) that comply with the [agentskills.io specification](https://agentskills.io/specification) and transpile cleanly across Claude Code, OpenAI Codex CLI, Cursor, OpenCode, Google Antigravity CLI, GitHub Copilot, and Pi.

## When to Use & Decision Matrix

Use this progressive-disclosure decision matrix to place every piece of knowledge in the right tier and compare token tradeoffs:

| Disclosure Tier | Target File Path | Size / Token Cap | When to Use & Tradeoff Comparison |
| :--- | :--- | :--- | :--- |
| **Tier 1: Metadata** | `SKILL.md` YAML frontmatter | `< 1,024` chars (`120-260` ideal) | Always resident in system prompt; routes user intent to the skill |
| **Tier 2: Core Brain** | `SKILL.md` markdown body | `200 - 240` lines (`<= 7,800` bytes) | Loaded on skill activation; stays safely under the 8,192-byte Codex cap |
| **Tier 3: Deep Reference** | `references/details.md` | Unlimited (loaded on demand) | Holds exhaustive tables, protocol schemas, and edge-case catalogs |
| **Tier 4: Deterministic Code** | `scripts/<helper>.py` or `.sh` | Executed via shell (0 prompt tokens) | Offloads exact validation, parsing, or math from LLM generation |

## Input Parameters & Output Format

A well-structured skill defines explicit task input parameters and deterministic output format contracts so calling agents know how to invoke it and parse results:

- **Input parameters**:
  - `skill_dir` (`string`, required): Target directory path matching `plugins/<plugin>/skills/<skill-name>/`.
  - `domain_triggers` (`array[string]`, required): Concrete user intents, file extensions, and error symptoms that activate the skill.
  - `target_harnesses` (`array[string]`, optional): Harnesses to validate against (defaults to all 7 supported coding tools).
- **Output format**:
  - Produces a validated directory tree (`SKILL.md`, `references/details.md`, and optional `scripts/`) and returns a `plugin-eval` JSON scorecard containing `overall_score` (`0-100`), `badge` (`gold`/`silver`/`bronze`), `dimensions`, and `issues`.

```yaml
---
name: my-skill-name
description: Use PROACTIVELY when [intent A], when debugging [symptom B], when testing [artifact C], or for configuring [system D]; automatically invoke when [trigger condition].
---
```

## Core Workflow: Design, Author, and Validate

### 1. Scaffold the Progressive-Disclosure Directory Tree

Create the skill directory so the folder name matches the lowercase hyphenated `name` field in `SKILL.md`:

```bash
# Create skill directory and on-demand reference folder
mkdir -p plugins/<plugin>/skills/<skill-name>/references

# Verify directory naming matches ^[a-z0-9-]+$
basename "plugins/<plugin>/skills/<skill-name>"
```

### 2. Author Portable Instructions & IO Wiring

Write the `SKILL.md` body using portable action verbs rather than single-vendor internal tool identifiers:

1. **Describe actions, not vendor-locked tool names**:
   - Write *"Inspect the configuration file"* or *"Search repository symbols"* instead of raw internal CamelCase tool identifiers.
   - Write *"Track checklist progress"* instead of referencing a single IDE's todo widget.
   - Write *"Delegate to a subagent"* instead of hardcoding a proprietary agent-spawn function.
2. **Enforce the Codex 8 KB body cap and line budget**:
   - Keep `SKILL.md` between `200` and `240` lines and `<= 7,800` bytes so OpenAI Codex CLI (`8,192`-byte limit) loads the complete brain without truncation.
   - Link deep lookup tables via backticked relative paths such as `` `references/details.md` ``.

### 3. Verify Byte Cap, Line Budget, and Static Score

Run a deterministic Python check followed by `plugin-eval` to verify structural completeness and Gold-tier readiness:

```python
from pathlib import Path

skill_file = Path("plugins/agent-efficiency/skills/skill-creator/SKILL.md")
raw = skill_file.read_text(encoding="utf-8")
_, _, body = raw.split("---", 2)
lines = len(raw.splitlines())
body_bytes = len(body.encode("utf-8"))
print({"total_lines": lines, "body_bytes": body_bytes, "within_codex_cap": body_bytes <= 8192})
```

```json
{
  "skill": "skill-creator",
  "overall_score": 92.4,
  "badge": "gold",
  "dimensions": {
    "frontmatter_quality": 1.0,
    "orchestration_wiring": 1.0,
    "progressive_disclosure": 0.75,
    "structural_completeness": 1.0
  }
}
```

## Examples

### Example 1: Refactoring an Oversized 14 KB Skill into Tiered Files

1. Measure the existing `SKILL.md` (`410` lines, `14,200` bytes — fails Codex `8,192`-byte cap).
2. Move the 180-line API schema catalog and language-specific boilerplate into `references/details.md`.
3. Retain the decision table, input/output specification, 5 core code examples, and troubleshooting checklist in `SKILL.md` (`215` lines, `6,900` bytes).
4. Re-run `make garden STRICT=1` and confirm zero `oversize_codex_skills` warnings.

### Example 2: Validating Cross-Harness Generation & Spec Compliance

```bash
# 1. Score the skill with plugin-eval
uv run --project plugins/plugin-eval plugin-eval score plugins/agent-efficiency/skills/skill-creator --depth quick

# 2. Regenerate all 7 harness targets and run strict validation
make generate-all && make validate STRICT=1 && make garden STRICT=1
```

### Pre-Commit Verification Checklist

1. Confirm `name` in YAML frontmatter matches the parent directory name (`^[a-z0-9-]+$`).
2. Confirm `description` includes `Use PROACTIVELY when ...`, `automatically invoke`, and 3+ gerund trigger clauses.
3. Confirm `SKILL.md` body is under `8,192` bytes and `references/details.md` exists for deep lookup tables.
4. Confirm `make generate-all` produces zero uncommitted diff and `make garden STRICT=1` passes cleanly.

## Troubleshooting

- **Symptom: `plugin-eval` reports `WEAK_TRIGGER` or low `frontmatter_quality`**
  - **Cause**: The `description` field is under 100 characters or lacks explicit `Use PROACTIVELY when ...` and comma-separated gerund trigger clauses.
  - **Fix**: Expand `description` to `150-240` characters starting with `Use PROACTIVELY when <gerund>ing ..., when debugging ..., when testing ..., or for configuring ...; automatically invoke ...`.
- **Symptom: `plugin-eval` reports `DEAD_CROSS_REF` on a prose `skills/<name>` path**
  - **Cause**: Un-backticked `skills/<name>` text in prose is resolved relative to `skills/`, producing `skills/skills/<name>`.
  - **Fix**: Wrap all `skills/<name>` and `references/<file>` paths in inline code backticks (`` `skills/<name>` ``).
- **Symptom: `make garden` fails with `oversize_codex_skills`**
  - **Cause**: The markdown body after YAML frontmatter exceeds `8,192` bytes without a `references/` split.
  - **Fix**: Trim `SKILL.md` body below `7,800` bytes and move secondary examples into `references/details.md`.

## See Also

- Per-harness transpilation paths and `plugin-eval` scoring weights live in `references/details.md`.
- Companion skills in this plugin include `skills/skill-discovery` for auditing upstream skills and `skills/context-mode` for token-bounded execution.
- Related repository authoring guidelines live in `docs/authoring.md` and `docs/harnesses.md`.
