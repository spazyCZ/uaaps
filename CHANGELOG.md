# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.8.0-draft] — 2026-03-13

### Added
- §3 Manifest: `files` field for archive content selection (packlist control)
- §3 Manifest: `dependencyGroups` — named, non-runtime opt-in dependency sets (`dev`, `docs`, `eval`)
- §3 Manifest: `extras` — optional consumer-facing feature bundles with dependencies, artifacts, and permissions
- §10 Migration: §11.2 Codex native build target spec — full artifact mapping table, skill/command/agent conversion rules
- §12 Packaging: Deterministic archive file selection algorithm and reproducible build spec
- §12 Packaging: Immutability: registries MUST reject duplicate `name@version` publishes
- §13 Dependencies: `dependencyGroups` and `extras` resolution semantics; install flags `--with`, `--only-group`, `--extras`
- §13 Dependencies: Lock file v2 — `source` becomes a structured object with `type`, `registry`, `tarball`, `display` fields
- §15 Validation: New rules for lock file source metadata v2, `files` field, archive completeness, `dependencyGroups`, `extras`, extra artifact refs
- §16 Security: Extra-selected permissions model (union at install time, tool MUST surface delta to user)

### Changed
- §2 Package Format: Commands directory gains optional `tests/` subdirectory; rules directory supports recursive grouping
- §4 Skills: `context`, `agent`, `disable-model-invocation` frontmatter fields marked as "Inferred" (not confirmed in public Claude Code docs); Copilot skills support corrected to "⚠️ Preview (not GA)"
- §5 Commands: Claude Code invocation syntax clarified (`/package-name:command-name`); Codex updated to generated-skill export path
- §6 Agents: Codex export path updated to `.agents/skills/agent-<name>/`
- §8 Hooks: Added "Inferred" caveat for Copilot hook event names; "Community" caveat for Cursor hook event names
- §10 Compatibility: Copilot skills/hooks/AGENTS.md confidence levels corrected; Cursor rules path corrected to `.cursor/rules/*.mdc`; Codex install target updated to `.agents/skills/` + `AGENTS.md`
- §16 Security: Root authority definition updated to include extra-selected permissions in the effective permission boundary

## [0.7.0-draft] — 2026-02-23

### Added
- §7 Rules: Full Universal Agent Rule Specification (UARS v1.3) — replaces the previous Cursor-centric stub
- Two CLI namespaces for rules: `aam pkg rules` (package authoring) and `aam convert rules` (standalone conversion)
- `aam pkg rules compile/import/diff/clean/list` sub-commands
- `aam convert rules -s <platform> -t <platform>` with standardised flags (`-s`, `-t`, `--type`, `--dry-run`, `--force`, `--verbose`)
- UARS `apply.mode` enum: `always` | `intelligent` | `files` | `manual`
- Mode mapping table and per-platform transformation tables for all four platforms
- Conflict detection with generation guards and `.aam/conflicts/` pending files
- Idempotency guarantee and watch mode for `aam pkg rules compile`
- AGENTS.md merge strategy (directory inference, root aggregation, guard-block management)
- Codex `AGENTS.override.md` support via `agents.codex.override: true`
- Validation rules, error exit codes (0–4), and worked migration examples
- `docs/schemas/rule-frontmatter.schema.json` updated to UARS v1.3 (`name`, `apply.*`, `agents.*`, `refs`)

### Changed
- `agents.*` override block now limited to platform-specific capabilities only (`enabled`, `alwaysApply`/`priority` for Cursor, `override` for Codex); scope/path fields removed
- Standalone conversion command renamed from `aam converter rules` → `aam convert rules`
- Flags renamed: `--from`/`--to` → `--source-platform`/`-s` and `--target-platform`/`-t`

## [0.6.0-draft] — 2026-02-23

### Added
- §1 Introduction: Specification versioning strategy, forward-compatibility parsing rules, conformance levels (L1–L3), ABNF identifier syntax
- §3 Manifest: YAML round-trip guarantee; `dist-tags`, `resolutions`, `deprecated` fields
- §10 Compatibility: Platform capability negotiation protocol and `.platform-capabilities.json`
- §12 Registry: Structured error codes, pagination, search/deprecation/yank endpoints; package lifecycle states (Published, Deprecated, Yanked)
- §13 Dependencies: Formalized resolution algorithm, `resolverVersion` field, workspace/monorepo support (`workspace.agent.json`)
- §14 Signing: SLSA-aligned provenance attestation (L0–L3) with in-toto format
- §15 Governance: Structured audit log format with JSON schema
- §16 Validation: New rules for resolver version, workspaces, forward-compat, permission grants, provenance
- §17 Security: Formal threat model; transitive permission composition with `permissionGrants`
- §18 Glossary: 12 new terms (Yank, Deprecation, Provenance, Conformance Level, Workspace, etc.)
- All JSON schemas upgraded from Draft-07 to JSON Schema 2020-12

### Changed
- §12 Packaging: Subsections reordered to numerical sequence
- Introduction section headings unnumbered

### Fixed
- Permissions schema in `package-manifest.schema.json` fixed to match prose spec structure

## [0.5.0-draft] — 2026-02-22

### Added
- Skill testing: `tests/` directory with deterministic assert runner (§4 Skills)
- Hook testing: `tests/` directory with fixtures and assert runner (§8 Hooks)
- Evals system: top-level `evals/` with agent sandbox execution model (§2 Package Format)
- Two-tier testing model: `tests/` (assert, CI-safe, $0) vs `evals/` (LLM-judged, agent sandbox)
- `aam test` and `aam eval` CLI commands

## [0.4.0-draft] — 2026-02-19

### Added
- Initial specification covering package format, manifest, skills, commands, prompts, agents, rules, hooks, MCP, compatibility matrix, migration guides, packaging & distribution, dependency resolution, signing & verification, governance, validation rules, security considerations, glossary, and appendix.
- MkDocs-based documentation site with Material theme.
- GitHub Actions workflow for automated GitHub Pages deployment.
