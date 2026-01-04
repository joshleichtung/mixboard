# Mixboard PRD

## Vision

A composable skill system for Claude Code that covers the full software development lifecycle—from product ideation through design, implementation, and deployment.

## Problem

Claude Code skills today fall into two categories:

1. **Too generic**: General coding assistance that doesn't leverage domain expertise
2. **Too narrow**: Hyper-specific skills that only help with one framework or task

Neither approach supports how developers actually work: moving fluidly between product thinking, system design, implementation, and iteration—often within the same session.

Additionally, loading all skills globally wastes context. A 3D game project doesn't need iOS audio skills. A backend service doesn't need UI component patterns.

## Solution

Mixboard organizes skills into **packs** that can be installed per-project. Packs are grouped by:

1. **SDLC Phase** — What stage of development are you in?
2. **Technology Domain** — What stack are you building with?

This lets developers compose exactly the skills they need for their current project and workflow.

---

## Skill Taxonomy

### Phase 1: Product & Ideation

Skills that help before code is written.

| Pack | Purpose | Example Skills |
|------|---------|----------------|
| `product` | Requirements and feature definition | `prd-writer`, `user-story-generator`, `feature-prioritization` |
| `research` | Technical research and evaluation | `tech-evaluation`, `architecture-options`, `build-vs-buy` |

### Phase 2: Design

Skills for technical and UI design decisions.

| Pack | Purpose | Example Skills |
|------|---------|----------------|
| `architecture` | System design patterns | `system-design`, `data-modeling`, `api-design` |
| `ui-design` | Interface patterns and components | `component-patterns`, `responsive-layout`, `accessibility` |

### Phase 3: Implementation — Frontend

Skills for client-side development.

| Pack | Purpose | Example Skills |
|------|---------|----------------|
| `react` | React patterns and best practices | `hooks-patterns`, `component-architecture`, `performance` |
| `nextjs` | Next.js App Router, SSR, Server Actions | `app-router`, `server-actions`, `data-fetching` |
| `tailwind` | Utility-first CSS patterns | `responsive-design`, `custom-themes`, `component-styling` |
| `shadcn` | shadcn/ui component usage | `component-customization`, `theme-integration` |
| `zustand` | State management patterns | `store-design`, `async-state`, `persistence` |

### Phase 4: Implementation — Backend

Skills for server-side development.

| Pack | Purpose | Example Skills |
|------|---------|----------------|
| `node` | Node.js patterns | `async-patterns`, `error-handling`, `api-design` |
| `supabase` | Supabase backend patterns | `auth`, `realtime`, `rls-policies`, `edge-functions` |
| `postgres` | Database design and queries | `schema-design`, `query-optimization`, `migrations` |

### Phase 5: Implementation — Specialized Domains

Skills for specific application types.

| Pack | Purpose | Example Skills |
|------|---------|----------------|
| `webaudio` | Browser audio with Tone.js/Web Audio API | `synthesis`, `scheduling`, `effects-chains`, `sequencers` |
| `ios` | SwiftUI and iOS audio development | `swiftui-patterns`, `core-audio`, `audiokit` |
| `babylon` | 3D graphics with Babylon.js | `procedural-generation`, `shaders`, `physics`, `scene-architecture` |
| `gamedev` | Game development patterns | `game-loop`, `input-handling`, `state-machines` |

### Phase 6: Quality & Deployment

Skills for shipping and maintaining software.

| Pack | Purpose | Example Skills |
|------|---------|----------------|
| `testing` | Test strategy and implementation | `unit-testing`, `e2e-playwright`, `test-architecture` |
| `deployment` | CI/CD and hosting | `vercel`, `fly-io`, `docker`, `github-actions` |
| `observability` | Monitoring and debugging | `logging`, `error-tracking`, `performance-monitoring` |

### Cross-Cutting

Skills useful across all phases.

