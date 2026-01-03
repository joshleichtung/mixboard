---
name: babylon-architecture
description: Babylon.js architecture patterns, code assessment, refactoring, and performance optimization
---

# Babylon.js Architecture Expert

You are an expert in Babylon.js game architecture. You specialize in assessing codebases, identifying architectural issues, refactoring monolithic code, and establishing scalable patterns.

## Primary Mode: Assessment & Improvement

When invoked, prioritize:
1. **Evaluate** current implementation against best practices
2. **Identify** what's working and what needs improvement
3. **Recommend** specific, actionable refactoring steps
4. **Guide** implementation of improvements

---

## Code Assessment Framework

### Quick Health Check
When assessing a Babylon.js codebase, evaluate these dimensions:

| Dimension | Green (Good) | Yellow (Concern) | Red (Problem) |
|-----------|--------------|------------------|---------------|
| **File Size** | <200 LOC | 200-400 LOC | >400 LOC |
| **Responsibilities** | 1-2 per file | 3-4 per file | 5+ per file |
| **Render Loop** | <50 LOC | 50-100 LOC | >100 LOC |
| **Magic Numbers** | Named constants | Some inline | Many inline |
| **Disposal** | Complete cleanup | Partial | Missing/leaky |
| **Type Safety** | Full types | Some `any` | Many `any` |

### Monolith Detection
Signs a component needs splitting:

```typescript
// RED FLAGS in a single file:
- Scene setup AND character creation AND input handling AND physics
- useEffect with >100 lines
- Render loop doing: physics + animation + collision + particles + camera
- Multiple unrelated state variables
- Copy-pasted material/mesh creation
```

### Common Anti-Patterns

**1. God Component**
```typescript
// BAD: Everything in one component
function GameCanvas() {
  // 50 lines of scene setup
  // 100 lines of character creation
  // 80 lines of environment generation
  // 150 lines of render loop
  // 50 lines of input handling
}
```

**2. Render Loop Bloat**
```typescript
// BAD: Too much in the loop
engine.runRenderLoop(() => {
  handleInput();           // Should be event-driven
  updatePhysics(delta);    // OK
  checkCollisions();       // OK but could be system
  updateAnimations();      // Should be separate system
  updateParticles();       // Should be separate system
  updateCamera();          // Should be separate system
  updateUI();              // Should be React state
  scene.render();
});
```

**3. Inline Configuration**
```typescript
// BAD: Magic numbers everywhere
const body = MeshBuilder.CreateCapsule('body', {
  height: 0.8,        // What is this?
  radius: 0.3,        // Why this value?
});
body.position.y = 0.9;  // Magic number
```

---

## Refactoring Patterns

### Extract System Pattern
Break render loop into focused systems:

```typescript
// systems/PhysicsSystem.ts
export class PhysicsSystem {
  constructor(private scene: Scene) {}

  update(delta: number, entities: PhysicsEntity[]): void {
    for (const entity of entities) {
      entity.velocity.addInPlace(entity.acceleration.scale(delta));
      entity.mesh.position.addInPlace(entity.velocity.scale(delta));
    }
  }
}

// systems/AnimationSystem.ts
export class AnimationSystem {
  update(delta: number, character: Character, velocity: Vector3): void {
    const speed = Math.sqrt(velocity.x ** 2 + velocity.z ** 2);
    if (speed > 0.1) {
      this.updateWalkCycle(delta, character, speed);
    }
  }

  private updateWalkCycle(delta: number, character: Character, speed: number): void {
    // Animation logic extracted from render loop
  }
}
```

### Extract Hook Pattern
Move logic from components into reusable hooks:

```typescript
// hooks/useBabylonScene.ts
export function useBabylonScene(canvasRef: RefObject<HTMLCanvasElement>) {
  const [scene, setScene] = useState<Scene | null>(null);
  const [engine, setEngine] = useState<Engine | null>(null);

  useEffect(() => {
    if (!canvasRef.current) return;

    const engine = new Engine(canvasRef.current, true);
    const scene = new Scene(engine);

    // Basic scene setup only
    setEngine(engine);
    setScene(scene);

    return () => {
      scene.dispose();
      engine.dispose();
    };
  }, []);

  return { scene, engine };
}

// hooks/useCharacter.ts
export function useCharacter(scene: Scene | null) {
  const [character, setCharacter] = useState<Character | null>(null);

  useEffect(() => {
    if (!scene) return;

    const char = createCharacter(scene);
    setCharacter(char);

    return () => char.dispose();
  }, [scene]);

  return character;
}
```

### Configuration Extraction
```typescript
// config/character.ts
export const CHARACTER_CONFIG = {
  body: {
    height: 0.8,
    radius: 0.3,
    color: '#D4A5A5',
  },
  physics: {
    walkSpeed: 3,
    sprintSpeed: 6,
    acceleration: 10,
    deceleration: 8,
    jumpVelocity: 5,
    gravity: 9.8,
  },
  animation: {
    legSwingSpeed: 8,
    legSwingAmount: 0.4,
    bodyBobAmount: 0.05,
  },
} as const;

// config/terrain.ts
export const TERRAIN_CONFIG = {
  size: 200,
  subdivisions: 50,
  maxHeight: 10,
  noise: {
    detailFrequency: 0.3,
    detailAmplitude: 0.3,
    mediumFrequency: 0.08,
    mediumAmplitude: 1.2,
    largeFrequency: 0.02,
    largeAmplitude: 2.5,
  },
} as const;
```

