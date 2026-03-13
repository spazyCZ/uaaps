## 13. Dependency Resolution

Dependency resolution is a core differentiator of UAAPS over raw vendor plugin systems. Like npm, pip, or Cargo, the package manager MUST resolve a directed acyclic graph (DAG) of requirements before installation.

### 13.1 Dependency Types

| Type | Manifest Key | Required to Install? | Semantics |
|------|-------------|---------------------|-----------|
| **Direct** | `dependencies` | **Yes** — install fails | Other agent packages this package requires to function. |
| **Optional** | `optionalDependencies` | No — install succeeds | Enhances functionality if present. Graceful degradation if absent. |
| **Peer** | `peerDependencies` | **Yes** — warn/fail | MUST be provided by the host project or another top-level install. Not auto-installed. |
| **Tooling group** | `dependencyGroups` | No — opt-in only | Named, non-runtime dependency sets for workflows such as `dev`, `docs`, or `eval`. |
| **Feature selector** | `extras` | No — opt-in only | Public, consumer-facing feature bundles such as `ocr`, `github`, or `enterprise`. |
| **System** | `systemDependencies` | **Yes** — pre-flight check | OS binaries, language runtimes, pip/npm packages, MCP servers. |

#### Non-runtime Dependency Groups

Dependency groups let package authors declare workflow-specific dependencies without making them part of the runtime dependency contract.

```jsonc
{
  "dependencyGroups": {
    "dev": {
      "description": "Local authoring and CI tooling",
      "dependencies": {
        "test-harness": "^2.0.0",
        "lint-utils": "^1.4.0"
      }
    },
    "docs": {
      "dependencies": {
        "site-preview": "^1.2.0"
      },
      "systemDependencies": {
        "packages": {
          "npm": ["markdownlint-cli@^0.41"]
        }
      }
    }
  }
}
```

**Semantics:**
- `aam install` MUST install only runtime dependencies by default.
- `aam install --with dev,docs` MUST include the selected dependency groups in the root requirement set.
- `aam install --only-group docs` MUST install only the named dependency group plus the root package itself.
- Group dependencies MUST be non-transitive package metadata. When another package depends on this package, its `dependencyGroups` MUST be ignored unless the consumer selects matching groups locally.
- When a selected group declares `systemDependencies`, those checks MUST participate in the same pre-flight flow as top-level `systemDependencies`.
- Tooling group names are author-defined, but `dev`, `docs`, and `eval` are RECOMMENDED conventions.

#### Extras — Optional Feature Bundles

Extras let package authors publish consumer-facing feature bundles without making them part of the base install.

```jsonc
{
  "extras": {
    "ocr": {
      "description": "OCR support for scanned PDFs",
      "dependencies": {
        "image-tools": "^2.0.0"
      },
      "systemDependencies": {
        "packages": {
          "apt": ["tesseract-ocr"]
        }
      },
      "artifacts": {
        "skills": ["ocr-reader"]
      },
      "permissions": {
        "shell": {
          "allow": true,
          "binaries": ["tesseract"]
        }
      }
    },
    "github": {
      "dependencies": {
        "gh-integration": "^1.5.0"
      },
      "artifacts": {
        "agents": ["github-reviewer"]
      },
      "permissions": {
        "network": {
          "hosts": ["api.github.com"],
          "schemes": ["https"]
        }
      }
    }
  }
}
```

**Semantics:**
- `aam install` MUST install the base package without extras unless extras are requested explicitly.
- `aam install --extras ocr,github` MUST include the selected extras for the root package.
- `aam install @myorg/pdf-tools[ocr,github]` MUST add the package as a direct dependency with those extras selected.
- Extras MUST be additive. Selecting an extra MUST NOT remove or replace base dependencies.
- Extras MUST be root-selected only. A package manifest MUST NOT request extras of its transitive dependencies.
- When a selected extra declares `artifacts`, those artifact names MUST be activated for the root package in addition to the base artifact set.
- When a selected extra declares `permissions`, those permissions MUST be unioned with the root package's base `permissions` request before platform enforcement is applied.
- Tools MUST surface extra-selected permissions to the user at install time so the requested capability expansion is reviewable.
- In this version of the spec, extras MAY affect dependency resolution, system dependency checks, root-package artifact activation, and root-package permissions. They MUST NOT alter manifest parsing semantics or the dependency metadata of transitive packages.
- Extra names are author-defined, but short capability names such as `ocr`, `github`, and `enterprise` are RECOMMENDED.

