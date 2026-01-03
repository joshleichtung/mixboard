---
name: audio-tonejs-advanced
description: "Advanced Tone.js audio synthesis and scheduling. Activates for: Transport timing, synth architecture, effect chains, AudioWorklet integration, Web Audio optimization, sequencer patterns, audio scheduling. Does NOT activate for: basic audio playback, HTML5 Audio API, or non-audio projects."
category: dev
---

# Advanced Tone.js Patterns

Use these patterns for audio/music applications in Josh's projects.

## Transport & Timing

### Precise Scheduling
Always schedule with Transport time, never `setTimeout`:

```typescript
import * as Tone from 'tone'

// Start audio context on user gesture
await Tone.start()

// Schedule relative to Transport
Tone.Transport.scheduleRepeat((time) => {
  synth.triggerAttackRelease('C4', '8n', time)
}, '4n')

// Use Time for musical notation
Tone.Transport.bpm.value = 120
Tone.Transport.start()
```

### Look-Ahead Pattern
For tight timing, use the scheduling window:

```typescript
const lookAhead = 0.1 // 100ms look-ahead

Tone.Transport.scheduleRepeat((time) => {
  // Schedule slightly ahead for accuracy
  synth.triggerAttackRelease('C4', '8n', time + lookAhead)
}, '4n')
```

## Synth Architecture

### Layer Pattern for Rich Sounds
```typescript
const bassSynth = new Tone.MonoSynth({
  oscillator: { type: 'sawtooth' },
  envelope: { attack: 0.01, decay: 0.2, sustain: 0.4, release: 0.8 }
})

const padSynth = new Tone.PolySynth(Tone.Synth, {
  oscillator: { type: 'triangle' },
  envelope: { attack: 0.5, decay: 0.1, sustain: 1, release: 2 }
})
```

### Effect Chain Pattern
```typescript
const reverb = new Tone.Reverb({ decay: 2.5, wet: 0.3 })
const delay = new Tone.FeedbackDelay('8n', 0.3)
const compressor = new Tone.Compressor(-30, 3)

// Chain: synth -> delay -> reverb -> compressor -> output
synth.chain(delay, reverb, compressor, Tone.Destination)
```

## Sequencer Pattern

### Step Sequencer with Tone.Sequence
```typescript
const sequence = new Tone.Sequence(
  (time, note) => {
    if (note !== null) {
      synth.triggerAttackRelease(note, '16n', time)
    }
  },
  ['C4', 'E4', 'G4', null, 'C5', null, 'G4', 'E4'],
  '8n'
)

sequence.start(0)
Tone.Transport.start()
```

### Pattern with Velocity
```typescript
const pattern = new Tone.Pattern(
  (time, { note, velocity }) => {
    synth.triggerAttackRelease(note, '8n', time, velocity)
  },
  [
    { note: 'C4', velocity: 1 },
    { note: 'E4', velocity: 0.7 },
    { note: 'G4', velocity: 0.5 }
  ],
  'up'
)
```

## Performance Optimization

### Dispose Pattern
Always clean up to prevent memory leaks:

```typescript
// React cleanup
useEffect(() => {
  const synth = new Tone.Synth().toDestination()

  return () => {
    synth.dispose()
  }
}, [])
```

### Lazy Loading
```typescript
// Only create audio graph when needed
const [audioReady, setAudioReady] = useState(false)

const initAudio = async () => {
  await Tone.start()
  // Build audio graph here
  setAudioReady(true)
}
```

### Buffer Pre-loading
```typescript
const player = new Tone.Player()
await player.load('/samples/kick.wav')
// Now ready for instant playback
```

## State Integration

Use Zustand for UI state (play/pause, volume sliders), but keep Tone.js instances in refs:

```typescript
const synthRef = useRef<Tone.Synth | null>(null)
const { isPlaying, setIsPlaying } = useAudioStore()

useEffect(() => {
  synthRef.current = new Tone.Synth().toDestination()
  return () => synthRef.current?.dispose()
}, [])

const togglePlay = () => {
  if (isPlaying) {
    Tone.Transport.pause()
  } else {
    Tone.Transport.start()
  }
  setIsPlaying(!isPlaying)
}
```

## Audio Context Resume

Always handle suspended context:

```typescript
const resumeAudio = async () => {
  if (Tone.context.state === 'suspended') {
    await Tone.context.resume()
  }
  await Tone.start()
}
```

## Common Gotchas

1. **Never schedule in render** - use effects or callbacks
2. **Dispose everything** - synths, effects, sequences
3. **User gesture required** - call `Tone.start()` on click
4. **Use refs for Tone objects** - don't put in React state
5. **Time is in seconds** - Transport time, not ms
