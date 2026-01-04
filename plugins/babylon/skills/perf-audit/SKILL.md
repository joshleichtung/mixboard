---
name: babylon-perf-audit
description: Profile and audit Babylon.js performance—FPS, mesh count, disposal, draw calls
---

# Babylon Performance Audit

## Purpose

Diagnose performance issues in a Babylon.js scene. Use when FPS drops, memory grows, or rendering feels slow.

## Inputs

- Scene reference
- Engine reference

## Outputs

- Performance report with metrics and recommendations
  - Verify: Report prints to console with all metrics
  - Verify: No NaN or undefined values
  - Verify: Disposal warnings appear for undisposed resources

## Assumptions

- Scene is running and rendering
- Can pause for audit if needed
- Access to browser dev tools for deeper analysis

## Procedure

### 1. Quick Metrics Snapshot

```typescript
import { Scene, Engine } from '@babylonjs/core';

interface PerfSnapshot {
  fps: number;
  meshCount: number;
  activeMeshCount: number;
  materialCount: number;
  textureCount: number;
  particleCount: number;
  drawCalls: number;
  triangles: number;
}

function takeSnapshot(scene: Scene, engine: Engine): PerfSnapshot {
  // Instrumentation may be null if not enabled—handle gracefully
  const instrumentation = scene.getInstrumentation?.();

  // Draw calls: -1 means unavailable (instrumentation not enabled or no frames yet)
  let drawCalls = -1;
  if (instrumentation?.drawCallsCounter?.current !== undefined) {
    drawCalls = instrumentation.drawCallsCounter.current;
  }

  // Triangles: estimate from vertex count (3 vertices per triangle)
  let triangles = -1;
  if (instrumentation?.totalVerticesCounter?.current !== undefined) {
    triangles = Math.floor(instrumentation.totalVerticesCounter.current / 3);
  }

  return {
    fps: Math.round(engine.getFps()),
    meshCount: scene.meshes.length,
    activeMeshCount: scene.getActiveMeshes().length,
    materialCount: scene.materials.length,
    textureCount: scene.textures.length,
    particleCount: scene.particleSystems.reduce(
      (sum, ps) => sum + (ps.getActiveCount?.() ?? 0), 0
    ),
    drawCalls,
    triangles,
  };
}
```

### 2. Enable Scene Instrumentation

Required for detailed metrics:

```typescript
function enableInstrumentation(scene: Scene): void {
  scene.performanceMonitor;  // Auto-created on access

  // Enable detailed counters
  const instrumentation = scene.getInstrumentation();
  if (instrumentation) {
    instrumentation.captureDrawCalls = true;
    instrumentation.captureRenderTime = true;
  }
}
```

### 3. Performance Report

```typescript
function generatePerfReport(scene: Scene, engine: Engine): void {
  const snap = takeSnapshot(scene, engine);

  console.group('🔍 Babylon Performance Audit');

  // FPS
  const fpsColor = snap.fps >= 55 ? '🟢' : snap.fps >= 30 ? '🟡' : '🔴';
  console.log(`${fpsColor} FPS: ${snap.fps}`);

  // Mesh counts
  console.log(`📦 Meshes: ${snap.meshCount} total, ${snap.activeMeshCount} active`);
  if (snap.meshCount > 500) {
    console.warn('⚠️ High mesh count. Consider instancing or merging.');
  }

  // Materials
  console.log(`🎨 Materials: ${snap.materialCount}`);
  if (snap.materialCount > 50) {
    console.warn('⚠️ Many materials. Consider material batching.');
  }

  // Textures
  console.log(`🖼️ Textures: ${snap.textureCount}`);

  // Particles
  console.log(`✨ Active particles: ${snap.particleCount}`);
  if (snap.particleCount > 10000) {
    console.warn('⚠️ High particle count. Consider reducing emission.');
  }

  // Draw calls (if instrumentation enabled)
  if (snap.drawCalls > 0) {
    console.log(`🖌️ Draw calls: ${snap.drawCalls}`);
    if (snap.drawCalls > 100) {
      console.warn('⚠️ High draw calls. Use instancing to batch.');
    }
  }

  // Triangles
  if (snap.triangles > 0) {
    console.log(`🔺 Triangles: ${snap.triangles.toLocaleString()}`);
  }

  console.groupEnd();
}
```

### 4. Memory Leak Detection