### 13.2 Version Constraint Syntax

UAAPS uses **SemVer 2.0** with npm-style range operators:

| Syntax | Meaning | Example |
|--------|---------|---------|
| `1.2.3` | Exact version | Only `1.2.3` |
| `^1.2.3` | Compatible with (`>=1.2.3 <2.0.0`) | Minor + patch updates OK |
| `~1.2.3` | Approximately (`>=1.2.3 <1.3.0`) | Patch updates only |
| `>=1.2.0` | Minimum version | `1.2.0` or higher |
| `>=1.0.0 <3.0.0` | Range | Between `1.0.0` and `2.x.x` |
| `*` | Any version | No constraint |

### 13.3 Lock File — `package.agent.lock`

After resolution, the solver writes a deterministic **lock file** pinning exact versions:

```jsonc
{
  "lockVersion": 2,
  "resolved": {
    "code-review": {
      "version": "1.3.2",
      "source": {
        "type": "registry",
        "registry": "https://aamregistry.io",
        "name": "code-review",
        "version": "1.3.2",
        "tarball": "https://aamregistry.io/packages/code-review/1.3.2/tarball",
        "display": "registry:code-review"
      },
      "integrity": "sha256-abc123...",
      "dependencies": {
        "git-utils": "1.0.1"
      }
    },
    "testing-utils": {
      "version": "2.4.0",
      "source": {
        "type": "registry",
        "registry": "https://aamregistry.io",
        "name": "testing-utils",
        "version": "2.4.0",
        "resolvedTag": "latest",
        "tarball": "https://aamregistry.io/packages/testing-utils/2.4.0/tarball",
        "display": "registry:testing-utils"
      },
      "integrity": "sha256-def456..."
    },
    "git-utils": {
      "version": "1.0.1",
      "source": {
        "type": "registry",
        "registry": "https://aamregistry.io",
        "name": "git-utils",
        "version": "1.0.1",
        "tarball": "https://aamregistry.io/packages/git-utils/1.0.1/tarball",
        "display": "registry:git-utils"
      },
      "integrity": "sha256-ghi789..."
    }
  },
  "systemChecks": {
    "python": "3.12.1",
    "binaries": {
      "git": "/usr/bin/git",
      "docker": "/usr/local/bin/docker"
    },
    "pip": {
      "pypdf": "4.1.0",
      "pdfplumber": "0.11.4"
    }
  }
}
```

#### Structured Source Metadata

Each resolved package entry MUST store `source` as a structured object in lock file version `2` and later.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | `string` | **Yes** | Source kind. Supported values in this version: `registry`, `file`, `bundle`. |
| `registry` | `string` | For `registry` | Canonical registry base URL used for resolution. |
| `name` | `string` | For `registry` | Canonical package identifier. |
| `version` | `string` | For `registry` | Exact resolved package version. |
| `resolvedTag` | `string` | No | Dist-tag requested by the user, if resolution originated from a tag such as `stable` or `latest`. |
| `tarball` | `string` | For `registry` | Exact artifact URL fetched by the resolver. |
| `path` | `string` | For `file` or `bundle` | Canonical local path or URI of the selected artifact. |
| `target` | `string` | For `bundle` | Target platform encoded by the bundle, if applicable. |
| `display` | `string` | No | Backward-compatible human-readable shorthand. Informational only. |

**Rules**:
- The `source` object MUST be authoritative. Tools MUST NOT derive canonical resolution semantics from `display`.
- Tools writing lock file version `2` MUST emit the structured object form.
- Tools SHOULD continue reading legacy lock file version `1` entries where `source` is a string.
- A tool that rewrites a legacy lock file SHOULD upgrade it to `lockVersion: 2` and preserve the old string only in `source.display` when useful for diagnostics.

