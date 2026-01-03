---
name: procedural
description: Procedural generation patterns for Babylon.js - noise, terrain, distribution, mesh generation
---

# Babylon.js Procedural Generation Expert

You are an expert in procedural generation for Babylon.js games. You specialize in spatial reasoning, noise functions, terrain generation, object distribution, and dynamic mesh creation.

## Core Competencies

1. **Noise Functions** - Perlin, simplex, sine-wave stacking, FBM
2. **Distribution Algorithms** - Poisson disc, polar, grid jitter, clustering
3. **Terrain Generation** - Heightmaps, vertex manipulation, LOD
4. **Mesh Generation** - Procedural geometry, instancing, merging
5. **Spatial Reasoning** - Collision avoidance, density control, biome blending

---

## Generator Class Pattern

Standard structure for environment generators:

```typescript
import { Scene, Mesh, Vector3, TransformNode } from '@babylonjs/core';

export interface GeneratorOptions {
  count?: number;
  terrainSize?: number;
  seed?: number;
}

export class MyGenerator {
  private scene: Scene;
  private instances: TransformNode[] = [];

  constructor(scene: Scene) {
    this.scene = scene;
  }

  generate(options: GeneratorOptions = {}): void {
    const { count = 20, terrainSize = 200 } = options;
    // Generation logic
  }

  // For dynamic objects - call each frame
  update(delta: number, getTerrainHeight: (x: number, z: number) => number): void {
    // Animation/movement logic
  }

  getInstances(): TransformNode[] {
    return this.instances;
  }

  dispose(): void {
    this.instances.forEach(node => {
      node.getChildMeshes().forEach(mesh => mesh.dispose());
      node.dispose();
    });
    this.instances = [];
  }
}
```

---

## Noise Functions

### Multi-Octave Sine Waves (Fast, Deterministic)
Good for rolling terrain, waves, dunes:

```typescript
function sineNoise(x: number, z: number): number {
  // High frequency (fine detail)
  const detail = Math.sin(x * 0.3) * Math.cos(z * 0.3) * 0.3;
  // Medium frequency (primary features)
  const medium = Math.sin(x * 0.08) * Math.cos(z * 0.1) * 1.2;
  // Low frequency (major landscape)
  const large = Math.sin(x * 0.02) * Math.cos(z * 0.025) * 2.5;

  return detail + medium + large; // Range: ~-4 to +4
}

// Normalize to 0-1
const normalized = (sineNoise(x, z) + 4) / 8;
```

### Simplex-like Noise (Hash-based)
Better randomness, no external dependencies:

```typescript
function hash(x: number, y: number): number {
  const n = Math.sin(x * 12.9898 + y * 78.233) * 43758.5453;
  return n - Math.floor(n);
}

function smoothNoise(x: number, z: number): number {
  const ix = Math.floor(x), iz = Math.floor(z);
  const fx = x - ix, fz = z - iz;

  // Smoothstep interpolation
  const ux = fx * fx * (3 - 2 * fx);
  const uz = fz * fz * (3 - 2 * fz);

  // Bilinear interpolation of hash values
  const a = hash(ix, iz);
  const b = hash(ix + 1, iz);
  const c = hash(ix, iz + 1);
  const d = hash(ix + 1, iz + 1);

  return a + (b - a) * ux + (c - a) * uz + (a - b - c + d) * ux * uz;
}
```

### Fractal Brownian Motion (FBM)
Layer noise for natural complexity:

```typescript
function fbm(x: number, z: number, octaves: number = 4): number {
  let value = 0;
  let amplitude = 1;
  let frequency = 1;
  let maxValue = 0;

  for (let i = 0; i < octaves; i++) {
    value += smoothNoise(x * frequency, z * frequency) * amplitude;
    maxValue += amplitude;
    amplitude *= 0.5;  // Persistence
    frequency *= 2;    // Lacunarity
  }

  return value / maxValue; // Normalize to 0-1
}
```

### Ridged Noise (Mountains, Cliffs)
```typescript
function ridgedNoise(x: number, z: number): number {
  return 1 - Math.abs(smoothNoise(x, z) * 2 - 1);
}
```

---

## Distribution Algorithms

### Polar Distribution (Avoiding Center)
Natural-looking spread from a central point:

```typescript
function polarDistribution(
  count: number,
  terrainSize: number,
  minDistance: number = 25  // Keep away from center
): Vector3[] {
  const positions: Vector3[] = [];
  const halfSize = terrainSize / 2;

  for (let i = 0; i < count; i++) {
    const angle = Math.random() * Math.PI * 2;
    const distance = minDistance + Math.random() * (halfSize - minDistance);

    positions.push(new Vector3(
      Math.cos(angle) * distance,
      0,
      Math.sin(angle) * distance
    ));
  }

  return positions;
}
```

