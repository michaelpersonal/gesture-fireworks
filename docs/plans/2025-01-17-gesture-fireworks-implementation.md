# Gesture Fireworks Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a webcam-based web app where users create fireworks with hand gestures using MediaPipe Hands.

**Architecture:** Single HTML file with embedded CSS/JS. MediaPipe for hand tracking, Canvas for rendering, Web Audio for sounds. Progressive enhancement - each task adds one working feature.

**Tech Stack:** HTML5, CSS3, Vanilla JavaScript, MediaPipe Hands (CDN), Web Audio API

---

## Task 1: Project Setup and Basic HTML Structure

**Files:**
- Create: `index.html`
- Create: `README.md`

**Step 1: Create the base HTML file with canvas and video elements**

Create `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gesture Fireworks</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            background: #000;
            overflow: hidden;
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
        }

        #container {
            position: relative;
            width: 100vw;
            height: 100vh;
        }

        #video {
            display: none;
        }

        #canvas {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
        }

        #start-screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
            cursor: pointer;
            z-index: 10;
        }

        #start-screen h1 {
            font-size: 3rem;
            color: #fff;
            margin-bottom: 1rem;
        }

        #start-screen p {
            font-size: 1.2rem;
            color: #888;
        }

        #start-screen.hidden {
            display: none;
        }

        #ui-overlay {
            position: absolute;
            top: 20px;
            left: 20px;
            background: rgba(0, 0, 0, 0.7);
            padding: 15px 20px;
            border-radius: 10px;
            color: #fff;
            z-index: 5;
            opacity: 0;
            transition: opacity 0.3s;
        }

        #ui-overlay.visible {
            opacity: 1;
        }

        #ui-overlay h2 {
            font-size: 1.2rem;
            margin-bottom: 10px;
        }

        #ui-overlay .status {
            font-size: 0.9rem;
            margin-bottom: 5px;
        }

        #ui-overlay .hint {
            font-size: 0.8rem;
            color: #888;
            margin-top: 10px;
        }

        .status-indicator {
            display: inline-block;
            width: 10px;
            height: 10px;
            border-radius: 50%;
            margin-right: 8px;
        }

        .status-indicator.waiting {
            background: #666;
        }

        .status-indicator.charging {
            background: #ff6b6b;
            animation: pulse 0.5s ease-in-out infinite;
        }

        @keyframes pulse {
            0%, 100% { transform: scale(1); opacity: 1; }
            50% { transform: scale(1.3); opacity: 0.7; }
        }

        #error-message {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(255, 0, 0, 0.2);
            border: 1px solid #ff6b6b;
            padding: 20px 30px;
            border-radius: 10px;
            color: #fff;
            text-align: center;
            z-index: 20;
            display: none;
        }

        #error-message.visible {
            display: block;
        }
    </style>
</head>
<body>
    <div id="container">
        <video id="video" playsinline></video>
        <canvas id="canvas"></canvas>
        
        <div id="start-screen">
            <h1>🎆 Gesture Fireworks</h1>
            <p>Click anywhere to start</p>
        </div>

        <div id="ui-overlay">
            <h2>🎆 Gesture Fireworks</h2>
            <div class="status" id="left-hand-status">
                <span class="status-indicator waiting"></span>
                Left hand: Waiting
            </div>
            <div class="status" id="right-hand-status">
                <span class="status-indicator waiting"></span>
                Right hand: Waiting
            </div>
            <div class="hint">Pinch and release to launch fireworks</div>
        </div>

        <div id="error-message">
            <p>Camera access needed for hand tracking</p>
            <p style="margin-top: 10px; font-size: 0.9rem; color: #888;">Please allow camera access and refresh</p>
        </div>
    </div>

    <script>
        // App will be implemented in subsequent tasks
        console.log('Gesture Fireworks - Ready for implementation');
    </script>
</body>
</html>
```

**Step 2: Create README**

Create `README.md`:

```markdown
# 🎆 Gesture Fireworks

A webcam-based interactive web app where you create fireworks with hand gestures.

## How to Use

1. Open the app in a browser
2. Allow camera access
3. Show your hands to the camera
4. Pinch (thumb + index finger together) to charge
5. Release to launch fireworks!

## Running Locally

```bash
# Using Python
python -m http.server 8000

# Using Node.js
npx serve .
```

Then open http://localhost:8000

## Browser Support

- Chrome, Edge, Firefox, Safari (latest versions)
- Mobile: Chrome/Safari on iOS and Android

## Credits

Built with [MediaPipe Hands](https://google.github.io/mediapipe/solutions/hands.html)
```

**Step 3: Verify the page loads**

Run:
```bash
cd /Users/zhisongguo/code/firework
python3 -m http.server 8000
```

Open http://localhost:8000 in browser.

Expected: See dark gradient background with "🎆 Gesture Fireworks" title and "Click anywhere to start"

**Step 4: Commit**

```bash
git add index.html README.md
git commit -m "feat: add basic HTML structure and styling"
```

