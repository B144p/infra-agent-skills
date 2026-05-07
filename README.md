# infra-agent-skills

Claude Code skills for infrastructure and development workflows.

## Skills

| Skill | Status | Description |
|---|---|---|
| [`auto-create-skill`](skills/auto-create-skill/SKILL.md) | **Deprecated** | Use the `skill-creator` plugin instead (see below) |

## Installation

Add a skill to your project:

```bash
npx skills add https://github.com/B144p/infra-agent-skills --skill <skill-name>
```

Or globally:

```bash
npx skills add https://github.com/B144p/infra-agent-skills --skill <skill-name> --global
```

## Auto-creating skills

> **`auto-create-skill` is deprecated.** Use the `skill-creator` plugin instead.

The `skill-creator` plugin captures reusable skills automatically — no manual commands needed. Configure a hook in `.claude/settings.json` to trigger it after a problem is resolved:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "error_resolved",
        "command": "/skill-creator capture"
      }
    ]
  }
}
```

After setup, Claude will automatically create skills at `.claude/skills/` when it solves non-trivial, repeatable problems.
