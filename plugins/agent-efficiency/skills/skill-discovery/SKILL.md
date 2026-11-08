---
name: skill-discovery
description: Use when searching for, previewing, auditing, or installing external Agent Skills (SKILL.md) from GitHub repositories, VoltAgent/awesome-agent-skills, anthropics/skills, vercel-labs/agent-skills, or supabase/agent-skills using gh skill and npx skills.
---

# Skill Discovery & Safe Installation Workflow

Discover, preview, security-audit, and install portable `SKILL.md` packages on demand without polluting baseline system-prompt context with hundreds of unused skills.

## When to Use

- Looking for an official or community skill for a specialized framework, cloud provider, or workflow.
- Auditing an external `SKILL.md` repository before installing it into a project or global agent config.
- Pinning installed skills to immutable Git commit SHAs or release tags for supply-chain safety.

## 1. Curated Upstream Skill Registries

Search these vetted repositories first before broad web queries:

| Repository | Domain Coverage |
| :--- | :--- |
| **`wshobson/agents` (`chun0li/agents`)** | 191 multi-harness skills across 53 plugins (Python, TS, K8s, LLM fine-tuning, security, governance, efficiency). |
| **`VoltAgent/awesome-agent-skills`** | Curated directory of 1,000+ official and community `SKILL.md` packages (Hugging Face, TestMu QA, cloud, data). |
| **`anthropics/skills`** | Official Anthropic reference skills (`skill-creator`, DOCX/PDF/PPTX/XLSX document skills, MCP builder). |
| **`vercel-labs/agent-skills`** | Official Vercel frontend, Next.js, React, and `web-design-guidelines` skills. |
| **`supabase/agent-skills`** | Official Supabase PostgreSQL Row-Level Security (RLS), Edge Functions, and migration skills. |
| **`NVIDIA/skills`** | Official NVIDIA CUDA, TensorRT, NIM, and GPU systems engineering skills. |
| **`hashicorp/agent-skills`** | Official Terraform HCL style guides, module scaffolding, and Vault workflows. |

## 2. CLI Discovery & Preview Commands

Use **GitHub CLI (`gh skill`, v2.90+)** or **Vercel Open Skills (`npx skills`)** to search and inspect skills before installation:

```bash
# 1. Search GitHub or the open skills index by keyword
gh skill search <keyword>
npx skills find <keyword>

# 2. List skills inside a specific repository
npx skills add <owner/repo> --list

# 3. Preview the SKILL.md content before installing (supply-chain check)
gh skill preview <owner/repo> <skill-name>
```

## 3. Supply-Chain Audit & Pinned Installation

Because `SKILL.md` files and bundled `scripts/` instruct agents with tool access, verify three invariants before installing:

1. **No prompt-injection or credential exfiltration**: Confirm `SKILL.md` and `scripts/` do not read `~/.ssh`, `~/.env`, or send environment variables to untrusted hosts.
2. **Pin to a commit SHA or release tag**: Use `--pin <sha>` so upstream branch mutations cannot silently alter agent behavior.
3. **Scope to the project or target agent**: Install specialized skills at the project scope rather than globally when they apply to a single repository.

```bash
# Install with GitHub CLI pinned to a verified commit SHA
gh skill install <owner/repo> <skill-name> --agent claude-code --pin <commit-sha>

# Or install via npx skills for a specific harness
npx skills add <owner/repo> --skill <skill-name> -a claude-code

# Check for upstream updates across installed skills
gh skill update
```