---

## Task 2: Camera Access and Video Feed

**Files:**
- Modify: `index.html`

**Step 1: Add camera initialization code**

Replace the `<script>` section in `index.html` with:

```javascript
<script>
    // ============================================
    // CAMERA MANAGER
    // ============================================
    class CameraManager {
        constructor(videoElement) {
            this.video = videoElement;
            this.stream = null;
        }

        async start() {
            try {
                this.stream = await navigator.mediaDevices.getUserMedia({
                    video: {
                        facingMode: 'user',
                        width: { ideal: 1280 },
                        height: { ideal: 720 }
                    }
                });
                this.video.srcObject = this.stream;
                await this.video.play();
                return true;
            } catch (error) {
                console.error('Camera error:', error);
                return false;
            }
        }

        getVideoDimensions() {
            return {
                width: this.video.videoWidth,
                height: this.video.videoHeight
            };
        }
    }

    // ============================================
    // RENDERER
    // ============================================
    class Renderer {
        constructor(canvas, video) {
            this.canvas = canvas;
            this.ctx = canvas.getContext('2d');
            this.video = video;
        }

        resize() {
            this.canvas.width = window.innerWidth;
            this.canvas.height = window.innerHeight;
        }

        clear() {
            this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
        }

        drawVideo() {
            const { width, height } = this.canvas;
            const videoAspect = this.video.videoWidth / this.video.videoHeight;
            const canvasAspect = width / height;

            let drawWidth, drawHeight, offsetX, offsetY;

            if (canvasAspect > videoAspect) {
                drawWidth = width;
                drawHeight = width / videoAspect;
                offsetX = 0;
                offsetY = (height - drawHeight) / 2;
            } else {
                drawHeight = height;
                drawWidth = height * videoAspect;
                offsetX = (width - drawWidth) / 2;
                offsetY = 0;
            }

            // Save context state
            this.ctx.save();
            
            // Mirror the video horizontally
            this.ctx.translate(width, 0);
            this.ctx.scale(-1, 1);

            // Draw dimmed video
            this.ctx.globalAlpha = 0.7;
            this.ctx.drawImage(this.video, offsetX, offsetY, drawWidth, drawHeight);
            this.ctx.globalAlpha = 1.0;

            // Restore context
            this.ctx.restore();

            // Store video bounds for coordinate conversion
            this.videoBounds = { offsetX, offsetY, drawWidth, drawHeight };
        }
    }

    // ============================================
    // MAIN APP
    // ============================================
    class GestureFireworksApp {
        constructor() {
            this.video = document.getElementById('video');
            this.canvas = document.getElementById('canvas');
            this.startScreen = document.getElementById('start-screen');
            this.uiOverlay = document.getElementById('ui-overlay');
            this.errorMessage = document.getElementById('error-message');

            this.camera = new CameraManager(this.video);
            this.renderer = new Renderer(this.canvas, this.video);

            this.isRunning = false;

            this.setupEventListeners();
        }

        setupEventListeners() {
            this.startScreen.addEventListener('click', () => this.start());
            window.addEventListener('resize', () => this.renderer.resize());
        }

        async start() {
            this.startScreen.classList.add('hidden');

            const success = await this.camera.start();
            if (!success) {
                this.errorMessage.classList.add('visible');
                return;
            }

            this.renderer.resize();
            this.uiOverlay.classList.add('visible');
            this.isRunning = true;
            this.loop();
        }

        loop() {
            if (!this.isRunning) return;

            this.renderer.clear();
            this.renderer.drawVideo();

            requestAnimationFrame(() => this.loop());
        }
    }

    // Initialize app
    const app = new GestureFireworksApp();
</script>
```

**Step 2: Verify camera works**

Run server and open http://localhost:8000

Click to start.

Expected: 
- Camera permission prompt appears
- After allowing, see mirrored webcam feed (slightly dimmed)
- UI overlay appears in top-left corner

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add camera access and video rendering"
```

---

## Task 3: MediaPipe Hand Tracking Integration

**Files:**
- Modify: `index.html`

**Step 1: Add MediaPipe script imports**

Add these lines inside `<head>`, before the `<style>` tag:

```html
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands@0.4.1675469240/hands.min.js" crossorigin="anonymous"></script>
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils@0.3.1675466862/camera_utils.min.js" crossorigin="anonymous"></script>
```

**Step 2: Add HandTracker class**

Add this class right after the CameraManager class in the script section:

```javascript
    // ============================================
    // HAND TRACKER
    // ============================================
    class HandTracker {
        constructor() {
            this.hands = new Hands({
                locateFile: (file) => {
                    return `https://cdn.jsdelivr.net/npm/@mediapipe/hands@0.4.1675469240/${file}`;
                }
            });

            this.hands.setOptions({
                maxNumHands: 2,
                modelComplexity: 1,
                minDetectionConfidence: 0.7,
                minTrackingConfidence: 0.5
            });

            this.latestResults = null;
            this.onResultsCallback = null;
        }

        async initialize() {
            return new Promise((resolve) => {
                this.hands.onResults((results) => {
                    this.latestResults = results;
                    if (this.onResultsCallback) {
                        this.onResultsCallback(results);
                    }
                });
                this.hands.initialize().then(() => {
                    console.log('MediaPipe Hands initialized');
                    resolve();
                });
            });
        }

        async detectHands(video) {
            await this.hands.send({ image: video });
        }

        onResults(callback) {
            this.onResultsCallback = callback;
        }
    }
