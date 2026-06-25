# Aura - Technical Documentation

> [!NOTE]
> This document provides an exhaustive, granular breakdown of the Aura Ambient Music Generator. It covers everything from the front-end architecture and the custom generative sequencing logic to the low-level Web Audio API synthesis parameters.

## 1. High-Level Architecture

Aura is a client-side, zero-backend web application. All computational load (both audio synthesis and UI rendering) runs directly in the user's browser. 

**Tech Stack:**
- **Framework:** React 18 (Functional Components, Hooks)
- **Build Tool:** Vite 5 (for hyper-fast HMR and optimized production bundling)
- **Audio Engine:** Native HTML5 Web Audio API (No external audio libraries like Tone.js)
- **Visuals:** HTML5 `<canvas>` (Procedural Particles) + CSS3 Animations (Gradients)
- **Styling:** Vanilla CSS (CSS Variables, Flexbox, Glassmorphism)
- **Icons:** `lucide-react`
- **Hosting:** Netlify (Global CDN, Continuous Deployment via GitHub)

---

## 2. The Audio Engine (`src/audioEngine.js`)

The `AudioEngine` class is a singleton that orchestrates all sound generation. It bypasses the need for downloading pre-rendered `.mp3` files by synthesizing sounds procedurally using mathematical waveforms.

### 2.1 The Master Bus & Global Effects
When the engine initializes, it creates an `AudioContext`. All sound generators eventually route into a Master Gain Node, which acts as the global volume control.

**Global Reverb (Convolution Reverb):**
Instead of mathematical algorithmic reverb, Aura uses a procedural *Convolution Reverb*. 
- It creates a synthetic "Impulse Response" (IR) buffer lasting 2.5 seconds.
- The buffer is filled with white noise (`Math.random() * 2 - 1`).
- An exponential decay curve (`Math.exp(-i / ...)`) is applied to the noise to simulate the acoustic tail of a physical room.
- This creates a massive, lush, cathedral-like atmosphere essential for ambient music.

### 2.2 The Generative Sequencer
Music generation is handled by a strictly timed 16-step algorithmic sequencer.

**Scheduling (The "Lookahead" Pattern):**
To ensure perfect timing without JavaScript thread blocking, Aura uses a standard Web Audio scheduling technique:
- A `setTimeout` loop fires every 25ms (the "lookahead" window).
- It checks if any 16th notes need to be played within the next 0.1 seconds (the "scheduleAheadTime").
- If so, it pre-schedules those audio events on the `AudioContext` timeline. This guarantees flawless rhythm even if the main UI thread stutters.

**Musical Theory & Progressions:**
The engine loops over 64 total steps (4 bars of 4/4 time). It dynamically switches chord progressions based on the selected Mood. Progressions are defined as arrays of MIDI note values:
- **Calm (Lo-Fi Chillhop / 75 BPM):** Uses Major 7th and Minor 9th jazz chords (e.g., Cmaj7 → Am9 → Fmaj7 → G7).
- **Deep (Synthwave / 85 BPM):** Uses darker minor 9th chords in lower registers (e.g., Cm9 → Fm9 → Abmaj7 → G7).
- **Ethereal (Dream Pop / 95 BPM):** Uses brighter, uplifting progressions (e.g., Fmaj7 → G → Em7 → Am).

### 2.3 Synthesizers & Instruments

The sequencer triggers specific instrument methods. Each method constructs a temporary synthesizer node chain, plays the note, and then garbage collects the nodes.

#### A. The Drum Machine
- **Kick (`playKick`):** A low-frequency sine wave oscillator starting at 100Hz and exponentially dropping to near 0Hz over 0.3 seconds. Coupled with a steep volume envelope, this creates a punchy, synthesized bass thud.
- **Snare (`playSnare`):** A buffer of pure white noise run through a High-Pass Biquad Filter (cutoff at 1200Hz). A sharp decay envelope shapes the noise into a "crack".
- **Hi-Hat (`playHihat`):** A square wave run through a severe High-Pass filter (7000Hz). Varying the decay envelope length differentiates between a "closed" hat (0.05s) and an "open" hat (0.2s).

