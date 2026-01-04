---
name: babylon-cel-shader
description: Create cel-shaded materials with quantized lighting and outlines
---

# Babylon Cel-Shader Material

## Purpose

Create a material that renders with discrete shading bands (cel-shading/toon shading). Use for stylized, comic-book, or animated aesthetics.

## Inputs

- Base color (hex string)
- Light direction (Vector3)
- Tone count (2-4 bands)
- Optional: outline width and color

## Outputs

- ShaderMaterial ready to apply to meshes
  - Verify: No shader compilation errors in console
  - Verify: Mesh shows discrete shading bands (not smooth gradient)
  - Verify: Light direction change affects all materials consistently

## Assumptions

- Babylon.js 6.x+
- WebGL2 or WebGL1 with required extensions
- Scene has at least one light source (for direction reference)

## Procedure

### 1. Basic Cel Shader

```typescript
import { ShaderMaterial, Scene, Vector3, Color3 } from '@babylonjs/core';

const CEL_VERTEX = `
  precision highp float;
  attribute vec3 position;
  attribute vec3 normal;
  uniform mat4 worldViewProjection;
  uniform mat4 world;
  varying vec3 vNormal;

  void main() {
    gl_Position = worldViewProjection * vec4(position, 1.0);
    vNormal = normalize((world * vec4(normal, 0.0)).xyz);
  }
`;

const CEL_FRAGMENT = `
  precision highp float;
  uniform vec3 baseColor;
  uniform vec3 lightDirection;
  varying vec3 vNormal;

  void main() {
    float diffuse = max(dot(vNormal, normalize(lightDirection)), 0.0);

    // 3-tone quantization
    float tone;
    if (diffuse > 0.6) tone = 1.0;
    else if (diffuse > 0.3) tone = 0.6;
    else tone = 0.3;

    gl_FragColor = vec4(baseColor * tone, 1.0);
  }
`;

function createCelMaterial(
  name: string,
  scene: Scene,
  color: string,
  lightDir: Vector3
): ShaderMaterial {
  const material = new ShaderMaterial(name, scene, {
    vertexSource: CEL_VERTEX,
    fragmentSource: CEL_FRAGMENT,
  }, {
    attributes: ['position', 'normal'],
    uniforms: ['worldViewProjection', 'world', 'baseColor', 'lightDirection'],
  });

  const c = Color3.FromHexString(color);
  material.setVector3('baseColor', new Vector3(c.r, c.g, c.b));
  material.setVector3('lightDirection', lightDir.normalize());

  return material;
}
```

### 2. Configurable Tone Count

```typescript
const CEL_FRAGMENT_CONFIGURABLE = `
  precision highp float;
  uniform vec3 baseColor;
  uniform vec3 lightDirection;
  uniform float toneCount;  // 2, 3, or 4
  varying vec3 vNormal;

  void main() {
    float diffuse = max(dot(vNormal, normalize(lightDirection)), 0.0);

    // Quantize to N tones
    float tone = floor(diffuse * toneCount) / (toneCount - 1.0);
    tone = max(tone, 0.2);  // Minimum brightness

    gl_FragColor = vec4(baseColor * tone, 1.0);
  }
`;
```

### 3. Apply Outline

Babylon's built-in outline (simple approach):

```typescript
import { Mesh, Color3 } from '@babylonjs/core';

function applyOutline(
  mesh: Mesh,
  width: number,
  color: Color3
): void {
  mesh.renderOutline = true;
  mesh.outlineWidth = width;
  mesh.outlineColor = color;
}

// Consistent outline config
const OUTLINE_CONFIG = {
  character: { width: 0.04, color: new Color3(0.1, 0.1, 0.1) },
  environment: { width: 0.03, color: new Color3(0.15, 0.15, 0.15) },
  props: { width: 0.02, color: new Color3(0.2, 0.2, 0.2) },
};
```

### 4. Material Factory Pattern

Centralize material creation for consistency:

```typescript
class CelMaterialFactory {
  private cache = new Map<string, ShaderMaterial>();
  private lightDir = new Vector3(5, 5, 5).normalize();

  constructor(private scene: Scene) {}

  get(color: string): ShaderMaterial {
    if (this.cache.has(color)) {
      return this.cache.get(color)!;
    }

    const material = createCelMaterial(`cel_${color}`, this.scene, color, this.lightDir);
    this.cache.set(color, material);
    return material;
  }

  setLightDirection(dir: Vector3): void {
    this.lightDir = dir.normalize();
    // Update all cached materials
    this.cache.forEach(mat => {
      mat.setVector3('lightDirection', this.lightDir);
    });
  }

  dispose(): void {
    this.cache.forEach(mat => mat.dispose());
    this.cache.clear();
  }
}
```

### 5. Color Palette

Define colors centrally:

```typescript
const PALETTE = {
  // Character
  skin: '#D4A5A5',
  clothing: '#8B7355',

  // Environment
  grass: '#7CAA6E',
  rock: '#8B8B7A',
  sand: '#E8D5B5',

  // Accent
  highlight: '#F4D35E',
  shadow: '#4A4A4A',
} as const;

// Usage
const material = factory.get(PALETTE.grass);
```

## Constraints

- No rim lighting or specular in this skill (keep minimal)
- Outline is post-effect—may not work with all render pipelines
- All cel materials should share light direction for consistency
- No transparency support (add if needed as variant)

## Activation

Activates when: cel shader, cel-shading, toon shader, quantized lighting, cartoon material, stylized rendering
