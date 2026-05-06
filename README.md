# infra-agent-skills

A library of Claude Code skills for automating common infrastructure and development workflows.

## What this is

Claude Code skills are reusable instruction sets that Claude loads on demand when a task matches a skill's trigger conditions. Each skill lives in its own directory with a `SKILL.md` file containing YAML frontmatter (name + description) and a structured body. The description is always in context; the body is loaded only when relevant — keeping token usage low across many skills.

This repo is the source of truth for skills shared across projects. Skills here can be copied or symlinked into a project's `.claude/skills/` directory or into `~/.claude/skills/` for global availability.

## Installation

Install a skill into your current project:

```bash
npx skills add https://github.com/B144p/infra-agent-skills --skill auto-create-skill
```

Install globally across all projects:

```bash
npx skills add https://github.com/B144p/infra-agent-skills --skill auto-create-skill --global
```

This copies the skill into `.claude/skills/` (project) or `~/.claude/skills/` (global).

## Skills

| Skill | Trigger | Description |
|---|---|---|
| [`auto-create-skill`](skills/auto-create-skill/SKILL.md) | After verifying a non-trivial fix | Captures working solutions as new reusable skills so the same problem solves faster next time |

## How `auto-create-skill` works

This is a meta-skill: it teaches Claude to capture other skills automatically. It runs as a final step inside a plan, not as a separate user command.

**Trigger conditions — all three must be true:**
1. The problem was non-trivial (took investigation, more than one attempt, non-obvious fix)
2. The fix is verified working (tests pass, error gone, behavior confirmed)
3. The problem could plausibly recur (same error class, same library, same pattern)

**The 7-step workflow:**
1. Diagnose root cause (see [`root-cause.md`](skills/auto-create-skill/root-cause.md))
2. Check for existing skills to avoid duplicates (see [`maintenance.md`](skills/auto-create-skill/maintenance.md))
3. Decide scope — project-level (`.claude/skills/`) vs. global (`~/.claude/skills/`)
4. Draft `SKILL.md` using the template in [`pattern.md`](skills/auto-create-skill/pattern.md)
5. Plan the file structure using progressive disclosure (see [`progressive-disclosure.md`](skills/auto-create-skill/progressive-disclosure.md))
6. Write the files using the safe creation commands in [`script.md`](skills/auto-create-skill/script.md)
7. Confirm with the user: "Captured as skill at `<path>`"

## Skill file structure

Skills use a three-stage progressive disclosure model to minimize token usage:

```
Stage 1 — Frontmatter (always in context, ~50–150 tokens)
  name: kebab-case-name
  description: specific trigger conditions + what the skill does

Stage 2 — SKILL.md body (loaded when skill is relevant, 30–150 lines)
  Symptoms, Root cause, Solution steps, Verification

Stage 3 — Reference files (loaded only when explicitly needed)
  examples.md, script.md, root-cause.md, maintenance.md, etc.
```

A project with 30 skills costs ~900 tokens permanently (frontmatter only) vs. ~30,000 tokens if full bodies were always loaded.

## Naming conventions

- **Format:** kebab-case only — `fix-undefined-property`, not `FixUndefinedProperty`
- **Verb-first** for action skills: `fix-`, `debug-`, `migrate-`, `setup-`, `resolve-`
- **Noun-based** for reference knowledge: `webpack-config-conventions`, `team-api-patterns`
- **Max 5 words** — if you need more, the skill is doing too much; split it

## Quality bar

**Capture when:**
- The fix required real investigation (multiple attempts, non-obvious cause)
- You are confident in the root cause
- The problem is generalizable or likely to recur

**Do NOT capture when:**
- It was a one-line typo, missing import, or obvious fix
- It is a one-off quirk specific to a single project
- You are not sure why the fix worked

**Description must include at least 3 of:**
- A specific error message or symptom
- A library, framework, or tool name
- A scenario keyword (when X happens, when doing Y)
- An action verb describing what the skill does

## Reference docs

All supporting documentation lives alongside the skill:

| File | Purpose |
|---|---|
| [`SKILL.md`](skills/auto-create-skill/SKILL.md) | Core skill — trigger conditions, workflow, anti-patterns |
| [`pattern.md`](skills/auto-create-skill/pattern.md) | SKILL.md template, naming rules, description writing guide |
| [`examples.md`](skills/auto-create-skill/examples.md) | Worked examples (JavaScript, Postgres) plus a counter-example |
| [`root-cause.md`](skills/auto-create-skill/root-cause.md) | 5-question diagnostic framework for identifying root causes |
| [`script.md`](skills/auto-create-skill/script.md) | Bash commands for safely creating skill directories and files |
| [`maintenance.md`](skills/auto-create-skill/maintenance.md) | Lifecycle guide — update, merge, delete, recognize stale skills |
| [`progressive-disclosure.md`](skills/auto-create-skill/progressive-disclosure.md) | Token efficiency strategy for structuring skill content |

## Adding new skills

Let `auto-create-skill` do it for you — if Claude is solving a non-trivial problem and the trigger conditions are met, it will create the skill automatically.

To create one manually:
1. Copy the template from [`pattern.md`](skills/auto-create-skill/pattern.md)
2. Follow the naming conventions above
3. Place it at `skills/<skill-name>/SKILL.md` (this repo) or `.claude/skills/<skill-name>/SKILL.md` (project-level)
4. Write a description specific enough to trigger on the right tasks and not on the wrong ones
