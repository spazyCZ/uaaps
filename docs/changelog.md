# Changelog

All notable changes to the UAAPS specification are documented here.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

---

## [0.8.0-draft] — 2026-03-13

### Added
- §3 Manifest: `files` field — restricts which files are included in the published `.aam` archive (see §12.1)
- §3 Manifest: `dependencyGroups` — named, non-runtime opt-in dependency sets for workflows (`dev`, `docs`, `eval`)
- §3 Manifest: `extras` — optional consumer-facing feature bundles that can add dependencies, activate artifacts, and extend permissions when selected at install time (`aam install --extras ocr,github`)
- §10 Migration §11.2: Codex native build target — full output layout spec, artifact mapping table, and conversion rules for skills, commands, and agents to `.agents/skills/` structure
- §12.1 Packaging: Deterministic archive file selection algorithm — `files`-field-driven packlist with always-include/always-exclude rules
- §12.1 Packaging: Reproducible archive build spec — lexicographic entry order, normalized owner/group/timestamps, `SOURCE_DATE_EPOCH` support
- §13.1 Dependencies: `dependencyGroups` semantics — non-transitive, `--with`/`--only-group` install flags, system dependency participation
- §13.1 Dependencies: `extras` semantics — additive, root-selected only, artifact activation, permission union at install time, `--extras` and `pkg[extra,…]` syntax
- §13.3 Lock file v2: `source` field becomes a structured object with `type`, `registry`, `name`, `version`, `tarball`, `display`, and optional `resolvedTag` fields
- §15 Validation: New rules — lock file source metadata v2, `files` field format, archive completeness (pack fails if referenced file omitted), `dependencyGroups` key format, `extras` key format, extra artifact ref resolution
- §15 Validation: Extra permission audit — same audit rules MUST apply to effective permission set after extra merge
- §16 Security: Extra-selected permissions model — union semantics, tool MUST surface permission delta at install time, platforms MUST audit effective (not base) permission set
- §16 Security: Package contents security practice — publishers SHOULD use `files` field and review packlist warnings for likely-secret files

### Changed
- §2 Package Format: Commands directory gains optional `tests/` subdirectory (deterministic command tests); rules directory supports recursive grouping (`group-name/rule-name/` at arbitrary depth)
- §4 Skills: `context`, `agent`, `disable-model-invocation` frontmatter fields annotated as "Inferred" — these fields are not confirmed in public Claude Code documentation
- §4 Skills: GitHub Copilot platform support corrected from "✅ Supported" to "⚠️ Preview (not GA)"
- §5 Commands: Claude Code invocation syntax clarified as `/package-name:command-name`; Codex updated from "⚠️ Via skills" to "⚠️ Via generated skills" with `.agents/skills/cmd-<name>/` path; migration guidance updated for Codex `openai.yaml` policy
- §6 Agents: Codex export path updated from "⚠️ Via skills" to "⚠️ Via generated skills" with `.agents/skills/agent-<name>/`
- §8 Hooks: Added "Inferred" caveat for Copilot hook event names (these are inferred mappings; GitHub Copilot Extensions use a distinct GitHub App webhook mechanism); "Community" caveat for Cursor hook event names
- §10 Compatibility matrix: Copilot skills confidence corrected to "Community"; Copilot/Cursor hooks confidence corrected to "Mixed"; AGENTS.md Copilot footnote clarifies it is read only in Copilot Coding Agent mode; Cursor rules path corrected to `.cursor/rules/*.mdc`
- §10 Migration: Codex install target updated from "`AGENTS.md` composite inject" to "`.agents/skills/` + `AGENTS.md`"
- §12.6 Registry: `DELETE /packages/:name/:version` endpoint removed; index regeneration trigger updated to cover all lifecycle state mutations
- §12.7 Packaging: Immutability guarantee strengthened — registries MUST reject any attempt to publish different archive bytes for an already-published `name@version`
- §16 Security: Root authority definition updated to include extra-selected permissions in the effective permission boundary (base `permissions` + selected extra permissions)

---

## [0.7.0-draft] — 2026-02-23