```

**Step 3: Integrate HandTracker into the app**

Update the `GestureFireworksApp` class:

```javascript
    // ============================================
    // MAIN APP
    // ============================================
    class GestureFireworksApp {
        constructor() {
            this.video = document.getElementById('video');
            this.canvas = document.getElementById('canvas');
            this.startScreen = document.getElementById('start-screen');
            this.uiOverlay = document.getElementById('ui-overlay');
            this.errorMessage = document.getElementById('error-message');

            this.camera = new CameraManager(this.video);
            this.renderer = new Renderer(this.canvas, this.video);
            this.handTracker = new HandTracker();

            this.isRunning = false;
            this.handsData = null;

            this.setupEventListeners();
        }

        setupEventListeners() {
            this.startScreen.addEventListener('click', () => this.start());
            window.addEventListener('resize', () => this.renderer.resize());
        }

        async start() {
            this.startScreen.classList.add('hidden');

            // Initialize hand tracker
            await this.handTracker.initialize();

            this.handTracker.onResults((results) => {
                this.handsData = results;
            });

            const success = await this.camera.start();
            if (!success) {
                this.errorMessage.classList.add('visible');
                return;
            }

            this.renderer.resize();
            this.uiOverlay.classList.add('visible');
            this.isRunning = true;
            this.loop();
        }

        async loop() {
            if (!this.isRunning) return;

            // Send frame to hand tracker
            await this.handTracker.detectHands(this.video);

            this.renderer.clear();
            this.renderer.drawVideo();

            // Log hand data for debugging
            if (this.handsData && this.handsData.multiHandLandmarks) {
                console.log(`Detected ${this.handsData.multiHandLandmarks.length} hand(s)`);
            }

            requestAnimationFrame(() => this.loop());
        }
    }
```

**Step 4: Verify hand tracking works**

Run server and open http://localhost:8000

Click to start, allow camera, show hands.

Expected:
- Console logs "MediaPipe Hands initialized"
- Console logs "Detected 1 hand(s)" or "Detected 2 hand(s)" when hands visible

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: integrate MediaPipe hand tracking"
```

---

## Task 4: Hand Skeleton Rendering

**Files:**
- Modify: `index.html`

**Step 1: Add skeleton drawing to Renderer class**

Add these methods to the `Renderer` class:

```javascript
        // Hand landmark connections for drawing skeleton
        static HAND_CONNECTIONS = [
            [0, 1], [1, 2], [2, 3], [3, 4],           // Thumb
            [0, 5], [5, 6], [6, 7], [7, 8],           // Index
            [0, 9], [9, 10], [10, 11], [11, 12],      // Middle
            [0, 13], [13, 14], [14, 15], [15, 16],    // Ring
            [0, 17], [17, 18], [18, 19], [19, 20],    // Pinky
            [5, 9], [9, 13], [13, 17]                  // Palm
        ];

        convertLandmarkToCanvas(landmark) {
            if (!this.videoBounds) return { x: 0, y: 0 };
            
            const { offsetX, offsetY, drawWidth, drawHeight } = this.videoBounds;
            
            // Mirror X coordinate (since video is mirrored)
            const x = this.canvas.width - (offsetX + landmark.x * drawWidth);
            const y = offsetY + landmark.y * drawHeight;
            
            return { x, y };
        }

        drawHandSkeleton(landmarks, isLeftHand, chargePercent = 0) {
            const baseColor = isLeftHand ? '#ff6b6b' : '#4ecdc4';
            const glowIntensity = chargePercent / 100;

            // Draw connections
            this.ctx.strokeStyle = baseColor;
            this.ctx.lineWidth = 3;
            this.ctx.lineCap = 'round';

            // Add glow effect
            if (glowIntensity > 0) {
                this.ctx.shadowColor = baseColor;
                this.ctx.shadowBlur = 10 + glowIntensity * 20;
            }

            for (const [start, end] of Renderer.HAND_CONNECTIONS) {
                const startPoint = this.convertLandmarkToCanvas(landmarks[start]);
                const endPoint = this.convertLandmarkToCanvas(landmarks[end]);

                this.ctx.beginPath();
                this.ctx.moveTo(startPoint.x, startPoint.y);
                this.ctx.lineTo(endPoint.x, endPoint.y);
                this.ctx.stroke();
            }

            // Draw landmarks
            for (let i = 0; i < landmarks.length; i++) {
                const point = this.convertLandmarkToCanvas(landmarks[i]);
                
                this.ctx.beginPath();
                this.ctx.arc(point.x, point.y, 5, 0, Math.PI * 2);
                this.ctx.fillStyle = baseColor;
                this.ctx.fill();
            }

            // Reset shadow
            this.ctx.shadowBlur = 0;
        }

        drawPinchIndicator(thumbTip, indexTip, isCharging) {
            const thumb = this.convertLandmarkToCanvas(thumbTip);
            const index = this.convertLandmarkToCanvas(indexTip);

            // Draw circles around thumb and index
            const color = isCharging ? '#ffd93d' : '#ffffff';
            
            this.ctx.strokeStyle = color;
            this.ctx.lineWidth = 3;
            this.ctx.shadowColor = color;
            this.ctx.shadowBlur = isCharging ? 15 : 5;

            // Thumb circle
            this.ctx.beginPath();
            this.ctx.arc(thumb.x, thumb.y, 15, 0, Math.PI * 2);
            this.ctx.stroke();

            // Index circle
            this.ctx.beginPath();
            this.ctx.arc(index.x, index.y, 15, 0, Math.PI * 2);
            this.ctx.stroke();

            // Line between them when charging
            if (isCharging) {
                this.ctx.beginPath();
                this.ctx.moveTo(thumb.x, thumb.y);
                this.ctx.lineTo(index.x, index.y);
                this.ctx.stroke();
            }

            this.ctx.shadowBlur = 0;
        }
```

