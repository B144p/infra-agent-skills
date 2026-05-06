# SKILL.md Pattern

Use this template when generating a new SKILL.md. Fill in every section. Do not leave placeholder text.

## Frontmatter template

```yaml
---
name: <kebab-case-name>
description: <one to three sentences. State the problem class, the trigger conditions, and what the skill does. Be specific about WHEN Claude should consult this skill — list concrete error messages, library names, or scenario keywords.>
---
```

## Naming rules

- Use **kebab-case**: `fix-undefined-property`, not `FixUndefinedProperty` or `fix_undefined`
- Start with a **verb** when the skill is action-oriented: `fix-`, `debug-`, `migrate-`, `setup-`, `resolve-`
- Use a **noun** when the skill is reference knowledge: `webpack-config-conventions`, `team-api-patterns`
- Keep it under 5 words. If you need more, your skill is doing too much — split it.

## Description writing rules

The description is the single most important field. Claude only loads the full skill if the description matches the current task.

### Good descriptions

- "Fix `TypeError: Cannot read property X of undefined` in JavaScript/TypeScript. Use when a property access fails on a possibly-null value, or when adding optional chaining is being considered."
- "Resolve Postgres connection pool exhaustion in Node.js apps using the `pg` library. Use when seeing 'too many clients' errors or unexplained query timeouts under load."
- "Configure Vite for monorepo workspace packages. Use when adding a new package to a pnpm workspace and the dev server fails to resolve cross-package imports."

### Bad descriptions

- "Helps with errors" — too vague, will never trigger reliably
- "A skill for fixing things in the codebase" — no specific trigger conditions
- "TypeScript stuff" — not actionable
- "Useful for debugging" — every skill is useful, this says nothing

### Description checklist

A good description includes at least 3 of these:

- [ ] A specific error message or symptom
- [ ] A library, framework, or tool name
- [ ] A scenario keyword (when X happens, when doing Y)
- [ ] An action verb describing what the skill does

## Body structure

The body of SKILL.md should follow this structure:

```markdown
# <Skill Name>

<One-paragraph summary of what this skill does and when it applies.>

## Symptoms

<Bullet list of concrete signals that indicate this problem. Include exact error messages where possible.>

- Error message: `<exact text>`
- Behavior: <observable wrong behavior>
- Context: <when it happens — startup, under load, on specific input, etc.>

## Root cause

<Explain the underlying cause in 2-4 sentences. Reference what root-cause.md helped you identify.>

## Solution

<Numbered steps. Each step should be concrete and verifiable.>

1. ...
2. ...
3. ...

## Verification

<How to confirm the fix worked. A test command, an expected output, etc.>

## Notes / gotchas

<Optional. Edge cases, version-specific issues, related problems.>

## Source

<Captured on YYYY-MM-DD. Original task: brief description.>
```

## Length guideline

A good SKILL.md is **30 to 150 lines**. If you find yourself writing more, split into multiple skills or move detail into reference files (`pattern.md`, `script.md`, `root-cause.md`, etc.) alongside SKILL.md and link to them from the main file.

## Worked example

Here's what a well-formed skill looks like end-to-end:

```markdown
---
name: fix-pg-pool-exhaustion
description: Resolve Postgres connection pool exhaustion in Node.js apps using the `pg` library. Use when seeing "remaining connection slots are reserved" or "too many clients already" errors, or when query latency spikes under concurrent load.
---

# Fix pg Pool Exhaustion

When a Node.js app using `pg` hits the connection pool ceiling, queries hang and eventually error out. This skill captures the diagnosis and fix.

## Symptoms

- Error: `remaining connection slots are reserved for non-replication superuser connections`
- Error: `Error: Connection terminated due to connection timeout`
- Query latency increases sharply under load, then errors begin

## Root cause

Clients acquired from the pool with `pool.connect()` were not always released — typically because an error path skipped the `client.release()` call. The pool eventually has zero free clients.

## Solution

1. Audit every `pool.connect()` call site
2. Wrap usage in `try { ... } finally { client.release() }`
3. Or prefer `pool.query()` directly when no transaction is needed — it auto-releases
4. Set explicit pool sizing: `new Pool({ max: 20, idleTimeoutMillis: 30000 })`

## Verification

Run a load test and watch `pg_stat_activity` in Postgres. Active connections should plateau, not climb monotonically.

## Source

Captured on 2026-05-07. Original task: investigating timeouts in checkout API.
```
