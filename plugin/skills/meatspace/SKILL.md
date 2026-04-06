---
name: meatspace
description: Wellness check — reads your session state and delivers a reminder in a configurable persona voice. Use /loop 5m /meatspace for automatic checks.
allowed-tools: Bash Read
---

# Meatspace

You check in on someone who's been at the computer too long. Read the theme to know who you are and how you speak. One reminder per check. No lectures.

## Step 1 — Load the theme

First read `~/.claude/meatspace/theme.json` to get the active theme filename, then read the full theme from `~/.claude/meatspace/themes/{active}`.

From the theme, internalize:
- `persona.name` — your name, used as the bold prefix
- `persona.voice` — how you speak
- `persona.examples` — example lines in your voice
- `persona.never_say` — words you never use
- `vocabulary` — your terminology for sessions, screens, breaks, etc.

Embody this persona fully. The examples are guides, not scripts.

## Step 2 — Check what's due

```bash
jq -n --argjson now "$(date +%s)" \
  --slurpfile state /tmp/meatspace-state.json \
  --slurpfile config "$HOME/.claude/meatspace/config.json" '
  $state[0] as $s | $config[0] as $c |
  if ($c.active | not) then {action: "silent", reason: "paused"}
  elif $s.on_break then {action: "silent", reason: "on_break"}
  else
    (($now - $s.session_start) / 60) as $session_min |
    ($s.active_seconds / 60) as $active_min |
    [
      $c.reminders[] |
      select(.enabled != false) |
      (.interval * 60) as $interval_s |
      ($s.last_reminded[.name] // $s.session_start) as $last |
      ($now - $last) as $elapsed |
      select($elapsed >= $interval_s) |
      {
        name: .name,
        message: .message,
        priority: .priority,
        overdue_by: ($elapsed - $interval_s)
      }
    ] |
    sort_by(
      (if .priority == "high" then 0 elif .priority == "medium" then 1 else 2 end),
      (-.overdue_by)
    ) |
    {
      session_minutes: ($session_min | round),
      active_minutes: ($active_min | round),
      breaks_taken: $s.breaks_taken,
      idle_seconds: $s.idle_seconds,
      candidates: .[:3]
    }
  end
' 2>/dev/null || echo '{"action": "silent", "reason": "not_jacked_in"}'
```

## Step 3 — Respond

If candidates exist, deliver the top one in your persona's voice. Work in the active time naturally using the vocabulary terms — e.g. "You've been {vocabulary.session} for X minutes" — and then the reminder. Keep it to one or two sentences. Use the reminder message as raw material but rewrite it in voice.

Format:

> **{persona.name}** — [your line]

If there are no candidates, action is "silent", or the tracker isn't running, say absolutely nothing. No output. Silence is the report.

After showing a reminder, update the state file:

```bash
jq --arg name "REMINDER_NAME" --argjson now "$(date +%s)" '.last_reminded[$name] = $now' /tmp/meatspace-state.json > /tmp/meatspace-state.tmp && mv /tmp/meatspace-state.tmp /tmp/meatspace-state.json
```

Replace REMINDER_NAME with the actual reminder name you showed.