**Step 2: Update the main loop to draw hands**

Update the `loop` method in `GestureFireworksApp`:

```javascript
        async loop() {
            if (!this.isRunning) return;

            // Send frame to hand tracker
            await this.handTracker.detectHands(this.video);

            this.renderer.clear();
            this.renderer.drawVideo();

            // Draw hand skeletons
            if (this.handsData && this.handsData.multiHandLandmarks) {
                for (let i = 0; i < this.handsData.multiHandLandmarks.length; i++) {
                    const landmarks = this.handsData.multiHandLandmarks[i];
                    const handedness = this.handsData.multiHandedness[i];
                    
                    // MediaPipe reports handedness from camera's perspective
                    // Since we mirror the video, we flip the label
                    const isLeftHand = handedness.label === 'Right';
                    
                    this.renderer.drawHandSkeleton(landmarks, isLeftHand, 0);
                    this.renderer.drawPinchIndicator(
                        landmarks[4],  // Thumb tip
                        landmarks[8],  // Index tip
                        false
                    );
                }
            }

            requestAnimationFrame(() => this.loop());
        }
```

**Step 3: Verify skeleton rendering**

Run server, click to start, show hands to camera.

Expected:
- Hand skeleton visible with connected lines and dots
- Left hand shows pink/red skeleton
- Right hand shows cyan/green skeleton
- Yellow/white circles around thumb and index finger tips

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add hand skeleton rendering"
```

---

## Task 5: Gesture Detection and Charge State

**Files:**
- Modify: `index.html`

**Step 1: Add GestureDetector class**

Add this class after the `HandTracker` class:

```javascript
    // ============================================
    // GESTURE DETECTOR
    // ============================================
    class GestureDetector {
        constructor() {
            this.PINCH_THRESHOLD = 0.08; // Normalized distance
            this.CHARGE_RATE = 0.02;     // Charge per frame
            this.MIN_CHARGE = 0.2;       // Minimum to trigger firework

            this.leftHand = { charging: false, charge: 0, wasPinching: false };
            this.rightHand = { charging: false, charge: 0, wasPinching: false };
        }

        calculatePinchDistance(landmarks) {
            const thumbTip = landmarks[4];
            const indexTip = landmarks[8];
            
            const dx = thumbTip.x - indexTip.x;
            const dy = thumbTip.y - indexTip.y;
            const dz = thumbTip.z - indexTip.z;
            
            return Math.sqrt(dx * dx + dy * dy + dz * dz);
        }

        getPinchPosition(landmarks) {
            const thumbTip = landmarks[4];
            const indexTip = landmarks[8];
            
            return {
                x: (thumbTip.x + indexTip.x) / 2,
                y: (thumbTip.y + indexTip.y) / 2,
                z: (thumbTip.z + indexTip.z) / 2
            };
        }

        update(handsData) {
            const events = [];

            // Reset tracking for hands not detected
            let leftDetected = false;
            let rightDetected = false;

            if (handsData && handsData.multiHandLandmarks) {
                for (let i = 0; i < handsData.multiHandLandmarks.length; i++) {
                    const landmarks = handsData.multiHandLandmarks[i];
                    const handedness = handsData.multiHandedness[i];
                    
                    // Flip due to mirrored video
                    const isLeftHand = handedness.label === 'Right';
                    const handState = isLeftHand ? this.leftHand : this.rightHand;
                    
                    if (isLeftHand) leftDetected = true;
                    else rightDetected = true;

                    const pinchDistance = this.calculatePinchDistance(landmarks);
                    const isPinching = pinchDistance < this.PINCH_THRESHOLD;

                    if (isPinching) {
                        // Charging
                        handState.charging = true;
                        handState.charge = Math.min(1, handState.charge + this.CHARGE_RATE);
                    } else if (handState.wasPinching && handState.charge >= this.MIN_CHARGE) {
                        // Released with enough charge - fire!
                        events.push({
                            type: 'firework',
                            hand: isLeftHand ? 'left' : 'right',
                            position: this.getPinchPosition(landmarks),
                            charge: handState.charge,
                            landmarks: landmarks
                        });
                        handState.charge = 0;
                        handState.charging = false;
                    } else {
                        // Not pinching, decay charge
                        handState.charge = Math.max(0, handState.charge - this.CHARGE_RATE * 0.5);
                        handState.charging = false;
                    }

                    handState.wasPinching = isPinching;
                }
            }

            // Reset hands that are not detected
            if (!leftDetected) {
                this.leftHand.charging = false;
                this.leftHand.charge = 0;
                this.leftHand.wasPinching = false;
            }
            if (!rightDetected) {
                this.rightHand.charging = false;
                this.rightHand.charge = 0;
                this.rightHand.wasPinching = false;
            }

            return events;
        }

        getHandState(isLeftHand) {
            return isLeftHand ? this.leftHand : this.rightHand;
        }
    }
