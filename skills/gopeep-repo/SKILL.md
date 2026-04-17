---
name: gopeep-repo
description: >
  Analyze any software repository and produce structured specification files
  that capture architecture, patterns, conventions, and deployment configuration.
  Output is a gopeep-spec/ directory designed to be consumed by another agent
  to faithfully recreate the project. Includes a human-editable 00-customization.md
  for adjusting architecture decisions, naming, data models, and tech choices before
  handing off to a recreating agent.
  Trigger: /gopeep-repo, /gopeep, "analyze this repo", "extract architecture",
  "document this codebase", "spec this project", when given a repo path to analyze.
---

Analyze the provided repository and produce a `gopeep-spec/` directory. The primary output is `00-customization.md` — a human-editable file where architectural decisions, naming conventions, tech stack choices, and data models can be adjusted. The remaining phase files are read-only analysis that a recreating agent uses alongside the customization file.

## Invocation

```
/gopeep-repo [repo_path] [--output dir] [--depth quick|standard|full] [--modules list] [--resume N] [--diff prev-spec]
```

Default depth: `full`. Default output: `./gopeep-spec/`

| Option | Default | Description |
|--------|---------|-------------|
| `--depth` | `full` | `quick` = phases 1-2, `standard` = phases 1-5, `full` = all phases |
| `--output` | `./gopeep-spec` | Output directory |
| `--modules` | auto | Comma-separated module names (large repos). `auto` = detect all |
| `--resume` | — | Resume from phase N |
| `--diff` | — | Path to previous gopeep-spec. Output only changes |

## Execution Order

Run phases 1-6 first to gather all data. Generate `00-customization.md` last, pulling editable values from all phases. Write each file before starting the next. Checkpoint after each phase — if interrupted, check which output files already exist in `gopeep-spec/` and resume from the first missing phase.

---

## Phase 1: Reconnaissance → `01-reconnaissance.md`

Without reading code:
- Count files, directories, total LOC
- Detect languages (file extensions, shebang lines)
- Identify build/config files (package.json, Cargo.toml, go.mod, Makefile, Dockerfile, etc.)
- Map directory tree (depth-limited for large repos — 3 levels + file counts; full for small)
- Detect monorepo patterns (workspaces, lerna, nx, turborepo, bazel)
- Identify VCS metadata (git history depth, branch count, contributor count if available)
- Detect CI/CD config files (.github/workflows, .gitlab-ci.yml, Jenkinsfile, etc.)
- Estimate complexity tier: `small` (<50 files) / `medium` (50-500) / `large` (500-5000) / `massive` (5000+)

**Scaling:** File counting and extension scanning is O(n) on filesystem — no context pressure. Massive repos: tree truncated to 3 levels with counts per directory.

---

## Phase 2: Stack & Dependency Analysis → `02-stack-and-dependencies.md`

- Parse all manifest/lock files (package.json, requirements.txt, Gemfile, go.sum, pom.xml, Cargo.toml, etc.)
- Categorize dependencies: runtime / dev / build / test / optional
- Identify framework(s) and version constraints
- Detect language versions (.nvmrc, .python-version, .tool-versions, rust-toolchain.toml)
- Identify infrastructure dependencies (databases, message queues, caches, object stores)
- Map internal dependency graph (monorepos: which packages depend on which)
- Detect code generation tools (protobuf, GraphQL codegen, OpenAPI generators)

**Scaling:** Manifest files are small — read all. For monorepos, process each workspace's manifests. Aggregate into unified dependency map.

---

## Phase 3: Architecture Mapping → `03-architecture.md`

Identify architectural patterns, component boundaries, and data flow.

**Architecture style:**
- Monolith / modular monolith / microservices / serverless / hybrid
- MVC / MVVM / hexagonal / clean architecture / event-driven / CQRS
- Client-server / SPA+API / SSR / SSG / hybrid rendering

**Service/module boundaries:**
- Entry points (main files, route definitions, handler registrations)
- Module/package structure and encapsulation boundaries
- API surface (REST endpoints, GraphQL schema, gRPC services, CLI commands)

**Data layer:**
- Database schemas (migrations, ORM models, raw SQL)
- Data access patterns (repository, active record, query builders)
- Caching strategy
- State management (frontend: Redux/Zustand/signals; backend: session stores)

