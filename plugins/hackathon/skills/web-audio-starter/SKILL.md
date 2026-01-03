---
name: web-audio-starter
description: Quick Web Audio API starter code for hackathons
---

Web Audio API quick-start code for {{project_name}}:

**1. Basic Audio Context Setup:**
```typescript
// lib/audio/context.ts
let audioContext: AudioContext | null = null;

export const getAudioContext = () => {
  if (!audioContext) {
    audioContext = new (window.AudioContext || window.webkitAudioContext)();
  }
  return audioContext;
};

export const resumeAudioContext = async () => {
  const ctx = getAudioContext();
  if (ctx.state === 'suspended') {
    await ctx.resume();
  }
};
```

**2. Simple Synth:**
```typescript
// lib/audio/synth.ts
export const playNote = (frequency: number, duration: number = 0.1) => {
  const ctx = getAudioContext();
  const oscillator = ctx.createOscillator();
  const gainNode = ctx.createGain();

  oscillator.connect(gainNode);
  gainNode.connect(ctx.destination);

  oscillator.frequency.value = frequency;
  gainNode.gain.value = 0.3;

  const now = ctx.currentTime;
  oscillator.start(now);
  gainNode.gain.exponentialRampToValueAtTime(0.01, now + duration);
  oscillator.stop(now + duration);
};
```

**3. Drum Machine Pattern:**
```typescript
// lib/audio/drum-machine.ts
import { playNote } from './synth';

const STEPS = 16;
let currentStep = 0;
let intervalId: number | null = null;

export const startSequence = (pattern: boolean[], bpm: number = 120) => {
  const interval = (60 / bpm) * 1000 / 4; // 16th notes

  intervalId = window.setInterval(() => {
    if (pattern[currentStep]) {
      playNote(440, 0.05); // Play on active steps
    }
    currentStep = (currentStep + 1) % STEPS;
  }, interval);
};

export const stopSequence = () => {
  if (intervalId) {
    clearInterval(intervalId);
    intervalId = null;
  }
  currentStep = 0;
};
```

**4. React Component:**
```typescript
// components/DrumMachine.tsx
'use client';

import { useState } from 'react';
import { startSequence, stopSequence } from '@/lib/audio/drum-machine';
import { resumeAudioContext } from '@/lib/audio/context';

export default function DrumMachine() {
  const [playing, setPlaying] = useState(false);
  const [pattern, setPattern] = useState(Array(16).fill(false));

  const togglePlay = async () => {
    await resumeAudioContext(); // Required for mobile

    if (playing) {
      stopSequence();
    } else {
      startSequence(pattern);
    }
    setPlaying(!playing);
  };

  const toggleStep = (index: number) => {
    const newPattern = [...pattern];
    newPattern[index] = !newPattern[index];
    setPattern(newPattern);
  };

  return (
    <div className="space-y-4">
      <button onClick={togglePlay}>
        {playing ? 'Stop' : 'Play'}
      </button>
      <div className="grid grid-cols-16 gap-1">
        {pattern.map((active, i) => (
          <button
            key={i}
            onClick={() => toggleStep(i)}
            className={active ? 'bg-blue-500' : 'bg-gray-200'}
          />
        ))}
      </div>
    </div>
  );
}
```

**Hackathon Pro Tips:**
- Use Tone.js instead of raw Web Audio for faster dev
- Copy drum sounds from freesound.org
- Make the grid look cool - judges love visual feedback
- Add keyboard shortcuts (spacebar = play/pause)

Ready to jam!
