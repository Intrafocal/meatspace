---
name: sitting
description: Tell Meatspace you've sat back down. Resumes normal stand reminders.
allowed-tools: Bash Read
---

# Sitting

The user has sat back down. Update the tracker state.

## Step 1 — Load the theme

First read `~/.claude/meatspace/theme.json` to get the active theme filename, then read the full theme from `~/.claude/meatspace/themes/{active}`.

## Step 2 — Update state

```bash
jq --argjson now "$(date +%s)" '
  .posture = "sitting" |
  .posture_since = $now |
  .last_reminded["sit"] = $now
' /tmp/meatspace-state.json > /tmp/meatspace-state.tmp && mv /tmp/meatspace-state.tmp /tmp/meatspace-state.json
```

## Step 3 — Respond

Use the persona voice. One short line. If they were standing for a while, mention how long.

Check how long they stood:

```bash
jq -r '
  if .posture == "standing" and .posture_since then
    ((now | floor) - .posture_since) / 60 | floor | tostring + " minutes standing"
  else
    "unknown"
  end
' /tmp/meatspace-state.json
```

> **{persona.name}** — [brief acknowledgment, optionally noting standing duration]
