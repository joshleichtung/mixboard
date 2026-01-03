# mixboard

Composable skill packs for Claude Code. Mix and match based on your project type.

## Installation

```bash
/plugin marketplace add josh/mixboard
```

## Available Packs

| Pack | Description |
|------|-------------|
| `nextjs` | Next.js 14+ App Router, Server Actions, SSR/ISR |
| `webaudio` | Tone.js, Web Audio API, synthesis, scheduling |
| `babylon` | Babylon.js procedural generation, shaders, physics |
| `ios` | iOS and SwiftUI development patterns |
| `hackathon` | Quick bootstrapping for time pressure |
| `workflow` | Track wins, friction, ideas, weekly reviews |

## Usage

Install packs for your project:

```bash
/plugin install nextjs@mixboard webaudio@mixboard
```

Or add to `.claude/settings.json` for auto-install:

```json
{
  "enabledPlugins": {
    "nextjs@mixboard": true,
    "webaudio@mixboard": true
  }
}
```

## Structure

```
plugins/
├── nextjs/
│   ├── .claude-plugin/plugin.json
│   └── skills/
│       └── nextjs-patterns/SKILL.md
├── webaudio/
│   └── ...
└── ...
```

Each pack contains skills that load on-demand when Claude detects relevant tasks.
