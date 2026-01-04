---
name: babylon-sky-gradient
description: Create gradient skybox with horizon control
---

# Babylon Gradient Sky

## Purpose

Create a gradient skybox that blends between top and bottom colors. Use for stylized backgrounds that complement cel-shaded aesthetics.

## Inputs

- Top color (hex string)
- Bottom color (hex string)
- Horizon sharpness (0.5-2.0, higher = sharper transition)

## Outputs

- Skybox mesh with gradient material
  - Verify: Sky renders behind all scene objects
  - Verify: Gradient visible from any camera angle
  - Verify: No seams or artifacts at sphere poles

## Assumptions

- Scene initialized
- Camera positioned
- No existing skybox (or will replace)

## Procedure

### 1. Gradient Sky Shader

```typescript
import { ShaderMaterial, MeshBuilder, Scene, Vector3, Color3 } from '@babylonjs/core';

const SKY_VERTEX = `
  precision highp float;
  attribute vec3 position;
  uniform mat4 worldViewProjection;
  varying vec3 vPosition;

  void main() {
    gl_Position = worldViewProjection * vec4(position, 1.0);
    vPosition = position;
  }
`;

const SKY_FRAGMENT = `
  precision highp float;
  uniform vec3 topColor;
  uniform vec3 bottomColor;
  uniform float horizonSharpness;
  varying vec3 vPosition;

  void main() {
    // Normalize Y to 0-1 range
    float t = (normalize(vPosition).y + 1.0) * 0.5;

    // Apply sharpness for horizon control
    t = smoothstep(0.0, horizonSharpness, t);

    vec3 color = mix(bottomColor, topColor, t);
    gl_FragColor = vec4(color, 1.0);
  }
`;

function createGradientSky(
  scene: Scene,
  topColor: string,
  bottomColor: string,
  horizonSharpness = 1.0
): { mesh: Mesh; material: ShaderMaterial } {
  // Create large sphere
  const sky = MeshBuilder.CreateSphere('sky', {
    diameter: 1000,
    segments: 32,
  }, scene);

  // Create material
  const material = new ShaderMaterial('skyMaterial', scene, {
    vertexSource: SKY_VERTEX,
    fragmentSource: SKY_FRAGMENT,
  }, {
    attributes: ['position'],
    uniforms: ['worldViewProjection', 'topColor', 'bottomColor', 'horizonSharpness'],
  });

  const top = Color3.FromHexString(topColor);
  const bottom = Color3.FromHexString(bottomColor);

  material.setVector3('topColor', new Vector3(top.r, top.g, top.b));
  material.setVector3('bottomColor', new Vector3(bottom.r, bottom.g, bottom.b));
  material.setFloat('horizonSharpness', horizonSharpness);

  // Render inside of sphere
  material.backFaceCulling = false;

  sky.material = material;

  // Render first (behind everything)
  sky.renderingGroupId = 0;

  return { mesh: sky, material };
}
```

### 2. Sky Color Presets

```typescript
const SKY_PRESETS = {
  daylight: {
    top: '#87CEEB',     // Sky blue
    bottom: '#E0F0FF',  // Light horizon
    sharpness: 1.0,
  },
  sunset: {
    top: '#1E3A5F',     // Deep blue
    bottom: '#FF7F50',  // Coral
    sharpness: 0.6,
  },
  stylized: {
    top: '#B8D4E8',     // Soft blue
    bottom: '#F5E6D3',  // Warm cream
    sharpness: 1.2,
  },
  night: {
    top: '#0D1B2A',     // Dark blue
    bottom: '#1B263B',  // Slightly lighter
    sharpness: 1.5,
  },
} as const;

// Usage
const { mesh, material } = createGradientSky(
  scene,
  SKY_PRESETS.stylized.top,
  SKY_PRESETS.stylized.bottom,
  SKY_PRESETS.stylized.sharpness
);
```

### 3. Update Sky Colors (Runtime)

```typescript
function updateSkyColors(
  material: ShaderMaterial,
  topColor: string,
  bottomColor: string
): void {
  const top = Color3.FromHexString(topColor);
  const bottom = Color3.FromHexString(bottomColor);

  material.setVector3('topColor', new Vector3(top.r, top.g, top.b));
  material.setVector3('bottomColor', new Vector3(bottom.r, bottom.g, bottom.b));
}
```

### 4. Integrate with Scene

```typescript
function setupSceneWithSky(scene: Scene): void {
  // Clear color (fallback)
  scene.clearColor = new Color4(0.5, 0.7, 0.9, 1.0);

  // Gradient sky
  const { mesh: sky } = createGradientSky(
    scene,
    '#B8D4E8',
    '#F5E6D3',
    1.0
  );

  // Ensure sky moves with camera (optional)
  scene.onBeforeRenderObservable.add(() => {
    if (scene.activeCamera) {
      sky.position.copyFrom(scene.activeCamera.position);
    }
  });
}
```

## Constraints

- Static gradient only—time-of-day cycling is game logic, not this skill
- No clouds or atmospheric effects (separate concern)
- Large sphere may affect frustum culling—position at camera if needed

## Activation

Activates when: skybox, gradient sky, background gradient, sky colors, horizon, stylized sky
