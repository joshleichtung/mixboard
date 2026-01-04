# mixboard

A personal plugin marketplace for Claude Code. Load skills per-project instead of globally.

## Why

Global skills load everywhere. A Babylon.js skill loads in your Python project. iOS patterns load when you're building a web app. Mixboard lets you install skill packs only where they're relevant.

## Structure

```
mixboard/
├── .claude-plugin/
│   └── marketplace.json     # Registry of available packs
└── plugins/
    └── <pack-name>/
        ├── .claude-plugin/plugin.json
        └── skills/
            └── <skill-name>/SKILL.md
```

## Available Packs

| Pack | Description |
|------|-------------|
| `babylon` | Babylon.js procedural generation, shaders, physics |
| `webaudio` | Tone.js, Web Audio API, synthesis patterns |
| `nextjs` | Next.js 14+ App Router, Server Actions, SSR/ISR |
| `ios` | SwiftUI and Core Audio development |
| `hackathon` | Quick bootstrapping under time pressure |
| `workflow` | Track wins, friction, ideas, weekly reviews |
| `healthtech` | HIPAA-focused architecture review |

## Recipes

Recipes are composition guides that combine multiple skills into a workflow. They're not skills themselves—they tell you *when* and *how* to apply skills together.

| Recipe | Purpose |
|--------|---------|
| [babylon-third-person-exploration](recipes/babylon-third-person-exploration.md) | Camera follow + orbit for exploration games |
| [babylon-stylized-desert-look](recipes/babylon-stylized-desert-look.md) | Cel shading + gradient sky for desert aesthetic |
| [babylon-procedural-biomes-v0](recipes/babylon-procedural-biomes-v0.md) | Height-based object distribution with instancing |

## Usage

Install packs for a project:

```bash
claude plugin install babylon@mixboard
```

Or add to your project's `.claude/settings.json`:

```json
{
  "plugins": ["babylon@mixboard", "webaudio@mixboard"]
}
```

## Creating a Pack

1. Create `plugins/<name>/.claude-plugin/plugin.json`:
```json
{
  "name": "my-pack",
  "description": "What this pack provides",
  "version": "0.1.0"
}
```

2. Add skills in `plugins/<name>/skills/<skill>/SKILL.md`:
```markdown
---
name: my-pack-skill-name
description: What this skill helps Claude do
---

Your skill prompt here...
```

3. Register in `marketplace.json`

## License

MIT
