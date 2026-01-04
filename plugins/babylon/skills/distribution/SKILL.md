---
name: babylon-distribution
description: Place objects across terrain using distribution algorithms (Poisson, grid jitter, polar, clustering)
---

# Babylon Object Distribution

## Purpose

Generate positions for scattering objects across terrain. Use when placing trees, rocks, grass clumps, or any repeated elements that need natural-looking distribution.

## Inputs

- Terrain bounds (size or min/max coordinates)
- Count or density target
- Spacing constraints (minimum distance, exclusion zones)
- Distribution algorithm choice

## Outputs

- Array of Vector3 positions (Y = 0, to be adjusted by terrain height)
  - Verify: `positions.length` matches expected count (±10% for Poisson)
  - Verify: All positions within bounds: `Math.abs(pos.x) <= halfSize`
  - Verify: Minimum spacing respected (sample 10 pairs, check distance)

## Assumptions

- Terrain bounds known
- Positions are XZ plane; Y will be set by terrain height query
- No collision with existing objects (handled separately)

## Procedure

### 1. Grid Jitter (Even Coverage)

Best for: Even spread with slight randomness, predictable count.

```typescript
import { Vector3 } from '@babylonjs/core';

function gridJitter(
  terrainSize: number,
  cellSize: number,
  jitter = 0.4  // 0-0.5, higher = more random
): Vector3[] {
  const positions: Vector3[] = [];
  const halfSize = terrainSize / 2;

  for (let x = -halfSize; x < halfSize; x += cellSize) {
    for (let z = -halfSize; z < halfSize; z += cellSize) {
      const jitterX = (Math.random() - 0.5) * cellSize * jitter * 2;
      const jitterZ = (Math.random() - 0.5) * cellSize * jitter * 2;

      positions.push(new Vector3(
        x + cellSize / 2 + jitterX,
        0,
        z + cellSize / 2 + jitterZ
      ));
    }
  }

  return positions;
}
```

### 2. Poisson Disc Sampling (Minimum Spacing)

Best for: Natural look with guaranteed minimum distance between objects.

```typescript
function poissonDisc(
  terrainSize: number,
  minDistance: number,
  maxAttempts = 30
): Vector3[] {
  const cellSize = minDistance / Math.sqrt(2);
  const gridWidth = Math.ceil(terrainSize / cellSize);
  const grid: (Vector3 | null)[][] = Array(gridWidth)
    .fill(null).map(() => Array(gridWidth).fill(null));

  const positions: Vector3[] = [];
  const active: Vector3[] = [];
  const halfSize = terrainSize / 2;

  const toGrid = (p: Vector3) => ({
    x: Math.floor((p.x + halfSize) / cellSize),
    z: Math.floor((p.z + halfSize) / cellSize)
  });

  // Seed with first point
  const first = new Vector3(
    (Math.random() - 0.5) * terrainSize,
    0,
    (Math.random() - 0.5) * terrainSize
  );
  positions.push(first);
  active.push(first);
  const fc = toGrid(first);
  grid[fc.x][fc.z] = first;

  while (active.length > 0) {
    const idx = Math.floor(Math.random() * active.length);
    const point = active[idx];
    let found = false;

    for (let attempt = 0; attempt < maxAttempts; attempt++) {
      const angle = Math.random() * Math.PI * 2;
      const dist = minDistance + Math.random() * minDistance;
      const candidate = new Vector3(
        point.x + Math.cos(angle) * dist,
        0,
        point.z + Math.sin(angle) * dist
      );

      if (Math.abs(candidate.x) > halfSize || Math.abs(candidate.z) > halfSize)
        continue;

      const cell = toGrid(candidate);
      let valid = true;

      for (let dx = -2; dx <= 2 && valid; dx++) {
        for (let dz = -2; dz <= 2 && valid; dz++) {
          const nx = cell.x + dx, nz = cell.z + dz;
          if (nx >= 0 && nx < gridWidth && nz >= 0 && nz < gridWidth) {
            const neighbor = grid[nx][nz];
            if (neighbor && Vector3.Distance(candidate, neighbor) < minDistance)
              valid = false;
          }
        }
      }

      if (valid) {
        positions.push(candidate);
        active.push(candidate);
        grid[cell.x][cell.z] = candidate;
        found = true;
        break;
      }
    }

    if (!found) active.splice(idx, 1);
  }

  return positions;
}
```

### 3. Polar Distribution (Avoid Center)

Best for: Clearing around player spawn, radial patterns.

```typescript
function polarDistribution(
  count: number,
  terrainSize: number,
  minRadius: number  // Keep this distance from center
): Vector3[] {
  const positions: Vector3[] = [];
  const maxRadius = terrainSize / 2;

  for (let i = 0; i < count; i++) {
    const angle = Math.random() * Math.PI * 2;
    const radius = minRadius + Math.random() * (maxRadius - minRadius);

    positions.push(new Vector3(
      Math.cos(angle) * radius,
      0,
      Math.sin(angle) * radius
    ));
  }

  return positions;
}
```

### 4. Clustering (Groups)

Best for: Forest groves, rock formations, grouped objects.

```typescript
function clusterDistribution(
  clusterCenters: Vector3[],
  itemsPerCluster: [number, number],  // [min, max]
  clusterRadius: number
): Vector3[] {
  const positions: Vector3[] = [];

  for (const center of clusterCenters) {
    const count = itemsPerCluster[0] +
      Math.floor(Math.random() * (itemsPerCluster[1] - itemsPerCluster[0] + 1));

    for (let i = 0; i < count; i++) {
      const angle = Math.random() * Math.PI * 2;
      const dist = Math.random() * clusterRadius;

      positions.push(new Vector3(
        center.x + Math.cos(angle) * dist,
        0,
        center.z + Math.sin(angle) * dist
      ));
    }
  }

  return positions;
}
```

### 5. Apply Terrain Height

After distribution, set Y from terrain:

```typescript
function applyTerrainHeight(
  positions: Vector3[],
  getHeight: (x: number, z: number) => number
): void {
  for (const pos of positions) {
    pos.y = getHeight(pos.x, pos.z);
  }
}
```

## Constraints

- Returns positions only—mesh creation is babylon-instancing
- Max 10k positions without spatial indexing
- No collision avoidance with existing objects (use babylon-collision for runtime)
- No slope filtering (add as post-filter if needed)

## Activation

Activates when: distribute objects, scatter, placement algorithm, Poisson disc, object positions, tree placement, rock distribution
