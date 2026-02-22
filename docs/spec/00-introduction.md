# Universal Agentic Artifact Package Specification (UAAPS)

> **Version**: 0.5.0-draft  
> **Date**: 2026-02-22 (revised)  
> **Purpose**: Define a universal, portable package standard for AI agent artifacts — analogous to Docker for containers or npm for JavaScript packages.  
> **Goal**: Write once, deploy to any agent platform. Simplify migration. Eliminate vendor lock-in.

---

## 1. Design Principles

| Principle | Description |
|-----------|-------------|
| **Portability** | A single package works across Claude Code, Cursor, Copilot, Codex, and any compliant platform. |
| **Progressive Disclosure** | Metadata loads first; full instructions load on activation; resources load on-demand. |
| **Convention over Configuration** | Standard directory names eliminate per-platform mapping; overrides are optional. |
| **Filesystem-First** | All artifacts are files on disk — no databases, no APIs required, no binary formats. |
| **Composability** | Packages can be installed together; namespacing prevents conflicts. |
| **Deterministic Resolution** | Lock files ensure reproducible installs across machines and CI. |
| **Backward Compatibility** | Existing Claude Code plugins and Cursor plugins work with minimal changes. |

---

## 2. Normative Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

| Keyword | Meaning |
|---------|---------|
| **MUST** / **REQUIRED** / **SHALL** | An absolute requirement of the specification. Implementations that do not satisfy this are non-conformant. |
| **MUST NOT** / **SHALL NOT** | An absolute prohibition. Implementations that violate this are non-conformant. |
| **SHOULD** / **RECOMMENDED** | There may exist valid reasons to ignore this in particular circumstances, but the full implications must be understood and carefully weighed before doing so. |
| **SHOULD NOT** / **NOT RECOMMENDED** | There may exist valid reasons when this behaviour is acceptable, but the implications must be understood and carefully weighed. |
| **MAY** / **OPTIONAL** | Truly optional. Implementations that omit this remain conformant; those that include it MUST interoperate with implementations that do not. |

Where this spec states a plain imperative (e.g., "Packages include a manifest") without one of these keywords, the statement is informative, not normative.
