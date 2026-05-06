---
name: auto-create-skill
description: Automatically capture solutions as reusable skills. Use whenever you solve a non-trivial error, debug a tricky issue, or work out a multi-step solution that is likely to recur. Triggers during plan execution after a fix is verified working — creates a new SKILL.md so the same problem solves faster next time.
---

# Auto Create Skill

This skill teaches you to capture working solutions as new skills automatically, without waiting for the user to ask. It runs as a step inside a plan, not as a separate user command.

## When to trigger this skill

Trigger this skill at the end of a task or plan step when ALL of the following are true:

1. You encountered a problem that was NOT trivial — it took more than one attempt, required investigation, or involved a non-obvious fix.
2. You verified the solution actually works — tests passed, error gone, behavior confirmed.
3. The problem could plausibly recur — same error class, same library, same pattern in this codebase or others.

If the fix was a one-line typo, a missing import, or something obvious in 10 seconds, do NOT create a skill. Skills are for repeatable knowledge, not for noise.

## When NOT to trigger

- Trivial fixes (typos, missing semicolons, obvious imports)
- One-off project-specific quirks that won't appear elsewhere
- Solutions you are not confident about — if it works "somehow," investigate further before capturing
- When the user has explicitly said they do not want skills created automatically in this session

## The workflow

1. **Diagnose root cause** — see `root-cause.md`. Do not skip this. A skill built on a misdiagnosis is worse than no skill.
2. **Check for existing skills** — see `maintenance.md`. If a related skill already exists, update it rather than creating a duplicate.
3. **Decide scope** — project-specific skills go to `.claude/skills/`. Generic skills go to `~/.claude/skills/`. When in doubt, default to project-level.
4. **Draft the SKILL.md** — use the template in `pattern.md` and reference real-world examples in `examples.md`. The description field is the single most important thing — it determines whether you will trigger this skill in future sessions.
5. **Plan the structure** — see `progressive-disclosure.md`. Decide what belongs in the frontmatter, what belongs in the body, and what belongs in reference files.
6. **Write the files** — use the commands in `script.md` to create the directory and SKILL.md safely.
7. **Confirm with the user** — at the end of your reply, mention briefly: "I captured this as a skill at `<path>` for future reference." Do not interrupt the plan to ask permission. Just inform.

## Reference files

- `root-cause.md` — how to identify whether a problem is worth capturing and what the real cause is
- `pattern.md` — templates and structure for the SKILL.md you are creating
- `script.md` — bash commands for creating the skill directory and writing the file safely
- `examples.md` — full worked examples of well-formed skills, plus a counter-example
- `maintenance.md` — handling existing skills: update, merge, or delete
- `progressive-disclosure.md` — how to structure content across SKILL.md and reference files for token efficiency

## Output expectations

After running this skill, the following must be true:

- A new directory exists at `.claude/skills/<skill-name>/` (or `~/.claude/skills/<skill-name>/` for global skills)
- A `SKILL.md` file exists in that directory with valid YAML frontmatter (two `---` delimiters)
- The description field is specific enough that Claude will recognize when to trigger it
- You mentioned the new skill in your reply so the user knows it was created

## Anti-patterns

- Creating a skill for every small fix — clutters the skills directory and dilutes trigger reliability
- Vague descriptions like "helps with errors" — these never trigger when needed
- Capturing a symptom-level fix instead of the root cause — see `root-cause.md`
- Overwriting an existing skill silently — always check for name collisions first (see `script.md`)
