---
name: babylon-character
description: Implement velocity-based character controller with acceleration and deceleration
---

# Babylon Character Controller

## Purpose

Implement responsive, physics-based character movement. Use when you need player-controlled movement with proper acceleration, deceleration, and air control.

## Inputs

- Input state (which keys are pressed)
- Delta time (seconds since last frame)
- Physics config (speeds, acceleration values)
- Camera reference (for camera-relative movement)

## Outputs

- Position delta to apply to character mesh
  - Verify: Character moves on WASD/arrows input
  - Verify: Character stops within ~0.2s of releasing keys
  - Verify: Diagonal movement not faster than cardinal (normalize test)
  - Verify: Movement feels responsive, not floaty or slippery

## Assumptions

- Input system provides boolean flags for each direction
- Delta time is calculated correctly (see babylon-render-loop)
- Ground detection exists (raycast or collision)
- Y-axis movement (jump/gravity) handled separately or integrated

## Procedure

### 1. Input State Type

```typescript
interface InputState {
  forward: boolean;
  backward: boolean;
  left: boolean;
  right: boolean;
  jump: boolean;
  sprint: boolean;
}
```

### 2. Physics Config

```typescript
interface CharacterPhysics {
  maxSpeed: number;        // Units per second
  sprintMultiplier: number;
  acceleration: number;    // How fast to reach max speed
  deceleration: number;    // How fast to stop
  gravity: number;
  jumpVelocity: number;
  airControlMultiplier: number;  // 0-1, reduces control in air
}

// Responsive defaults (action game feel)
const DEFAULT_PHYSICS: CharacterPhysics = {
  maxSpeed: 5,
  sprintMultiplier: 1.5,
  acceleration: 25,
  deceleration: 20,
  gravity: 25,
  jumpVelocity: 10,
  airControlMultiplier: 0.3,
};
```

### 3. Character Controller Class

```typescript
import { Vector3 } from '@babylonjs/core';

class CharacterController {
  velocity = Vector3.Zero();
  isGrounded = true;

  constructor(private physics: CharacterPhysics = DEFAULT_PHYSICS) {}

  update(delta: number, input: InputState, cameraForward: Vector3): Vector3 {
    // 1. Get desired direction (camera-relative)
    const desired = this.getDesiredDirection(input, cameraForward);

    // 2. Calculate effective max speed
    const maxSpeed = input.sprint
      ? this.physics.maxSpeed * this.physics.sprintMultiplier
      : this.physics.maxSpeed;

    // 3. Apply horizontal movement
    const control = this.isGrounded ? 1 : this.physics.airControlMultiplier;
    this.applyHorizontalMovement(desired, maxSpeed, delta, control);

    // 4. Apply gravity
    if (!this.isGrounded) {
      this.velocity.y -= this.physics.gravity * delta;
    }

    // 5. Handle jump
    if (input.jump && this.isGrounded) {
      this.velocity.y = this.physics.jumpVelocity;
      this.isGrounded = false;
    }

    // 6. Return position delta
    return this.velocity.scale(delta);
  }

  private getDesiredDirection(input: InputState, cameraForward: Vector3): Vector3 {
    // Flatten camera forward to XZ plane
    const forward = new Vector3(cameraForward.x, 0, cameraForward.z).normalize();
    const right = Vector3.Cross(forward, Vector3.Up()).normalize();

    const direction = Vector3.Zero();

    if (input.forward) direction.addInPlace(forward);
    if (input.backward) direction.subtractInPlace(forward);
    if (input.right) direction.addInPlace(right);
    if (input.left) direction.subtractInPlace(right);

    // CRITICAL: Normalize to prevent diagonal speed boost
    if (direction.length() > 0.1) {
      direction.normalize();
    }

    return direction;
  }

  private applyHorizontalMovement(
    desired: Vector3,
    maxSpeed: number,
    delta: number,
    control: number
  ): void {
    const currentH = new Vector3(this.velocity.x, 0, this.velocity.z);
    const desiredH = desired.scale(maxSpeed);

    if (desired.length() > 0.1) {
      // Accelerate toward desired velocity
      const diff = desiredH.subtract(currentH);
      const accelAmount = Math.min(
        this.physics.acceleration * delta * control,
        diff.length()
      );
      const change = diff.normalize().scale(accelAmount);
      this.velocity.x += change.x;
      this.velocity.z += change.z;
    } else {
      // Decelerate to stop
      const speed = currentH.length();
      const decelAmount = this.physics.deceleration * delta;

      if (speed > decelAmount) {
        const factor = (speed - decelAmount) / speed;
        this.velocity.x *= factor;
        this.velocity.z *= factor;
      } else {
        this.velocity.x = 0;
        this.velocity.z = 0;
      }
    }

    // Clamp to max speed
    const hSpeed = Math.sqrt(this.velocity.x ** 2 + this.velocity.z ** 2);
    if (hSpeed > maxSpeed) {
      const factor = maxSpeed / hSpeed;
      this.velocity.x *= factor;
      this.velocity.z *= factor;
    }
  }

  // Call when character lands
  onGrounded(): void {
    this.isGrounded = true;
    if (this.velocity.y < 0) {
      this.velocity.y = 0;
    }
  }

  // Call when character leaves ground
  onAirborne(): void {
    this.isGrounded = false;
  }
}
```

### 4. Integration with Mesh

```typescript
function updateCharacter(
  mesh: TransformNode,
  controller: CharacterController,
  input: InputState,
  camera: ArcRotateCamera,
  delta: number,
  getGroundHeight: (x: number, z: number) => number
): void {
  // Get camera forward
  const cameraForward = camera.getTarget().subtract(camera.position);

  // Calculate movement
  const positionDelta = controller.update(delta, input, cameraForward);

  // Apply to mesh
  mesh.position.addInPlace(positionDelta);

  // Ground check
  const groundY = getGroundHeight(mesh.position.x, mesh.position.z);
  if (mesh.position.y <= groundY) {
    mesh.position.y = groundY;
    controller.onGrounded();
  } else if (mesh.position.y > groundY + 0.1) {
    controller.onAirborne();
  }
}
```

### 5. Tuning Guide

| Feel | Acceleration | Deceleration | Notes |
|------|--------------|--------------|-------|
| Tight/responsive | 40 | 35 | Action games |
| Weighted | 15 | 12 | Exploration |
| Floaty | 8 | 5 | Platformers with momentum |

## Constraints

- This skill handles horizontal movement + jump initiation
- Gravity is simple—for complex physics, use a physics engine
- No collision detection (use babylon-collision)
- No animation triggering (that's a separate system)

## Activation

Activates when: character controller, player movement, WASD movement, velocity-based movement, acceleration, deceleration
