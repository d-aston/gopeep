

<h1 align="center">gopeep</h1>

<p align="center">
  <strong>nothing to see here</strong>
</p>

<p align="center">
  <a href="https://github.com/d-aston/gopeep/stargazers"><img src="https://img.shields.io/github/stars/JuliusBrussee/gopeep?style=flat&color=yellow" alt="Stars"></a>
  <a href="https://github.com/JuliusBrussee/gopeep/commits/main"><img src="https://img.shields.io/github/last-commit/JuliusBrussee/gopeep?style=flat" alt="Last Commit"></a>
  <a href="LICENSE"><img src="https://img.shields.io/github/license/JuliusBrussee/gopeep?style=flat" alt="License"></a>
</p>

<p align="center">
  <a href="#install">Install</a> •
  <a href="#gopeep-repo">gopeep-repo</a> •
  <a href="#gopeep-web">gopeep-web</a> •
  <a href="#output">Output</a> •
  <a href="#customization">Customization</a>
</p>

---

Two agent skills that analyze any codebase or website and produce structured specification files — plus a human-editable `00-customization.md` where you can adjust architecture decisions, colors, data models, and tech choices before handing the spec to an agent to recreate the project.

**Input:** A repo path, website URL, or pasted HTML.  
**Output:** A `gopeep-spec/` directory ready to hand to a recreating agent.

## Before / After

**Without gopeep:**
> "Recreate this project" → agent reads some files, guesses at structure, misses patterns, asks 20 questions, produces something that vaguely resembles the original.

**With gopeep:**
> `/gopeep-repo ./my-project` → structured spec with architecture diagrams, pattern catalog, deployment config, ordered build plan, and a `00-customization.md` you can edit before recreation. Hand the folder to any agent. Done.

---

## Install

Pick your agent. One command.

| Agent | Install |
|-------|---------|
| **Claude Code** | `claude plugin marketplace add d-aston/gopeep && claude plugin install gopeep@gopeep-repo` |
| **Cursor** | `npx skills add d-aston/gopeep -a cursor` |
| **Windsurf** | `npx skills add d-aston/gopeep -a windsurf` |
| **Copilot** | `npx skills add d-aston/gopeep -a github-copilot` |
| **Cline** | `npx skills add d-aston/gopeep -a cline` |
| **Any other** | `npx skills add d-aston/gopeep` |

Uninstall: `npx skills remove gopeep`

> **Windows note:** `npx skills` uses symlinks by default. If symlinks fail, add `--copy`: `npx skills add d-aston/gopeep --copy`

---

## gopeep-repo

Analyze any software repository — local path or remote URL.

```bash
/gopeep-repo ./my-project
/gopeep-repo ./my-project --depth quick          # recon + deps only
/gopeep-repo ./my-project --output ./spec        # custom output dir
/gopeep-repo ./mono --modules auth,api,worker    # specific modules only
/gopeep-repo ./my-project --resume 4             # resume from phase 4
/gopeep-repo ./my-project --diff ./spec-v1       # diff against previous spec
```

**Six analysis phases:**

| Phase | Output | What it captures |
|-------|--------|-----------------|
| 1 | `01-reconnaissance.md` | Repo shape, languages, CI/CD, complexity tier |
| 2 | `02-stack-and-dependencies.md` | All frameworks, deps, language versions, infra deps |
| 3 | `03-architecture.md` | Architecture style, module boundaries, data layer, API surface |
| 4 | `04-patterns-and-conventions.md` | Naming, error handling, testing patterns, type system |
| 5 | `05-infrastructure-and-deployment.md` | Docker, K8s, CI/CD, env vars, observability |
| 6 | `06-reproduction-blueprint.md` | Ordered build plan with exact commands and verification steps |

Plus `00-customization.md` — the human-editable override file.

**Scale-aware:** Works on 10-file scripts and million-line monorepos. Massive repos use hierarchical decomposition with sub-agents per module and grep-first pattern detection. Never reads more than ~100 files per agent pass.

---

## gopeep-web

Analyze any website — source files, a URL, or pasted HTML.

```bash
/gopeep-web ./src                          # from source directory
/gopeep-web ./index.html                   # single HTML file
/gopeep-web https://example.com            # live URL
# or paste HTML directly into the conversation
```

**Five analysis phases:**

| Phase | Output | What it captures |
|-------|--------|-----------------|
| 1 | `01-reconnaissance.md` | Page type, rendering model, framework signals, design system |
| 2 | `02-stack-and-dependencies.md` | Framework, build tool, CSS approach, state management |
| 3 | `03-architecture.md` | Component tree, layout regions, data patterns, routing |
| 4 | `04-style-system.md` | Complete design token system — colors, type, spacing, motion |
| 5 | `05-reproduction-blueprint.md` | Ordered component build plan with exact interfaces |

Plus `00-customization.md` — swap colors, fonts, routes, and data shapes before recreation.

---

## Output

```
gopeep-spec/
├── 00-customization.md          ← edit this before recreation
├── 01-reconnaissance.md
├── 02-stack-and-dependencies.md
├── 03-architecture.md
├── 04-patterns-and-conventions.md   (repo) / 04-style-system.md (web)
├── 05-infrastructure-and-deployment.md   (repo) / 05-reproduction-blueprint.md (web)
├── 06-reproduction-blueprint.md   (repo only)
└── diagrams/
    ├── architecture-overview.mmd
    ├── data-flow.mmd
    └── ...
```

All architectural diagrams use Mermaid. Embedded inline in `.md` files and saved as standalone `.mmd` files.

---

## Customization

The `00-customization.md` file is the key innovation. Every value gopeep extracts can be overridden before recreation — without touching the analysis files.

**For repos:**
- Swap databases, frameworks, cloud providers
- Rename entities and API endpoints
- Change deployment targets and CI/CD platforms
- Override naming conventions
- Edit data models

**For websites:**
- Change the entire color palette in one table
- Swap fonts
- Adjust spacing and border radius
- Change component behavior (animations, loading states, error handling)
- Edit UX copy
- Rename routes

**How it works:**

```
1. Run /gopeep-repo or /gopeep-web
2. Open gopeep-spec/00-customization.md
3. Change what you want, leave the rest blank
4. Hand gopeep-spec/ to any agent: "recreate this using gopeep-spec/"
5. Done — the agent reads your customizations and applies them
```

The recreation agent always reads `00-customization.md` first. Your overrides win over the analyzed values.

---

## Success criteria

Gopeep output is successful when:

1. **Completeness** — An agent reading only `gopeep-spec/` (no access to original) could recreate a project with the same architecture, patterns, and deployment topology.
2. **Accuracy** — Architecture descriptions match actual code. No hallucinated components.
3. **Actionability** — Every section includes concrete reproduction instructions, not just observations.
4. **Customizability** — `00-customization.md` covers every decision a human might want to change.
5. **Scale** — Analysis completes in reasonable time for repos up to 100K+ files.

---

## License

Apache 2.0. See [LICENSE](LICENSE).