```

**Step 2: Add UI update method**

Add this method to `GestureFireworksApp`:

```javascript
        updateUI() {
            const leftStatus = document.getElementById('left-hand-status');
            const rightStatus = document.getElementById('right-hand-status');

            const updateHand = (element, state, label) => {
                const indicator = element.querySelector('.status-indicator');
                if (state.charging) {
                    const percent = Math.round(state.charge * 100);
                    element.innerHTML = `
                        <span class="status-indicator charging"></span>
                        ${label}: Charging... ${percent}%
                    `;
                } else {
                    element.innerHTML = `
                        <span class="status-indicator waiting"></span>
                        ${label}: Waiting
                    `;
                }
            };

            updateHand(leftStatus, this.gestureDetector.leftHand, 'Left hand');
            updateHand(rightStatus, this.gestureDetector.rightHand, 'Right hand');
        }
```

**Step 3: Integrate gesture detection into the app**

Update the `GestureFireworksApp` constructor and methods:

In constructor, add:
```javascript
            this.gestureDetector = new GestureDetector();
```

Update the `loop` method:

```javascript
        async loop() {
            if (!this.isRunning) return;

            // Send frame to hand tracker
            await this.handTracker.detectHands(this.video);

            // Update gesture detection
            const events = this.gestureDetector.update(this.handsData);

            // Handle events (fireworks - implemented in next task)
            for (const event of events) {
                console.log('Firework triggered!', event.hand, 'charge:', event.charge);
            }

            this.renderer.clear();
            this.renderer.drawVideo();

            // Draw hand skeletons with charge effect
            if (this.handsData && this.handsData.multiHandLandmarks) {
                for (let i = 0; i < this.handsData.multiHandLandmarks.length; i++) {
                    const landmarks = this.handsData.multiHandLandmarks[i];
                    const handedness = this.handsData.multiHandedness[i];
                    const isLeftHand = handedness.label === 'Right';
                    
                    const handState = this.gestureDetector.getHandState(isLeftHand);
                    const chargePercent = handState.charge * 100;
                    
                    this.renderer.drawHandSkeleton(landmarks, isLeftHand, chargePercent);
                    this.renderer.drawPinchIndicator(
                        landmarks[4],
                        landmarks[8],
                        handState.charging
                    );
                }
            }

            // Update UI
            this.updateUI();

            requestAnimationFrame(() => this.loop());
        }
```

**Step 4: Verify gesture detection**

Run server, click to start, pinch fingers together.

Expected:
- UI updates to show "Charging... X%"
- Hand skeleton glows brighter while charging
- Yellow circles appear around pinch fingers
- Console logs "Firework triggered!" when releasing after charging

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add gesture detection and charge state"
```

---

## Task 6: Particle System

**Files:**
- Modify: `index.html`

**Step 1: Add ParticleSystem class**

Add this class after `GestureDetector`:

