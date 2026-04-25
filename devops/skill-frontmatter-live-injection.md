---
name: skill-frontmatter-live-injection
description: >
  Inject live/dynamic state into a Hermes Skill via YAML frontmatter updates.
  Uses a cron script to regenerate the frontmatter with fresh data,
  avoiding any modification to config.yaml or other system files.
trigger:
  - Building a skill that needs live state (cost, quota, health)
  - User says "don't touch config.yaml" or "system files are read-only"
  - Need dynamic data without session auto-load mechanism
related_skills:
  - decision-engine
---

# Skill Frontmatter Live Injection Pattern

## Problem

You have a skill that needs **live/dynamic data** (e.g., today's API cost,
remaining quota, system health status). You want this data available when
the skill is loaded, but:

- **Config.yaml patching is dangerous** — Hermes updates overwrite it
- **Hermes has no session auto-load** — skills don't load automatically at session start
- **You don't want to modify system files** — user's explicit preference

## Solution

Inject live state into the **skill's own YAML frontmatter** via a cron script.
When the skill is loaded (`skill_view()`), the agent sees the current state
embedded in the frontmatter — no config modification needed.

```
Cron Job (every hour)
    │
    ▼
Generate fresh YAML frontmatter with live data
    │
    ▼
Replace old frontmatter in SKILL.md
    │
    ▼
Agent loads skill → sees live state in frontmatter
```

## Why This Works

| Approach | Safe from updates? | Auto-load? | Needs config change? |
|----------|-------------------|------------|---------------------|
| Patch config.yaml | ❌ No | ✅ Yes | ❌ Already done |
| Status file only | ✅ Yes | ❌ No | ✅ No |
| **Skill frontmatter** | ✅ Yes | ❌ No* | ✅ No |

*Auto-load limitation: agent must still be told to load the skill, but when
they do, the data is already there.

## Reference Implementation

### Python: Frontmatter Replacer

```python
from pathlib import Path
from datetime import datetime

def update_skill_frontmatter(skill_path: Path, data: dict) -> None:
    """
    Replace YAML frontmatter in a SKILL.md file.

    CRITICAL: Use split("---", 2) NOT partition("---").
    partition only strips the first ---, leaving double frontmatter.
    """
    now = datetime.now().strftime("%Y-%m-%d %H:%M")

    # Build new frontmatter
    lines = ["---", f'name: {data["name"]}']
    for key, value in data["live_state"].items():
        lines.append(f"{key}: {value}")
    lines.append(f'updated_at: "{now}"')
    lines.append("---")
    frontmatter = "\n".join(lines) + "\n"

    # Read existing content
    content = skill_path.read_text(encoding="utf-8")

    # Strip old frontmatter
    if content.startswith("---"):
        parts = content.split("---", 2)
        if len(parts) >= 3:
            rest = parts[2].lstrip("\n")
        else:
            rest = content
    else:
        rest = content

    # Write back
    skill_path.write_text(frontmatter + rest, encoding="utf-8")
```

### Key Pitfall: `split` vs `partition`

```python
# ❌ WRONG — strips only first ---, leaves second --- in body
_, _, rest = content.partition("---")
# Result: "name: x\n---\n# Title" → double frontmatter

# ✅ CORRECT — strips both opening and closing ---
parts = content.split("---", 2)
rest = parts[2].lstrip("\n") if len(parts) >= 3 else content
# Result: "# Title" → clean body
```

### Cron Integration

```python
#!/usr/bin/env python3
"""Update skill frontmatter every hour via cron."""

from pathlib import Path

def main():
    # 1. Gather live data
    live_data = gather_data()  # your data source

    # 2. Update skill
    skill_path = Path.home() / ".agent" / "skills" / "my-skill" / "SKILL.md"
    update_skill_frontmatter(skill_path, {
        "name": "my-skill",
        "live_state": live_data
    })

    # 3. Also write to standalone cache (optional)
    cache_path = Path.home() / ".agent" / "data" / "my_skill_status.md"
    cache_path.write_text(format_status(live_data), encoding="utf-8")

if __name__ == "__main__":
    main()
```

Cron:
```bash
0 * * * * /usr/bin/python3 /home/user/.agent/scripts/my_skill/update_frontmatter.py
```

## Frontmatter Structure

Keep it flat and readable. The agent parses this on skill load.

```yaml
---
name: my-skill
live_state:
  today_cost: 13.77
  budget_limit: 2.00
  status: CRITICAL
  top_model: kimi-k2.6
updated_at: "2026-04-23 16:00"
---
```

## Detection Strategy

Since the skill doesn't auto-load, add **natural language triggers** to help
the agent detect when to load it:

```yaml
trigger:
  - "status" / "check" / "ready"
  - "budget" / "cost" / "quota"
  - "用咩" / "點解" / "邊個"
```

And document them in the skill body so the agent knows what keywords to watch for.

## User Preference

> **"我唔想改動系統文件，唔好搞config，呢個隨時會被覆蓋"**

This pattern respects that preference completely:
- No config.yaml modification
- No gateway restart required
- Survives Hermes updates
- All state lives in user-managed files (skills/ + data/)

## Limitations

1. **Skill must be explicitly loaded** — no auto-load mechanism exists
2. **Data freshness = cron frequency** — default 1 hour lag
3. **Frontmatter bloat** — don't dump large datasets; keep it under ~50 lines
4. **YAML syntax risk** — escaped strings, colons in values, etc.

## When NOT to Use

- If you need real-time (<1 minute) state updates
- If the skill is loaded so rarely that stale data is useless
- If the data is too large for YAML frontmatter (use separate cache file)
- If Hermes adds native session auto-load (then this pattern becomes obsolete)

## Related

- `decision-engine` — production example of this pattern
- `hermes-cron-metrics-state-db` — data source for live metrics