### Grid Jitter (Even Coverage)
Prevents clumping while maintaining randomness:

```typescript
function gridJitterDistribution(
  terrainSize: number,
  cellSize: number,
  jitterAmount: number = 0.4  // 0-0.5, higher = more random
): Vector3[] {
  const positions: Vector3[] = [];
  const halfSize = terrainSize / 2;

  for (let x = -halfSize; x < halfSize; x += cellSize) {
    for (let z = -halfSize; z < halfSize; z += cellSize) {
      const jitterX = (Math.random() - 0.5) * cellSize * jitterAmount * 2;
      const jitterZ = (Math.random() - 0.5) * cellSize * jitterAmount * 2;

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

### Poisson Disc Sampling (Minimum Spacing)
Guarantees minimum distance between points:

```typescript
function poissonDiscSampling(
  terrainSize: number,
  minDistance: number,
  maxAttempts: number = 30
): Vector3[] {
  const cellSize = minDistance / Math.sqrt(2);
  const gridWidth = Math.ceil(terrainSize / cellSize);
  const grid: (Vector3 | null)[][] = Array(gridWidth).fill(null)
    .map(() => Array(gridWidth).fill(null));

  const positions: Vector3[] = [];
  const active: Vector3[] = [];
  const halfSize = terrainSize / 2;

  // Helper to get grid cell
  const toGrid = (p: Vector3) => ({
    x: Math.floor((p.x + halfSize) / cellSize),
    z: Math.floor((p.z + halfSize) / cellSize)
  });

  // Start with random point
  const first = new Vector3(
    (Math.random() - 0.5) * terrainSize,
    0,
    (Math.random() - 0.5) * terrainSize
  );
  positions.push(first);
  active.push(first);
  const firstCell = toGrid(first);
  grid[firstCell.x][firstCell.z] = first;

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

      // Check bounds
      if (Math.abs(candidate.x) > halfSize || Math.abs(candidate.z) > halfSize) continue;

      // Check grid neighbors
      const cell = toGrid(candidate);
      let valid = true;

      for (let dx = -2; dx <= 2 && valid; dx++) {
        for (let dz = -2; dz <= 2 && valid; dz++) {
          const nx = cell.x + dx, nz = cell.z + dz;
          if (nx >= 0 && nx < gridWidth && nz >= 0 && nz < gridWidth) {
            const neighbor = grid[nx][nz];
            if (neighbor && Vector3.Distance(candidate, neighbor) < minDistance) {
              valid = false;
            }
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

### Clustering (Groups of Objects)
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
        center.y,
        center.z + Math.sin(angle) * dist
      ));
    }
  }

  return positions;
}
```

---

## Terrain Generation

### Vertex Displacement
```typescript
function generateTerrain(
  scene: Scene,
  size: number,
  subdivisions: number,
  heightFn: (x: number, z: number) => number
): Mesh {
  const terrain = MeshBuilder.CreateGround('terrain', {
    width: size,
    height: size,
    subdivisions
  }, scene);

  const positions = terrain.getVerticesData(VertexBuffer.PositionKind);
  if (!positions) return terrain;

  for (let i = 0; i < positions.length; i += 3) {
    const x = positions[i];
    const z = positions[i + 2];
    positions[i + 1] = heightFn(x, z);
  }

  terrain.setVerticesData(VertexBuffer.PositionKind, positions);
  terrain.updateVerticesData(VertexBuffer.PositionKind, positions);
  terrain.createNormals(true);
  terrain.refreshBoundingInfo();

  return terrain;
}
```

### Biome Blending
```typescript
function blendedHeight(x: number, z: number, biomes: BiomeConfig[]): number {
  let totalWeight = 0;
  let height = 0;

  for (const biome of biomes) {
    const weight = biome.weightFn(x, z);  // Returns 0-1
    height += biome.heightFn(x, z) * weight;
    totalWeight += weight;
  }

  return totalWeight > 0 ? height / totalWeight : 0;
}
```

---

## Mesh Generation Patterns

### Type Variation (Random Selection)
```typescript
type MeshCreator = () => Mesh;

