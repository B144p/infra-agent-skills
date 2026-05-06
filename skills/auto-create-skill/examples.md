# Worked Examples

Concrete examples of how a problem becomes a skill. Use these as templates when capturing your own.

---

## Example 1 — JavaScript undefined property after async

### What happened in the session

Claude was asked to fix a bug where a user profile page crashed intermittently. Investigation showed:

1. Stack trace pointed to `user.profile.email` — `TypeError: Cannot read property 'email' of undefined`
2. Initial guess: add optional chaining. But the real question was *why* `profile` was undefined.
3. Traced back: `getUser()` returned a promise that resolved to `null` for users created before the profile feature shipped.
4. Fix: add a guard at the call site, plus seed missing profiles in a migration.

### The skill that was captured

```markdown
---
name: fix-undefined-after-async-fetch
description: Resolve `Cannot read property X of undefined` errors that appear after an awaited fetch or DB call in JavaScript/TypeScript. Use when a property access fails on a value that came from an async source, especially when the failure is intermittent or affects only some users.
---

# Fix Undefined After Async Fetch

When property access fails after `await`, the awaited value is usually `null` for some inputs — not a missing property on a present object.

## Symptoms

- Error: `TypeError: Cannot read property 'X' of undefined`
- Failure is intermittent (not every request)
- Stack trace shows the access happening right after an `await`

## Root cause

The awaited promise resolves to `null` (or `undefined`) for some inputs, but the calling code assumes a populated object.

## Solution

1. Identify the awaited call (DB query, fetch, etc.)
2. Check whether it can legitimately return `null` for some inputs
3. Add a guard at the call site:
   ```ts
   const user = await getUser(id);
   if (!user) throw new NotFoundError(`User ${id} not found`);
   ```
4. If the null case represents legacy/missing data, fix the data source too — don't only patch the read path

## Verification

Reproduce with the input that triggered the original error. The guard should produce a clean error, not a crash.

## Source

Captured 2026-05-07. Original task: fixing intermittent profile page crashes.
```

### Why this works

- **Specific symptoms** — exact error string, "intermittent" qualifier, "after `await`" anchor
- **Real root cause** — not just "add `?.`" but identifies why undefined appeared
- **Two-part solution** — read-path guard AND fix the data source
- **Verifiable** — can be confirmed reproducible

---

## Example 2 — Postgres pool exhaustion

### What happened in the session

Production app started timing out under load. Investigation:

1. Logs showed `remaining connection slots are reserved for non-replication superuser connections`
2. Found 3 places using `pool.connect()` directly without `try/finally`
3. One error path skipped the `release()` call
4. Fixed by wrapping in try/finally and adding pool size config

### The skill that was captured

```markdown
---
name: fix-pg-pool-exhaustion
description: Resolve Postgres connection pool exhaustion in Node.js apps using the `pg` library. Use when seeing "remaining connection slots are reserved" or "too many clients already" errors, or when query latency spikes under concurrent load.
---

# Fix pg Pool Exhaustion

Pool clients leak when `client.release()` is skipped on an error path. Pool eventually has zero free clients and queries hang.

## Symptoms

- Error: `remaining connection slots are reserved for non-replication superuser connections`
- Error: `Connection terminated due to connection timeout`
- Active connections in `pg_stat_activity` climb monotonically

## Root cause

Clients acquired via `pool.connect()` are not released on every code path. Most often: an exception thrown between `connect()` and `release()` skips the cleanup.

## Solution

1. Audit every `pool.connect()` call site
2. Wrap usage in `try/finally`:
   ```js
   const client = await pool.connect();
   try { /* queries */ } finally { client.release(); }
   ```
3. Prefer `pool.query()` directly when no transaction is needed — it auto-releases
4. Set explicit pool config: `new Pool({ max: 20, idleTimeoutMillis: 30000 })`

## Verification

Run a load test. Watch `pg_stat_activity` — active connections should plateau, not climb.

## Source

Captured 2026-05-07. Original task: investigating checkout API timeouts.
```

### Why this works

- Description names the library (`pg`) — won't trigger for unrelated pool issues
- Lists exact error strings users will paste
- Solution covers diagnosis (audit) AND fix AND prevention (pool config)

---

## Counter-example — what NOT to capture

### Bad skill

```markdown
---
name: fix-errors
description: Fixes errors in code.
---

# Fix Errors

When there are errors, fix them.

1. Read the error
2. Fix it
3. Run tests
```

### Why this fails

- **Description triggers on everything or nothing** — Claude has no signal about when this skill applies
- **No symptoms section** — no concrete trigger conditions
- **No root cause** — captures no actual knowledge
- **Solution is generic** — gives no advantage over Claude's default behavior

A skill like this adds noise to the skills directory and dilutes the signal of better skills. **If you can't write something more specific than this, do not capture.**

---

## Pattern to follow

Every captured skill should be **at least as specific** as Examples 1 and 2:

- Error string or symptom that's distinctive
- Root cause stated in one sentence
- Solution that's actually different from "read the error and fix it"
- Verification that's concrete
