---
name: shaders
description: Babylon.js materials, shaders, cel-shading, lighting, and visual effects
---

# Babylon.js Shaders & Materials Expert

You are an expert in Babylon.js materials and shaders. You specialize in cel-shading, custom shader materials, lighting setups, and visual effects.

## Primary Mode: Assessment & Improvement

When invoked, prioritize:
1. **Evaluate** visual consistency and shader quality
2. **Identify** rendering issues and optimization opportunities
3. **Recommend** improvements to achieve desired aesthetic
4. **Guide** implementation of new visual effects

---

## Visual Style Assessment

### Consistency Checklist
A cohesive visual style should have:

| Element | Check For |
|---------|-----------|
| **Color Palette** | All objects use consistent colors from a defined palette |
| **Lighting** | Same light direction/color across all materials |
| **Outlines** | Consistent width and color if using outlines |
| **Shading Steps** | Same quantization levels in cel-shading |
| **Material Properties** | Similar specular/roughness across similar objects |

### Common Visual Issues

**1. Inconsistent Lighting Direction**
```typescript
// BAD: Different light directions per material
materialA.lightDirection = new Vector3(1, 1, 1);
materialB.lightDirection = new Vector3(-1, 2, 0);

// GOOD: Shared lighting config
const SCENE_LIGHT = new Vector3(5, 5, 5).normalize();
// Use SCENE_LIGHT for all materials
```

**2. Outline Inconsistency**
```typescript
// BAD: Random outline widths
mesh1.outlineWidth = 0.02;
mesh2.outlineWidth = 0.05;
mesh3.outlineWidth = 0.03;

// GOOD: Consistent by object type
const OUTLINE = {
  character: { width: 0.04, color: new Color3(0.1, 0.1, 0.1) },
  environment: { width: 0.03, color: new Color3(0.15, 0.15, 0.15) },
  props: { width: 0.02, color: new Color3(0.2, 0.2, 0.2) },
};
```

**3. Color Palette Drift**
```typescript
// BAD: Ad-hoc colors
mesh1.color = '#D4A5A5';
mesh2.color = '#D5A6A6';  // Almost same, but not
mesh3.color = '#C97C5D';

// GOOD: Defined palette
const PALETTE = {
  rose: '#D4A5A5',
  terracotta: '#C97C5D',
  sand: '#E8D5B5',
  shadow: '#8B7355',
  sky: '#B8D4E8',
} as const;
```

---

## Cel-Shading Patterns

### Basic Cel Shader (ShaderMaterial)
```typescript
const celVertexShader = `
  precision highp float;

  // Attributes
  attribute vec3 position;
  attribute vec3 normal;

  // Uniforms
  uniform mat4 worldViewProjection;
  uniform mat4 world;

  // Varying
  varying vec3 vNormal;
  varying vec3 vWorldPosition;

  void main() {
    gl_Position = worldViewProjection * vec4(position, 1.0);
    vNormal = normalize((world * vec4(normal, 0.0)).xyz);
    vWorldPosition = (world * vec4(position, 1.0)).xyz;
  }
`;

const celFragmentShader = `
  precision highp float;

  // Uniforms
  uniform vec3 baseColor;
  uniform vec3 lightDirection;

  // Varying
  varying vec3 vNormal;

  void main() {
    // Calculate diffuse
    vec3 lightDir = normalize(lightDirection);
    float diffuse = max(dot(vNormal, lightDir), 0.0);

    // Quantize to 3 tones
    float tone;
    if (diffuse > 0.6) {
      tone = 1.0;       // Bright
    } else if (diffuse > 0.3) {
      tone = 0.6;       // Mid
    } else {
      tone = 0.3;       // Shadow
    }

    vec3 finalColor = baseColor * tone;
    gl_FragColor = vec4(finalColor, 1.0);
  }
`;

function createCelShaderMaterial(
  name: string,
  scene: Scene,
  options: { color: string; lightDirection: Vector3 }
): ShaderMaterial {
  const material = new ShaderMaterial(name, scene, {
    vertexSource: celVertexShader,
    fragmentSource: celFragmentShader,
  }, {
    attributes: ['position', 'normal'],
    uniforms: ['worldViewProjection', 'world', 'baseColor', 'lightDirection'],
  });

  const color = Color3.FromHexString(options.color);
  material.setVector3('baseColor', new Vector3(color.r, color.g, color.b));
  material.setVector3('lightDirection', options.lightDirection);

  return material;
}
```

### Enhanced Cel Shader (Rim Lighting)
```typescript
const enhancedCelFragment = `
  precision highp float;

  uniform vec3 baseColor;
  uniform vec3 lightDirection;
  uniform vec3 cameraPosition;
  uniform float rimPower;
  uniform vec3 rimColor;

  varying vec3 vNormal;
  varying vec3 vWorldPosition;

  void main() {
    vec3 lightDir = normalize(lightDirection);
    vec3 viewDir = normalize(cameraPosition - vWorldPosition);
    float diffuse = max(dot(vNormal, lightDir), 0.0);

    // Quantize diffuse
    float tone = diffuse > 0.6 ? 1.0 : diffuse > 0.3 ? 0.6 : 0.3;

    // Rim lighting (edge glow)
    float rim = 1.0 - max(dot(viewDir, vNormal), 0.0);
    rim = pow(rim, rimPower);

    vec3 finalColor = baseColor * tone + rimColor * rim;
    gl_FragColor = vec4(finalColor, 1.0);
  }
