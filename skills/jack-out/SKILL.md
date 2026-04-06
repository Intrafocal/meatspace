---
name: jack-out
description: Stop the Meatspace tracker. Disconnects you with a configurable persona.
allowed-tools: Bash Read
---

# Jack Out

Stop the Meatspace background tracker and clean up state.

## Step 1 — Load the theme

First read `~/.claude/meatspace/theme.json` to get the active theme filename, then read the full theme from `~/.claude/meatspace/themes/{active}`.

## Step 2 — Stop the tracker

```bash
meatspace-stop
```

Verify:

```bash
if [ -f /tmp/meatspace-tracker.pid ]; then
  echo "STALE_PID"
else
  echo "CLEAN"
fi
```

If STALE_PID, force cleanup:

```bash
kill "$(cat /tmp/meatspace-tracker.pid)" 2>/dev/null; rm -f /tmp/meatspace-tracker.pid /tmp/meatspace-state.json
```

## Response

Use the persona voice from the theme. Prefix with the persona name in bold.

If CLEAN:

> **{persona.name}** — {responses.jack_out}

If forced cleanup was needed:

> **{persona.name}** — {responses.jack_out_forced}
