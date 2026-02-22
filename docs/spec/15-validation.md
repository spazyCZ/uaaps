## 16. Validation Rules

| Rule | Constraint |
|------|-----------|
| Package `name` | `[a-z0-9-]` max 64 chars, or scoped `@scope/name` max 130 chars |
| Package `version` | Valid SemVer 2.0 |
| Skill `description` | Max 1024 chars, MUST describe WHAT + WHEN |
| SKILL.md body | RECOMMENDED < 5,000 tokens / 500 lines |
| Frontmatter | Valid YAML, no tabs |
| Scripts | MUST be self-contained or document dependencies |
| Hook commands | MUST handle JSON on stdin |
| File references | Forward slashes only, relative paths |
| Dependency versions | Valid SemVer range syntax (`^`, `~`, `>=`, exact) |
| Dependency graph | MUST be acyclic — circular dependencies are an error |
| Peer dependencies | MUST be satisfied by root or ancestor package |
| System dependencies | Pre-flight check MUST pass (or explicit `--skip-checks`) |
| Lock file integrity | Hash MUST match on `--frozen` install |
| Namespace uniqueness | No two skills with same `name` within a single package |
| Resolution overrides | `resolutions` entries MUST reference packages in the tree |
