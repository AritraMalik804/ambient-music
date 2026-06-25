# Aura - Infinite Background Music for Focus & Flow

[![Netlify Status](https://api.netlify.com/api/v1/badges/c46c8e5a-34f5-41d2-a172-a6646a0e9c0d/deploy-status)](https://app.netlify.com/projects/ambient-music/deploys)

**Listen Live:** [https://ambient-music.netlify.app/](https://ambient-music.netlify.app/)

Finding the perfect background music for deep work, coding, studying, or just relaxing can be frustrating. Playlists eventually repeat, algorithms serve distracting tracks, and streaming services eat up bandwidth. 

**Aura** solves this by mathematically generating an infinite, non-repeating stream of lo-fi, synthwave, and ambient music directly in your browser. It never repeats, it never ends, and it requires absolutely zero internet bandwidth after the initial page load.

## Why Use Aura?
- 🎧 **Unbroken Focus:** Infinite, lyric-free music explicitly designed to keep you in the flow state.
- 📴 **100% Offline Capable:** Once loaded, the audio is generated entirely by your device's processor using the Web Audio API. No streaming required.
- 🔋 **Ultra-Lightweight:** Runs smoothly in a background tab without draining your battery or eating up RAM.
- 🎛️ **Choose Your Vibe:** 
  - **Calm**: Smooth Lo-Fi Chillhop for reading and studying.
  - **Deep**: Dark, driving Synthwave for coding and late-night focus.
  - **Ethereal**: Uplifting Dream Pop for creative thinking and relaxation.

## For Developers (Under the Hood)
Aura is a client-side, zero-backend application. It uses custom statistical sequencing and native browser APIs to synthesize rich soundscapes on the fly, entirely eliminating the need for audio files.

For a deep dive into the algorithmic sequencing, Web Audio API math, and application architecture, please read the [Technical Documentation](TECHNICAL_DOCS.md).

## Running Locally

To run this project on your own machine:

```bash
git clone https://github.com/AritraMalik804/ambient-music.git
cd ambient-music
npm install
npm run dev
```

## License & Attribution

This project is open-source and free to use. However, if you use, host, or distribute this application (or a modified version of it), **you must provide clear attribution and a link back to the original website**: 
👉 [https://ambient-music.netlify.app/](https://ambient-music.netlify.app/)

See the [LICENSE](LICENSE) file for exact terms.
