# Skill Creation Scripts

Use these commands to create the skill files safely. Do not write directly to the skills directory without first checking what exists.

## Step 1 — Decide the path

```bash
# Project-level (default)
SKILL_DIR=".claude/skills/<skill-name>"

# Global (only for truly generic skills)
SKILL_DIR="$HOME/.claude/skills/<skill-name>"
```

Default to **project-level**. Only use global when the skill is clearly applicable across projects (language-level patterns, generic library issues, etc.).

## Step 2 — Check for collision

```bash
if [ -d "$SKILL_DIR" ]; then
  echo "Skill already exists at $SKILL_DIR"
  echo "Existing content:"
  cat "$SKILL_DIR/SKILL.md"
fi
```

If a skill with the same name exists, choose one:

- **Update it** — read the existing SKILL.md, merge new knowledge in, preserve the original `name` field in frontmatter
- **Rename** — pick a more specific name for the new skill (e.g., `fix-undefined-property` → `fix-undefined-after-async`)
- **Skip** — if the existing skill already covers this case, do not duplicate

## Step 3 — Create directory and file

```bash
mkdir -p "$SKILL_DIR"

cat > "$SKILL_DIR/SKILL.md" << 'EOF'
---
name: <skill-name>
description: <description>
---

# <Skill Name>

<body content>
EOF
```

**Important:** use a here-doc with single-quoted `'EOF'` to prevent shell variable interpolation inside the skill body. If the body contains `$` or backticks meant literally, this is required.

## Step 4 — Validate

```bash
# Check the file exists and starts with frontmatter
head -5 "$SKILL_DIR/SKILL.md"

# Confirm YAML frontmatter has both delimiters
DELIM_COUNT=$(grep -c "^---$" "$SKILL_DIR/SKILL.md")
if [ "$DELIM_COUNT" -ne 2 ]; then
  echo "ERROR: SKILL.md must have exactly two '---' delimiters, found $DELIM_COUNT"
fi

# Confirm name and description fields exist
grep -q "^name:" "$SKILL_DIR/SKILL.md" || echo "ERROR: missing name field"
grep -q "^description:" "$SKILL_DIR/SKILL.md" || echo "ERROR: missing description field"
```

If any check fails, the skill will not load. Fix and re-run validation.

## Step 5 — Inform the user

End your response with a short note:

> Captured the fix as a skill at `.claude/skills/<skill-name>/SKILL.md` so this resolves faster next time.

Do not include a long explanation. The user is at the end of a plan and just wants confirmation that the capture happened.

## Adding reference files (optional)

If the skill is complex enough to need supporting files (like this one does), add them in the same directory:

```bash
mkdir -p "$SKILL_DIR"

# Main skill
cat > "$SKILL_DIR/SKILL.md" << 'EOF'
...
EOF

# Reference files
cat > "$SKILL_DIR/details.md" << 'EOF'
...
EOF
```

Reference them from the main SKILL.md so Claude knows to load them when needed:

```markdown
## Reference files
- `details.md` — extended explanation and edge cases
```

## Common mistakes to avoid

- **Writing to read-only paths** — some plugin skill paths are read-only. Always write to `.claude/skills/` (project) or `~/.claude/skills/` (global).
- **Forgetting the frontmatter delimiters** — without two `---` markers, the skill will not be loaded.
- **Missing the description field** — Claude triggers skills based on description. No description, no trigger.
- **Creating skills inside other skills' directories** — each skill gets its own directory at the top level of `skills/`.
- **Committing global skills to project git** — `.claude/skills/` is project; `~/.claude/skills/` is your home and should not be in any project repo.
- **Using shell variable interpolation in here-docs** — always quote the EOF (`'EOF'`) when the body contains literal `$` or backticks.

## Quick one-liner template

For simple cases, this single block does everything:

```bash
SKILL_NAME="fix-something"
SKILL_DIR=".claude/skills/$SKILL_NAME"
mkdir -p "$SKILL_DIR" && cat > "$SKILL_DIR/SKILL.md" << 'EOF'
---
name: fix-something
description: <fill in>
---

# Fix Something

<fill in body>
EOF
echo "Created skill at $SKILL_DIR"
```
