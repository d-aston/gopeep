# gopeep

Two agent skills for architecture extraction and frontend reverse-engineering.

## gopeep-repo

Analyze any software repository and produce structured `gopeep-spec/` files capturing architecture, patterns, conventions, and deployment config — plus a human-editable `00-customization.md` for adjusting the recreation before handing off to an agent.

**Trigger:** `/gopeep-repo [path]`, "analyze this repo", "extract architecture", "document this codebase"

## gopeep-web

Analyze a website's HTML, CSS, and JavaScript and produce structured `gopeep-spec/` files — plus a human-editable `00-customization.md` for adjusting colors, fonts, components, and data shapes.

**Trigger:** `/gopeep-web [source]`, "analyze this website", "reverse engineer this UI", "recreate this page"

---

Skills defined in `skills/gopeep-repo/SKILL.md` and `skills/gopeep-web/SKILL.md`.