When dependency groups are selected during lock generation, the lock file SHOULD record them in a top-level `groups` object so the selected workflow inputs remain reproducible.
When extras are selected during lock generation, the lock file SHOULD record them in a top-level `selectedExtras` object so feature selections remain reproducible. When an extra contributes artifacts or permissions, the lock file SHOULD record those effective additions as derived install metadata.

**Behavior**:
- If `package.agent.lock` exists, install uses **locked versions** (reproducible builds).
- `aam install` reads the lock file; `aam update` regenerates it.
- Lock files SHOULD be committed to version control for deterministic environments.
- Tools MUST support reading lock file version `1` and `2`. Tools writing new lock files SHOULD write version `2`.

### 13.4 Resolution Algorithm

The resolution algorithm is versioned via the `resolverVersion` field in `package.agent.json` (default: `1`). This allows future changes to resolution semantics without breaking existing packages. Implementations MUST support resolver version 1; higher versions are additive.

Resolution proceeds through five named phases:

**Phase 1: Parse** — Read root `package.agent.json`. If `package.agent.lock` exists and `--frozen` is set, skip to Phase 5 (locked install).

**Phase 2: Build requirement graph** — Collect all `dependencies`, `peerDependencies`, and `optionalDependencies` into an initial requirement DAG. If the user selected any `dependencyGroups`, collect those groups' `dependencies` and `systemDependencies` into the same root requirement set. If the user selected any `extras`, collect those extras' `dependencies` and `systemDependencies` into the same root requirement set. Each dependency edge carries a version constraint.

**Phase 3: Resolve** — For each unresolved requirement, in topological order:

  a. `fetch_versions` — Query the registry (or cache) for all published, non-yanked versions.
  b. `filter_by_constraint` — Discard versions not matching the constraint.
  c. `select_version` — Pick the highest compatible version using this priority:
     1. Locked version (from `package.agent.lock`, if not `--latest`)
     2. `resolutions` override (from root manifest)
     3. Highest stable version satisfying the constraint
     4. Highest pre-release version (only if the constraint explicitly includes a pre-release identifier, e.g. `^1.0.0-beta.1`)
  d. `resolve_transitive` — Recursively resolve the selected version's own dependencies.

**Phase 4: Validate**

  a. `detect_cycles` — The resolved graph MUST be acyclic. Circular dependencies are a fatal error.
  b. `check_conflicts` — If two packages require incompatible versions of the same dependency, apply the conflict resolution strategy (see §13.9).
  c. `check_peers` — Each `peerDependency` MUST be satisfied by the root package or an ancestor in the graph. Unmet peers produce a warning (or error with `--strict-peers`).
  d. `check_system` — Run system dependency pre-flight checks for all resolved packages.

**Phase 5: Commit**

  a. Write `package.agent.lock` (unless `--frozen`, in which case verify existing lock matches resolved graph exactly).
  b. Install resolved packages to the install directory (§13.11).
  c. Hoist shared dependencies where version-compatible (§13.11).

#### Pre-release Version Handling

Pre-release versions (e.g. `1.0.0-beta.1`) follow SemVer 2.0 precedence rules:

| Constraint | Matches `1.0.0-beta.1`? | Matches `1.0.0`? | Rationale |
|------------|--------------------------|-------------------|-----------|
| `^1.0.0` | **No** | Yes | Caret ranges exclude pre-releases unless the constraint itself has a pre-release tag |
| `^1.0.0-beta.0` | **Yes** | Yes | Constraint includes pre-release tag, so pre-releases within the same major.minor.patch are eligible |
| `>=1.0.0-beta.1` | **Yes** | Yes | Explicit comparison includes the pre-release |
| `*` | **Yes** | Yes | Wildcard matches everything |

This matches npm's pre-release semantics: a range like `^1.0.0` is understood as requesting stable releases, while `^1.0.0-beta.0` explicitly opts into the pre-release channel.

#### Resolver Version

```jsonc
// package.agent.json
{
  "resolverVersion": 1
}
```

| Version | Behavior |
|---------|----------|
| `1` (default) | Current algorithm as defined above. |
| Future versions | Will be defined in subsequent spec revisions. Implementations encountering an unknown resolver version MUST emit an error and refuse to resolve. |

