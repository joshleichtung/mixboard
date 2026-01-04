# Recipe: Third-Person Exploration Camera

## Purpose

Set up a smooth third-person camera that follows a character across procedural terrain, with orbit controls for player look-around.

## When to Use

- You have a character controller moving across terrain
- You want the camera to follow smoothly without snapping
- Player should be able to orbit/zoom while exploring

## Required Skills

| Pack | Skill | Role |
|------|-------|------|
| babylon | `babylon-camera` | Follow + orbit camera setup |
| babylon | `babylon-character` | Velocity-based movement (already in place) |
| babylon | `babylon-terrain` | Height query for camera ground awareness |

## Preconditions

- Scene initialized with terrain mesh
- Character controller exists and moves via velocity
- Input system captures mouse/touch for orbit

## Flow

### 1. Explore

**Goal**: Understand current camera and character setup.

- [ ] Identify current camera type (`FreeCamera`, `ArcRotateCamera`, or none)
- [ ] Find where character position updates each frame
- [ ] Check if terrain exposes a `getHeightAt(x, z)` function

**Exit**: Know camera type, character update location, terrain height API.

### 2. Architect

**Goal**: Decide camera strategy.

| Decision | Default | Escape Hatch |
|----------|---------|--------------|
| Camera type | `ArcRotateCamera` for orbit + follow | `FreeCamera` if no orbit needed |
| Follow target | Character mesh position | Custom offset node |
| Height clamping | Camera Y >= terrain + 2 | Disable for flying characters |

- [ ] Sketch the update order: character moves → camera target updates → camera orbits
- [ ] Decide smoothing speed (default: 8)

**Exit**: Camera type chosen, update order documented.

### 3. Implement

**Goal**: Wire up the camera.

**Step 1**: Create orbit camera targeting character
```
babylon-camera → setupOrbitCamera() with character.position as target
```

**Step 2**: Add per-frame target tracking
```
babylon-camera → OrbitFollowCamera class
Call orbitFollowCamera.update(delta) in render loop
```

**Step 3**: (Optional) Clamp camera above terrain
```
babylon-terrain → getHeightAt(camera.position.x, camera.position.z)
Ensure camera.position.y >= terrainHeight + minClearance
```

**Exit**: Camera follows character smoothly with orbit controls.

### 4. Review

- [ ] Camera doesn't snap when character changes direction quickly
- [ ] Orbit limits prevent looking underground (`lowerBetaLimit > 0`)
- [ ] Zoom limits are reasonable (can't clip into character or zoom infinitely)
- [ ] No jitter at high speeds

### 5. Verify

- [ ] Move character across terrain—camera follows smoothly
- [ ] Orbit with mouse/touch—camera rotates around character
- [ ] Zoom in/out—respects distance limits
- [ ] Walk to terrain edge—camera doesn't clip through ground
- [ ] FPS stable (check with `babylon-perf-audit`)

## Verification Checklist

```
[ ] Camera type: ArcRotateCamera with target tracking
[ ] Smooth follow: no snapping on direction change
[ ] Orbit works: mouse/touch rotates view
[ ] Zoom limits: min/max distance enforced
[ ] Ground clearance: camera never below terrain
[ ] Performance: no FPS regression
```
