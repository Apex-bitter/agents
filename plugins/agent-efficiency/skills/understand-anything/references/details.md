# Understand Anything Reference Details (`Lum1104/Understand-Anything`)

## Semantic Knowledge Graph Schema

`Understand-Anything` represents repository architecture as a multi-layered semantic graph stored under `.understand/graph.json`:

```json
{
  "version": "1.0.0",
  "commit_sha": "6d659bb",
  "layers": [
    {
      "name": "domain-core",
      "modules": ["src/domain/**", "src/models/**"],
      "allowed_dependencies": []
    },
    {
      "name": "application-services",
      "modules": ["src/services/**", "src/usecases/**"],
      "allowed_dependencies": ["domain-core"]
    },
    {
      "name": "adapters-and-http",
      "modules": ["src/api/**", "src/cli/**"],
      "allowed_dependencies": ["application-services", "domain-core"]
    }
  ]
}
```

## Delta Scan Classification Matrix

When executing a delta scan (`git diff <baseline_sha>..HEAD --name-only`), classify each changed path before touching `AGENTS.md` or `project-doc.md`:

| Diff Pattern | Classification | Documentation Action |
| :--- | :--- | :--- |
| Internal function body edit | Implementation detail | Update `.understand/graph.json` timestamp only; skip `AGENTS.md` |
| New public CLI flag or API endpoint | Interface expansion | Update `project-doc.md` endpoint table; keep `AGENTS.md` unchanged |
| New top-level directory or plugin | Structural shift | Propose `AGENTS.md` map update and request human confirmation |
| Dependency swap in `pyproject.toml` / `package.json` | Tech stack drift | Update `project-doc.md` and `conductor/tech-stack.md` |
| Auth middleware or DB migration change | Cross-cutting shift | Update security/data sections in `project-doc.md` and `AGENTS.md` |
