# Skill Maintenance

Creating a skill is only the start. Skills become stale, overlap with each other, or get superseded as the codebase changes. This file covers the lifecycle.

## Before creating — check for existing skills

Always check the skills directory for related skills before creating a new one:

```bash
# List all project skills
ls .claude/skills/

# Search for related keywords
grep -r "undefined" .claude/skills/ --include="SKILL.md" -l
grep -r "pool" ~/.claude/skills/ --include="SKILL.md" -l
```

If a related skill already exists, you have three options.

## Decision tree — existing skill found

```
Is the existing skill addressing the SAME root cause?
├── Yes → UPDATE the existing skill (don't create new)
│        Add new symptoms, refine description, expand solution
│
└── No, but it's the same general topic
    │
    ├── Is the new case a specific subset of the existing skill?
    │   └── Yes → ADD a "Variants" section to the existing skill
    │
    └── Is it a genuinely different problem in the same area?
        └── Yes → CREATE new skill, but make the description more
                  specific so the two don't both trigger on every case
```

## How to update an existing skill

1. **Read the existing skill in full** — don't assume you know what it says
2. **Preserve the `name` field** — never change a skill's name during update; it would break references
3. **Append new information rather than replace** — the original capture had a reason
4. **Update the `description`** if the skill now covers more cases — but keep it specific
5. **Add a note in the body** about the update:

```markdown
## Update history

- 2026-05-07: original capture — covered the basic null-from-await case
- 2026-05-15: added handling for nested optional chains
```

## How to merge two skills that overlap

If you find two existing skills that overlap (often happens when skills accumulate):

1. Pick the skill with the more general name as the survivor
2. Move unique content from the other into it
3. Delete the redundant skill's directory entirely:
   ```bash
   rm -rf .claude/skills/<redundant-name>
   ```
4. If the deleted skill might be referenced elsewhere, leave a stub for one session:
   ```markdown
   ---
   name: <old-name>
   description: DEPRECATED. Merged into <new-name>. Will be removed.
   ---
   This skill has been merged into `<new-name>`. See that skill instead.
   ```

## When to delete a skill

Delete (don't just ignore) a skill when:

- The library it covers is no longer used in the codebase
- The framework version it targets has been upgraded past the issue
- The pattern it documents is now handled by a lint rule or type check
- You've noticed it triggering on the wrong cases repeatedly

```bash
rm -rf .claude/skills/<skill-name>
```

Don't keep skills "just in case." A stale skill is worse than no skill — it pollutes the available_skills list and may trigger spuriously.

## Recognizing a stale skill

Signals that a skill needs review:

- The error it covers no longer occurs in the codebase
- The dependency it references has been removed from `package.json`
- The solution it suggests is now the default behavior in newer versions
- You've seen it trigger and then Claude ignored its advice (means description matches but content is no longer relevant)

## Updating during plan execution

If during a plan you find that an existing skill *almost* covers the current problem but is missing something:

1. Solve the current problem first — don't get sidetracked
2. After verifying the fix works, update the existing skill to incorporate what you learned
3. Inform the user briefly: "Updated existing skill `<name>` with handling for this variant"

## Anti-patterns

- **Creating a new skill when an existing one applies** — clutter
- **Editing the `name` field during update** — breaks the directory→name correspondence
- **Letting two skills cover the exact same case** — both will trigger, wasting context
- **Keeping skills for code that no longer exists** — false memory
- **Aggressive deletion without checking** — losing knowledge that took real investigation to acquire