### Added
- §7 Rules: Full Universal Agent Rule Specification (UARS v1.3) replacing the previous stub.
  - UARS file format: `<name>.rule.md` with YAML frontmatter (`name`, `version`, `description`, `apply.*`, `agents.*`, `refs`) plus free-form Markdown body.
  - `apply.mode` enum: `always` | `intelligent` | `files` | `manual` with defined semantics per platform.
  - Mode mapping table: how each `apply.mode` value compiles to Copilot `applyTo`, Cursor `alwaysApply`/`globs`, Claude Code `paths:`, and Codex directory placement.
  - `agents.*` per-platform override block limited to platform-specific capabilities: `enabled` (all), `alwaysApply` + `priority` (Cursor), `override` (Codex).
  - `refs` field for attaching other rule names or template paths (Cursor `@file` style).
  - Compiled output examples for all four platforms (Copilot, Cursor, Claude Code, AGENTS.md).
  - Full field reference table for top-level fields and `agents.*` override fields.
- §7 Rules: Converter Specification — normative two-namespace CLI design:
  - **`aam pkg rules` namespace** (package authoring): `compile`, `import`, `diff`, `clean`, `list` sub-commands for rules living in `.aam/rules/`.
  - **`aam convert rules` namespace** (standalone conversion): direct in-memory platform-to-platform translation without a UAAPS package.
- §7 Rules: `aam pkg rules compile` processing pipeline (9 stages: Discovery → Parse → Validate → Filter → Transform → Group → Merge → Conflict → Write).
- §7 Rules: Per-platform transformation tables for Copilot, Cursor, Claude Code, and Codex.
- §7 Rules: `aam pkg rules import` inverse field mapping and per-platform discovery paths.
- §7 Rules: Conflict Detection — generation guard patterns per platform; `.aam/conflicts/` pending files; `--force` behaviour.
- §7 Rules: Idempotency guarantee — deterministic alphabetical rule ordering, no volatile metadata in generated content.
- §7 Rules: Watch mode (`--watch`) with filesystem event subscription and full-pass recompilation.
- §7 Rules: `aam pkg rules diff` and `aam pkg rules clean` with guard-based deletion safety.
- §7 Rules: `aam pkg rules list` formatted output.
- §7 Rules: Error exit codes `0`–`4` for `aam pkg rules` commands.
- §7 Rules: `aam convert rules` standardised options table (`--source-platform` / `-s`, `--target-platform` / `-t`, `--type`, `--source`, `--out`, `--dry-run`, `--force`, `--verbose`).
- §7 Rules: AGENTS.md merge strategies — directory inference from globs, root aggregation for `always`/`intelligent`, guard-block management for Claude Code.
- §7 Rules: Codex `AGENTS.override.md` support via `agents.codex.override: true`.
- §7 Rules: Export section: UARS → AGENTS.md directory tree with section-splitting and guard comments.
- §7 Rules: Import section: AGENTS.md → UARS reverse pipeline including H1/H2 section splitting.
- §7 Rules: Validation rules (required fields, glob requirement for `files` mode, slug format, semver format).
- §7 Rules: Worked migration examples (Cursor `.mdc` → UARS `.rule.md`, AGENTS.md → UARS).
- `docs/schemas/rule-frontmatter.schema.json`: Updated to UARS v1.3 schema with `name`, `version`, `description`, `apply` (with `mode` + `globs`), `agents` per-platform overrides, and `refs`. Schema authority: informational only; prose spec is normative.

### Changed
- §7 Rules: Replaced the previous 40-line stub (Cursor-centric `RULE.md` format with `description`, `globs`, `alwaysApply`) with the full UARS specification.
- §7 Rules: `agents.*` override block no longer allows re-defining `scope`, `paths`, `applyTo`, or `globs` (these are controlled by `apply.*`). Only platform-specific features not expressible in `apply:` are permitted in `agents:`.
- `docs/schemas/rule-frontmatter.schema.json`: Schema required fields changed from `["description"]` to `["name", "description", "apply"]`.
- CLI command naming: standalone conversion uses `aam convert rules` (not `aam converter rules`, `aam rules compile`, or `aam rules convert`).
- CLI flag naming: standalone conversion uses `--source-platform` / `-s` and `--target-platform` / `-t` (not `--from` / `--to`).

---

## [0.6.0-draft] — 2026-02-23