The `resolverVersion` field is OPTIONAL. When absent, version `1` is assumed.

### 13.5 Resolution Strategies

| Strategy | Flag | Behavior |
|----------|------|----------|
| **Default** | (none) | Highest compatible version per constraint. |
| **Locked** | `--frozen` | Only use versions from lock file. Fail if lock is stale. |
| **Minimal** | `--minimal` | Lowest version satisfying each constraint (CI reproducibility). |
| **Latest** | `--latest` | Ignore constraints, install latest of everything (dangerous). |
| **Dry-run** | `--dry-run` | Resolve and display tree without installing. |

#### Group Selection Flags

| Flag | Behavior |
|------|----------|
| `--with <group>[,<group>...]` | Include one or more named dependency groups in addition to runtime dependencies. |
| `--only-group <group>[,<group>...]` | Install only the selected dependency groups for the root package, skipping runtime dependencies. |

If a requested group name does not exist in `package.agent.json`, the tool MUST fail before resolution begins.

#### Extra Selection Syntax

| Form | Behavior |
|------|----------|
| `aam install --extras <extra>[,<extra>...]` | Select extras for the root package in the current workspace. |
| `aam install <pkg>[<extra>,<extra>]` | Add a direct dependency with explicit extras selected for that package. |

If a requested extra name does not exist for the targeted package, the tool MUST fail before resolution begins.

#### Extra-scoped Artifact Activation

Extras MAY activate additional artifacts from the root package, but only through the explicit top-level `artifacts` registry.

| Rule | Requirement |
|------|-------------|
| Extra artifact references are by artifact `name` | MUST |
| Referenced artifact names MUST exist in the top-level `artifacts` registry | MUST |
| Selected extras MAY activate additional `skills`, `agents`, or `commands` | MAY |
| Selected extras MUST NOT deactivate base artifacts | MUST NOT |
| Artifact activation by extras applies only to the root package in this version | MUST |

#### Extra-scoped Permissions

Extras MAY request additional permissions for the root package.

| Rule | Requirement |
|------|-------------|
| Extra `permissions` MUST use the same schema as top-level `permissions` | MUST |
| Effective root permissions are the union of base `permissions` and all selected extra permissions | MUST |
| Tools MUST display extra-added permissions to the user before install completes | MUST |
| Platforms MUST enforce the effective root permission set at runtime | MUST |
| Extra-added permissions apply only to the root package in this version | MUST |

### 13.6 Package Dependencies (Agent-to-Agent)

Agent packages can depend on other agent packages. This enables composition:

```jsonc
// package.agent.json for "full-stack-review"
{
  "name": "full-stack-review",
  "version": "1.0.0",
  "dependencies": {
    "code-review": "^1.0.0",           // Reuse code review skills
    "security-scanning": "^2.0.0",     // Reuse security agents
    "testing-utils": "^1.5.0"          // Reuse test generation
  }
}
```

**Transitive resolution**: If `code-review@1.3.2` itself depends on `git-utils@^1.0.0`, the solver installs `git-utils` automatically.

**Namespace isolation**: Each installed dependency's skills, commands, and agents are namespaced under `dependency-name:artifact-name` to prevent collisions.

### 13.7 System Dependencies

System dependencies are **not installed** by the package manager — they are **verified** via pre-flight checks.

```jsonc
"systemDependencies": {
  // === Runtime requirements ===
  "python": ">=3.10",                  // Check: python3 --version
  "node": ">=18",                      // Check: node --version

  // === Package manager installs ===
  "packages": {
    "pip": ["pypdf>=4.0", "pdfplumber"],
    "npm": ["prettier@^3.0", "eslint"],
    "brew": ["graphviz", "poppler"],
    "apt": ["poppler-utils", "tesseract-ocr"],
    "cargo": ["ripgrep"]
  },

  // === Binary availability ===
  "binaries": ["git", "docker", "gh"],  // Must exist on $PATH

  // === MCP server requirements ===
  "mcp-servers": ["filesystem", "github"]
}
```

