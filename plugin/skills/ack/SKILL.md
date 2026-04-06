---
name: ack
description: Acknowledge a Meatspace reminder to reset its timer. Usage — /ack water, /ack stand, /ack eyes, etc.
allowed-tools: Bash Read
argument-hint: [reminder-name]
---

# Ack

Reset a reminder's timer after the user has done the thing.

## Step 1 — Load the theme

First read `~/.claude/meatspace/theme.json` to get the active theme filename, then read the full theme from `~/.claude/meatspace/themes/{active}`.

## Step 2 — Validate and update

The user should provide a reminder name as the argument (e.g. `water`, `stand`, `eyes`, `stretch`, `outside`, `breathe`).

If no argument, list the available reminders:

```bash
jq -r '.reminders[] | select(.enabled != false) | .name' "$HOME/.claude/meatspace/config.json"
```

Then ask which one to acknowledge.

If a valid reminder name is provided, update the state file:

```bash
jq --arg name "REMINDER_NAME" --argjson now "$(date +%s)" '.last_reminded[$name] = $now' /tmp/meatspace-state.json > /tmp/meatspace-state.tmp && mv /tmp/meatspace-state.tmp /tmp/meatspace-state.json
```

## Step 3 — Respond

Use the persona voice from the theme. Keep it to one short line — a quick acknowledgment.

> **{persona.name}** — [brief acknowledgment in voice]

If the tracker isn't running (no state file), say so in voice.
