---
name: workflow-skill-authoring
description: Contract for writing Mixboard skills—structure, scope, declarations, and prohibitions
---

# Mixboard Skill Authoring Contract

This skill defines how all Mixboard skills must be written. It is the meta-contract that all other skills follow.

## SKILL.md Structure

Every skill file follows this format:

```markdown
---
name: <pack>-<descriptor>
description: <one-line summary of what the skill helps Claude do>
---

# <Skill Title>

## Purpose
<What problem this skill solves and when to use it>

## Inputs
<What context, information, or state the skill expects>

## Outputs
<What artifacts or results the skill produces>

## Assumptions
<What must be true for this skill to apply correctly>

## Procedure
<The actual guidance, patterns, or instructions>

## Constraints
<Boundaries and prohibitions specific to this skill>
```

### Naming Convention

- **Pack name**: lowercase, single word (`babylon`, `webaudio`, `ios`)
- **Skill name**: `<pack>-<descriptor>` (e.g., `babylon-procedural`, `webaudio-tonejs`)
- **Descriptor**: concise, action-oriented when possible

## Scope Boundaries

### What Belongs in a Skill

- Domain-specific procedures and patterns
- Best practices for a technology or technique
- Reusable guidance that applies across projects
- Explicit inputs, outputs, and assumptions

### What Does NOT Belong in a Skill

| Instead Use | For |
|-------------|-----|
| **Recipe** | Composition patterns spanning multiple packs |
| **CLAUDE.md** | Project identity, constraints, quality gates |
| **Conversation** | Ephemeral working memory, session-specific context |

### Decision Tree

```
Is it project-specific? → CLAUDE.md
Is it about combining multiple packs? → Recipe
Is it reusable domain knowledge? → Skill
Is it temporary/session-specific? → Conversation (don't persist)
```

## Declaration Model

### Inputs

Declare what the skill expects to be available:

```markdown
## Inputs
- Active Babylon.js project with scene initialized
- Zustand store for game state
- Target terrain dimensions (width, depth)
```

Inputs should be verifiable. If an input is missing, the skill should not proceed blindly.

### Outputs

Declare what the skill produces:

```markdown
## Outputs
- Terrain mesh added to scene
- Zustand store updated with terrain metadata
- Performance budget validated (< 16ms frame time)
```

Outputs should be concrete and checkable.

### Assumptions

Declare what must be true for the skill to work correctly:

```markdown
## Assumptions
- TypeScript strict mode enabled
- Babylon.js version 6.x or higher
- Scene uses right-handed coordinate system
```

Assumptions are preconditions. If violated, results are undefined.

## Prohibitions

All Mixboard skills must observe these constraints:

### 1. No Project-Specific Logic

Skills are reusable across projects. Never include:
- Hardcoded paths, names, or identifiers
- References to specific project structure
- Assumptions about particular codebase layout

**Wrong**: "Update the `src/game/terrain.ts` file"
**Right**: "Create or update the terrain generation module"

### 2. No Conversational Memory Reliance

Skills cannot depend on Claude "remembering" prior context. All required knowledge must be:
- Declared as Inputs
- Available in CLAUDE.md
- Discoverable via Explore mode

**Wrong**: "As discussed earlier, use the noise function"
**Right**: "Inputs: Noise function implementation (or use simplex-noise package)"

### 3. No Tool Calls Unless Explicit

Skills are guidance, not automation. They do not invoke tools unless:
- The skill is explicitly marked as "active" (executes actions)
- Tool usage is documented in the Procedure section
- The action is bounded and reversible

Most skills are "passive"—they inform Claude's approach but don't take actions.

### 4. No Architectural Decisions

Skills provide implementation patterns, not design choices. If a skill encounters an architectural question:
- Stop and note the decision needed
- Defer to Architect mode
- Do not embed opinions about system design

**Wrong**: "Use Redux for state management"
**Right**: "Integrate with the project's state management solution (see CLAUDE.md)"

### 5. No Unbounded Scope

Skills must be completable. They should not:
- Require indefinite exploration
- Depend on external resources that may not exist
- Grow in scope during execution

A skill that cannot be completed is not a skill—it's a project.

## Activation Patterns

Skills should specify when they're relevant:

### Keyword Triggers
```markdown
Activates when: procedural generation, terrain, heightmap, biome
```

### Context Triggers
```markdown
Activates when: working in Babylon.js project with terrain requirements
```

### Explicit Invocation
```markdown
Invoke with: /babylon:procedural or "use babylon-procedural skill"
```

## Quality Checklist

Before finalizing a skill, verify:

- [ ] Name follows `<pack>-<descriptor>` convention
- [ ] Description is one line and actionable
- [ ] Inputs are explicit and verifiable
- [ ] Outputs are concrete and checkable
- [ ] Assumptions are stated as preconditions
- [ ] No project-specific logic
- [ ] No conversational memory reliance
- [ ] No unauthorized tool calls
- [ ] No architectural decisions embedded
- [ ] Scope is bounded and completable
