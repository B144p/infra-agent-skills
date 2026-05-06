# Progressive Disclosure

Skills work in three loading stages. Putting the right content at the right stage saves context and makes triggering reliable. Get this wrong and skills bloat the context window even when they're not relevant.

## The three stages

```
Stage 1 — ALWAYS in context (every turn, every conversation)
  ↓ Frontmatter only: name + description (~50–150 tokens)
  ↓ Claude decides whether to load Stage 2 based on this alone
  ↓
Stage 2 — Loaded when Claude decides skill is relevant
  ↓ Body of SKILL.md (target: 30–150 lines)
  ↓ Should contain everything needed for the common case
  ↓
Stage 3 — Loaded only when explicitly referenced from Stage 2
  ↓ Reference files (root-cause.md, examples.md, script.md, etc.)
  ↓ Should contain detail that's only needed sometimes
```

## What goes where

### Stage 1 — Frontmatter

**Include:**
- `name` (kebab-case)
- `description` — one to three sentences with concrete trigger conditions

**Do NOT include:**
- Long instructions
- Code examples
- Anything Claude needs to read every turn

The frontmatter is permanently in context. Every word here costs tokens forever. Be ruthless.

### Stage 2 — SKILL.md body

**Include:**
- Symptoms (what does this problem look like)
- Root cause (one paragraph)
- Solution steps (the common case, end-to-end)
- Verification (how to know it worked)
- Pointers to reference files for deeper detail

**Do NOT include:**
- Multiple long worked examples (move to `examples.md`)
- Extensive bash scripts (move to `script.md`)
- Edge case enumeration (move to a reference file)
- Background reading or theory (move to a reference file)

Target length: **30 to 150 lines**. If you're going over, that's a signal to split.

### Stage 3 — Reference files

**Good candidates for reference files:**
- Worked examples (1–3 full examples, with explanation)
- Detailed scripts or commands
- Edge cases and gotchas
- Background context or theory
- Maintenance / lifecycle guidance

**Naming convention:** descriptive lowercase filenames — `examples.md`, `script.md`, `root-cause.md`, `maintenance.md`. Avoid generic names like `notes.md` or `extra.md`.

## When to split into reference files

Split if any of these are true:

1. SKILL.md is over 150 lines
2. A section is only relevant to a subset of triggers
3. You have multiple full examples (move them to `examples.md`)
4. You have substantial code blocks (move them to `script.md`)
5. The skill has a methodology that's reusable across uses (move to its own reference file)

## How to reference Stage 3 files from SKILL.md

Always link explicitly so Claude knows the file exists and what it contains:

```markdown
## Reference files

- `examples.md` — three worked examples with explanations
- `script.md` — bash commands for creating files safely
- `root-cause.md` — methodology for diagnosis before capture
```

Without explicit references, Stage 3 files are invisible to Claude.

## When NOT to split

Don't split if any of these are true:

- The skill is under 100 lines
- The content is genuinely needed every time the skill triggers
- The split would create files under 20 lines (those should just be inline)

Splitting too aggressively means Claude has to load multiple files for every trigger, which is *worse* than just having one focused file.

## Token economics — why this matters

A typical project has ~10–30 skills. If every skill's full body were in context permanently:

- 30 skills × 100 lines × ~10 tokens/line = ~30,000 tokens used by skills alone

With progressive disclosure:

- 30 skills × ~30 tokens (frontmatter only) = ~900 tokens permanent
- Full body of 1–2 skills loaded per turn (when relevant) = ~2,000–4,000 tokens dynamic

The dynamic approach uses 5–10% of the static approach. Skills only work at scale because of this.

## Self-check before finalizing

Before finishing skill capture, ask:

- [ ] Is the frontmatter description specific enough to trigger reliably AND not trigger spuriously?
- [ ] Is the SKILL.md body scoped to the common case, with edge cases moved to reference files?
- [ ] Are reference files explicitly linked from SKILL.md?
- [ ] Is the total skill (all files) under ~500 lines? If more, the skill is probably doing too much — consider splitting into multiple skills.
