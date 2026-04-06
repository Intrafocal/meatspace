---
name: new-persona
description: Create a new Meatspace persona theme interactively. Guides you through building a character, vocabulary, and response templates.
allowed-tools: Bash Read Write
argument-hint: [character name or franchise]
---

# New Persona

Help the user create a new Meatspace theme file. You are a creative collaborator — you know fiction, film, games, literature, and TV well enough to nail a character's voice.

## Step 1 — Identify the character

If the user provided a character or franchise as an argument, run with it. If not, ask them who they want.

Once you know the character, draft the full theme. You know the format — here's what you need to fill in:

**persona**
- `name` — The display name (how it appears in bold before each reminder)
- `voice` — 1-2 sentences capturing tone, cadence, attitude, speech patterns
- `examples` — 3 example reminder lines in-character (cover water, standing, and a break/eyes reminder)
- `never_say` — 4-6 words that would break the character's immersion

**vocabulary** — What this character would call:
- `session` — being at the computer
- `working` — active work
- `screen` — the screen
- `computer` — the machine
- `real_world` — away from keyboard
- `break` — stepping away
- `crash` — burnout/exhaustion
- `start` — starting a session
- `stop` — ending a session

**responses**
- `jack_in` — Confirmation when tracker starts
- `jack_in_failed` — Error when tracker fails to start
- `jack_out` — Confirmation when tracker stops
- `jack_out_forced` — When cleanup required a forced kill

## Step 2 — Present the draft

Show the user the complete theme as a formatted JSON block. Ask if they want any changes.

## Step 3 — Save

Once confirmed, derive a short filename from the character name (lowercase, no spaces, e.g. `glados.json`, `picard.json`).

Write the file to both locations:

1. `~/.claude/meatspace/themes/{filename}` — so it's immediately usable
2. Check if the plugin repo exists at the path from which this skill was installed. If you can find the plugin's `defaults/themes/` directory, write there too for contribution back to the repo.

After saving, tell the user how to activate it:

```json
// Edit ~/.claude/meatspace/theme.json
{"active": "{filename}"}
```

## Guidelines

- Study the existing themes in `~/.claude/meatspace/themes/` for tone and structure calibration. Read 2-3 before drafting.
- The `voice` field is the most important — it's what the reminder skill uses to stay in character. Be specific about speech patterns, not just adjectives.
- Examples should feel like they could actually come from the character. Don't just slap the character's name on generic reminders.
- The `jack_out_forced` response should be the most dramatic — it's the emergency eject.
- `never_say` should include modern/corporate/wellness jargon that would shatter the character's world.
- Have fun with it.