```javascript
    // ============================================
    // PARTICLE SYSTEM
    // ============================================
    class ParticleSystem {
        constructor() {
            this.particles = [];
            this.gravity = 0.15;
            this.maxParticles = 500;

            this.palettes = [
                ['#ff006e', '#fb5607', '#ffbe0b', '#ff006e'],  // Warm
                ['#00f5d4', '#00bbf9', '#9b5de5', '#00f5d4'],  // Cool
                ['#ff006e', '#8338ec', '#3a86ff', '#ff006e'],  // Neon
                ['#06d6a0', '#118ab2', '#073b4c', '#ffd166'],  // Ocean
            ];
        }

        createFirework(x, y, charge) {
            const particleCount = Math.floor(80 + charge * 70);
            const palette = this.palettes[Math.floor(Math.random() * this.palettes.length)];
            const baseSpeed = 3 + charge * 5;

            for (let i = 0; i < particleCount; i++) {
                if (this.particles.length >= this.maxParticles) break;

                const angle = (Math.PI * 2 * i) / particleCount + (Math.random() - 0.5) * 0.5;
                const speed = baseSpeed * (0.5 + Math.random() * 0.5);

                this.particles.push({
                    x: x,
                    y: y,
                    vx: Math.cos(angle) * speed,
                    vy: Math.sin(angle) * speed,
                    color: palette[Math.floor(Math.random() * palette.length)],
                    size: 2 + Math.random() * 3,
                    life: 1,
                    decay: 0.01 + Math.random() * 0.015,
                    gravity: this.gravity * (0.8 + Math.random() * 0.4)
                });
            }
        }

        update() {
            for (let i = this.particles.length - 1; i >= 0; i--) {
                const p = this.particles[i];

                // Update position
                p.x += p.vx;
                p.y += p.vy;
                p.vy += p.gravity;

                // Apply drag
                p.vx *= 0.98;
                p.vy *= 0.98;

                // Decrease life
                p.life -= p.decay;
                p.size *= 0.98;

                // Remove dead particles
                if (p.life <= 0 || p.size < 0.5) {
                    this.particles.splice(i, 1);
                }
            }
        }

        draw(ctx) {
            ctx.save();
            
            for (const p of this.particles) {
                ctx.globalAlpha = p.life;
                ctx.fillStyle = p.color;
                
                // Draw with glow
                ctx.shadowColor = p.color;
                ctx.shadowBlur = 10;
                
                ctx.beginPath();
                ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
                ctx.fill();
            }
            
            ctx.restore();
        }

        clear() {
            this.particles = [];
        }
    }
```

**Step 2: Add particle drawing method to Renderer**

Add this method to the `Renderer` class:

```javascript
        drawParticles(particleSystem) {
            particleSystem.draw(this.ctx);
        }
```

**Step 3: Integrate particles into the app**

In `GestureFireworksApp` constructor, add:
```javascript
            this.particleSystem = new ParticleSystem();
```

Update the event handling and render in the `loop` method:

```javascript
        async loop() {
            if (!this.isRunning) return;

            // Send frame to hand tracker
            await this.handTracker.detectHands(this.video);

            // Update gesture detection
            const events = this.gestureDetector.update(this.handsData);

            // Handle firework events
            for (const event of events) {
                if (event.type === 'firework') {
                    // Convert normalized position to canvas coordinates
                    const canvasPos = this.renderer.convertLandmarkToCanvas(event.position);
                    this.particleSystem.createFirework(canvasPos.x, canvasPos.y, event.charge);
                }
            }

            // Update particles
            this.particleSystem.update();

            this.renderer.clear();
            this.renderer.drawVideo();

            // Draw hand skeletons with charge effect
            if (this.handsData && this.handsData.multiHandLandmarks) {
                for (let i = 0; i < this.handsData.multiHandLandmarks.length; i++) {
                    const landmarks = this.handsData.multiHandLandmarks[i];
                    const handedness = this.handsData.multiHandedness[i];
                    const isLeftHand = handedness.label === 'Right';
                    
                    const handState = this.gestureDetector.getHandState(isLeftHand);
                    const chargePercent = handState.charge * 100;
                    
                    this.renderer.drawHandSkeleton(landmarks, isLeftHand, chargePercent);
                    this.renderer.drawPinchIndicator(
                        landmarks[4],
                        landmarks[8],
                        handState.charging
                    );
                }
            }

            // Draw particles on top
            this.renderer.drawParticles(this.particleSystem);

            // Update UI
            this.updateUI();

            requestAnimationFrame(() => this.loop());
        }
```

**Step 4: Verify fireworks work**

Run server, click to start, pinch and release.

