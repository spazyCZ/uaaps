# AGENTS.md — Contribution Instructions for AI Agents

This file governs how AI agents (GitHub Copilot, Claude, Cursor, Codex, etc.) MUST behave when editing this repository.

---

## 1. Repository Purpose

This repository contains the **Universal Agentic Artifact Package Specification (UAAPS)** — a formal technical specification written in Markdown under `docs/spec/`. The spec defines a portable standard for AI agent artifact packages.

---

## 2. Normative Language (RFC 2119) — CRITICAL

All spec prose in `docs/spec/` MUST follow [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) normative keyword conventions as defined in `docs/spec/00-introduction.md § 2`.

**Rules you MUST follow when writing or editing any spec file:**

| Keyword | Case | Use when |
|---------|------|----------|
| `MUST` / `MUST NOT` | UPPERCASE | Absolute requirement or prohibition |
| `SHOULD` / `SHOULD NOT` | UPPERCASE | Strong recommendation with acknowledged exceptions |
| `MAY` / `OPTIONAL` | UPPERCASE | Genuinely optional behaviour |
| `REQUIRED` / `RECOMMENDED` | UPPERCASE | Synonyms for MUST / SHOULD respectively |

- **Never** write `must`, `should`, `may`, `required`, `recommended`, or `optional` in lowercase in normative spec prose.
- Lowercase is only acceptable inside fenced code blocks (YAML/JSON/shell examples) where it is user-authored example content, not spec text.
- Plain imperatives without a keyword (e.g. "Packages include a manifest") are informative — use them sparingly and only when you intentionally want non-normative wording.

---

## 3. Spec File Conventions

- All spec files live in `docs/spec/` and are numbered `00-` through `17-` plus `appendix.md` and `full.md`.
- `full.md` is a generated concatenation — **do not edit it directly**. Edit the numbered source file instead.
- Section heading numbers inside each file MUST match the file's position in the sequence (e.g. `02-manifest.md` contains `## 3. Manifest`).
- Field tables MUST use the column order: `Field | Type | Required | Description`.
- The `Required` column value MUST use one of: `**Yes**`, `No`, `RECOMMENDED`, or a conditional note.

---

## 4. Writing Style

- Use second-person imperative ("Implementations MUST…", "Tools SHOULD…") not first-person.
- Keep prose concise. Prefer tables and code blocks over long paragraphs.
- Use em dashes (`—`) not double hyphens (`--`) in prose.
- Forward slashes only in file paths — never backslashes.
- Do not add new sections without a corresponding schema update in `docs/schemas/` if structured data is involved.

---

## 5. What Requires Human Review

Before committing changes to the following, flag them for human review:

- `docs/schemas/` — JSON Schema changes affect validation tooling.
- `docs/spec/00-introduction.md` — changes to normative language definitions affect the entire spec.
- `VERSION` and `CHANGELOG.md` — version bumps follow SemVer and require deliberate author decision.
- Any removal of an existing normative MUST/MUST NOT rule — this is a breaking change.
