# Meatspace

Themed wellness reminders for [Claude Code](https://claude.ai/claude-code). A background tracker monitors your session activity and idle time, then a configurable persona nudges you to hydrate, stand, stretch, and step away.

## Install

```
/plugin marketplace add Intrafocal/meatspace
/plugin install meatspace
```

Or test locally:

```
claude --plugin-dir /path/to/meatspace
```

## Usage

```
/jack-in              # Start tracker + reminder loop
/jack-out             # Stop tracker
/meatspace            # Manual check (runs automatically via loop)
/standing             # Tell Meatspace you're standing (suppresses stand reminders, starts a sit timer)
/sitting              # Tell Meatspace you've sat back down
/ack <reminder>       # Acknowledge a reminder to reset its timer (e.g. /ack water)
/new-persona [name]   # Create a new theme interactively
```

`/jack-in` starts a background process that tracks your active time and idle breaks via macOS HID input, then sets up a recurring check every 5 minutes. When a reminder is due, your chosen persona delivers it in character.

## Themes

Ships with 13 personas. Set the active theme in `~/.claude/meatspace/theme.json`:

```json
{"active": "molly.json"}
```

| File | Persona | Source |
|---|---|---|
| `molly.json` | Molly Millions | Neuromancer |
| `stilgar.json` | Stilgar | Dune |
| `mother.json` | MU-TH-UR 6000 | Alien |
| `mithrandir.json` | Mithrandir | Lord of the Rings |
| `firekeeper.json` | Fire Keeper | Elden Ring / Dark Souls |
| `dungeonmaster.json` | Dungeon Master | D&D |
| `systemrom.json` | System ROM | 8-Bit Arcade |
| `mcclane.json` | McClane | Die Hard |
| `emh.json` | The Doctor | Star Trek Voyager |
| `goose.json` | Goose | Top Gun |
| `cooper.json` | Agent Cooper | Twin Peaks |
| `iroh.json` | Uncle Iroh | Avatar: The Last Airbender |
| `zardoz.json` | ZARDOZ | Zardoz |

### Create your own with `/new-persona`

The easiest way to create a theme is interactively:

```
/new-persona GLaDOS
/new-persona Captain Picard
/new-persona                   # asks you who you want
```

This walks you through building the character voice, vocabulary, and response templates, then saves the theme file. You can also create one manually — drop a JSON file in `~/.claude/meatspace/themes/` with this structure:

```json
{
  "persona": {
    "name": "Display Name",
    "voice": "Description of how this persona speaks...",
    "examples": [
      "Example reminder line in voice",
      "Another example line"
    ],
    "never_say": ["words", "to", "avoid"]
  },
  "vocabulary": {
    "session": "what you call being at the computer",
    "working": "what you call active work",
    "screen": "what you call the screen",
    "computer": "what you call the machine",
    "real_world": "what you call away-from-keyboard",
    "break": "what you call stepping away",
    "crash": "what you call burnout",
    "start": "what you call starting",
    "stop": "what you call stopping"
  },
  "responses": {
    "jack_in": "Confirmation when tracker starts",
    "jack_in_failed": "Error when tracker fails",
    "jack_out": "Confirmation when tracker stops",
    "jack_out_forced": "When cleanup needed force"
  }
}
```

## Configuration

Edit `~/.claude/meatspace/config.json` to customize reminders:

```json
{
  "reminders": [
    {
      "name": "water",
      "message": "Time to hydrate.",
      "interval": 30,
      "priority": "high",
      "enabled": true
    }
  ],
  "idle_threshold": 300,
  "active": true
}
```

- **interval**: minutes between reminders
- **priority**: `high` | `medium` | `low` (high always shows, low skips if you recently took a break)
- **enabled**: toggle individual reminders
- **posture** (optional): `"sitting"` or `"standing"` — only fires when you're in that posture (set via `/standing` and `/sitting`)
- **idle_threshold**: seconds of inactivity before you're considered "on break"
- **active**: `false` to pause all reminders

## Auto-start on session

By default, Meatspace is opt-in — you run `/jack-in` each session. To start automatically, add a `SessionStart` hook to your `~/.claude/settings.json`:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "meatspace-start"
          }
        ]
      }
    ]
  }
}
```

This starts the background tracker on every session. You'll still need to run `/loop 5m /meatspace` once to start the reminder loop (or add it to a [SessionStart prompt hook](https://docs.anthropic.com/en/docs/claude-code/hooks) if you want it fully hands-free).

## Renaming commands

The skill names (`/jack-in`, `/jack-out`, `/meatspace`, `/new-persona`) are just directory names. To rename a command, rename the directory in your local plugin install:

```bash
# Example: rename /jack-in to /plug-in
mv ~/.claude/plugins/cache/meatspace/skills/jack-in ~/.claude/plugins/cache/meatspace/skills/plug-in
```

Or if you've forked the repo, rename the directories in `skills/` and reinstall.

## Requirements

- macOS (uses `ioreg` for idle detection)
- `jq` (pre-installed on most systems, `brew install jq` if needed)

## How it works

1. `/jack-in` runs `meatspace-start`, which launches `meatspace-tracker` as a background process
2. The tracker writes `/tmp/meatspace-state.json` every 60 seconds with idle time, active time, and break detection
3. A cron loop runs `/meatspace` every 5 minutes, which reads the state + config + theme and delivers the most overdue reminder in the persona's voice
4. `/jack-out` (or session end) kills the tracker and cleans up

## License

MIT
