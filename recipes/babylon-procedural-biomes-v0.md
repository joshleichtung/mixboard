# Recipe: Procedural Biomes v0

## Purpose

Add procedural object placement to terrain with biome-aware distribution. Objects cluster naturally and vary by terrain region.

## When to Use

- Terrain exists but feels empty
- You want rocks, plants, or props scattered naturally
- Different terrain zones should have different object types/densities

## Required Skills

| Pack | Skill | Role |
|------|-------|------|
| babylon | `babylon-terrain` | Height queries for placement |
| babylon | `babylon-distribution` | Poisson/jitter placement algorithms |
| babylon | `babylon-instancing` | Efficient rendering of many objects |
| babylon | `babylon-perf-audit` | Verify draw calls and FPS |

## Preconditions

- Terrain mesh exists with `getHeightAt(x, z)` function
- Source meshes for props exist (rocks, plants, etc.)
- Materials assigned to source meshes

## Flow

### 1. Explore

**Goal**: Understand terrain and available props.

- [ ] Get terrain bounds (min/max X, Z)
- [ ] List source meshes to scatter (names, vertex counts)
- [ ] Identify biome zones if any (e.g., dunes vs rocky outcrops)

**Exit**: Know terrain bounds, prop inventory, zone definitions.

### 2. Architect

**Goal**: Define distribution strategy per biome/zone.

**Biome Map** (v0 approach—height-based):

| Height Range | Biome | Objects | Density |
|--------------|-------|---------|---------|
| Low (< 2) | Valley | Dry grass, small rocks | High |
| Mid (2–5) | Slope | Medium rocks, sparse grass | Medium |
| High (> 5) | Ridge | Large rocks, no grass | Low |

**Distribution Decisions**:

| Decision | Default | Escape Hatch |
|----------|---------|--------------|
| Algorithm | Poisson disk | Grid jitter for uniform |
| Min spacing | 2 units | Tighter for grass |
| Scale variation | 0.8–1.2 | Wider for organic feel |
| Rotation | Random Y | Fixed for aligned objects |

**Exit**: Biome rules documented, density targets set.

### 3. Implement

**Goal**: Generate positions and render instances.

**Step 1**: Generate all candidate positions once
```
babylon-distribution → poissonDisc() or gridJitter()
Generate positions across entire terrain bounds
```

**Step 2**: Snap all positions to terrain height
```
babylon-distribution → applyTerrainHeight(positions, getHeightAt)
Now each position.y reflects actual terrain elevation
```

**Step 3**: Partition positions into biomes by height
```
valleyPositions = positions.filter(p => p.y < 2)
slopePositions  = positions.filter(p => p.y >= 2 && p.y < 5)
ridgePositions  = positions.filter(p => p.y >= 5)
```

**Step 4**: Create instances per biome/object type
```
babylon-instancing → createThinInstances()
For each biome, sample positions and apply to object meshes
```

**Step 5**: (Optional) Add per-instance color variation
```
babylon-instancing → createColoredInstances()
Subtle hue shifts for natural look
```

**Pseudo-structure**:
```
// Generate once, partition by height
allPositions = poissonDisc(terrainSize, minSpacing)
applyTerrainHeight(allPositions, getHeightAt)

biomePositions = {
  valley: allPositions.filter(p => p.y < 2),
  slope:  allPositions.filter(p => p.y >= 2 && p.y < 5),
  ridge:  allPositions.filter(p => p.y >= 5),
}

for each biome in biomePositions:
  for each objectType in biome.objects:
    subset = sample(biomePositions[biome], objectType.ratio)
    createThinInstances(objectType.mesh, subset, {
      scaleRange: [0.8, 1.2],
      randomRotationY: true
    })
```

**Exit**: Objects scattered across terrain, rendered as instances.

### 4. Review

- [ ] Objects don't float above or sink into terrain
- [ ] Spacing feels natural (no grid patterns visible)
- [ ] Biome transitions aren't too abrupt
- [ ] No objects placed in playable paths (if applicable)
- [ ] Total instance count reasonable (< 50k combined)

### 5. Verify

- [ ] Walk through each biome—objects match expected types
- [ ] Check from above—no visible grid patterns
- [ ] Check close-up—objects sit on terrain surface
- [ ] Run `babylon-perf-audit`—draw calls reduced via instancing
- [ ] FPS stable with all objects visible

## Rollback/Safety

Keep prior scene state on a feature branch. Add one biome at a time and verify placement before adding the next.

## Verification Checklist

```
[ ] Positions: Y matches terrain height
[ ] Spacing: no visible grid, natural clustering
[ ] Biomes: correct objects in correct zones
[ ] Instancing: 1 draw call per object type (not per instance)
[ ] Performance: FPS stable, draw calls < 50
[ ] Collisions: objects don't block player (if applicable)
```

## Future (v1+)

- Noise-based biome map instead of height
- Exclusion zones (paths, water, structures)
- LOD for distant objects
- Chunked loading for large worlds
