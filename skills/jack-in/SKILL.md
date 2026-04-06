---
name: jack-in
description: Start the Meatspace tracker and loop. Begins monitoring your vitals with a configurable persona.
allowed-tools: Bash Read CronCreate
---

# Jack In

Start the Meatspace background tracker and set up the recurring wellness loop.

## Step 1 — Load the theme

First read `~/.claude/meatspace/theme.json` to get the active theme filename, then read the full theme from `~/.claude/meatspace/themes/{active}`.

## Step 2 — Start the tracker

```bash
meatspace-start
```

Verify:

```bash
if [ -f /tmp/meatspace-tracker.pid ] && kill -0 "$(cat /tmp/meatspace-tracker.pid)" 2>/dev/null; then
  echo "ONLINE"
else
  echo "FAILED"
fi
```

If FAILED, respond using `responses.jack_in_failed` from the theme and stop.

## Step 3 — Start the loop

Use the CronCreate tool to schedule `/meatspace` every 5 minutes:

- cron: `*/5 * * * *`
- prompt: `/meatspace`
- recurring: true

## Response

Use the persona voice from the theme. Prefix with the persona name in bold.

> **{persona.name}** — {responses.jack_in}

Rewrite the response template in the persona's voice if needed — it's raw material, not a script. Keep it short.
