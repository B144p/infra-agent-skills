# Root Cause Analysis

Before you capture a skill, you must understand what actually went wrong. A skill that documents the wrong cause is worse than nothing — it will trigger on the wrong situations and waste future context.

## The 5-question check

Ask these in order. If you cannot answer any of them clearly, do NOT create the skill yet.

1. **What was the observable failure?** — the exact error message, the wrong behavior, the stack trace.
2. **What was the immediate cause?** — the line or function that failed. The "what."
3. **What was the upstream cause?** — why did that line fail? What state, input, or condition led to it? The "why."
4. **What is the general class of this problem?** — null-handling, race condition, config mismatch, version incompatibility, misunderstood API contract, etc.
5. **What is the smallest fix that addresses the upstream cause** (not just the symptom)?

If your fix only addresses question 2 and not question 3, you patched a symptom. The skill you write will be brittle and trigger on the wrong situations.

## Symptom vs cause — examples

| Symptom fix (bad skill) | Root cause fix (good skill) |
|---|---|
| Wrap call in try/catch to swallow error | Validate input shape before the call |
| Add `?? ''` to silence undefined warning | Trace where the undefined value originates and fix the source |
| Bump dependency version until error stops | Identify the breaking API change and adapt usage |
| Add `await` randomly until it works | Map out the actual async flow and identify the missing await |
| Increase timeout until requests succeed | Diagnose why requests are slow and fix the bottleneck |

## Generalizability test

Once you understand the cause, ask:

- Could this exact class of problem appear in a different file in this codebase? → if yes, **project-level** skill
- Could it appear in a different project entirely? → if yes, **global** skill (`~/.claude/skills/`)
- Is it tied to one specific library or framework? → name the library in the description
- Is it tied to a specific version? → note the version in the skill body
- Is it truly one-off and unlikely to repeat? → **do not create a skill**

## Confidence check

Before writing the skill, write one sentence in your head:

> "Next time I see <X>, I should <Y>."

If you cannot fill in X and Y precisely, your understanding is not yet skill-worthy. Investigate more or skip the capture.

## Examples of well-formed conclusions

- "Next time I see `Cannot read property 'X' of undefined` after an async call, I should check whether the awaited promise resolved to null and add a guard at the call site."
- "Next time `pnpm install` fails with `ERR_PNPM_PEER_DEP_ISSUES` on a workspace project, I should check the root `package.json` for `pnpm.peerDependencyRules.allowedVersions` before trying version bumps."
- "Next time a Postgres query hangs in a Node.js app under load, I should check the `pg` connection pool size and confirm clients are released after queries."

## Red flags that mean "don't capture yet"

- The fix worked but you cannot explain why
- You tried several things and one of them worked, but you don't know which
- The error message is vague and you fixed it by trial and error
- You suspect there might be a deeper issue you didn't fully resolve

In any of these cases, capturing a skill will encode your guess as if it were knowledge. Skip the capture and either investigate more, or just leave the fix in place without writing a skill.