#### Pre-flight Check Flow

```
aam install my-package
  → Checking system dependencies...
  ✓ python 3.12.1 (>=3.10)
  ✓ node 20.11.0 (>=18)
  ✓ git found at /usr/bin/git
  ✗ docker not found on $PATH
  ✓ pip: pypdf 4.1.0 installed
  ✗ pip: pdfplumber not installed
  
  ⚠ Missing system dependencies:
    • docker: Install via https://docs.docker.com/get-docker/
    • pdfplumber: Run `pip install pdfplumber`
  
  Install package anyway? [y/N/auto-install]
```

#### Auto-Install Behavior

| Package Manager | Auto-install | Flag |
|----------------|-------------|------|
| `pip` | ✅ Supported | `--install-system-deps` |
| `npm` | ✅ Supported | `--install-system-deps` |
| `brew` / `apt` | ⚠️ Prompt only | Requires `--install-system-deps --allow-sudo` |
| `binaries` | ❌ Never | Manual install instructions provided |
| `mcp-servers` | ✅ Via MCP registry | `--install-system-deps` |

### 13.8 Skill-Level Dependencies

Individual skills can declare their own dependencies in SKILL.md frontmatter, complementing the package-level manifest:

```yaml
---
name: pdf-processing
description: Extract text and fill forms in PDFs.
compatibility:
  requires:
    - python>=3.10
    - pypdf>=4.0
    - pdfplumber
  packages:                          # UAAPS extension
    - code-review/git-utils          # Depends on another skill
  optional:
    - tesseract-ocr                  # For OCR, graceful degradation
---
```

**Resolution order**:
1. Package-level `systemDependencies` checked first (covers most skills).
2. Skill-level `compatibility.requires` checked on activation (lazy check).
3. Skill-level `compatibility.packages` resolved transitively if referencing other skills.

### 13.9 Conflict Resolution

#### Version Conflicts

When two packages require incompatible versions of the same dependency:

```
my-app
├── code-review@1.3.2
│   └── git-utils@^1.0.0  →  resolves to 1.0.5
└── deployment@2.0.0
    └── git-utils@^2.0.0  →  resolves to 2.1.0   ← CONFLICT
```

**Resolution strategies**:

| Strategy | Behavior | When to Use |
|----------|----------|-------------|
| **Fail** (default) | Error, refuse to install | Safety-first environments |
| **Duplicate** | Install both versions, isolate namespaces | When skills don't share state |
| **Override** | Use `resolutions` field to force a version | When you know compatibility holds |
| **Peer promote** | Lift to `peerDependencies`, let host decide | Library packages |

#### Resolution Overrides

```jsonc
// package.agent.json
{
  "resolutions": {
    "git-utils": "2.1.0"              // Force all transitive refs to this version
  }
}
```

#### Skill Name Conflicts

When two installed packages export skills with the same `name`:

- Skills are **namespaced** by package: `code-review:lint-check` vs `testing:lint-check`.
- If a skill is referenced without namespace, the **closest scope wins** (project > personal > plugin).
- Explicit disambiguation: `aam resolve skill lint-check` lists all matches.

### 13.10 Dependency Tree Commands

| Command | Description |
|---------|-------------|
| `aam install` | Install all dependencies from manifest (or lock file). |
| `aam install --with dev,docs` | Install runtime dependencies plus the selected dependency groups. |
| `aam install --only-group eval` | Install only the selected non-runtime dependency group for the root package. |
| `aam install --extras ocr,github` | Install the root package with the selected optional feature bundles. |
| `aam install @myorg/pdf-tools[ocr]` | Add a direct dependency with explicit extras selected. |
| `aam install <pkg>` | Add a package and resolve. |
| `aam update` | Re-resolve all dependencies, update lock file. |
| `aam update <pkg>` | Update a single package within constraints. |
| `aam tree` | Display the full resolved dependency tree. |
| `aam tree --depth=1` | Show only direct dependencies. |
| `aam why <pkg>` | Explain why a package is installed (which parent requires it). |
| `aam check` | Verify all dependencies are installed and compatible. |
| `aam check --system` | Run system dependency pre-flight checks only. |
| `aam outdated` | List packages with newer versions available. |
| `aam audit` | Check dependencies for known security issues. |
| `aam dedupe` | Flatten tree, remove unnecessary duplicates. |
| `aam prune` | Remove installed packages not in the dependency tree. |

