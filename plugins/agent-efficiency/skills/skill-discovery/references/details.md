# Skill Discovery Reference Details

## Supply-Chain Security Inspection Patterns

Before installing a third-party `SKILL.md` package or executing its `scripts/` directory, run static pattern checks for credential access, network exfiltration, and hidden prompt-injection directives:

```python
import pathlib
import re
import sys

RISKY_PATTERNS = {
    "credential_read": re.compile(r"(\.ssh|\.aws|\.env|\.netrc|\.gitcookies|API_KEY|SECRET_KEY)"),
    "network_exfil": re.compile(r"(curl\s+.*-d|wget\s+--post|requests\.post|urllib\.request)"),
    "shell_pipe": re.compile(r"curl\s+[^\n]+\|\s*(ba)?sh"),
    "invisible_unicode": re.compile(r"[\u200b-\u200f\u202a-\u202e\u2060-\u206f]"),
}

def audit_skill_tree(skill_root: pathlib.Path) -> list[dict[str, str]]:
    findings: list[dict[str, str]] = []
    for path in sorted(skill_root.rglob("*")):
        if not path.is_file():
            continue
        text = path.read_text(encoding="utf-8", errors="replace")
        for label, pattern in RISKY_PATTERNS.items():
            for match in pattern.finditer(text):
                findings.append({
                    "file": str(path.relative_to(skill_root)),
                    "rule": label,
                    "match": match.group(0),
                })
    return findings
```

## Multi-Agent Target Flag Reference (`npx skills` & `gh skill`)

| Target Coding Tool | `gh skill install --agent` | `npx skills add -a` | Default Project Install Path |
| :--- | :--- | :--- | :--- |
| Claude Code | `claude-code` | `claude-code` | `.claude/skills/<skill-name>/` |
| OpenAI Codex CLI | `codex` | `codex` | `.codex/skills/<skill-name>/` |
| Cursor | `cursor` | `cursor` | `.cursor/skills/<skill-name>/` |
| OpenCode | `opencode` | `opencode` | `.opencode/skills/<skill-name>/` |
| GitHub Copilot | `github-copilot` | `github-copilot` | `.github/skills/<skill-name>/` |
| Google Antigravity | `antigravity` | `antigravity` | `.agent/skills/<skill-name>/` |
