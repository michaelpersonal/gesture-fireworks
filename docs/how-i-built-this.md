# How I Built Gesture Fireworks

## The Idea

I saw a cool hand-gesture fireworks app on X/Twitter and wanted to recreate it. The original was by [@libukai](https://x.com/libukai/status/2012466547399487584) - a webcam app where you pinch your fingers to launch fireworks.

I wanted to build my own version using vibe coding with an AI agent.

## Tech Stack Decisions

| Choice | Why |
|--------|-----|
| **Vanilla JavaScript** | No framework overhead, runs anywhere |
| **MediaPipe Hands** | Google's ML library for hand tracking - accurate and fast |
| **HTML5 Canvas** | Hardware-accelerated rendering for particles |
| **Web Audio API** | Synthesized sounds without external files |
| **Single HTML file** | Easy to deploy, no build step |

## The Build Process

### Phase 1: Setup & Brainstorming

I started by setting up the AI workflow rules, then kicked off brainstorming with a simple prompt:

> "I'd like to create an application like this one [link to X post]"

The AI asked one question at a time:
- Web app, mobile, or desktop? → **Web**
- Vanilla JS, React, or Vue? → **Vanilla**
- Sound effects? → **Yes**
- Match original visual style? → **Yes**

This took about 5 minutes of picking options.

### Phase 2: Core Implementation

The AI created a detailed design doc, then built:
1. Camera access and video rendering
2. MediaPipe hand tracking integration
3. Gesture detection (pinch = charge, release = explode)
4. Basic particle system
5. Sound effects

First working version was ready in ~30 minutes.

### Phase 3: Polish & Iteration

I gave feedback:

> "improvements: 1) the fireworks too small, make it bigger and more explosive; 2) right hand doesn't seem as responsive; 3) remove the noise while hands charging"

The AI made each fix. When the right-hand issue persisted, I asked:

> "can you double check why right hand doesn't behave the same as left hand"

This triggered deeper debugging - turned out MediaPipe's handedness labels were unreliable. The fix was to use wrist position instead.

### Phase 4: Visual Upgrade

I wasn't happy with the basic fireworks:

> "can you research if there is a better fireworks library, your currently is a little boring, not as colorful"

The AI presented 3 options. We upgraded the custom particle system with:
- 8 vibrant color palettes
- Trails and sparkles
- Glow effects
- Additive blending

### Phase 5: Audio & Deployment

> "great now add background music /Downloads/684.mp3"

Then deployment:

> "deploy this app to vercel (reuse my website if possible), change my personal website to add this project link"

Hit some issues with audio paths in production - the AI debugged systematically.

### Phase 6: Rocket Launch Feature

Final feature request:

> "now is it possible if I move my hand from bottom to top, you can simulate a fireworks shoot out and pop in the sky?"

The first attempt wasn't right. I clarified:

> "I would think you first need a firework on the ground, light up, then shoot up, then explode"

We brainstormed and I suggested:

> "maybe left hand pinch trigger this, but right hand keeps the old way"

This became the final design: right hand = instant explosion, left hand = ground-launched rocket.

## Iterations & Improvements

| Feedback I Gave | What Changed |
|-----------------|--------------|
| "fireworks too small" | Increased particle count and spread |
| "right hand not responsive" | Fixed handedness detection using wrist position |
| "remove charging noise" | Removed the charge sound effect |
| "boring, not colorful" | Added 8 color palettes, trails, glow |
| "rocket should light up first" | Added ignition sequence with fuse sound |

## Challenges & Solutions

### Challenge 1: Hand Detection Reliability
**Problem:** Right hand wasn't being detected as reliably as left hand.
**Solution:** MediaPipe's `handedness.label` was unreliable. Switched to using wrist X-coordinate to determine which hand is which.

### Challenge 2: Audio in Production
**Problem:** Background music worked locally but not on Vercel.
**Solution:** Fixed relative paths (`background.mp3` → `/fireworks/background.mp3`) and handled browser autoplay policies with `audioContext.resume()`.

### Challenge 3: Rocket Launch Sequence
**Problem:** Initial implementation just exploded at hand position.
**Solution:** Created proper `Rocket` class with ignition → ascent → explosion phases.

## Key Prompts That Worked

| Prompt | Why It Worked |
|--------|---------------|
| `"I'd like to create an application like this one [link]"` | Started with concrete inspiration |
| `"improvements: 1) ... 2) ... 3) ..."` | Numbered feedback is clear and actionable |
| `"can you double check why..."` | Asked for investigation, not just fixes |
| `"can you research better libraries"` | Let AI explore options |
| `"maybe left hand does X, right hand does Y"` | Proposed my own solution during brainstorm |

## Lessons Learned

1. **Start with inspiration** - Sharing a link/video beats describing from scratch
2. **Short answers work** - Just pick `a`, `b`, or `yes` during brainstorming
3. **Numbered feedback** - Makes issues clear and trackable
4. **Ask for investigation** - "double check why" triggers deeper analysis
5. **Propose ideas during brainstorming** - I'm the creative director, AI executes
6. **Iterate fast** - Try it, give feedback, improve - don't over-plan

## Project Links

- **Live Demo:** https://michaelguo.vercel.app/fireworks/
- **GitHub:** https://github.com/michaelpersonal/gesture-fireworks