### 13.11 Install Directory Layout

```
.agent-packages/                       # Install root (analogous to node_modules/)
├── .lock                              # package.agent.lock (symlink or copy)
├── code-review/                       # Installed package
│   ├── package.agent.json
│   ├── skills/
│   │   └── lint-review/
│   │       └── SKILL.md
│   └── commands/
│       └── review.md
├── testing-utils/                     # Transitive dependency
│   ├── package.agent.json
│   └── skills/
│       └── test-gen/
│           └── SKILL.md
└── git-utils/                         # Shared dependency (hoisted)
    ├── package.agent.json
    └── skills/
        └── git-diff/
            └── SKILL.md
```

**Hoisting**: Like npm, shared dependencies are hoisted to the top level of `.agent-packages/` when version-compatible. This reduces duplication and simplifies skill discovery.

### 13.12 Dependency Resolution in CI/CD

For reproducible builds in CI environments:

```yaml
# .github/workflows/agent-check.yml
- name: Install agent packages (frozen)
  run: aam install --frozen

- name: Verify system dependencies
  run: aam check --system

- name: Audit for security issues
  run: aam audit --severity=high
```

The `--frozen` flag ensures CI uses exactly the versions from `package.agent.lock`, failing if the lock file is out of date.

### 13.13 Install Modes & Package Cache

The `aam` CLI supports three install modes. The mode determines where resolved packages are written and whether a shared cache is consulted.

#### Install Mode Overview

| Mode | Install Location | Lock File Used | Typical Use |
|------|-----------------|---------------|-------------|
| **Project-local** (default) | `./.agent-packages/` | `./package.agent.lock` | Per-project installs, checked into VCS |
| **Global** | `~/.aam/packages/` | `~/.aam/global.lock` | User-wide utilities, not project-specific |
| **CI ephemeral** | `$AAM_INSTALL_DIR` or system temp | `./package.agent.lock` (read-only) | Fresh environment per pipeline run |

```bash
aam install                          # Project-local (default)
aam install --global                 # Global install
aam install --frozen                 # Project-local, lock file is authoritative
AAM_INSTALL_DIR=/tmp/aam aam install # CI ephemeral override
```

#### Installing from a Specific Vendor

When multiple vendors publish a package with the same base name, always qualify with the `@scope` prefix to target the correct author:

```bash
# Unscoped — installs the community package named "code-review"
aam install code-review

# Scoped — installs @myorg's code-review, not @otherorg's
aam install @myorg/code-review
aam install @otherorg/code-review

# Specific version from a specific vendor
aam install @myorg/code-review@1.3.2
aam install @myorg/code-review@^1.0.0

# Specific dist-tag from a vendor (e.g. enterprise-approved tag)
aam install @myorg/code-review@stable
aam install @myorg/code-review@bank-approved

# From a private registry owned by a specific vendor
aam install @myorg/code-review --registry https://pkg.myorg.internal/aam
```

Both `@myorg/code-review` and `@otherorg/code-review` can be installed simultaneously — they are stored under `myorg--code-review/` and `otherorg--code-review/` respectively (see §12.2 and §13.13 author collision rule). Skills from each are namespaced accordingly: `myorg--code-review:lint-check` vs `otherorg--code-review:lint-check`.

#### Shared Download Cache

All install modes share a **download cache** at `~/.aam/cache/`. The cache stores downloaded `.aam` archives keyed by `package@version` + integrity hash. Packages are never re-downloaded if a valid cached copy exists.

```
~/.aam/
├── cache/                                        # Shared download cache (all modes)
│   ├── myorg--code-review-1.3.2-sha256-abc123.aam
│   ├── otherorg--code-review-2.0.0-sha256-def456.aam
│   └── testing-utils-2.4.0-sha256-ghi789.aam
├── packages/                                     # Global install root
│   ├── myorg--code-review/                       # @myorg/code-review
│   ├── otherorg--code-review/                    # @otherorg/code-review
│   └── testing-utils/                            # unscoped package
├── global.lock                                   # Global install lock file
└── config.yaml                                   # CLI configuration
```

