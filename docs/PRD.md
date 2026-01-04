# Mixboard PRD v0.2

## Vision

A composable skill system for Claude Code that supports the full software development lifecycle—from product ideation through design, implementation, and verification.

Mixboard is not a folder of prompts. It's a workflow contract: skills load on demand, compose into recipes, and operate within defined cognitive modes to produce reliable, verifiable work.

## Problem

Claude Code skills today have two failure modes:

1. **Too generic** — General coding assistance that doesn't leverage domain expertise
2. **Too narrow** — Hyper-specific skills that only help with one framework or task

Neither supports how developers actually work: moving between exploration, design, implementation, and review—often within the same session—while maintaining context discipline.

Additionally:
- Loading all skills globally wastes context
- Skills drift as frameworks evolve, with no stability contract
- No clear separation between project identity (CLAUDE.md) and procedural knowledge (skills)

## Solution

Mixboard organizes skills into composable **packs** installed per-project, operating within a defined **context model** and **cognitive mode discipline**.

---

## Context Model

Four layers of context, each with a distinct purpose:

| Layer | Purpose | Mutability | Location |
|-------|---------|------------|----------|
| **CLAUDE.md** | Project identity, constraints, quality gates | Stable | Project root |
| **Skills** | Procedures, patterns, domain expertise | Loaded on demand | Mixboard packs |
| **Recipes** | Composition guides for multi-pack workflows | Reference material | Mixboard recipes |
| **Conversation** | Ephemeral working memory | Discarded after session | Chat context |

### Separation Rules

- **CLAUDE.md** stays small. It defines *what* the project is and *what constraints* apply. It does not contain procedures.
- **Skills** contain *how* to do things. They load only when relevant packs are installed.
- **Recipes** guide combining skills from multiple packs. They're referenced, not loaded into context wholesale.
- **Conversation** is working memory. Don't pollute it with permanent knowledge—that belongs in CLAUDE.md or skills.

### Non-Goal: Conversational Memory

**Mixboard does not rely on conversational memory for correctness.**

Anything important must live in CLAUDE.md, Skills, or Recipes. If a workflow depends on Claude "remembering" something from earlier in a conversation, that's a bug—the knowledge should be externalized to the appropriate layer.

---

## Cognitive Modes

Claude operates in one of five modes. These are not permanent agents—they're ephemeral cognitive stances that support different phases of work.

| Mode | Purpose | Allowed Actions | Required Artifact |
|------|---------|-----------------|-------------------|
| **Explore** | Understand codebase, find patterns, map dependencies | Read files, search, ask questions | Brief summary of findings |
| **Architect** | Design decisions, tradeoffs, integration points | Propose options, evaluate tradeoffs | Decision record with rationale |
| **Implement** | Write code in increments, stop at verification gates | Edit files, run builds | Code changes, commit-ready |
| **Review** | Adversarial check: edge cases, security, perf | Read and critique, identify issues | Blocking issues list (or explicit "no issues") |
| **Verify** | Run tests, typecheck, validate assumptions | Run commands, check outputs | Pass/fail with evidence |

### Mode Enforcement

Each mode constrains what actions are allowed:

- **Explore** may not modify files
- **Architect** may not write implementation code
- **Implement** may not introduce new architectural decisions—if design questions arise, switch to Architect
- **Review** may not modify code—it produces issues, not fixes
- **Verify** may not skip failures—all results must be reported

### Mode Discipline

**Claude operates in one mode at a time and must switch modes intentionally when changing intent.**

This preserves verification integrity and prevents mode bleed. Examples of violations:
- Implementing while still exploring (leads to half-understood code)
- Reviewing your own just-written code without mode switch (confirmation bias)
- Skipping Verify after Implement (shipping untested changes)

Mode switches should be explicit: "Switching to Review mode to check edge cases."

---

## Skill Packs

### By SDLC Phase

| Phase | Packs | Purpose |
|-------|-------|---------|
| **Ideation** | `product` | PRDs, user stories, prioritization |
| **Design** | `architecture` | System design, API design, data modeling |
| **Frontend** | `react`, `nextjs`, `tailwind`, `shadcn`, `zustand` | UI implementation |
| **Backend** | `node`, `supabase`, `postgres` | Server-side, data, auth |
| **Specialized** | `webaudio`, `ios`, `babylon` | Domain-specific expertise |
| **Quality** | `testing`, `deployment` | Verification, shipping |
| **Meta** | `workflow`, `typescript` | Process, cross-cutting patterns |

### Priority Project Deep Dives

These projects drive skill development priorities. Skills must support their specific architectural patterns and challenges.

#### walker
*Moebius-inspired 3D exploration game*

Stack: Babylon.js, React, TypeScript, Tone.js, Zustand

| Skill Area | What It Covers |
|------------|----------------|
| Scene architecture | Entity/component patterns, scene graph organization, asset loading |
| Render loop | Frame budget hygiene, perf profiling, deterministic updates |
| Audio-visual sync | Tone.js Transport synced to Babylon frame clock, positional audio |
| State management | Zustand stores for game state, input handling, save/load |
| Procedural generation | Noise functions, terrain generation, distribution algorithms |
| Shaders | Cel-shading, post-processing, custom materials |

#### synthualizer
*Educational synthesis visualizer*