Expected:
- When releasing a charged pinch, colorful particles explode from the pinch point
- Particles fall with gravity and fade out
- Multiple colors in each explosion
- Works with both hands

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add particle system and firework explosions"
```

---

## Task 7: Sound Effects

**Files:**
- Create: `sounds/` directory with audio files
- Modify: `index.html`

**Step 1: Create sound files**

Since we can't generate actual audio files, we'll use Web Audio API to synthesize sounds:

Add the `AudioManager` class after `ParticleSystem`:

```javascript
    // ============================================
    // AUDIO MANAGER (Synthesized sounds)
    // ============================================
    class AudioManager {
        constructor() {
            this.audioContext = null;
            this.initialized = false;
            this.chargeSources = new Map(); // Track charging sounds per hand
        }

        async initialize() {
            if (this.initialized) return;
            
            try {
                this.audioContext = new (window.AudioContext || window.webkitAudioContext)();
                this.initialized = true;
                console.log('Audio initialized');
            } catch (error) {
                console.warn('Audio not available:', error);
            }
        }

        playCharge(handId) {
            if (!this.audioContext || this.chargeSources.has(handId)) return;

            const oscillator = this.audioContext.createOscillator();
            const gainNode = this.audioContext.createGain();

            oscillator.type = 'sine';
            oscillator.frequency.setValueAtTime(100, this.audioContext.currentTime);
            oscillator.frequency.linearRampToValueAtTime(300, this.audioContext.currentTime + 2);

            gainNode.gain.setValueAtTime(0, this.audioContext.currentTime);
            gainNode.gain.linearRampToValueAtTime(0.15, this.audioContext.currentTime + 0.1);

            oscillator.connect(gainNode);
            gainNode.connect(this.audioContext.destination);

            oscillator.start();

            this.chargeSources.set(handId, { oscillator, gainNode });
        }

        stopCharge(handId) {
            const source = this.chargeSources.get(handId);
            if (source) {
                source.gainNode.gain.linearRampToValueAtTime(0, this.audioContext.currentTime + 0.1);
                setTimeout(() => {
                    source.oscillator.stop();
                }, 150);
                this.chargeSources.delete(handId);
            }
        }

        updateChargeIntensity(handId, charge) {
            const source = this.chargeSources.get(handId);
            if (source) {
                // Increase volume and pitch with charge
                source.gainNode.gain.setValueAtTime(0.1 + charge * 0.2, this.audioContext.currentTime);
                source.oscillator.frequency.setValueAtTime(100 + charge * 300, this.audioContext.currentTime);
            }
        }

        playExplosion(intensity = 1) {
            if (!this.audioContext) return;

            // Create noise burst
            const bufferSize = this.audioContext.sampleRate * 0.3;
            const buffer = this.audioContext.createBuffer(1, bufferSize, this.audioContext.sampleRate);
            const data = buffer.getChannelData(0);

            for (let i = 0; i < bufferSize; i++) {
                data[i] = (Math.random() * 2 - 1) * Math.exp(-i / (bufferSize * 0.1));
            }

            const noise = this.audioContext.createBufferSource();
            noise.buffer = buffer;

            // Filter for more "pop" sound
            const filter = this.audioContext.createBiquadFilter();
            filter.type = 'lowpass';
            filter.frequency.setValueAtTime(1000 + intensity * 2000, this.audioContext.currentTime);
            filter.frequency.linearRampToValueAtTime(200, this.audioContext.currentTime + 0.3);

            const gainNode = this.audioContext.createGain();
            gainNode.gain.setValueAtTime(0.3 * intensity, this.audioContext.currentTime);
            gainNode.gain.linearRampToValueAtTime(0, this.audioContext.currentTime + 0.3);

            noise.connect(filter);
            filter.connect(gainNode);
            gainNode.connect(this.audioContext.destination);

            noise.start();

            // Add a "sparkle" high frequency
            const sparkle = this.audioContext.createOscillator();
            sparkle.type = 'sine';
            sparkle.frequency.setValueAtTime(2000 + Math.random() * 1000, this.audioContext.currentTime);
            sparkle.frequency.linearRampToValueAtTime(500, this.audioContext.currentTime + 0.2);

            const sparkleGain = this.audioContext.createGain();
            sparkleGain.gain.setValueAtTime(0.1 * intensity, this.audioContext.currentTime);
            sparkleGain.gain.linearRampToValueAtTime(0, this.audioContext.currentTime + 0.2);

            sparkle.connect(sparkleGain);
            sparkleGain.connect(this.audioContext.destination);

            sparkle.start();
            sparkle.stop(this.audioContext.currentTime + 0.3);
        }
    }
```

**Step 2: Integrate audio into the app**

In `GestureFireworksApp` constructor, add:
```javascript
            this.audioManager = new AudioManager();
```

Update the `start` method to initialize audio:
```javascript
        async start() {
            this.startScreen.classList.add('hidden');

            // Initialize audio (requires user interaction)
            await this.audioManager.initialize();

            // Initialize hand tracker
            await this.handTracker.initialize();

            this.handTracker.onResults((results) => {
                this.handsData = results;
            });

            const success = await this.camera.start();
            if (!success) {
                this.errorMessage.classList.add('visible');
                return;
            }

            this.renderer.resize();
            this.uiOverlay.classList.add('visible');
            this.isRunning = true;
            this.loop();
        }
