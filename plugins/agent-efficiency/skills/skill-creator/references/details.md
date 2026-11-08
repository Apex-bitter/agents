# Skill Creator Reference Details (`agentskills.io` & Cross-Harness Spec)

## Per-Harness Skill Adaptation Matrix

When a skill is authored in `plugins/<plugin>/skills/<skill-name>/SKILL.md`, the repository generators transform it for seven target harnesses:

| Harness | Generated Path | Naming Convention | Special Constraints |
| :--- | :--- | :--- | :--- |
| Claude Code | `plugins/<p>/skills/<s>/SKILL.md` | Bare `<skill>` | Native source of truth |
| OpenAI Codex CLI | `.codex/skills/<p>__<s>/SKILL.md` | `<plugin>__<skill>` | **8,192-byte body cap**; excess overflows to `references/` |
| OpenCode | `.opencode/skills/<p>-<s>/SKILL.md` | `<plugin>-<skill>` | Hyphenated namespace for global discovery |
| Google Antigravity CLI | `.antigravity/plugins/<p>/skills/<s>/SKILL.md` | Bare `<skill>` | Scoped by parent plugin directory |
| GitHub Copilot | `.copilot/skills/<p>__<s>/SKILL.md` | `<plugin>__<skill>` | Commands also emit as invocable skills |
| Cursor | `.claude/skills/` | Native read | Reads Claude Code format directly |
| Pi | `.pi/skills/<p>/<s>/SKILL.md` | Bare `<skill>` | Recursive directory discovery |

## `plugin-eval` Static Scoring Checklist

Use this checklist to reach Gold (`>= 85/100`) on `plugin-eval`:

1. **Frontmatter Quality (32% weight)**:
   - `name` matches directory name (`^[a-z0-9-]+$`).
   - `description` is `>= 120` characters and starts with `Use PROACTIVELY when ...`.
   - Includes `automatically invoke` and 3+ comma-separated gerund/trigger clauses (`, when configuring ...`, `, when debugging ...`, `, for validating ...`).
2. **Task IO Wiring (23% weight)**:
   - Document explicit `input` `parameters` that the workflow `accepts`.
   - Document the `output format` that the workflow `returns` or `produces`.
3. **Progressive Disclosure (14% weight)**:
   - Keep `SKILL.md` between `200` and `240` lines (`<= 7,800` bytes).
   - Provide deep reference material in `references/details.md`.
4. **Structural Completeness (10% weight)**:
   - Include `>= 6` `##`/`###` headings, `>= 5` fenced code blocks across `>= 3` languages (`bash`, `json`, `python`, `yaml`), a markdown comparison table, `## Examples`, and `## Troubleshooting`.
