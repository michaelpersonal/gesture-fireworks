# Gesture Fireworks - Design Document

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:writing-plans to create implementation plan from this design.

**Goal:** Build a webcam-based web app where users create fireworks with hand gestures - pinch to charge, release to explode.

**Architecture:** Browser app using MediaPipe Hands for real-time hand tracking, HTML5 Canvas for rendering particles and overlays, Web Audio API for sound effects.

**Tech Stack:** Vanilla HTML/CSS/JavaScript, MediaPipe Hands, Canvas 2D, Web Audio API

---

## 1. Architecture Overview

### Core Flow
```
Camera Feed → MediaPipe Hands → Gesture Detection → Particle System → Canvas Render
                                      ↓
                               Sound Effects
```

### File Structure
```
firework/
├── index.html      # Main app (contains all CSS/JS)
├── sounds/
│   ├── charge.mp3  # Charging loop sound
│   └── explode.mp3 # Firework explosion
└── README.md       # How to run
```

### Key Components
1. **CameraManager** - Handles webcam access and video feed
2. **HandTracker** - MediaPipe integration, extracts hand landmarks
3. **GestureDetector** - Detects pinch gesture, tracks charge state per hand
4. **ParticleSystem** - Creates and animates firework particles
5. **Renderer** - Draws everything to canvas (video, skeleton, particles, UI)
6. **AudioManager** - Handles sound loading and playback

---

## 2. Gesture Detection & Charge Mechanic

### Pinch Gesture Detection

MediaPipe provides 21 landmarks per hand. For pinch detection:
- **Landmark 4** - Thumb tip
- **Landmark 8** - Index finger tip

```
Pinch detected when: distance(thumb_tip, index_tip) < threshold (~40px)
Pinch released when: distance > threshold
```

### Charge State Machine (Per Hand)

```
┌─────────┐  pinch   ┌──────────┐  release  ┌───────────┐
│  IDLE   │ ──────→  │ CHARGING │ ────────→ │ EXPLODE!  │
└─────────┘          └──────────┘           └───────────┘
     ↑                    │                       │
     └────────────────────┴───────────────────────┘
                    (reset after explosion)
```

### Charge Accumulation
- Charge builds over 1-2 seconds while pinching
- Visual feedback: hand skeleton pulses/glows brighter as charge increases
- Audio feedback: charging sound increases in intensity
- UI shows: "Left hand: Charging... 45%"

### Release Trigger
- When pinch releases AND charge > minimum threshold (~20%)
- Firework spawns at the midpoint between thumb and index finger
- Charge percentage affects firework size and particle count
- Full charge = bigger, more spectacular explosion

### Both Hands Independent
- Each hand has its own charge state
- Can charge both simultaneously for dual fireworks
- Different skeleton colors: Left = pink/red, Right = green/cyan

---

## 3. Particle System & Visual Effects

### Firework Explosion

When a firework triggers at position (x, y):

1. **Initial Burst** - 80-150 particles spawn at once
2. **Particle Properties** (each particle has):
   - Position (x, y)
   - Velocity (random direction, random speed 2-8px/frame)
   - Color (randomly picked from a palette)
   - Size (starts 3-5px, shrinks over time)
   - Life (1-2 seconds, then fades out)
   - Gravity (slight downward pull for realistic arc)

### Color Palettes
```javascript
const palettes = [
  ['#ff006e', '#fb5607', '#ffbe0b'],  // Warm (pink/orange/yellow)
  ['#00f5d4', '#00bbf9', '#9b5de5'],  // Cool (cyan/blue/purple)
  ['#ff006e', '#8338ec', '#3a86ff'],  // Neon (pink/purple/blue)
]
```

### Rendering Layers (bottom to top)
1. **Camera feed** - Slightly dimmed (opacity ~0.7) for contrast
2. **Hand skeletons** - 21 landmarks connected with lines
3. **Charge indicators** - Yellow circles on thumb/index during pinch
4. **Particles** - Firework explosions with glow effect
5. **UI overlay** - Status text in corner

### Hand Skeleton Drawing
- Lines connecting landmarks (palm, fingers)
- Small circles at each joint
- Left hand: Pink/red (#ff6b6b)
- Right hand: Cyan/green (#4ecdc4)
- Glow effect intensifies during charge

### Performance Target
- 60 FPS with up to 500 active particles
- Use `requestAnimationFrame` for smooth animation
- Object pooling to reduce garbage collection

---

## 4. Sound Effects & UI

### Sound Design

**Charge Sound** (`charge.mp3`)
- Low rumbling whoosh that builds in intensity
- Loops while charging, pitch/volume increases with charge %
- Cuts off when released

**Explosion Sound** (`explode.mp3`)
- Satisfying "pop" with sparkle tail
- Slight variation in pitch each time (0.9-1.1x) for variety
- Multiple can overlap (both hands exploding)

**Implementation**
- Web Audio API for precise control
- Preload sounds on first user interaction (browser requirement)
- Use AudioContext for pitch shifting and volume control
- Keep sounds short (<1s) for responsive feel

### UI Elements

**Top-left status panel**:
```
🎆 Gesture Fireworks
Left hand: 🔴 Charging... 67%    Right hand: ⚪ Waiting
Pinch and release to launch fireworks
```
- Title with emoji
- Per-hand status (Charging % / Waiting)
- Instruction hint
- Semi-transparent dark background for readability

**Top-right**: "Exit Fullscreen" button

**Center prompt** (on first load): "Click anywhere to start camera"

### Responsive Behavior
- Works on desktop and mobile (front camera)
- Canvas scales to fill viewport
- UI text scales with screen size
- Touch-friendly buttons (44px+ tap targets)

---

## 5. Error Handling

**Camera Access Denied**
- Show friendly message: "Camera access needed for hand tracking"
- Provide button to retry permission request

**No Hands Detected**
- After 3 seconds of no detection, show hint: "Show your hands to the camera 👋"
- Subtle pulsing animation to draw attention

**MediaPipe Load Failure**
- Fallback message: "Failed to load hand tracking. Please refresh."
- Log detailed error to console for debugging

**Low Performance**
- If FPS drops below 30, reduce max particles
- Disable glow effects if needed

---

## 6. How to Run

**Local Development**
```bash
cd firework
python -m http.server 8000
# Open http://localhost:8000
```

**Deployment Options**
- GitHub Pages (free, just push to repo)
- Netlify/Vercel (drag and drop)
- Any static file host

**Browser Support**
- Chrome, Edge, Firefox, Safari (latest versions)
- Mobile: Chrome/Safari on iOS and Android
- Requires: WebGL, getUserMedia, ES6 modules

---

## Summary

| Aspect | Decision |
|--------|----------|
| Platform | Web browser |
| Tech | Vanilla HTML/CSS/JS + MediaPipe |
| Features | Core + sound effects |
| Style | Dark theme, visible hand skeleton |
| Language | English only |