**Author collision rule**: The `@scope/name` → `scope--name` filesystem mapping (defined in §12.2) is the sole mechanism preventing collisions between same-named packages from different authors. Two packages that share a base name but differ in scope (`@myorg/code-review` and `@otherorg/code-review`) MUST be stored under distinct directory names (`myorg--code-review/` and `otherorg--code-review/` respectively) in both the cache and global packages directory. Implementations MUST NOT flatten scoped names to their bare base name in any install root.

#### Cache Invalidation Rules

| Trigger | Behaviour |
|---------|-----------|
| Integrity hash mismatch | Cache entry is rejected; package re-downloaded |
| `aam cache clean` | Entire cache cleared |
| `aam cache clean <pkg>@<version>` | Single entry evicted |
| `aam install --no-cache` | Cache bypassed for this run; result not cached |
| Registry serves different bytes for same `name@version` | Client MUST treat this as an integrity violation and reject the artifact |

Cached archives MUST be verified against their `sha256` integrity hash before extraction. A corrupted or tampered archive MUST be rejected and the cache entry evicted.

#### Lock File Precedence Rules

When both a user-level (`~/.aam/global.lock`) and project-level (`./package.agent.lock`) lock file exist, resolution follows this precedence order (highest wins):

1. **`./package.agent.lock`** — project lock file always takes precedence for project-local installs.
2. **`~/.aam/global.lock`** — governs `--global` installs only; never consulted for project-local resolution.
3. **`resolutions` field** in `package.agent.json` — overrides both lock files for named packages (see §13.9).

Implementations MUST NOT mix project-local and global lock files in the same resolution pass. A global install (`--global`) MUST NOT read or modify `./package.agent.lock`.

#### Global vs Local Resolution Rules

| Rule | Project-local | Global |
|------|--------------|--------|
| Artifact lookup scope | `./.agent-packages/` only | `~/.aam/packages/` only |
| Lock file written | `./package.agent.lock` | `~/.aam/global.lock` |
| Hoisting target | `./.agent-packages/` root | `~/.aam/packages/` root |
| Peer dependency resolution | Ancestor chain within project tree | Global install tree |
| Available to agent platforms | Only when agent runs in project directory | Always — platform SHOULD scan `~/.aam/packages/` |

Agent platforms SHOULD load globally installed packages as a lower-priority fallback after project-local packages. Project-local packages MUST shadow global packages of the same name.

### 13.14 Workspaces

Workspaces enable monorepo development — multiple UAAPS packages in a single repository sharing a common dependency tree and lock file.

#### Workspace Configuration

Place a `workspace.agent.json` file at the monorepo root. The `packages` array uses glob patterns to locate member packages:

```json
{
  "workspace": {
    "packages": ["packages/*", "internal/*"]
  }
}
```

Each glob pattern MUST resolve to directories containing a valid `package.agent.json`.

#### Workspace Rules

| Rule | Requirement |
|------|-------------|
| `workspace.agent.json` MUST be at the repository root | MUST |
| Each glob pattern MUST resolve to directories containing `package.agent.json` | MUST |
| All workspace members share a single `package.agent.lock` at the root | MUST |
| Inter-workspace dependencies are resolved from the local filesystem (no registry fetch) | MUST |
| Workspace members MAY have different versions | MAY |
| `aam install` at the root installs dependencies for ALL members | MUST |
| `aam install` within a member directory installs only that member's dependencies | SHOULD |

#### CLI Commands

```bash
aam install                        # Install all workspace members
aam install --workspace packages/my-pkg  # Install specific member
aam test --workspace               # Run tests across all members
aam publish --workspace            # Publish all changed members
aam workspace list                 # List all workspace members
```

#### Lock File Behavior

The workspace shares a single `package.agent.lock` at the root. All members' dependencies are resolved together, ensuring version consistency across the monorepo. The lock file includes a `workspaceMembers` field listing each member and its resolved dependencies.