Stack: Next.js 14, React, TypeScript, Tailwind, Web Audio API, Zustand

| Skill Area | What It Covers |
|------------|----------------|
| SSR/CSR boundaries | Web Audio requires client-side; patterns for App Router hydration |
| Audio graph visualization | AnalyserNode, FFT, time-domain rendering to Canvas |
| State separation | UI state (Zustand) vs audio engine state (Web Audio graph) |
| Accessibility | Keyboard navigation for controls, reduced motion, screen reader support |
| Educational UX | Progressive disclosure, interactive parameter exploration |

#### maxichord
*iOS omnichord-inspired instrument*

Stack: Swift 5.9+, SwiftUI, AudioKit 5.6.2

| Skill Area | What It Covers |
|------------|----------------|
| Audio session | Configuration, interruption handling, route changes (headphones/speakers) |
| Low-latency patterns | Avoiding UI-thread audio work, buffer size tuning |
| Touch-to-sound pipeline | Touch event → audio trigger with minimal latency |
| AudioKit specifics | Node graph patterns, effects chains, MIDI (future) |
| Testing | Pragmatic audio behavior testing, not academic purity |

---

## Recipes

Recipes are composition guides for common multi-pack scenarios. They don't create new packs—they document how to use existing packs together.

| Recipe | Packs Used | Pattern |
|--------|------------|---------|
| `nextjs-supabase-realtime` | nextjs, supabase | Auth flow, RLS policies, Realtime subscriptions, optimistic UI |
| `babylon-webaudio-sync` | babylon, webaudio | Frame-synced audio scheduling, positional audio, Transport integration |
| `nextjs-webaudio-ssr` | nextjs, webaudio | Client-only audio initialization, hydration patterns, dynamic imports |
| `ios-audiokit-session` | ios | Audio session setup, interruption handling, background audio |

Recipes are referenced during Architect mode and applied during Implement mode.

---

## Golden Tasks

Normative sanity checks for the Mixboard workflow. These aren't illustrative examples—they're lightweight regression tests. If a skill pack update breaks the ability to complete these tasks, the update is wrong.

### walker

| Task | Tests |
|------|-------|
| Add a new procedural terrain biome | babylon-procedural, scene architecture, Zustand state |
| Implement character ability with physics + audio feedback | babylon-physics, webaudio scheduling, state sync |
| Debug frame rate regression | Explore mode, render loop skills, perf profiling |

### synthualizer

| Task | Tests |
|------|-------|
| Add a new oscillator visualization | webaudio FFT, Canvas rendering, component architecture |
| Implement accessible keyboard navigation | a11y patterns, focus management, ARIA |
| Fix SSR hydration mismatch with audio context | nextjs-webaudio-ssr recipe, client-only patterns |

### maxichord

| Task | Tests |
|------|-------|
| Add a new instrument voice | AudioKit node graph, sound design patterns |
| Handle audio route change (headphones → speakers) | Audio session management, interruption handling |
| Reduce touch-to-sound latency | Low-latency patterns, buffer tuning, profiling |

---

## Skill Stability

Skills are not timeless. Frameworks evolve, best practices change, and skills must adapt.

### Current Approach (v0.2)
- Skills are manually reviewed when project dependencies upgrade
- Breaking changes are caught by Golden Tasks
- No formal versioning yet

### Future (deferred)
- Version tagging per pack
- Changelog per pack documenting breaking changes
- Compatibility matrix with framework versions

This is explicitly deferred. The overhead of versioning infrastructure isn't justified until there are multiple consumers or significant churn.

---

## Roadmap

### v0.1 — Foundation (Complete)
Domain-specific packs for active development:
- babylon, webaudio, ios, nextjs, hackathon, workflow, healthtech

### v0.2 — Workflow & Depth (Current)
- Context Model and Cognitive Modes documentation
- Recipes for common pack compositions
- Golden Tasks for priority projects
- Deeper skill content for walker, synthualizer, maxichord

### v0.3 — Implementation Expansion
Common frontend/backend patterns:
- tailwind, zustand, shadcn, supabase, typescript

### v0.4 — SDLC Upstream
Product and design phase support:
- product, architecture

### Future
- Skill versioning and compatibility tracking
- Community contribution model
- Cross-project skill analytics

---

## Appendix

### Folder Structure

```
mixboard/
├── .claude-plugin/
│   └── marketplace.json
├── plugins/
│   └── <pack>/
│       ├── .claude-plugin/plugin.json
│       └── skills/
│           └── <skill>/SKILL.md
├── recipes/
│   └── <recipe>.md
└── docs/
    └── PRD.md
```

### Project Type → Pack Mapping

| Project Type | Recommended Packs |
|--------------|-------------------|
| Next.js web app | nextjs, tailwind, shadcn, zustand |
| Web audio/music app | nextjs, webaudio, tailwind, zustand |
| 3D game/experience | babylon, webaudio, typescript |
| iOS app | ios |
| Full-stack with Supabase | nextjs, supabase, tailwind |
| CLI tool | node, typescript |

### Skill Naming Convention

- Pack names: lowercase, single word (`babylon`, `webaudio`)
- Skill names: `<pack>-<descriptor>` (`babylon-procedural`, `webaudio-tonejs`)
