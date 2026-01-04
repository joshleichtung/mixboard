---
name: babylon-render-loop
description: Set up system-based render loop with delta time and frame budget awareness
---

# Babylon Render Loop

## Purpose

Orchestrate game systems in a structured render loop with accurate delta time. Use to organize physics, animation, camera, and other per-frame updates.

## Inputs

- Engine reference
- Scene reference
- List of systems to update

## Outputs

- Running render loop that calls systems in order
  - Verify: FPS displayed in console or overlay
  - Verify: Systems called each frame in registration order
  - Verify: Delta time is accurate (~16ms at 60fps, log to verify)

## Assumptions

- Engine and scene already initialized
- Systems implement an `update(delta: number)` method
- React state updates happen outside the render loop

## Procedure

### 1. System Interface

```typescript
interface GameSystem {
  update(delta: number): void;
  dispose?(): void;
}
```

### 2. Render Loop Manager

```typescript
import { Engine, Scene } from '@babylonjs/core';

class RenderLoop {
  private systems: GameSystem[] = [];
  private lastTime = 0;
  private isRunning = false;

  constructor(
    private engine: Engine,
    private scene: Scene
  ) {}

  register(system: GameSystem): void {
    this.systems.push(system);
  }

  unregister(system: GameSystem): void {
    const index = this.systems.indexOf(system);
    if (index !== -1) {
      this.systems.splice(index, 1);
    }
  }

  start(): void {
    if (this.isRunning) return;
    this.isRunning = true;
    this.lastTime = performance.now();

    this.engine.runRenderLoop(() => {
      const now = performance.now();
      const delta = (now - this.lastTime) / 1000;  // Seconds
      this.lastTime = now;

      // Cap delta to prevent spiral of death
      const cappedDelta = Math.min(delta, 0.1);  // Max 100ms

      // Update all systems
      for (const system of this.systems) {
        system.update(cappedDelta);
      }

      this.scene.render();
    });
  }

  stop(): void {
    this.engine.stopRenderLoop();
    this.isRunning = false;
  }

  dispose(): void {
    this.stop();
    for (const system of this.systems) {
      system.dispose?.();
    }
    this.systems = [];
  }
}
```

### 3. Example Systems

```typescript
// Physics system
class PhysicsSystem implements GameSystem {
  constructor(
    private character: CharacterController,
    private input: InputState,
    private camera: ArcRotateCamera
  ) {}

  update(delta: number): void {
    const movement = this.character.update(delta, this.input, this.getCameraForward());
    // Apply movement...
  }

  private getCameraForward(): Vector3 {
    return this.camera.getTarget().subtract(this.camera.position);
  }
}

// Camera system
class CameraSystem implements GameSystem {
  constructor(
    private followCamera: FollowCamera
  ) {}

  update(delta: number): void {
    this.followCamera.update(delta);
  }
}

// Animation system
class AnimationSystem implements GameSystem {
  constructor(
    private character: TransformNode,
    private velocity: () => Vector3
  ) {}

  update(delta: number): void {
    const speed = this.velocity().length();
    // Update walk cycle based on speed...
  }
}
```

### 4. Usage

```typescript
// Setup
const loop = new RenderLoop(engine, scene);

loop.register(new InputSystem(inputRef));
loop.register(new PhysicsSystem(controller, inputRef.current, camera));
loop.register(new CollisionSystem(character, obstacles));
loop.register(new AnimationSystem(character, () => controller.velocity));
loop.register(new CameraSystem(followCamera));

// Start the loop
loop.start();

// Cleanup on unmount
return () => loop.dispose();
```

### 5. Debug Overlay

```typescript
class DebugSystem implements GameSystem {
  private frameCount = 0;
  private lastFpsUpdate = 0;
  private fps = 0;

  constructor(private engine: Engine) {}

  update(delta: number): void {
    this.frameCount++;
    const now = performance.now();

    if (now - this.lastFpsUpdate > 1000) {
      this.fps = this.frameCount;
      this.frameCount = 0;
      this.lastFpsUpdate = now;
      console.log(`FPS: ${this.fps}, Delta: ${(delta * 1000).toFixed(1)}ms`);
    }
  }
}
```

### 6. Frame Budget Awareness

```typescript
class FrameBudgetMonitor implements GameSystem {
  private targetFrameTime = 16.67;  // 60fps in ms

  update(delta: number): void {
    const frameTime = delta * 1000;

    if (frameTime > this.targetFrameTime * 1.5) {
      console.warn(`Frame budget exceeded: ${frameTime.toFixed(1)}ms`);
    }
  }
}
```

### 7. System Ordering

Systems run in registration order. Typical order:

1. **Input** - Capture current input state
2. **Physics** - Calculate movement, apply forces
3. **Collision** - Detect and resolve overlaps
4. **Animation** - Update visual state based on physics
5. **Camera** - Follow updated character position
6. **Particles** - Update effects
7. **Debug** - Logging, overlays

## Constraints

- No React integration in this skill (hooks are separate)
- Systems should be stateless or manage their own state
- Delta time is capped to prevent physics explosion on lag spikes
- No fixed timestep (use interpolation if needed for determinism)

## Activation

Activates when: render loop, game loop, system update, delta time, frame rate, system orchestration