```

Update the `loop` method to handle audio:

```javascript
        async loop() {
            if (!this.isRunning) return;

            // Send frame to hand tracker
            await this.handTracker.detectHands(this.video);

            // Update gesture detection
            const events = this.gestureDetector.update(this.handsData);

            // Handle charge sounds
            for (const hand of ['left', 'right']) {
                const state = hand === 'left' ? this.gestureDetector.leftHand : this.gestureDetector.rightHand;
                
                if (state.charging && state.charge > 0.05) {
                    this.audioManager.playCharge(hand);
                    this.audioManager.updateChargeIntensity(hand, state.charge);
                } else {
                    this.audioManager.stopCharge(hand);
                }
            }

            // Handle firework events
            for (const event of events) {
                if (event.type === 'firework') {
                    const canvasPos = this.renderer.convertLandmarkToCanvas(event.position);
                    this.particleSystem.createFirework(canvasPos.x, canvasPos.y, event.charge);
                    this.audioManager.playExplosion(event.charge);
                }
            }

            // Update particles
            this.particleSystem.update();

            this.renderer.clear();
            this.renderer.drawVideo();

            // Draw hand skeletons with charge effect
            if (this.handsData && this.handsData.multiHandLandmarks) {
                for (let i = 0; i < this.handsData.multiHandLandmarks.length; i++) {
                    const landmarks = this.handsData.multiHandLandmarks[i];
                    const handedness = this.handsData.multiHandedness[i];
                    const isLeftHand = handedness.label === 'Right';
                    
                    const handState = this.gestureDetector.getHandState(isLeftHand);
                    const chargePercent = handState.charge * 100;
                    
                    this.renderer.drawHandSkeleton(landmarks, isLeftHand, chargePercent);
                    this.renderer.drawPinchIndicator(
                        landmarks[4],
                        landmarks[8],
                        handState.charging
                    );
                }
            }

            // Draw particles on top
            this.renderer.drawParticles(this.particleSystem);

            // Update UI
            this.updateUI();

            requestAnimationFrame(() => this.loop());
        }
```

**Step 3: Verify sound works**

Run server, click to start, pinch and release.

Expected:
- Hear rising tone while charging (pitch and volume increase)
- Hear "pop" with sparkle when firework explodes
- Sounds vary slightly each time

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add synthesized sound effects"
```

---

## Task 8: Polish and Error Handling

**Files:**
- Modify: `index.html`

**Step 1: Add "no hands detected" hint**

Add these properties to `GestureFireworksApp` constructor:
```javascript
            this.lastHandsDetectedTime = 0;
            this.showHandsHint = false;
```

Add this method to `GestureFireworksApp`:
```javascript
        checkHandsHint() {
            const now = Date.now();
            const hasHands = this.handsData && 
                            this.handsData.multiHandLandmarks && 
                            this.handsData.multiHandLandmarks.length > 0;

            if (hasHands) {
                this.lastHandsDetectedTime = now;
                this.showHandsHint = false;
            } else if (now - this.lastHandsDetectedTime > 3000) {
                this.showHandsHint = true;
            }
        }
```

Add hint drawing to `Renderer` class:
```javascript
        drawHint(message) {
            const x = this.canvas.width / 2;
            const y = this.canvas.height / 2 + 100;

            this.ctx.save();
            this.ctx.font = '24px -apple-system, BlinkMacSystemFont, sans-serif';
            this.ctx.textAlign = 'center';
            this.ctx.fillStyle = 'rgba(255, 255, 255, 0.8)';
            
            // Pulsing animation
            const pulse = 0.7 + 0.3 * Math.sin(Date.now() / 300);
            this.ctx.globalAlpha = pulse;
            
            this.ctx.fillText(message, x, y);
            this.ctx.restore();
        }
```

Update the `loop` method to show hint:

After `this.updateUI();` add:
```javascript
            // Check and show hands hint
            this.checkHandsHint();
            if (this.showHandsHint) {
                this.renderer.drawHint('👋 Show your hands to the camera');
            }
```

Also update the `start` method to initialize the timer:
```javascript
            this.lastHandsDetectedTime = Date.now();
```

**Step 2: Add FPS monitoring for performance**

Add to `GestureFireworksApp` constructor:
```javascript
            this.frameCount = 0;
            this.lastFpsUpdate = 0;
            this.fps = 60;
```

Add FPS tracking in the `loop` method (at the start):
```javascript
            // FPS tracking
            this.frameCount++;
            const now = Date.now();
            if (now - this.lastFpsUpdate > 1000) {
                this.fps = this.frameCount;
                this.frameCount = 0;
                this.lastFpsUpdate = now;
                
                // Reduce particles if performance is poor
                if (this.fps < 30) {
                    this.particleSystem.maxParticles = Math.max(200, this.particleSystem.maxParticles - 50);
                }
            }
```

**Step 3: Verify everything works**

Run server and test:
- Start the app
- Wait 3 seconds without showing hands → hint appears
- Show hands → hint disappears
- Pinch and release → fireworks with sound
- Both hands work independently
- UI shows correct status

**Step 4: Final commit**

```bash
git add index.html
git commit -m "feat: add polish - hands hint, FPS monitoring, performance optimization"
```

---

## Summary

After completing all tasks, you will have:

| Component | Status |
|-----------|--------|
| Camera access | ✅ |
| Hand tracking | ✅ |
| Skeleton rendering | ✅ |
| Pinch gesture detection | ✅ |
| Charge state machine | ✅ |
| Particle system | ✅ |
| Sound effects | ✅ |
| UI overlay | ✅ |
| Error handling | ✅ |

**To test the complete app:**
```bash
cd /Users/zhisongguo/code/firework
python3 -m http.server 8000
# Open http://localhost:8000
```