### Added
- §1 Introduction: Specification versioning strategy (major/minor/patch rules for the spec itself)
- §1 Introduction: Forward-compatibility parsing rules (consumers MUST ignore unknown fields)
- §1 Introduction: Conformance levels — Level 1 (Reader), Level 2 (Installer), Level 3 (Publisher)
- §1 Introduction: ABNF syntax definitions for all identifiers (package names, version ranges, skill refs, etc.)
- §3 Manifest: YAML round-trip guarantee (YAML 1.2, JSON-compatible types only)
- §3 Manifest: `dist-tags`, `resolutions`, and `deprecated` fields added to manifest schema
- §5 Commands: Namespace collision resolution rules
- §6 Agents: `system-prompt.md` frontmatter field reference table
- §9 MCP: `servers.json` field reference table
- §10 Compatibility: Platform capability negotiation protocol and `.platform-capabilities.json`
- §12.6 Registry: Structured error code constants, content negotiation, pagination, search/deprecation/yank endpoints
- §12.7 Packaging: Package lifecycle states (Published, Deprecated, Yanked) with immutability guarantee
- §13.4 Dependencies: Formalized resolution algorithm with named phases, priority ordering, pre-release handling
- §13.4 Dependencies: Resolver versioning (`resolverVersion` manifest field)
- §13.14 Dependencies: Workspace / monorepo support (`workspace.agent.json`)
- §14.5 Signing: SLSA-aligned provenance attestation (L0–L3) with in-toto format
- §15.3 Governance: Structured audit log format with JSON schema
- §16 Validation: New validation rules for resolver version, workspaces, forward-compat, permission grants, provenance
- §17.2 Security: Formal threat model (actors, surfaces, mitigations, residual risks)
- §17.3 Security: Transitive permission composition model with `permissionGrants`
- §18 Glossary: 12 new terms (Yank, Deprecation, Provenance, Conformance Level, Workspace, etc.)
- Implementers guide expanded from stub to full conformance guide
- All JSON schemas upgraded from Draft-07 to JSON Schema 2020-12
- Schema authority statement: prose spec is normative, schemas are informational

### Changed
- §12 Packaging: Subsections reordered to numerical sequence (§12.1–§12.6)
- §12.6 Registry: Status upgraded from "Skeleton" to "Draft" with substantially expanded content
- Introduction section headings unnumbered (resolves duplicate §2 in combined full.md)
- Permissions schema in `package-manifest.schema.json` fixed to match prose spec structure

### Fixed
- Broken `#evals-evals` anchor links verified correct (not broken — Python-Markdown collapses em-dash correctly)
- Stale changelog reference to `§5.5 Prompts` → `§5 Commands`
- Copilot issue `#1157` citation replaced with generic reference (unverifiable URL)
- Self-referential `§17.1` citation clarified in security section
- Appendix headings made consistent (Appendix A, Appendix B)
- Eval `engine` field: `copilot` and `cursor` values marked as reserved (unsupported until headless modes available)
- Model IDs in eval examples noted as illustrative

---

## [0.5.0-draft] — 2026-02-22

### Added
- §4 Skills: `tests/` directory structure for deterministic (assert) skill script testing
- §8 Hooks: `tests/` directory structure with fixtures and deterministic hook testing
- §2 Package Format: `evals/` directory expanded with full eval config, case format, and execution flow
- Evals system: LLM-judged integration tests via agent sandbox (`engine`, `judge`, `sandbox` config)
- Tests vs Evals comparison table clarifying the two-tier testing model
- `aam test` CLI for deterministic assert tests (skills + hooks)
- `aam eval` CLI for LLM-judged sandbox evaluations

---

## [0.4.0-draft] — 2026-02-19

### Added
- §12 Packaging & Distribution: archive format (`.aam`), scoped package names, dist-tags, portable bundles
- §13 Dependency Resolution: lock file format, resolution algorithm, conflict handling, CI/CD patterns
- §14 Package Signing & Verification: Sigstore, GPG, registry attestation, verification policy
- §15 Governance & Policy Gates: client-side policy gates, approval workflows, audit log events
- §5 Commands: reusable parameterized prompt templates with variable interpolation
- MkDocs-based documentation structure replacing monolithic `SPECIFICATION.md`

### Changed
- Merged `DESIGN.md` content into main specification
- Agent definitions restructured as two-file directories (`agent.yaml` + `system-prompt.md`)

---

## [0.3.0] — 2026-01-15

### Added
- Initial hook event unification across Claude Code, Cursor, and Copilot
- `AGENTS.md` designated as universal fallback instruction format
- Cross-platform compatibility matrix

---

*Older entries pending backfill.*