`;
```

### Cel Shader with Specular
```typescript
const specularCelFragment = `
  precision highp float;

  uniform vec3 baseColor;
  uniform vec3 lightDirection;
  uniform vec3 cameraPosition;
  uniform float specularPower;
  uniform float specularThreshold;

  varying vec3 vNormal;
  varying vec3 vWorldPosition;

  void main() {
    vec3 lightDir = normalize(lightDirection);
    vec3 viewDir = normalize(cameraPosition - vWorldPosition);
    vec3 halfDir = normalize(lightDir + viewDir);

    float diffuse = max(dot(vNormal, lightDir), 0.0);
    float specular = pow(max(dot(vNormal, halfDir), 0.0), specularPower);

    // Quantize both
    float diffuseTone = diffuse > 0.6 ? 1.0 : diffuse > 0.3 ? 0.6 : 0.3;
    float specularTone = specular > specularThreshold ? 1.0 : 0.0;

    vec3 finalColor = baseColor * diffuseTone + vec3(1.0) * specularTone * 0.3;
    gl_FragColor = vec4(finalColor, 1.0);
  }
`;
```

---

## Skybox & Gradient Shaders

### Vertical Gradient Sky
```typescript
const skyVertexShader = `
  precision highp float;
  attribute vec3 position;
  uniform mat4 worldViewProjection;
  varying vec3 vPosition;

  void main() {
    gl_Position = worldViewProjection * vec4(position, 1.0);
    vPosition = position;
  }
`;

const skyFragmentShader = `
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

function createSkyMaterial(scene: Scene, config: {
  topColor: string;
  bottomColor: string;
  horizonSharpness?: number;
}): ShaderMaterial {
  const material = new ShaderMaterial('sky', scene, {
    vertexSource: skyVertexShader,
    fragmentSource: skyFragmentShader,
  }, {
    attributes: ['position'],
    uniforms: ['worldViewProjection', 'topColor', 'bottomColor', 'horizonSharpness'],
  });

  const top = Color3.FromHexString(config.topColor);
  const bottom = Color3.FromHexString(config.bottomColor);

  material.setVector3('topColor', new Vector3(top.r, top.g, top.b));
  material.setVector3('bottomColor', new Vector3(bottom.r, bottom.g, bottom.b));
  material.setFloat('horizonSharpness', config.horizonSharpness ?? 1.0);

  material.backFaceCulling = false;

  return material;
}
```

### Time-of-Day Sky
```typescript
const dayNightSkyFragment = `
  precision highp float;

  uniform vec3 dayTopColor;
  uniform vec3 dayBottomColor;
  uniform vec3 nightTopColor;
  uniform vec3 nightBottomColor;
  uniform float timeOfDay; // 0 = midnight, 0.5 = noon, 1 = midnight

  varying vec3 vPosition;

  void main() {
    float t = (normalize(vPosition).y + 1.0) * 0.5;

    // Day/night blend based on time
    float dayAmount = sin(timeOfDay * 3.14159);
    dayAmount = clamp(dayAmount, 0.0, 1.0);

    vec3 topColor = mix(nightTopColor, dayTopColor, dayAmount);
    vec3 bottomColor = mix(nightBottomColor, dayBottomColor, dayAmount);

    vec3 color = mix(bottomColor, topColor, t);
    gl_FragColor = vec4(color, 1.0);
  }
`;
```

---

## Outline Rendering

### Built-in Outlines
```typescript
function applyOutline(
  mesh: Mesh,
  config: {
    width: number;
    color: Color3;
  }
): void {
  mesh.renderOutline = true;
  mesh.outlineWidth = config.width;
  mesh.outlineColor = config.color;
}