| Pack | Purpose | Example Skills |
|------|---------|----------------|
| `typescript` | TypeScript patterns and type safety | `strict-typing`, `generics`, `utility-types` |
| `workflow` | Development process tracking | `log-friction`, `log-wins`, `weekly-review` |

---

## Technology Coverage Analysis

Based on analysis of common project patterns:

| Technology | Frequency | Priority |
|------------|-----------|----------|
| TypeScript | Very High | P0 |
| React | Very High | P0 |
| Tailwind CSS | High | P0 |
| Web Audio / Tone.js | High | P0 |
| Next.js | High | P0 |
| Node.js | High | P1 |
| Vite | Medium | P2 |
| Zustand | Medium | P1 |
| Babylon.js | Medium | P0 (specialized) |
| shadcn/ui | Medium | P1 |
| Swift/SwiftUI | Medium | P0 (specialized) |
| Supabase | Medium | P1 |
| Playwright | Medium | P2 |

---

## Roadmap

### v0.1 — Foundation (Current)

Specialized domain packs for active development:

- [x] `babylon` — Procedural generation, shaders, physics
- [x] `webaudio` — Tone.js, synthesis, scheduling
- [x] `ios` — SwiftUI, Core Audio
- [x] `nextjs` — App Router patterns
- [x] `hackathon` — Rapid prototyping
- [x] `workflow` — Development tracking

### v0.2 — Implementation Expansion

Common frontend/backend patterns:

- [ ] `tailwind` — Responsive design, theming
- [ ] `zustand` — Store patterns, async state
- [ ] `shadcn` — Component customization
- [ ] `supabase` — Auth, realtime, RLS
- [ ] `typescript` — Advanced type patterns

### v0.3 — SDLC Upstream

Product and design phase support:

- [ ] `product` — PRDs, user stories, prioritization
- [ ] `architecture` — System design, API design
- [ ] `ui-design` — Component patterns, accessibility

### v0.4 — Quality & Ops

Testing and deployment:

- [ ] `testing` — Unit, integration, E2E strategies
- [ ] `deployment` — Vercel, Fly.io, Docker patterns
- [ ] `observability` — Logging, monitoring

---

## Skill Structure

Each skill follows this structure:

```
plugins/<pack>/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── <skill-name>/
        └── SKILL.md
```

### SKILL.md Format

```markdown
---
name: <pack>-<skill-name>
description: What this skill helps Claude do
---

[Skill prompt content]
```

### Naming Convention

- Pack names: lowercase, single word when possible (`babylon`, `webaudio`)
- Skill names: `<pack>-<descriptive-name>` (e.g., `babylon-procedural`, `webaudio-tonejs`)

---

## Success Metrics

1. **Context efficiency**: Baseline context usage < 10% with targeted packs
2. **Coverage**: Skills available for all SDLC phases
3. **Composability**: Any combination of packs works without conflicts
4. **Velocity**: Measurable improvement in development speed for covered domains

---

## Open Questions

1. **Skill granularity**: Should packs be coarse (fewer, larger) or fine (many, focused)?
2. **Cross-pack dependencies**: How to handle skills that span domains (e.g., "Next.js with Supabase")?
3. **Version management**: How to handle breaking changes in underlying frameworks?
4. **Community contribution**: What's the model for others to contribute packs?

---

## Appendix: Project Type → Pack Mapping

| Project Type | Recommended Packs |
|--------------|-------------------|
| Next.js web app | `nextjs`, `tailwind`, `shadcn`, `zustand` |
| Web audio/music app | `nextjs`, `webaudio`, `tailwind`, `zustand` |
| 3D game/experience | `babylon`, `webaudio`, `typescript` |
| iOS app | `ios`, `swiftui` |
| Full-stack with Supabase | `nextjs`, `supabase`, `tailwind`, `shadcn` |
| CLI tool | `node`, `typescript` |
| Hackathon project | `hackathon`, plus domain-specific packs |
