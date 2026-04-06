---
name: standing
description: Tell Meatspace you're standing. Suppresses stand reminders and starts a sit-down timer.
allowed-tools: Bash Read
---

# Standing

The user is now standing. Update the tracker state to reflect this.

## Step 1 — Load the theme

First read `~/.claude/meatspace/theme.json` to get the active theme filename, then read the full theme from `~/.claude/meatspace/themes/{active}`.

## Step 2 — Update state

```bash
jq --argjson now "$(date +%s)" '
  .posture = "standing" |
  .posture_since = $now |
  .last_reminded["stand"] = $now
' /tmp/meatspace-state.json > /tmp/meatspace-state.tmp && mv /tmp/meatspace-state.tmp /tmp/meatspace-state.json
```

## Step 3 — Respond

Use the persona voice. One short line acknowledging they're on their feet.

> **{persona.name}** — [brief acknowledgment in voice]