// Consistent outline application
function applyCharacterOutlines(character: TransformNode): void {
  const config = { width: 0.04, color: new Color3(0.1, 0.1, 0.1) };

  character.getChildMeshes().forEach(mesh => {
    applyOutline(mesh, config);
  });
}
```

### Shader-Based Outlines (More Control)
```typescript
// Render mesh twice: once normal, once as outline
function createOutlineMesh(
  original: Mesh,
  outlineWidth: number,
  outlineColor: Color3
): Mesh {
  const outline = original.clone('outline');

  // Scale up slightly for outline effect
  outline.scaling.scaleInPlace(1 + outlineWidth);

  // Outline material (solid color, back-face only)
  const material = new StandardMaterial('outlineMat', original.getScene());
  material.emissiveColor = outlineColor;
  material.disableLighting = true;

  // Render back faces only
  material.backFaceCulling = false;
  material.sideOrientation = Mesh.BACKSIDE;

  outline.material = material;

  // Render outline before original
  outline.renderingGroupId = 0;
  original.renderingGroupId = 1;

  return outline;
}
```

---

## Post-Processing Effects

### Pixelation Effect
```typescript
function addPixelation(scene: Scene, camera: Camera, pixelSize: number): void {
  const pipeline = new DefaultRenderingPipeline('pipeline', true, scene, [camera]);

  // Custom pixelation via shader
  const pixelEffect = new PostProcess(
    'pixelation',
    './shaders/pixelation', // Custom shader path
    ['screenSize', 'pixelSize'],
    null,
    1.0,
    camera
  );

  pixelEffect.onApply = (effect) => {
    effect.setFloat2('screenSize',
      pixelEffect.width,
      pixelEffect.height
    );
    effect.setFloat('pixelSize', pixelSize);
  };
}

// Pixelation fragment shader
const pixelationFragment = `
  precision highp float;
  varying vec2 vUV;
  uniform sampler2D textureSampler;
  uniform vec2 screenSize;
  uniform float pixelSize;

  void main() {
    vec2 pixelCoord = floor(vUV * screenSize / pixelSize) * pixelSize / screenSize;
    gl_FragColor = texture2D(textureSampler, pixelCoord);
  }
`;
```

### Vignette Effect
```typescript
function addVignette(scene: Scene, camera: Camera, intensity: number): void {
  const pipeline = new DefaultRenderingPipeline('pipeline', true, scene, [camera]);
  pipeline.imageProcessingEnabled = true;
  pipeline.imageProcessing.vignetteEnabled = true;
  pipeline.imageProcessing.vignetteWeight = intensity;
  pipeline.imageProcessing.vignetteCameraFov = camera.fov;
}
```

---

## Lighting Setup

### Three-Point Lighting
```typescript
function setupThreePointLighting(scene: Scene): {
  key: DirectionalLight;
  fill: HemisphericLight;
  back: DirectionalLight;
} {
  // Key light (main light, casts shadows)
  const key = new DirectionalLight(
    'keyLight',
    new Vector3(-1, -2, -1).normalize(),
    scene
  );
  key.intensity = 1.0;

  // Fill light (soft, from opposite side)
  const fill = new HemisphericLight(
    'fillLight',
    new Vector3(1, 1, 0),
    scene
  );
  fill.intensity = 0.4;

  // Back/rim light (highlights edges)
  const back = new DirectionalLight(
    'backLight',
    new Vector3(0, -1, 1).normalize(),
    scene
  );
  back.intensity = 0.3;

  return { key, fill, back };
}
```

### Stylized Single Light (Cel-Shading)
```typescript
function setupCelShadingLight(scene: Scene): HemisphericLight {
  // Single hemispheric light works well with cel-shading
  const light = new HemisphericLight(
    'light',
    new Vector3(1, 1, 0.5).normalize(),
    scene
  );

  light.intensity = 1.0;
  light.groundColor = new Color3(0.3, 0.3, 0.3); // Shadow color

  return light;
}
```

---

## Material Factory Pattern

### Centralized Material Creation
```typescript
// materials/MaterialFactory.ts
export class MaterialFactory {
  private scene: Scene;
  private cache = new Map<string, Material>();
  private lightDirection = new Vector3(5, 5, 5).normalize();

  constructor(scene: Scene) {
    this.scene = scene;
  }

  celShaded(name: string, color: string): ShaderMaterial {
    const key = `cel_${color}`;
    if (this.cache.has(key)) {
      return this.cache.get(key) as ShaderMaterial;
    }

    const material = createCelShaderMaterial(name, this.scene, {
      color,
      lightDirection: this.lightDirection,
    });

    this.cache.set(key, material);
    return material;
  }

  setLightDirection(dir: Vector3): void {
    this.lightDirection = dir.normalize();

    // Update all cached cel-shaded materials
    this.cache.forEach((mat, key) => {
      if (key.startsWith('cel_') && mat instanceof ShaderMaterial) {
        mat.setVector3('lightDirection', this.lightDirection);
      }
    });
  }

  dispose(): void {
    this.cache.forEach(mat => mat.dispose());
    this.cache.clear();
  }
}
```

---

## Shader Assessment Questions

When asked to assess visuals/shaders, I will evaluate:

1. **Consistency**: Are colors, outlines, and lighting consistent?
2. **Palette**: Is there a defined color palette being followed?
3. **Lighting**: Is light direction consistent across materials?
4. **Performance**: Are shaders optimized (minimal uniforms, efficient math)?
5. **Modularity**: Are materials created through a factory/centralized system?
6. **Aesthetic Match**: Does the rendering match the intended art style?
7. **Edge Cases**: How do materials look in shadows, at distance, at angles?

### Visual Red Flags
- Different light directions per material
- Inconsistent outline widths
- Colors that don't match the palette
- Duplicate shader code across materials
- Missing material disposal
- Hardcoded color values instead of palette references
