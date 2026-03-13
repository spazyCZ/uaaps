# 📦 UAAPS — Universal Agentic Artifact Package Specification

> **Version:** 0.8.0-draft &nbsp;|&nbsp; **Status:** 🌐 Open Standard, seeking collaborators

✍️ **Write once. Deploy to any agent platform.**

The ecosystem for AI agents is fragmenting fast. Every major platform — Claude Code, GitHub Copilot, Cursor, OpenAI Codex, Google Gemini, Amazon Q — requires its own format for instructions, skills, and prompts. We are rewriting the same artifacts over and over again, and there is no lock file, no registry, no reproducibility.

🐳 **UAAPS is the open standard that fixes this.** It is to AI agent artifacts what Docker is to containers and npm is to JavaScript packages.

---

## 🚧 The Problem

Managing AI agent artifacts at scale is a structural mess:

- 🔒 **No portability** — skills written for Claude Code cannot be dropped into Copilot or Cursor without manual rework.
- 🎲 **No reproducibility** — there is no lock file ensuring an agent behaves identically on your laptop and in CI.
- 💥 **No composability** — loading multiple packages causes silent instruction conflicts with no namespacing.
- ⚙️ **No lifecycle management** — platforms lack hooks for events like `pre-tool-use` or `permission-request` that deterministic agent validation requires.
- 🔄 **No update management** — there is no standard mechanism to propagate fixes or new versions of skills across projects; every consumer must update manually and in isolation.
- 🧪 **No testability** — there is no standard way to verify that a skill or hook works correctly. Scripts lack test harnesses, and LLM-driven behavior has no eval framework — breakage is discovered in production.
- 🛡️ **Uncontrolled duplication creates security risk** — skills are copy-pasted across repositories with no traceability. Since skills can contain executable scripts, a single compromised or outdated copy can go undetected across dozens of projects, expanding the attack surface with every duplicate.

### 🤔 Why "Hacking" npm Is Not Enough

It is tempting to drop prompts into a JavaScript package, but npm fails three critical agent requirements:

| Requirement | npm | UAAPS |
|---|---|---|
| ⚡ **Context efficiency** | Installs everything locally at once | Progressive Disclosure — metadata first, full instructions only on activation |
| 🪝 **Agent lifecycle hooks** | Limited to `postinstall` scripts | First-class hooks for `pre-tool-use`, `permission-request`, and more |
| 🌐 **Cross-language dependencies** | JS packages only | System Dependencies with pre-flight checks for `python`, MCP servers, and OS binaries |

---

## 💡 The Solution

UAAPS defines a **filesystem-first, portable standard** for packaging AI agent artifacts:

```
package-name/
├── package.agent.json   # Single required manifest
├── package.agent.lock   # Reproducible installs
├── skills/              # Agent skills
├── commands/            # Slash commands
├── agents/              # Sub-agent definitions
├── rules/               # Project rules / instructions
├── hooks/               # Lifecycle hooks
└── mcp/                 # MCP server configs
```

A single `package.agent.json` manifest declares your artifact and makes it consumable by any compliant platform:

```jsonc
{
  "name": "@my-org/code-review",
  "version": "1.0.0",
  "description": "Code review skills and commands",
  "engines": {
    "claude-code": ">=1.0",
    "cursor": ">=2.2",
    "copilot": "*",
    "codex": "*"
  },
  "dependencies": {
    "testing-utils": "^2.1.0"
  }
}
```

### 🏛️ Core Design Principles

| Principle | What It Means |
|---|---|
| 🌍 **Portability** | One package works across all compliant platforms |
| 🔍 **Progressive Disclosure** | Metadata loads first; instructions load on activation |
| 🔒 **Deterministic Resolution** | Lock files guarantee reproducible agent behavior in CI |
| 🧩 **Composability** | Namespaced skills prevent conflicts between packages |
| 📁 **Filesystem-First** | No databases, no APIs required — just files |
| 🧪 **Testability** | Two-tier testing: deterministic `tests/` for CI + LLM-judged `evals/` for integration |

---



## 🏗️ Reference Implementation: Skills Concentrator

To prove the standard works, a reference implementation and management layer are being built under the **Agent Package Manager (AAM)** project.

🚀 The **Skills Concentrator** is now implemented and ready for testing. It allows agents to consume remote skills seamlessly — solving the problem of distributing and versioning capabilities across multiple providers without requiring a local copy of every artifact.

- 🛠️ **AAM repository:** [github.com/spazyCZ/agent-package-manager](https://github.com/spazyCZ/agent-package-manager/tree/test?tab=readme-ov-file#1-use-remote-skills)


---

## 📖 Full Specification

The specification is published at **[uaaps.github.io/uaaps_docs](https://uaaps.github.io/uaaps_docs/)** and covers:

- 📄 Package manifest schema (`package.agent.json`)
- 🗂️ Directory structure conventions
- 🔗 Dependency resolution and lock file format
- 🪝 Lifecycle hooks
- ✅ System dependency pre-flight checks
- 🛡️ Permission model
- 🧪 Quality / eval definitions
- 🔌 Vendor extension points (`x-claude`, `x-cursor`, …)

The spec source lives in [`docs/spec/`](docs/spec/) — each chapter is a separate Markdown file. To build and preview locally:

```bash
.venv/bin/python -m pip install mkdocs-material
.venv/bin/python -m mkdocs serve
```

---

## 🤝 Call for Collaboration

Agent artifact management is the next foundational layer of AI-augmented software engineering. **This standard only succeeds if it is built together.**

We are looking for collaborators who are:

- 🤖 **Building with AI agents** — and hitting the same portability and reproducibility walls.
- 🏢 **Platform developers** — working on agent runtimes who want to adopt or shape a common standard.
- 🔧 **Tooling authors** — building registries, package managers, or IDEs for the agentic ecosystem.
- 🔬 **Spec reviewers** — with experience in package management, developer tooling, or open standards.

### 🚀 How to Contribute

1. 📖 **Read the spec** — [docs/spec/full.md](docs/spec/full.md)
2. 💬 **Open an issue** — Feedback, edge cases, missing scenarios
3. 🔀 **Submit a PR** — Spec improvements, examples, reference implementations
4. 🧪 **Test the Skills Concentrator** — [How to use remote skills](https://github.com/spazyCZ/agent-package-manager/tree/main?tab=readme-ov-file#quick-start)
5. 🗣️ **Start a discussion** — Architecture decisions, cross-platform compatibility

Let's stop building silos and start building a standard. 🌱

---

## 🔬 Research & Background

Additional context and prior-art analysis is available in [work/research_list_of_sources.md](docs/research_list_of_sources.md).

---

## 📜 License

Open specification. See [LICENSE](LICENSE) for details.
