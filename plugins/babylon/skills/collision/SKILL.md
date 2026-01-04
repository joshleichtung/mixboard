---
name: babylon-collision
description: Detect and resolve simple collisions using circle/sphere checks and raycasts
---

# Babylon Collision Detection

## Purpose

Detect overlaps between entities and resolve them with pushout vectors. Use for character-obstacle collision, trigger zones, and ground detection.

## Inputs

- Entity position and radius
- List of obstacles (position + radius)
- For raycasts: origin, direction, max distance

## Outputs

- Collision result with pushout vector
  - Verify: No entity-obstacle overlap after applying pushout
  - Verify: No jitter on sustained contact (stable resolution)
  - Verify: Raycast returns correct hit point on terrain

## Assumptions

- Entities approximated as spheres/circles
- Obstacles are static (or updated each frame)
- Positions are world-space

## Procedure

### 1. Circle Collision (2D, XZ Plane)

Fast check ignoring Y—good for top-down or when heights are similar:

```typescript
import { Vector3 } from '@babylonjs/core';

interface CircleCollider {
  position: Vector3;
  radius: number;
}

interface CollisionResult {
  collided: boolean;
  pushout: Vector3;
}

function checkCircleCollision(
  entity: CircleCollider,
  obstacles: CircleCollider[]
): CollisionResult {
  const pushout = Vector3.Zero();

  for (const obstacle of obstacles) {
    const dx = entity.position.x - obstacle.position.x;
    const dz = entity.position.z - obstacle.position.z;
    const dist = Math.sqrt(dx * dx + dz * dz);
    const minDist = entity.radius + obstacle.radius;

    if (dist < minDist && dist > 0.001) {
      const overlap = minDist - dist;
      pushout.x += (dx / dist) * overlap;
      pushout.z += (dz / dist) * overlap;
    }
  }

  return {
    collided: pushout.length() > 0.001,
    pushout,
  };
}
```

### 2. Sphere Collision (Full 3D)

When Y matters:

```typescript
function checkSphereCollision(
  entity: CircleCollider,
  obstacles: CircleCollider[]
): CollisionResult {
  const pushout = Vector3.Zero();

  for (const obstacle of obstacles) {
    const diff = entity.position.subtract(obstacle.position);
    const dist = diff.length();
    const minDist = entity.radius + obstacle.radius;

    if (dist < minDist && dist > 0.001) {
      const overlap = minDist - dist;
      pushout.addInPlace(diff.normalize().scale(overlap));
    }
  }

  return {
    collided: pushout.length() > 0.001,
    pushout,
  };
}
```

### 3. Apply Collision Resolution

```typescript
function resolveCollision(
  entityPosition: Vector3,
  obstacles: CircleCollider[],
  entityRadius: number
): Vector3 {
  const result = checkCircleCollision(
    { position: entityPosition, radius: entityRadius },
    obstacles
  );

  if (result.collided) {
    return entityPosition.add(result.pushout);
  }

  return entityPosition;
}

// Usage in update loop
newPosition = resolveCollision(newPosition, obstacles, characterRadius);
mesh.position.copyFrom(newPosition);
```

### 4. Raycast Ground Detection

```typescript
import { Ray, Scene, Mesh } from '@babylonjs/core';

function getGroundHeight(
  scene: Scene,
  x: number,
  z: number,
  terrainMesh: Mesh
): number {
  const ray = new Ray(
    new Vector3(x, 1000, z),  // Start high above
    Vector3.Down(),
    2000  // Max distance
  );

  const hit = scene.pickWithRay(ray, (mesh) => mesh === terrainMesh);

  if (hit?.hit && hit.pickedPoint) {
    return hit.pickedPoint.y;
  }

  return 0;  // Default ground level
}
```

### 5. Raycast with Multiple Meshes

```typescript
function raycastTerrain(
  scene: Scene,
  origin: Vector3,
  direction: Vector3,
  maxDistance: number,
  includeMeshes: Mesh[]
): { hit: boolean; point: Vector3 | null; mesh: Mesh | null } {
  const ray = new Ray(origin, direction.normalize(), maxDistance);

  const hit = scene.pickWithRay(ray, (mesh) =>
    includeMeshes.includes(mesh as Mesh)
  );

  if (hit?.hit && hit.pickedPoint) {
    return {
      hit: true,
      point: hit.pickedPoint,
      mesh: hit.pickedMesh as Mesh,
    };
  }

  return { hit: false, point: null, mesh: null };
}
```

### 6. Trigger Zone Detection

Non-resolving collision for triggers:

```typescript
function checkTriggerZone(
  entityPosition: Vector3,
  triggerCenter: Vector3,
  triggerRadius: number
): boolean {
  const dist = Vector3.Distance(entityPosition, triggerCenter);
  return dist < triggerRadius;
}

// Usage
if (checkTriggerZone(player.position, checkpointPosition, 2)) {
  onCheckpointReached();
}
```

### 7. Obstacle Management

```typescript
class CollisionManager {
  private obstacles: CircleCollider[] = [];

  addObstacle(position: Vector3, radius: number): void {
    this.obstacles.push({ position, radius });
  }

  removeObstaclesInRadius(center: Vector3, radius: number): void {
    this.obstacles = this.obstacles.filter(
      obs => Vector3.Distance(obs.position, center) > radius
    );
  }

  checkCollision(entity: CircleCollider): CollisionResult {
    return checkCircleCollision(entity, this.obstacles);
  }

  clear(): void {
    this.obstacles = [];
  }
}
```

## Constraints

- O(n) complexity—use spatial partitioning for >100 obstacles
- No continuous collision detection (tunneling possible at high speeds)
- Pushout may cause secondary collisions—iterate if needed
- No physics engine integration (this is simple custom collision)

## Activation

Activates when: collision detection, obstacle avoidance, pushout, overlap, raycast, ground detection, trigger zone
