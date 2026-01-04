---
name: babylon-camera
description: Implement smooth follow camera and orbit camera with constraints
---

# Babylon Camera Systems

## Purpose

Set up cameras that follow or orbit around a target with smooth interpolation and sensible constraints. Use for third-person character cameras or exploration cameras.

## Inputs

- Camera reference:
  - **Follow camera**: Use `FreeCamera` or `UniversalCamera` (position-based)
  - **Orbit camera**: Use `ArcRotateCamera` (angle/radius-based)
- Target to follow (position or TransformNode)
- Smoothing parameters
- Distance/angle constraints (orbit only)

## Outputs

- Camera positioned and oriented correctly each frame
  - Verify: Camera follows target without snapping
  - Verify: Camera respects distance limits (orbit: zoom in/out)
  - Verify: Camera respects angle limits (orbit: can't look underground)
  - Verify: Movement feels smooth, not jerky

## Assumptions

- **Follow camera**: FreeCamera or UniversalCamera (direct position/rotation control)
- **Orbit camera**: ArcRotateCamera (alpha/beta/radius control)
- Target exists and has a position
- Update called every frame with delta time

## Procedure

### 1. Smooth Follow Camera

Uses `FreeCamera` or `UniversalCamera` for direct position control:

```typescript
import { Vector3, FreeCamera, TransformNode, Scene } from '@babylonjs/core';

function createFollowCamera(scene: Scene, target: Vector3): FreeCamera {
  const camera = new FreeCamera('followCamera', new Vector3(0, 5, -10), scene);
  camera.setTarget(target);
  // Do NOT attach controls—we manage position manually
  return camera;
}

class FollowCamera {
  private offset: Vector3;
  private smoothSpeed: number;

  constructor(
    private camera: FreeCamera,  // NOT ArcRotateCamera
    private target: TransformNode,
    options: {
      offset?: Vector3;
      smoothSpeed?: number;
    } = {}
  ) {
    this.offset = options.offset ?? new Vector3(0, 5, -10);
    this.smoothSpeed = options.smoothSpeed ?? 5;
  }

  update(delta: number): void {
    // Desired position = target + offset
    const desiredPosition = this.target.position.add(this.offset);

    // Smooth interpolation
    const t = Math.min(1, this.smoothSpeed * delta);
    this.camera.position = Vector3.Lerp(
      this.camera.position,
      desiredPosition,
      t
    );

    // Look at target (slightly above feet)
    const lookTarget = this.target.position.add(new Vector3(0, 1, 0));
    this.camera.setTarget(lookTarget);
  }

  setOffset(offset: Vector3): void {
    this.offset = offset;
  }
}
```

### 2. Orbit Camera with Constraints

```typescript
import { Scene, ArcRotateCamera, Vector3 } from '@babylonjs/core';

interface OrbitConfig {
  distance: number;
  minDistance: number;
  maxDistance: number;
  minBeta: number;   // Min vertical angle (radians)
  maxBeta: number;   // Max vertical angle (radians)
  sensitivity: number;
}

function setupOrbitCamera(
  scene: Scene,
  target: Vector3,
  config: OrbitConfig
): ArcRotateCamera {
  const camera = new ArcRotateCamera(
    'orbitCamera',
    Math.PI / 2,      // Alpha: horizontal angle
    Math.PI / 4,      // Beta: vertical angle
    config.distance,
    target,
    scene
  );

  // Distance constraints
  camera.lowerRadiusLimit = config.minDistance;
  camera.upperRadiusLimit = config.maxDistance;

  // Vertical angle constraints
  camera.lowerBetaLimit = config.minBeta;
  camera.upperBetaLimit = config.maxBeta;

  // Sensitivity (higher = slower)
  camera.angularSensibilityX = 4000 / config.sensitivity;
  camera.angularSensibilityY = 4000 / config.sensitivity;

  // Smooth zoom
  camera.wheelPrecision = 50;

  // Attach controls
  camera.attachControl(scene.getEngine().getRenderingCanvas(), true);

  return camera;
}

// Example config
const CAMERA_CONFIG: OrbitConfig = {
  distance: 15,
  minDistance: 5,
  maxDistance: 30,
  minBeta: 0.1,           // Don't go below horizon
  maxBeta: Math.PI / 2.2, // Don't go directly overhead
  sensitivity: 1,
};
```

### 3. Camera Target Tracking

Update orbit camera target to follow moving entity:

```typescript
class OrbitFollowCamera {
  constructor(
    private camera: ArcRotateCamera,
    private target: TransformNode,
    private smoothSpeed: number = 8
  ) {}

  update(delta: number): void {
    const targetPos = this.target.position;
    const t = Math.min(1, this.smoothSpeed * delta);

    // Smoothly move camera target
    this.camera.target = Vector3.Lerp(
      this.camera.target,
      targetPos,
      t
    );
  }
}
```

### 4. Camera Collision Avoidance

Prevent camera from going through walls:

```typescript
function adjustCameraForCollision(
  camera: ArcRotateCamera,
  scene: Scene,
  collisionMeshes: Mesh[]
): void {
  const target = camera.target;
  const direction = camera.position.subtract(target).normalize();
  const maxDistance = camera.radius;

  const ray = new Ray(target, direction, maxDistance);
  const hit = scene.pickWithRay(ray, mesh =>
    collisionMeshes.includes(mesh as Mesh)
  );

  if (hit?.hit && hit.distance < maxDistance) {
    // Move camera closer to avoid obstruction
    camera.radius = hit.distance - 0.5;
  }
}
```

### 5. Camera Presets

```typescript
const CAMERA_PRESETS = {
  exploration: {
    distance: 20,
    minDistance: 10,
    maxDistance: 40,
    minBeta: 0.2,
    maxBeta: Math.PI / 2.5,
    sensitivity: 0.8,
  },
  action: {
    distance: 10,
    minDistance: 5,
    maxDistance: 15,
    minBeta: 0.3,
    maxBeta: Math.PI / 3,
    sensitivity: 1.2,
  },
  cinematic: {
    distance: 30,
    minDistance: 20,
    maxDistance: 50,
    minBeta: 0.1,
    maxBeta: Math.PI / 4,
    sensitivity: 0.5,
  },
} as const;
```

### 6. Transition Between Camera Modes

```typescript
function transitionCamera(
  camera: ArcRotateCamera,
  targetRadius: number,
  targetBeta: number,
  duration: number,
  scene: Scene
): void {
  const startRadius = camera.radius;
  const startBeta = camera.beta;
  let elapsed = 0;

  const observer = scene.onBeforeRenderObservable.add(() => {
    elapsed += scene.getEngine().getDeltaTime() / 1000;
    const t = Math.min(1, elapsed / duration);

    // Smooth step
    const smoothT = t * t * (3 - 2 * t);

    camera.radius = startRadius + (targetRadius - startRadius) * smoothT;
    camera.beta = startBeta + (targetBeta - startBeta) * smoothT;

    if (t >= 1) {
      scene.onBeforeRenderObservable.remove(observer);
    }
  });
}
```

## Constraints

- No camera shake (separate concern)
- No input handling beyond built-in ArcRotateCamera controls
- Collision avoidance is optional and simple (raycast only)
- No cutscene/rail camera support

## Activation

Activates when: follow camera, orbit camera, camera smoothing, camera constraints, third-person camera, camera zoom limits

---

## Changelog

- **Fix**: FollowCamera now uses `FreeCamera` (position-based), not `ArcRotateCamera`
- **Clarify**: Inputs/Assumptions now distinguish follow (FreeCamera) vs orbit (ArcRotateCamera)
- **Add**: `createFollowCamera()` helper with note about not attaching controls