**Integration points:**
- External API calls (HTTP clients, SDK usage)
- Message queue producers/consumers
- Webhook handlers
- File I/O patterns

**Cross-cutting concerns:**
- Authentication/authorization approach
- Logging/observability instrumentation
- Error handling strategy
- Middleware/interceptor chains
- Configuration management (env vars, config files, feature flags)

Include Mermaid diagrams of architecture overview, data flow, and dependency graph.

Mark editable items inline:
> **✏️ Editable** — see `00-customization.md` › [section name]

**Scaling strategy:**
- **Small/Medium:** Read entry points, route files, models, key modules. Usually fits in context.
- **Large:** Sample-based. Read directory structures per module, then selectively read entry points, route definitions, model definitions, config. Use grep for pattern detection.
- **Massive:** Hierarchical decomposition. Map top-level modules first. Analyze each module independently (parallelize with sub-agents). Never attempt full codebase read — budget ~20 key files per module.

---

## Phase 4: Pattern & Convention Extraction → `04-patterns-and-conventions.md`

**Naming conventions:**
- File naming (kebab-case, PascalCase, snake_case, co-location patterns)
- Variable/function/class naming patterns
- Directory naming conventions

**Structural patterns:**
- Component/module template (what does a typical module look like?)
- File organization within modules (index barrel files, co-located tests, etc.)
- Import ordering conventions

**Code patterns:**
- Error handling idiom (Result types, try/catch style, error codes, Either)
- Async patterns (async/await, promises, callbacks, channels, actors)
- DI/IoC approach (constructor injection, container, manual wiring)
- Validation approach (schema validation, decorators, manual checks)

**Testing patterns:**
- Test organization (co-located, separate tree, both)
- Test naming conventions
- Fixture/factory patterns
- Mock/stub approach
- Integration vs unit test separation

**API patterns:**
- Request/response shape conventions
- Error response format
- Pagination approach
- Versioning strategy

**Type system usage:**
- Strictness level (strict TypeScript, mypy strict, etc.)
- Custom type patterns (branded types, newtypes, opaque types)

**Scaling:** Pattern extraction works by sampling. Read 3-5 representative files per detected pattern category. Grep for common idioms. Cross-validate for consistency vs intentional deviation.

---

## Phase 5: Infrastructure & Deployment → `05-infrastructure-and-deployment.md`

**Containerization:**
- Dockerfile analysis (base images, multi-stage builds, layers)
- Docker Compose / container orchestration configs

**Orchestration:**
- Kubernetes manifests (deployments, services, ingress, configmaps, secrets)
- Helm charts / Kustomize overlays
- Terraform / Pulumi / CloudFormation / CDK
- Serverless configs (serverless.yml, SAM templates, Vercel/Netlify config)

**CI/CD:**
- Pipeline definitions (stages, jobs, triggers)
- Build steps and artifacts
- Deployment strategy (blue/green, canary, rolling, recreate)
- Environment promotion flow (dev → staging → prod)

**Environment configuration:**
- Environment variables (names, not values — extract from .env.example, docker-compose, CI config)
- Secrets management approach (vault, AWS secrets manager, sealed secrets, etc.)

**Observability:**
- Monitoring setup (Prometheus, Datadog, CloudWatch, etc.)
- Logging configuration
- Tracing instrumentation (OpenTelemetry, Jaeger, etc.)

**Scaling:** Infrastructure files are typically few and self-contained. Read all Dockerfiles, CI configs, and IaC files. For large Terraform/K8s directories, map module structure first, then read key modules.

---

## Phase 6: Reproduction Blueprint → `06-reproduction-blueprint.md`

Synthesize phases 1-5 into an ordered build plan:

1. Project scaffolding (language, framework, tooling setup)
2. Directory structure and module skeleton
3. Configuration and environment setup
4. Data layer (schemas, migrations, ORM setup)
5. Core business logic modules (dependency order)
6. API/service layer
7. Authentication and authorization
8. Background jobs / workers / queues
9. Frontend (if applicable)
10. Infrastructure and deployment config
11. CI/CD pipeline
12. Observability setup

**For each phase:**
- Exact commands to run
- Files to create with their structure
- Configuration values to set (reference `00-customization.md` token names)
- Verification steps

**Critical path:** Minimum steps for a running skeleton.