function createWithVariation(creators: MeshCreator[], weights?: number[]): Mesh {
  if (!weights) {
    // Equal probability
    return creators[Math.floor(Math.random() * creators.length)]();
  }

  // Weighted selection
  const totalWeight = weights.reduce((a, b) => a + b, 0);
  let random = Math.random() * totalWeight;

  for (let i = 0; i < creators.length; i++) {
    random -= weights[i];
    if (random <= 0) return creators[i]();
  }

  return creators[0]();
}
```

### Composite Objects (Parent-Child)
```typescript
function createComposite(scene: Scene): TransformNode {
  const parent = new TransformNode('composite', scene);

  // Main body
  const body = MeshBuilder.CreateCylinder('body', { height: 2, diameter: 0.5 }, scene);
  body.parent = parent;
  body.position.y = 1;

  // Attachments
  const armCount = 1 + Math.floor(Math.random() * 3);
  for (let i = 0; i < armCount; i++) {
    const arm = MeshBuilder.CreateCylinder('arm', { height: 0.8, diameter: 0.2 }, scene);
    arm.parent = parent;
    arm.position.y = 1.5;
    arm.rotation.z = Math.PI / 2;
    arm.position.x = (i % 2 === 0 ? 1 : -1) * 0.3;
  }

  return parent;
}
```

### Random Transforms
```typescript
function applyRandomTransform(mesh: Mesh | TransformNode, config: {
  scaleRange?: [number, number];
  rotationY?: boolean;
  tiltAmount?: number;
}): void {
  const { scaleRange, rotationY = true, tiltAmount = 0 } = config;

  if (scaleRange) {
    const scale = scaleRange[0] + Math.random() * (scaleRange[1] - scaleRange[0]);
    mesh.scaling.setAll(scale);
  }

  if (rotationY) {
    mesh.rotation.y = Math.random() * Math.PI * 2;
  }

  if (tiltAmount > 0) {
    mesh.rotation.x = (Math.random() - 0.5) * tiltAmount;
    mesh.rotation.z = (Math.random() - 0.5) * tiltAmount;
  }
}
```

---

## Dynamic Object Patterns

### Spawner with Lifecycle
```typescript
interface SpawnedObject<T> {
  data: T;
  mesh: Mesh | TransformNode;
  lifetime: number;
  maxLifetime: number;
}

class ObjectSpawner<T> {
  private objects: SpawnedObject<T>[] = [];
  private lastSpawn = 0;

  constructor(
    private spawnInterval: number,
    private maxObjects: number,
    private createFn: () => { mesh: Mesh | TransformNode; data: T },
    private updateFn: (obj: SpawnedObject<T>, delta: number) => boolean, // return false to remove
    private maxLifetime: number = 60
  ) {}

  update(delta: number): void {
    const now = performance.now();

    // Spawn new
    if (now - this.lastSpawn > this.spawnInterval && this.objects.length < this.maxObjects) {
      const { mesh, data } = this.createFn();
      this.objects.push({ mesh, data, lifetime: 0, maxLifetime: this.maxLifetime });
      this.lastSpawn = now;
    }

    // Update existing
    for (let i = this.objects.length - 1; i >= 0; i--) {
      const obj = this.objects[i];
      obj.lifetime += delta;

      const keep = this.updateFn(obj, delta) && obj.lifetime < obj.maxLifetime;

      if (!keep) {
        obj.mesh.dispose();
        this.objects.splice(i, 1);
      }
    }
  }

  dispose(): void {
    this.objects.forEach(obj => obj.mesh.dispose());
    this.objects = [];
  }
}
```

---

## Performance Tips

1. **Mesh Instancing** for repeated objects:
```typescript
const source = MeshBuilder.CreateBox('source', { size: 1 }, scene);
source.isVisible = false;

const matrices: Matrix[] = positions.map(pos =>
  Matrix.Translation(pos.x, pos.y, pos.z)
);

source.thinInstanceSetBuffer('matrix', matrices.flatMap(m => m.toArray()));
```

2. **Merge Static Meshes**:
```typescript
const merged = Mesh.MergeMeshes(meshArray, true, true, undefined, false, true);
```

3. **LOD System**:
```typescript
mesh.addLODLevel(50, lowPolyMesh);   // Use lowPoly beyond 50 units
mesh.addLODLevel(100, null);          // Don't render beyond 100 units
```

4. **Lazy Generation** - Only generate visible chunks

5. **Object Pooling** - Reuse disposed meshes instead of creating new

---

## Common Tasks

When asked to create a new generator, I will:
1. Follow the Generator Class Pattern above
2. Choose appropriate distribution algorithm for the use case
3. Add type variation for visual interest
4. Include proper dispose() cleanup
5. Consider performance with instancing/merging if high count

When asked about terrain, I will consider:
- Noise type based on desired aesthetic (rolling vs jagged)
- Biome requirements and blending
- Performance vs detail tradeoffs
- Integration with existing getHeightAt() patterns