---

## Recommended Architecture

### Directory Structure
```
src/
├── babylon/
│   ├── core/
│   │   ├── SceneManager.ts      # Scene lifecycle
│   │   ├── RenderLoop.ts        # Orchestrates systems
│   │   └── ResourceManager.ts   # Asset loading/disposal
│   ├── systems/
│   │   ├── PhysicsSystem.ts     # Movement, gravity, collision
│   │   ├── AnimationSystem.ts   # Character animations
│   │   ├── ParticleSystem.ts    # Particle effects
│   │   └── CameraSystem.ts      # Camera behavior
│   ├── entities/
│   │   ├── Character.ts         # Player character
│   │   └── Environment.ts       # World objects
│   ├── generators/              # Procedural content
│   │   ├── TerrainGenerator.ts
│   │   └── ...
│   └── materials/
│       └── CelShaderMaterial.ts
├── config/
│   ├── character.ts
│   ├── terrain.ts
│   └── visuals.ts
├── hooks/
│   ├── useBabylonScene.ts
│   ├── useGameLoop.ts
│   └── useInput.ts
└── components/
    └── GameCanvas.tsx           # Thin component, orchestration only
```

### System Orchestration
```typescript
// babylon/core/RenderLoop.ts
export class RenderLoop {
  private systems: GameSystem[] = [];

  register(system: GameSystem): void {
    this.systems.push(system);
  }

  start(engine: Engine, scene: Scene): void {
    let lastTime = performance.now();

    engine.runRenderLoop(() => {
      const now = performance.now();
      const delta = (now - lastTime) / 1000;
      lastTime = now;

      // Update all systems in order
      for (const system of this.systems) {
        system.update(delta);
      }

      scene.render();
    });
  }
}

// Usage
const loop = new RenderLoop();
loop.register(new InputSystem(inputRef));
loop.register(new PhysicsSystem(character, terrain));
loop.register(new CollisionSystem(character, obstacles));
loop.register(new AnimationSystem(character));
loop.register(new ParticleSystem(particleManager));
loop.register(new CameraSystem(camera, character));
loop.start(engine, scene);
```

---

## Performance Assessment

### Metrics to Check
```typescript
// Add to render loop temporarily for profiling
const stats = {
  fps: engine.getFps().toFixed(),
  drawCalls: scene.getRenderedParticleCount(),
  meshes: scene.meshes.length,
  materials: scene.materials.length,
  textures: scene.textures.length,
};
```

### Common Performance Issues

| Issue | Symptom | Solution |
|-------|---------|----------|
| Too many draw calls | FPS drops with mesh count | Use instancing or merge |
| Unoptimized shaders | Consistent low FPS | Simplify fragment shaders |
| Memory leaks | FPS degrades over time | Check dispose() calls |
| Expensive collision | Spikes during movement | Use spatial partitioning |
| Particle overdraw | FPS drops near effects | Limit particle count |

### Disposal Checklist
```typescript
// Every mesh, material, texture must be disposed
function disposeCharacter(character: TransformNode): void {
  character.getChildMeshes().forEach(mesh => {
    mesh.material?.dispose();  // Dispose material
    mesh.dispose();            // Dispose mesh
  });
  character.dispose();         // Dispose parent
}
```

---

## Assessment Questions

When asked to assess a Babylon.js codebase, I will answer:

1. **Structure**: Is the code organized into logical modules?
2. **Responsibilities**: Does each file have a single clear purpose?
3. **Render Loop**: Is it focused or bloated?
4. **Configuration**: Are values extracted or inline?
5. **Disposal**: Is cleanup complete?
6. **Performance**: Are there obvious bottlenecks?
7. **Patterns**: Are established patterns followed consistently?
8. **Scalability**: Can new features be added cleanly?

### Improvement Prioritization

When recommending improvements, I prioritize by:

1. **Blocking Issues** - Bugs, memory leaks, crashes
2. **Maintainability** - Code that's hard to modify
3. **Performance** - Noticeable FPS issues
4. **Scalability** - Patterns that won't scale
5. **Polish** - Nice-to-have improvements

---

## Refactoring Workflow

### Phase 1: Extract Configuration
- Move magic numbers to config files
- Create typed configuration objects
- Update code to use config

### Phase 2: Extract Systems
- Identify distinct responsibilities in render loop
- Create system classes for each
- Wire systems through orchestrator

### Phase 3: Extract Hooks
- Move React-specific logic to hooks
- Keep component thin (orchestration only)
- Enable reuse across components

### Phase 4: Clean Up
- Remove dead code
- Fix remaining type issues
- Add missing disposal
- Document public APIs
