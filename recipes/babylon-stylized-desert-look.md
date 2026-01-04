# Recipe: Stylized Desert Look

## Purpose

Create a cohesive stylized visual style for a desert environment: cel-shaded materials, gradient sky, and warm color palette.

## When to Use

- You have terrain and objects rendering but they look flat or default
- You want a non-photorealistic, illustrated aesthetic
- Desert/dune setting with warm tones and clear sky

## Required Skills

| Pack | Skill | Role |
|------|-------|------|
| babylon | `babylon-cel-shader` | Quantized lighting for stylized materials |
| babylon | `babylon-sky-gradient` | Procedural gradient skybox |
| babylon | `babylon-terrain` | Normal recomputation if lighting looks flat |

## Preconditions

- Scene has terrain mesh and any objects to shade
- Single directional light (sun) exists or will be created
- No existing post-processing that conflicts with cel shading

## Flow

### 1. Explore

**Goal**: Inventory what needs shading and current lighting.

- [ ] List all meshes that need cel-shaded materials
- [ ] Find existing light(s)—type, direction, intensity
- [ ] Check if meshes have normals computed (required for cel shading)

**Exit**: Know which meshes to shade, light direction, normal status.

### 2. Architect

**Goal**: Define the visual palette and shading strategy.

**Color Palette** (desert defaults):

| Element | Color | Hex |
|---------|-------|-----|
| Sand base | Warm tan | `#D4A574` |
| Sand shadow | Deeper ochre | `#A67B5B` |
| Sky top | Deep blue | `#1E3A5F` |
| Sky horizon | Pale orange | `#F5D6BA` |
| Light direction | 45° elevation (to-light vector, normalized) | `(0.41, 0.82, 0.41)` |

**Shading Decisions**:

| Decision | Default | Escape Hatch |
|----------|---------|--------------|
| Tone count | 3 (hardcoded in shader) | Modify shader for 2 or 4 |
| Outline | Optional via `applyOutline()` | Skip for softer feel |
| Sky gradient | 2-color vertical | 3-color for sunset |

**Exit**: Palette documented, band count chosen.

### 3. Implement

**Goal**: Apply sky and materials.

**Step 1**: Create gradient sky
```
babylon-sky-gradient → createGradientSky(scene, topColor, bottomColor)
Top: #1E3A5F, Horizon: #F5D6BA
```

**Step 2**: Set up light direction for cel shader
```
lightDir = new Vector3(0.41, 0.82, 0.41)  // normalized, pointing TO light
Pass to createCelMaterial() for all materials
```

**Step 3**: Apply cel shader to terrain
```
babylon-cel-shader → createCelMaterial(name, scene, '#D4A574', lightDir)
Apply to terrain mesh
```

**Step 4**: Apply cel shader to objects
```
Same pattern for rocks, plants, character
Consider slightly different hues per object type
```

**Step 5**: (Optional) Recompute normals if lighting looks flat
```
babylon-terrain → recomputeNormalsManually() if terrain shading is uniform
```

**Exit**: All meshes cel-shaded, sky gradient visible.

### 4. Review

- [ ] Shadow bands visible on terrain undulations (not uniformly lit)
- [ ] Sky gradient blends smoothly at horizon
- [ ] Character/objects don't clash with terrain palette
- [ ] No harsh edges or banding artifacts
- [ ] Sun direction creates readable shadows

### 5. Verify

- [ ] Rotate camera 360°—sky gradient consistent
- [ ] Walk into shadows—band transitions visible
- [ ] Check from far distance—silhouettes read clearly
- [ ] Check normals—terrain isn't flat-shaded unintentionally
- [ ] Performance stable (cel shader shouldn't impact FPS significantly)

## Rollback/Safety

Keep prior materials on a feature branch. Apply one material at a time and verify visually before proceeding.

## Verification Checklist

```
[ ] Sky: gradient renders top-to-horizon
[ ] Terrain: 3 visible tone bands under sun
[ ] Objects: consistent cel-shaded style
[ ] Palette: warm desert tones throughout
[ ] Normals: terrain shows shading variation
[ ] Performance: no FPS drop from materials
```

## Palette Reference

```
Sand:       #D4A574 → #A67B5B (lit → shadow)
Rock:       #8B7355 → #5C4A3D
Dry grass:  #C4A35A → #8B7542
Character:  Project-specific
Sky top:    #1E3A5F
Sky horizon:#F5D6BA
```
