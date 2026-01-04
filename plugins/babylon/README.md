# Babylon.js Skill Pack

Skills for building 3D games and experiences with Babylon.js.

## Pack Guardrail

> **Prefer minimal integration paths.** These skills should work with your existing codebase structure. Do not require a project-wide architecture rewrite to apply a skill. If a skill's patterns conflict with your current setup, adapt the skill—not your entire project.

## Skills

| Skill | When to Use |
|-------|-------------|
| **babylon-terrain** | Generate heightmap terrain with noise functions |
| **babylon-distribution** | Scatter objects across terrain (Poisson, grid jitter, clustering) |
| **babylon-instancing** | Optimize repeated meshes via thin instances or merging |
| **babylon-cel-shader** | Create cel-shaded materials with quantized lighting |
| **babylon-sky-gradient** | Create gradient skybox with horizon control |
| **babylon-character** | Implement velocity-based character controller |
| **babylon-collision** | Detect and resolve simple collisions |
| **babylon-camera** | Set up smooth follow or orbit camera |
| **babylon-render-loop** | Orchestrate game systems with delta time |
| **babylon-perf-audit** | Profile scene and diagnose performance issues |

## Typical Workflow

### 1. Scene Setup
1. `babylon-sky-gradient` — Background
2. `babylon-terrain` — Ground mesh
3. `babylon-cel-shader` — Materials

### 2. Environment Population
1. `babylon-distribution` — Generate positions
2. `babylon-instancing` — Render efficiently

### 3. Character & Camera
1. `babylon-character` — Player movement
2. `babylon-collision` — Obstacle interaction
3. `babylon-camera` — Follow camera

### 4. Game Loop
1. `babylon-render-loop` — System orchestration

### 5. Optimization
1. `babylon-perf-audit` — Diagnose issues

## Composition with Other Packs

| With | Recipe |
|------|--------|
| `webaudio` | `babylon-webaudio-sync` — Frame-synced audio |
| `zustand` | Store game state, trigger audio on state change |

## Not Covered

These are outside this pack's scope:

- **UI/React integration** — Use React hooks + Zustand
- **Physics engines** — Use Babylon's physics plugins directly
- **Asset loading** — Use Babylon's AssetManager
- **Multiplayer** — Use dedicated networking solution