**Decision points:** Choices the original made that the recreating agent must replicate.

**Anti-patterns noted:** Any tech debt detected — mark as `replicate as-is` vs `known issue`.

**Important:** When values in `00-customization.md` differ from what was analyzed, the customization file wins. Always read `00-customization.md` before starting any recreation task.

---

## Final Step (after Phase 6): Human Customization File → `00-customization.md`

> This step does not produce a numbered phase file. Output is always `00-customization.md`.

**Generate this file last.** Pull editable values from all prior phases. Write it to be readable without technical background.

### Format

```markdown
<!--
  GOPEEP CUSTOMIZATION FILE
  ═══════════════════════════════════════════════════════════════
  Edit this file to customize the recreation before handing it
  to an agent. Change the values — not the labels or structure.

  Rules:
  - This file OVERRIDES the analyzed spec files
  - Change only the VALUE columns / content blocks below
  - Leave labels, section headers, and comments alone
  - If unsure about a value, leave it as-is (the original will be used)
  ═══════════════════════════════════════════════════════════════
-->

---
gopeep_version: "1.0"
source: <repo_path or url>
analyzed_at: <ISO-8601>
complexity_tier: <small|medium|large|massive>
---

# Customization

## How to use this file

Gopeep analyzed the repository and filled in the values below. Change anything you want adapted, leave the rest as-is. When you hand the `gopeep-spec/` folder to an agent to recreate the project, it will read this file first and apply your changes.

---

## 🏗️ Architecture

> Change these to adjust the overall architecture of the recreated project.

| Decision | Analyzed Value | Your Override |
|----------|---------------|---------------|
| Architecture style | `______` | *(leave blank to keep)* |
| Primary language | `______` | *(leave blank to keep)* |
| Framework | `______` | *(leave blank to keep)* |
| Database | `______` | *(leave blank to keep)* |
| API style | `______` | *(leave blank to keep)* |
| Auth approach | `______` | *(leave blank to keep)* |

---

## 📦 Tech Stack

> Override specific technologies. Leave blank to keep what was analyzed.

\`\`\`
Language version:     ______  (leave blank to keep analyzed: ______)
Framework version:    ______  (leave blank to keep analyzed: ______)
Database:             ______  (leave blank to keep analyzed: ______)
Cache:                ______  (leave blank to keep analyzed: ______)
Message queue:        ______  (leave blank to keep analyzed: ______)
Container runtime:    ______  (leave blank to keep analyzed: ______)
CI/CD platform:       ______  (leave blank to keep analyzed: ______)
Cloud provider:       ______  (leave blank to keep analyzed: ______)
\`\`\`

---

## 🗄️ Data Models

> Edit field names, types, add/remove fields.
> The agent will use these shapes when generating schema and migration code.

<!-- Gopeep fills these in from analyzed models/schemas -->

### [Primary entity — e.g. User, Product, Order]

\`\`\`typescript
// EDIT THIS — change field names, types, add/remove fields
interface ______ {
  id: ______;
  // ... (gopeep fills from analysis)
}
\`\`\`

<!-- Add more interfaces/models as needed -->

---

## 🌐 API Design

> Add, remove, or rename endpoints.
> Mark endpoints as public or authenticated.

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| `GET` | `/______` | ______ | public |
| `______` | `______` | ______ | ______ |

<!-- Add rows as needed -->

---

## 🏷️ Naming Conventions

> Override naming conventions if you want the recreation to use different patterns.

\`\`\`
File naming:          ______  (leave blank to keep analyzed: ______)
Function naming:      ______  (leave blank to keep analyzed: ______)
Class naming:         ______  (leave blank to keep analyzed: ______)
Database tables:      ______  (leave blank to keep analyzed: ______)
API endpoints:        ______  (leave blank to keep analyzed: ______)
Environment vars:     ______  (leave blank to keep analyzed: ______)
\`\`\`

---

## ⚙️ Configuration & Environment

> List environment variables the recreation should expect.
> Add any variables not detected in the original.

| Variable | Purpose | Required | Default |
|----------|---------|----------|---------|
| `______` | ______ | yes/no | ______ |

<!-- Gopeep fills from .env.example, docker-compose, CI config -->

---

## 🚀 Deployment Target

> Describe where and how the recreation should be deployed.

\`\`\`
Deploy target:        ______  (e.g. "AWS ECS", "Vercel", "bare metal", "same as analyzed")
Container strategy:   ______  (e.g. "Docker Compose", "K8s", "none")
CI/CD:                ______  (e.g. "GitHub Actions", "GitLab CI", "same as analyzed")
Environments:         ______  (e.g. "dev + prod", "dev + staging + prod")
\`\`\`

---

## 🔧 Behavior & Quality

> Plain English — describe what you want. The agent interprets these.

\`\`\`
Error handling style:   ______  (e.g. "exceptions", "Result types", "same as analyzed")
Logging approach:       ______  (e.g. "structured JSON", "same as analyzed")
Test coverage:          ______  (e.g. "unit + integration", "same as analyzed")
Code style:             ______  (e.g. "same as analyzed", "stricter TypeScript")
\`\`\`
```

