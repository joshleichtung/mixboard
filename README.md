# 🎛️ mixboard

**Stop loading every skill everywhere. Start mixing what you need.**

Claude Code is powerful, but context is finite. If you've got 30 skills loaded globally, you're burning tokens before you type a single character. Mixboard fixes that.

## The Problem

```
Your Claude Code session starts...
├── Loading vim-tips... (you're in a web project)
├── Loading babylon-shaders... (it's a Node CLI)
├── Loading ios-audio... (there's no Xcode in sight)
└── 35% of context gone. You haven't done anything yet.
```

## The Solution

Mixboard is a personal plugin marketplace. Install skill packs per-project. Load babylon skills in your game project. Load webaudio in your sequencer. Load nothing you don't need.

```
Your Claude Code session starts...
├── Loading webaudio-bootstrap... (it's a sequencer project)
├── Loading webaudio-tonejs... (perfect)
└── 4% of context used. Let's build.
```

## Skill Packs

| Pack | What It Does | Skills |
|------|--------------|--------|
| **babylon** | Procedural 3D with Babylon.js | `procedural` `shaders` `physics` `architecture` |
| **webaudio** | Browser audio with Tone.js | `tonejs-advanced` `web-bootstrap` |
| **nextjs** | Next.js 14+ patterns | `nextjs-patterns` |
| **ios** | SwiftUI + Core Audio | `audio-help` |
| **hackathon** | Ship fast under pressure | `bootstrap` `quick-feature` `web-audio-starter` |
| **workflow** | Track friction, wins, ideas | `log-win` `log-friction` `log-idea` `weekly-review` `checklist` |
| **healthtech** | HIPAA-focused dev | `architecture-review` |

## Installation

Add mixboard as a marketplace:

```bash
claude plugin marketplace add josh/mixboard
```

Install packs you need:

```bash
claude plugin install babylon@mixboard
claude plugin install webaudio@mixboard hackathon@mixboard
```

## Per-Project Setup

Add to your project's `.claude/settings.json`:

```json
{
  "plugins": ["babylon@mixboard", "webaudio@mixboard"]
}
```

Now those skills only load in that project.

## Why "Mixboard"?

Like a mixing board routes audio signals, this routes Claude skills. Slide up the faders you need. Mute everything else. Your mix, your session.

## Structure

```
mixboard/
├── .claude-plugin/
│   └── marketplace.json     # Registry of all packs
└── plugins/
    ├── babylon/
    │   ├── .claude-plugin/plugin.json
    │   └── skills/
    │       ├── procedural/SKILL.md
    │       ├── shaders/SKILL.md
    │       ├── physics/SKILL.md
    │       └── architecture/SKILL.md
    ├── webaudio/
    ├── nextjs/
    ├── ios/
    ├── hackathon/
    ├── workflow/
    └── healthtech/
```

## Make Your Own Pack

1. Create a directory in `plugins/`
2. Add `.claude-plugin/plugin.json`
3. Add skills in `skills/<name>/SKILL.md`
4. Update `marketplace.json`

Skills use frontmatter:

```markdown
---
name: my-pack-skill-name
description: What this skill helps Claude do
---

Your skill prompt here...
```

## Results

Before mixboard: **35% context used at baseline**
After mixboard: **~5% context used at baseline**

That's 60k tokens back. Use them for actual work.

---

*Built to make Claude Code sessions leaner. Fork it, remix it, make it yours.*
