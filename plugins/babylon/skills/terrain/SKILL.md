---
name: babylon-terrain
description: Generate heightmap-based terrain with noise functions and vertex displacement
---

# Babylon Terrain Generation

## Purpose

Generate a ground mesh with height variation using noise functions. Use when you need rolling hills, dunes, or any terrain with smooth elevation changes.

## Inputs

- Scene reference (initialized)
- Terrain size (width/depth in world units)
- Subdivision count (vertices per axis)
- Height function or noise configuration

## Outputs

- Ground mesh with displaced vertices
  - Verify: Mesh appears in scene inspector with name "terrain"
  - Verify: Vertex count = (subdivisions + 1)²
  - Verify: Normals computed (mesh not flat-shaded unintentionally)
  - Verify: `mesh.getBoundingInfo()` shows expected Y range

## Assumptions

- Babylon.js 6.x or higher
- Scene already initialized
- Right-handed coordinate system (Babylon default)
- Y is up

## Procedure

### 1. Create Base Ground

```typescript
import { MeshBuilder, VertexBuffer, Scene, Mesh } from '@babylonjs/core';

function createTerrain(
  scene: Scene,
  size: number,
  subdivisions: number,
  heightFn: (x: number, z: number) => number
): Mesh {
  const terrain = MeshBuilder.CreateGround('terrain', {
    width: size,
    height: size,  // "height" is depth in CreateGround
    subdivisions,
  }, scene);

  const positions = terrain.getVerticesData(VertexBuffer.PositionKind);
  if (!positions) return terrain;

  // Displace Y based on height function
  for (let i = 0; i < positions.length; i += 3) {
    const x = positions[i];
    const z = positions[i + 2];
    positions[i + 1] = heightFn(x, z);
  }

  terrain.setVerticesData(VertexBuffer.PositionKind, positions);
  terrain.createNormals(true);  // Recompute normals
  terrain.refreshBoundingInfo();

  return terrain;
}
```

### 2. Noise Functions

**Multi-octave sine (fast, deterministic):**

```typescript
function sineNoise(x: number, z: number): number {
  const detail = Math.sin(x * 0.3) * Math.cos(z * 0.3) * 0.3;
  const medium = Math.sin(x * 0.08) * Math.cos(z * 0.1) * 1.2;
  const large = Math.sin(x * 0.02) * Math.cos(z * 0.025) * 2.5;
  return detail + medium + large;
}
```

**Hash-based smooth noise (better randomness):**

```typescript
function hash(x: number, y: number): number {
  const n = Math.sin(x * 12.9898 + y * 78.233) * 43758.5453;
  return n - Math.floor(n);
}

function smoothNoise(x: number, z: number): number {
  const ix = Math.floor(x), iz = Math.floor(z);
  const fx = x - ix, fz = z - iz;
  const ux = fx * fx * (3 - 2 * fx);
  const uz = fz * fz * (3 - 2 * fz);

  const a = hash(ix, iz), b = hash(ix + 1, iz);
  const c = hash(ix, iz + 1), d = hash(ix + 1, iz + 1);

  return a + (b - a) * ux + (c - a) * uz + (a - b - c + d) * ux * uz;
}
```

**Fractal Brownian Motion (layered detail):**

```typescript
function fbm(x: number, z: number, octaves = 4): number {
  let value = 0, amplitude = 1, frequency = 1, maxValue = 0;

  for (let i = 0; i < octaves; i++) {
    value += smoothNoise(x * frequency, z * frequency) * amplitude;
    maxValue += amplitude;
    amplitude *= 0.5;
    frequency *= 2;
  }

  return value / maxValue;  // Normalized 0-1
}
```

### 3. Height Query Function

Expose a function for other systems to query terrain height:

```typescript
function createHeightQuery(
  heightFn: (x: number, z: number) => number
): (x: number, z: number) => number {
  return (x, z) => heightFn(x, z);
}

// Usage: character Y = getTerrainHeight(character.x, character.z)
```

## Constraints

- Max 100×100 subdivisions without LOD (10k vertices)
- For larger terrains, use chunking or LOD (separate concern)
- No biome blending in this skill—use babylon-distribution for varied placement
- No material assignment—that's babylon-cel-shader or babylon-materials

## Activation

Activates when: terrain generation, heightmap, ground mesh, noise terrain, rolling hills, procedural ground