**When generating this file:**
- Fill all `______` placeholders with extracted values from phases 1-6
- For data models: use actual TypeScript/language interfaces matching analyzed shapes
- For environment variables: use exact variable names from analysis
- Write `[not detected]` for values that couldn't be determined — the human must fill in
- Keep all comments and instructions intact — they guide the human editor

---

## Output File Structure

```
gopeep-spec/
├── 00-customization.md              ← PRIMARY HUMAN-EDIT FILE
├── 01-reconnaissance.md
├── 02-stack-and-dependencies.md
├── 03-architecture.md
├── 04-patterns-and-conventions.md
├── 05-infrastructure-and-deployment.md
├── 06-reproduction-blueprint.md
└── diagrams/
    ├── architecture-overview.mmd
    ├── data-flow.mmd
    ├── dependency-graph.mmd
    └── deployment-topology.mmd
```

For large repos, additional per-module deep-dives:
```
gopeep-spec/
└── modules/
    ├── module-auth.md
    ├── module-api.md
    └── module-worker.md
```

## Phase File Format

```markdown
---
gopeep_version: "1.0"
phase: <N>
repo: <repo_name>
analyzed_at: <ISO-8601>
complexity_tier: <small|medium|large|massive>
editable: false
---

# <Phase Title>

> **Note for recreating agents:** Read `00-customization.md` first.
> Values there override anything in this file.

## Summary

## Details

## Confidence Notes
<!-- Sampled vs exhaustive. Ambiguous patterns. What was skipped and why. -->

## Reproduction Instructions
```

## Scaling Strategy

| Tier | Files | Strategy |
|------|-------|----------|
| Small | <50 | Read everything. Single-pass. |
| Medium | 50-500 | Read key files. Grep for patterns. |
| Large | 500-5000 | Sample-based. Module-by-module. Parallel sub-agents for phases 3-4. |
| Massive | 5000+ | Hierarchical decomposition. Top-level mapping first, then per-service sub-agents. Heavy grep. |

**Large repo tactics:**
1. Never read more than ~100 files per agent context. Prioritize entry points, configs, models.
2. Grep over read — find patterns, then selectively read matching files.
3. Directory names carry architectural intent. Map structure before reading contents.
4. For phase 3, spawn sub-agents per module. Each produces `modules/module-<name>.md`. Parent synthesizes.
5. Each phase writes output before next phase begins. If interrupted, resume from last completed phase.

## Edge Cases

| Scenario | Behavior |
|----------|----------|
| Binary-heavy repo | Skip binary files. Note presence and purpose in recon. |
| Generated code | Identify generator config. Document generation process, not generated output. |
| Vendored dependencies | Detect vendor dirs. Document vendoring approach, skip vendored code analysis. |
| Git submodules | Document references. Analyze submodule repos separately if `--depth full`. |
| Encrypted/obfuscated files | Note presence. Skip analysis. Flag in confidence notes. |
| Multiple languages in same service | Document polyglot nature. Analyze each language's patterns separately. |
| No clear entry point | Use heuristics: main files, index files, exported modules. Flag uncertainty. |
| Conflicting patterns | Document both with locations. Note inconsistency vs intentional variation. |

## Success Criteria

A human can open `00-customization.md`, swap the database, rename entities, change the deployment target, override naming conventions — then hand the entire `gopeep-spec/` folder to an agent, which recreates the project using the customized values throughout.
