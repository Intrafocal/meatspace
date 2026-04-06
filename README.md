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
/jack-in          # Start tracker + reminder loop
/jack-out         # Stop tracker
/meatspace        # Manual check (runs automatically via loop)
```

`/jack-in` starts a background process that tracks your active time and idle breaks via macOS HID input, then sets up a recurring check every 5 minutes. When a reminder is due, your chosen persona delivers it in character.

## Themes

Ships with 10 personas. Set the active theme in `~/.claude/meatspace/theme.json`:

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

### Create your own

Drop a JSON file in `~/.claude/meatspace/themes/` with this structure:

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
- **idle_threshold**: seconds of inactivity before you're considered "on break"
- **active**: `false` to pause all reminders

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
