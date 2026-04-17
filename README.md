<p align="center">
  <img src="https://em-content.zobj.net/source/apple/391/ewe_1f411.png" width="120" />
</p>

<h1 align="center">gopeep 𓋾</h1>

<p align="center">
  <strong>extract architecture. customize it. hand it to an agent. done.</strong>
</p>

<p align="center">
  <a href="https://github.com/d-aston/gopeep/stargazers"><img src="https://img.shields.io/github/stars/d-aston/gopeep?style=flat&color=yellow" alt="Stars"></a>
  <a href="https://github.com/d-aston/gopeep/commits/main"><img src="https://img.shields.io/github/last-commit/d-aston/gopeep?style=flat" alt="Last Commit"></a>
  <a href="LICENSE"><img src="https://img.shields.io/github/license/d-aston/gopeep?style=flat" alt="License"></a>
</p>

<p align="center">
  <a href="#whats-included">What's Included</a> •
  <a href="#install">Install</a> •
  <a href="#usage">Usage</a> •
  <a href="#customization">Customization</a>
</p>

---

Two agent skills that analyze any codebase or website and produce structured specification files — plus a human-editable `00-customization.md` where you can adjust architecture decisions, colors, data models, and tech choices before handing the spec to an agent to recreate the project.

**Input:** A repo path, website URL, or pasted HTML.  
**Output:** A `gopeep-spec/` directory ready to hand to a recreating agent.

---

## What's Included

### The Skills: gopeep-repo + gopeep-web

**gopeep-repo** — 6-phase analysis of any software repository ([view skill](skills/gopeep-repo/SKILL.md)):

| Phase | Covers |
|-------|--------|
| reconnaissance | Repo shape, file counts, languages, CI/CD detection, complexity tier |
| stack-and-dependencies | Frameworks, package manifests, language versions, infra dependencies |
| architecture | Architecture style, module boundaries, data layer, API surface, cross-cutting concerns |
| patterns-and-conventions | Naming, error handling, async patterns, testing conventions, type system |
| infrastructure-and-deployment | Docker, K8s, CI/CD pipelines, env vars, secrets, observability |
| reproduction-blueprint | Ordered build plan with exact commands, critical path, verification steps |

**gopeep-web** — 5-phase analysis of any website or frontend ([view skill](skills/gopeep-web/SKILL.md)):

| Phase | Covers |
|-------|--------|
| reconnaissance | Page type, rendering model, framework signals, design system detection |
| stack-and-dependencies | Framework, build tool, CSS approach, state management, routing |
| architecture | Component tree, layout regions, data patterns, routing, API calls |
| style-system | Complete design tokens — colors, typography, spacing, shape, motion |
| reproduction-blueprint | Ordered component build plan with exact interfaces and verification steps |

Both skills produce a `00-customization.md` — a human-editable file that overrides any analyzed value before recreation.

---

### 2 Commands

| Command | What it does |
|---------|-------------|
| `/gopeep-repo [path]` | Analyze a local or remote software repository |
| `/gopeep-web [source]` | Analyze a website — URL, file, directory, or pasted HTML |

**gopeep-repo options:**

| Option | Default | Description |
|--------|---------|-------------|
| `--depth quick\|standard\|full` | `full` | `quick` = phases 1-2, `standard` = phases 1-5, `full` = all |
| `--output [dir]` | `./gopeep-spec` | Output directory |
| `--modules [list]` | auto | Comma-separated module names for large repos |
| `--resume [N]` | — | Resume from phase N after interruption |
| `--diff [prev-spec]` | — | Output only changes against a previous spec |

---

## Install

| Agent | Install |
|-------|---------|
| **Claude Code** | `npx skills add d-aston/gopeep -a claude-code` |
| **Cursor** | `npx skills add d-aston/gopeep -a cursor` |
| **Windsurf** | `npx skills add d-aston/gopeep -a windsurf` |
| **Copilot** | `npx skills add d-aston/gopeep -a github-copilot` |
| **Cline** | `npx skills add d-aston/gopeep -a cline` |
| **Any other** | `npx skills add d-aston/gopeep` |

Uninstall: `npx skills remove gopeep`

> **Windows note:** `npx skills` uses symlinks by default. If symlinks fail, add `--copy`: `npx skills add d-aston/gopeep --copy`

---

## Usage

### /gopeep-repo — Analyze a repository

```bash
/gopeep-repo ./my-project
```

Full analysis of a local repo. Produces `gopeep-spec/` with all 6 phase files and `00-customization.md`.

```bash
/gopeep-repo ./my-project --depth quick     # recon + deps only, fast overview
/gopeep-repo ./my-project --depth standard  # phases 1-5, omits phase 6 (reproduction blueprint)
```

When to use: Before handing a project to an agent to recreate or migrate.

```bash
/gopeep-repo ./monorepo --modules auth,api  # analyze specific modules only
```

When to use: Large repos where you only need a subset analyzed.

```bash
/gopeep-repo ./my-project --resume 4        # interrupted mid-run, pick up from phase 4
/gopeep-repo ./my-project --diff ./spec-v1  # what changed since last analysis
```

### /gopeep-web — Analyze a website

```bash
/gopeep-web https://example.com            # live URL
/gopeep-web ./src                          # local source directory
/gopeep-web ./index.html                   # single HTML file
# or paste HTML directly into the conversation
```

Produces `gopeep-spec/` with design tokens, component tree, and `00-customization.md` pre-filled with extracted colors, fonts, and data shapes.

When to use: Reverse-engineering a UI to recreate it with different branding or stack.

### Combining with recreation

```bash
/gopeep-repo ./old-project                 # analyze
# edit gopeep-spec/00-customization.md     # customize
# "recreate this project using gopeep-spec/"  # hand to agent
```

```bash
/gopeep-web https://example.com           # extract design
# swap colors + fonts in 00-customization.md
# "build this UI using gopeep-spec/"
```

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

All architectural diagrams use Mermaid — embedded inline in `.md` files and saved as standalone `.mmd` files.

---

## Customization

`00-customization.md` is the key file. Every value gopeep extracts can be overridden before recreation — without touching the analysis files. **The customization file always wins.**

**For repos** — override:
- Database, framework, cloud provider
- Entity names and API endpoints
- Deployment targets and CI/CD platforms
- Naming conventions
- Data models

**For websites** — override:
- Entire color palette (one table)
- Fonts
- Spacing and border radius
- Component behavior (animations, loading states, error handling)
- UX copy
- Routes and data shapes

---

## Anti-Patterns

gopeep is designed to avoid these in the specs it produces:

- Don't reference values from the analysis files in reproduction — always read `00-customization.md` first
- Don't read the entire codebase for large repos — grep first, read selectively
- Don't skip the customization file when recreating — it's the source of truth
- Don't analyze generated or vendored code — document the generator, skip the output

---

## License

Apache 2.0. See [LICENSE](LICENSE).
