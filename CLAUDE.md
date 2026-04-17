# CLAUDE.md — gopeep

## Project overview

Gopeep is two agent skills — `gopeep-repo` and `gopeep-web` — that analyze codebases and websites, producing structured `gopeep-spec/` directories. Key differentiator: `00-customization.md` lets humans adjust architecture decisions before handing the spec to a recreating agent.

---

## File structure

### Single source of truth — edit only these

| File | What it controls |
|------|-----------------|
| `skills/gopeep-repo/SKILL.md` | Full gopeep-repo behavior: 6 phases, scaling strategy, edge cases, output format, customization file format. |
| `skills/gopeep-web/SKILL.md` | Full gopeep-web behavior: 5 phases, URL/file/paste input, style system extraction, customization file format. |

### Auto-synced — do not edit directly

Overwritten when sources change.

| File | Synced from |
|------|-------------|
| `.cursor/skills/gopeep-repo/SKILL.md` | `skills/gopeep-repo/SKILL.md` |
| `.cursor/skills/gopeep-web/SKILL.md` | `skills/gopeep-web/SKILL.md` |
| `.windsurf/skills/gopeep-repo/SKILL.md` | `skills/gopeep-repo/SKILL.md` |
| `.windsurf/skills/gopeep-web/SKILL.md` | `skills/gopeep-web/SKILL.md` |

---

## Key design decisions

**`00-customization.md` is the primary output.** The analysis phase files are inputs to the customization file. A human edits customization. A recreating agent reads customization first — its values override phase files.

**Customization overrides analysis.** Everywhere in both SKILL.md files: "when values in `00-customization.md` differ from what was analyzed, the customization file wins." Don't break this invariant.

**Scale-aware, not scale-limited.** gopeep-repo handles everything from small scripts to massive monorepos via complexity-tier detection in phase 1. Preserve the scaling strategy in any SKILL.md edit.

**Phases checkpoint.** Each phase writes its output file before the next phase starts. If interrupted, user can `--resume N`. Don't change phase output file names.

---

## README rules

README is the product front door.

- Keep "Before / After" section as the pitch
- Install table must be accurate — one broken install command costs a real user
- Don't explain the skill files in the README — link to them
- Keep language plain — non-technical people should understand and install in 60 seconds

---

## Key rules for agents working here

- Edit `skills/gopeep-repo/SKILL.md` or `skills/gopeep-web/SKILL.md` for behavior changes. Never edit the agent-specific copies.
- Never break the "customization file wins" invariant in either skill.
- Phase output filenames are stable — recreating agents reference them by name. Don't rename.
- `00-customization.md` is always the first file in `gopeep-spec/` (alphabetically and conceptually). Preserve this.
- Any new phase added to gopeep-repo gets a numbered file `0N-<name>.md`. Blueprint is always last.
