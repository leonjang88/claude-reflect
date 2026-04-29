# Reflect Routing Manifest (Example)

Hand-curated routing table for `/retro`. Maps learning categories to their target files.

---

## CLAUDE.md Files

### `~/CLAUDE.md`
Root-level personal rules. Covers: response preferences, tone/style, working conventions.
**Routes:** Cross-project working conventions, response format preferences, tone corrections, how Claude should behave universally.

### `./CLAUDE.md`
Project-level rules. Covers: file organization, naming conventions, project-specific workflows.
**Routes:** File/directory conventions, project structure rules, workflow corrections.

---

## Skills

### `.claude/skills/retro/SKILL.md`
The /retro skill itself.
**Routes:** Corrections to how the retro/reflect workflow runs, detection patterns, routing logic, approval flow.

### `.claude/skills/deploy/SKILL.md`
Your deploy workflow skill.
**Routes:** Deployment process corrections, environment-specific rules, pre-deploy checks.

### `.claude/skills/review/SKILL.md`
Your code review skill.
**Routes:** Review checklist changes, what to flag, approval criteria.

---

## Memory Files

### `~/.claude/projects/<your-project>/memory/`
Persistent cross-session memory.
**Routes:** New persistent facts (role, preferences, goals), project context that doesn't belong in a CLAUDE.md, reference pointers to external systems, behavioral feedback that applies across sessions.

**To add a new memory:** Create a new `.md` file in this directory and add a one-line pointer to `MEMORY.md`.
