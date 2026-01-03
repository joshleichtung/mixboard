---
name: audio-web-bootstrap
description: Bootstrap a web-based audio/music sequencer app
---

Create web-based audio app: {{app_name}}

Based on your vibeseq/vibeloop experience, here's the optimal setup:

**Stack:**
```bash
bun create next-app {{app_name}} --typescript --tailwind
cd {{app_name}}
bun add tone zustand
bun add -d @types/node
```

**Web Audio Setup:**
1. **Tone.js for audio engine:**
   - Handles timing and scheduling
   - Built-in synthesizers and effects
   - Transport for sequencer control

2. **Zustand for state:**
   - Pattern/sequence state
   - Playback state
   - UI interactions

3. **React for UI:**
   - Grid-based sequencer UI
   - Real-time visual feedback
   - Responsive controls

**Core Architecture:**
```
/app
  /components
    /sequencer
      Grid.tsx          # Main sequencer grid
      Controls.tsx      # Play/pause/tempo
      SoundEngine.tsx   # Tone.js wrapper
  /lib
    /audio
      synth.ts          # Sound synthesis
      sequencer.ts      # Pattern playback
    /store
      sequencer-store.ts # Zustand state
```

**Key Patterns from your previous work:**
- Web Audio Context initialization
- RequestAnimationFrame for visual sync
- Touch-friendly grid interactions
- Mobile-responsive design

**Optional: Real-time collaboration (like vibeseq):**
```bash
bun add socket.io-client
```

**Creative UI inspiration:**
- LCARS interface (like vibeloop)
- Tenori-On grid aesthetic
- Custom color schemes

Ready to build! What type of sequencer?