#### B. The Polyphonic Chords (`playChord`)
When a chord triggers, the engine loops through the MIDI array and spawns a distinct oscillator for each note:
- **Waveforms:** Uses triangle waves for smooth pads (Calm/Ethereal) and sawtooth waves for grittier pads (Deep).
- **Filter Envelopes:** Each chord routes through a Low-Pass filter. In the "Deep" mood, the filter cutoff smoothly animates downward during the chord, creating a classic "wobble" or "filter sweep" effect.

#### C. The Bassline (`playBass`)
Spawns a sine or square wave tuned exactly two octaves (24 MIDI notes) below the root note of the current chord. It runs through a heavy 250Hz Low-Pass filter to ensure it remains a deep sub-bass without interfering with the mid-range chords.

#### D. The Generative Arpeggiator (`playMelody`)
The engine features a stochastic (randomized) lead synthesizer. 
- **Pitch Selection:** It randomly selects a note from the *currently active chord* and shifts it up one octave. By constraining the random selection to the current chord tones, it guarantees the melody is mathematically impossible to be dissonant.
- **Rhythm:** It has a 30% chance to skip playing on any given beat, and is hard-coded to avoid playing on downbeats. This creates syncopated, human-feeling improvisations.

---

## 3. The User Interface (`src/App.jsx`)

The UI is built with React and prioritizes a frictionless, immersive aesthetic.

**State Management:**
- `isPlaying` (boolean): Tracks global playback status.
- `mood` (string): Tracks the current vibe.
- `volume` (float): Tracks the master gain level (0.0 to 1.0).

**Security & Browser Policies:**
Modern browsers block the Web Audio API until a user physically interacts with the page (to prevent autoplaying spam). The `useEffect` hook ensures the `AudioContext` remains suspended until the user clicks the central Play button.

**Volume Control:**
The volume slider maps directly to the `masterGain` node. Adjustments use `setTargetAtTime` rather than immediate value changes to prevent "clicking" or "popping" artifacts in the audio stream when dragging the slider.

---

## 4. Visual Engine

### 4.1 Glassmorphism & CSS (`src/index.css`)
The UI heavily relies on "Glassmorphism"—the illusion of frosted glass floating in 3D space.
- Achieved using `backdrop-filter: blur(16px)` layered over semi-transparent white backgrounds (`rgba(255, 255, 255, 0.05)`).
- The main background is an oversized `radial-gradient` that uses CSS `@keyframes` to slowly rotate a full 360 degrees every 80 seconds, providing a hypnotic, ambient backdrop.

### 4.2 HTML5 Particle Canvas (`src/ParticleBackground.jsx`)
A secondary visual layer generated by standard JavaScript `<canvas>`.
- **Initialization:** Dynamically spawns up to 80 Particle objects (scaling down for smaller mobile screens to save battery).
- **Physics:** Each particle is assigned a random X/Y coordinate, size, opacity, and drift vector. 
- **Animation Loop:** Uses `requestAnimationFrame` to update positions 60 times a second. Particles wrap around the edges of the screen (e.g., floating off the right edge spawns them on the left edge), creating an infinite universe effect.

---

## 5. Deployment Architecture

While currently hosted on Netlify, the application underwent a rapid prototyping pipeline:
1. **Local Vite Server:** Initial development ran on `localhost:5173`. Node.js compatibility required pinning Vite to version 5.
2. **Local Nginx:** A production build (`dist`) was generated and served via a bare-metal Nginx binary running on port `8080` to simulate a true production environment.
3. **LocalTunnel:** The Nginx server was briefly exposed to the public internet using `localtunnel`, intercepting headers (`bypass-tunnel-reminder`) via the Nginx `nginx.conf` to bypass proxy splash pages.
4. **Git & GitHub:** The codebase (excluding large Nginx binaries via `.gitignore`) was committed and pushed to a public GitHub repository.
5. **Netlify:** The GitHub repository is linked to Netlify, which triggers an automated `npm run build` and global CDN invalidation every time new code is pushed to the `main` branch.
