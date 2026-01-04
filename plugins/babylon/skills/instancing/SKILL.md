---
name: babylon-instancing
description: Optimize repeated meshes via thin instances or mesh merging
---

# Babylon Mesh Instancing

## Purpose

Render many copies of a mesh efficiently using thin instances (GPU instancing) or mesh merging. Use when you have >10 identical objects to reduce draw calls.

## Inputs

- Source mesh (the template)
- Position array (from babylon-distribution)
- Transform options (scale range, rotation, tilt)

## Outputs

- Single mesh with instances, or merged mesh
  - Verify: Inspector shows 1 mesh with N instances (not N separate meshes)
  - Verify: Draw calls reduced (check `scene.getActiveMeshes().length`)
  - Verify: Source mesh hidden (`isVisible = false`)

## Assumptions

- Positions already computed (use babylon-distribution)
- Source mesh has material assigned
- All instances share the same material

## Procedure

### 1. Thin Instances (Recommended for >50 copies)

GPU-instanced rendering—one draw call for all copies:

```typescript
import { Mesh, Matrix, Vector3 } from '@babylonjs/core';

function createThinInstances(
  source: Mesh,
  positions: Vector3[],
  options: {
    scaleRange?: [number, number];
    randomRotationY?: boolean;
    tilt?: number;
  } = {}
): Mesh {
  const { scaleRange, randomRotationY = true, tilt = 0 } = options;

  // Hide source mesh
  source.isVisible = false;

  // Build transformation matrices
  const matrices: number[] = [];

  for (const pos of positions) {
    // Scale
    const scale = scaleRange
      ? scaleRange[0] + Math.random() * (scaleRange[1] - scaleRange[0])
      : 1;

    // Rotation
    const rotY = randomRotationY ? Math.random() * Math.PI * 2 : 0;
    const rotX = tilt ? (Math.random() - 0.5) * tilt : 0;
    const rotZ = tilt ? (Math.random() - 0.5) * tilt : 0;

    // Build matrix: scale → rotate → translate
    const matrix = Matrix.Compose(
      new Vector3(scale, scale, scale),
      Quaternion.FromEulerAngles(rotX, rotY, rotZ),
      pos
    );

    matrices.push(...matrix.toArray());
  }

  // Apply to mesh
  source.thinInstanceSetBuffer('matrix', new Float32Array(matrices), 16);

  return source;
}
```

### 2. Thin Instances with Per-Instance Color

For varied colors without separate materials:

```typescript
import { Color4 } from '@babylonjs/core';

function createColoredInstances(
  source: Mesh,
  positions: Vector3[],
  colors: Color4[]  // One per position
): Mesh {
  source.isVisible = false;

  const matrices: number[] = [];
  const colorData: number[] = [];

  for (let i = 0; i < positions.length; i++) {
    const matrix = Matrix.Translation(
      positions[i].x,
      positions[i].y,
      positions[i].z
    );
    matrices.push(...matrix.toArray());
    colorData.push(colors[i].r, colors[i].g, colors[i].b, colors[i].a);
  }

  source.thinInstanceSetBuffer('matrix', new Float32Array(matrices), 16);
  source.thinInstanceSetBuffer('color', new Float32Array(colorData), 4);

  // Material must use instance color:
  // material.useInstancing = true; (for StandardMaterial)

  return source;
}
```

### 3. Mesh Merging (For <50 Static Objects)

Combines geometry into single mesh—good for static scenery:

```typescript
function mergeStaticMeshes(meshes: Mesh[]): Mesh | null {
  if (meshes.length === 0) return null;

  const merged = Mesh.MergeMeshes(
    meshes,
    true,   // disposeSource
    true,   // allow32BitsIndices
    undefined,
    false,  // subdivideWithSubMeshes
    true    // multiMultiMaterials
  );

  return merged;
}
```

### 4. LOD with Instances

Reduce detail at distance:

```typescript
function setupInstanceLOD(
  source: Mesh,
  lowPolySource: Mesh,
  positions: Vector3[],
  lodDistance: number
): void {
  // High detail instances
  createThinInstances(source, positions);

  // Low detail instances (same positions)
  createThinInstances(lowPolySource, positions);
  lowPolySource.isVisible = false;

  // LOD switching (simplified—real LOD needs more setup)
  source.addLODLevel(lodDistance, lowPolySource);
  source.addLODLevel(lodDistance * 2, null);  // Cull beyond
}
```

### 5. Update Instance Transforms

For moving instances:

```typescript
function updateInstancePosition(
  mesh: Mesh,
  index: number,
  newPosition: Vector3
): void {
  const matrix = Matrix.Translation(newPosition.x, newPosition.y, newPosition.z);
  mesh.thinInstanceSetMatrixAt(index, matrix, true);  // true = mark dirty
}
```

## Constraints

- Thin instances require same material for all copies
- No per-instance physics without custom handling
- Merged meshes are static—can't move individual parts
- Max ~100k instances before GPU limits

## Activation

Activates when: instancing, thin instances, merge meshes, optimize draw calls, many objects, batch rendering, GPU instancing