```typescript
function checkDisposal(scene: Scene): void {
  console.group('🗑️ Disposal Check');

  // Check for meshes without materials
  const orphanMeshes = scene.meshes.filter(m => !m.material && m.isVisible);
  if (orphanMeshes.length > 0) {
    console.warn(`⚠️ ${orphanMeshes.length} visible meshes without materials`);
  }

  // Check total counts over time
  console.log(`Total meshes: ${scene.meshes.length}`);
  console.log(`Total materials: ${scene.materials.length}`);
  console.log(`Total textures: ${scene.textures.length}`);

  console.log('💡 Run this periodically. Growing counts indicate leaks.');

  console.groupEnd();
}
```

### 5. Disposal Helper

```typescript
import { TransformNode, Scene } from '@babylonjs/core';

function disposeEntitySafely(node: TransformNode, scene: Scene): void {
  // Dispose child meshes first
  const meshes = node.getChildMeshes();
  for (const mesh of meshes) {
    // Dispose material only if this is the sole user
    if (mesh.material) {
      const users = scene.meshes.filter(m => m.material === mesh.material);
      if (users.length <= 1) {
        mesh.material.dispose();
      }
    }
    mesh.dispose();
  }

  // Dispose the node itself
  node.dispose();
}

// Usage: disposeEntitySafely(myEntity, scene);
```

### 6. Common Issues Checklist

```typescript
function runAuditChecklist(scene: Scene): void {
  console.group('📋 Common Issues Checklist');

  // 1. Too many meshes
  if (scene.meshes.length > 200) {
    const instanceable = scene.meshes.filter(m =>
      scene.meshes.filter(m2 => m2.name.startsWith(m.name.split('_')[0])).length > 5
    );
    if (instanceable.length > 0) {
      console.warn(`⚠️ Consider instancing: ${instanceable.length} meshes with similar names`);
    }
  }

  // 2. No frustum culling
  const alwaysActive = scene.meshes.filter(m => m.alwaysSelectAsActiveMesh);
  if (alwaysActive.length > 10) {
    console.warn(`⚠️ ${alwaysActive.length} meshes bypass frustum culling`);
  }

  // 3. Large textures
  const largeTextures = scene.textures.filter(t => {
    const size = t.getSize();
    return size.width > 2048 || size.height > 2048;
  });
  if (largeTextures.length > 0) {
    console.warn(`⚠️ ${largeTextures.length} textures > 2048px`);
  }

  // 4. Particle systems
  const heavyParticles = scene.particleSystems.filter(ps => ps.getCapacity() > 5000);
  if (heavyParticles.length > 0) {
    console.warn(`⚠️ ${heavyParticles.length} particle systems with capacity > 5000`);
  }

  console.log('✅ Checklist complete');
  console.groupEnd();
}
```

### 7. Full Audit Function

**Important**: Instrumentation counters need at least one render frame to populate after being enabled.

```typescript
function runFullAudit(scene: Scene, engine: Engine): void {
  // Step 1: Enable instrumentation BEFORE rendering
  enableInstrumentation(scene);

  // Step 2: Wait for at least one render frame to populate counters
  // Using requestAnimationFrame ensures we wait for an actual frame
  requestAnimationFrame(() => {
    requestAnimationFrame(() => {
      // Now counters should be populated
      generatePerfReport(scene, engine);
      checkDisposal(scene);
      runAuditChecklist(scene);
    });
  });
}

// Usage: call once scene is running
// runFullAudit(scene, engine);
```

**Note**: If drawCalls or triangles show -1, instrumentation wasn't enabled in time. Call `enableInstrumentation()` earlier (e.g., right after scene creation).

## Constraints

- Read-only audit—no automatic fixes
- Recommendations only; implementation is manual
- Some metrics require instrumentation to be enabled first
- Not a replacement for browser profiler (use together)

## Activation

Activates when: performance audit, perf check, profile babylon, slow rendering, FPS drop, memory leak, draw calls

---

## Changelog

- **Fix**: `disposeEntitySafely()` now accepts `scene` parameter (was using undefined `scene`)
- **Fix**: `takeSnapshot()` now safely handles missing instrumentation (returns -1 vs crashing)
- **Clarify**: `runFullAudit()` uses `requestAnimationFrame` to ensure counters populate
- **Add**: Note explaining -1 values for drawCalls/triangles when instrumentation isn't ready
