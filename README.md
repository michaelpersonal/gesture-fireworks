# 🎆 Gesture Fireworks

Create stunning fireworks with hand gestures using your webcam! Features real-time hand tracking, colorful particle explosions with trails & glow effects, synthesized sound effects, and background music.

**Pinch your fingers to charge, release to launch!**

## 🚀 [Live Demo](https://michaelguo.vercel.app/fireworks/)

![Demo](https://img.shields.io/badge/demo-live-brightgreen) ![JavaScript](https://img.shields.io/badge/javascript-vanilla-yellow) ![MediaPipe](https://img.shields.io/badge/mediapipe-hands-blue)

## ✨ Features

- 🖐️ **Real-time hand tracking** - Uses MediaPipe Hands for accurate gesture detection
- 🎨 **8 vibrant color palettes** - Golden, Electric Blue, Pink Paradise, Green Aurora, Purple Galaxy, Rainbow, Fire Storm, Ice Crystal
- ✨ **Advanced particle effects** - Trails, sparkles, glow, additive blending
- 🔊 **Synthesized sound effects** - Explosion pops with sparkle sounds
- 🎵 **Background music** - Immersive audio experience
- 📱 **Works on mobile** - Use your phone's front camera
- 🚀 **No installation** - Runs directly in the browser

## 🎮 How to Use

1. Open the app in a modern browser
2. Click anywhere to start
3. Allow camera access when prompted
4. **Pinch** your thumb and index finger together to charge
5. **Release** to launch fireworks!
6. Use both hands for double the fun! 🎆🎆

## 🛠️ Tech Stack

- **Vanilla JavaScript** - No frameworks, just clean code
- **MediaPipe Hands** - Google's hand tracking ML model
- **HTML5 Canvas** - Hardware-accelerated 2D rendering
- **Web Audio API** - Synthesized sound effects

## 🚀 Running Locally

```bash
# Clone the repository
git clone https://github.com/michaelpersonal/gesture-fireworks.git
cd gesture-fireworks

# Start a local server (required for MediaPipe)
python -m http.server 8000

# Or using Node.js
npx serve .
```

Then open http://localhost:8000 in your browser.

## 🌐 Browser Support

| Browser | Support |
|---------|---------|
| Chrome | ✅ Full |
| Edge | ✅ Full |
| Firefox | ✅ Full |
| Safari | ✅ Full |
| Mobile Chrome | ✅ Full |
| Mobile Safari | ✅ Full |

Requires: WebGL, getUserMedia (camera), ES6 modules

## 📁 Project Structure

```
gesture-fireworks/
├── index.html      # Main app (all-in-one HTML/CSS/JS)
├── background.mp3  # Background music
├── README.md       # This file
└── docs/
    └── plans/      # Design & implementation docs
```

## 🎨 Color Palettes

| Palette | Colors |
|---------|--------|
| 🌟 Golden Celebration | Gold, Orange, Red, White |
| 💎 Electric Blue | Cyan, Sky Blue, Royal Blue, White |
| 🌸 Pink Paradise | Deep Pink, Hot Pink, Magenta, White |
| 🌲 Green Aurora | Lime, Green, Yellow-Green, White |
| 🔮 Purple Galaxy | Violet, Blue-Violet, Purple, White |
| 🌈 Rainbow Burst | Red, Orange, Yellow, Green, Blue, Violet |
| 🔥 Fire Storm | Orange-Red, Tomato, Coral, Orange, Gold, White |
| ❄️ Ice Crystal | Light Cyan, Powder Blue, Sky Blue, Teal, White |

## 🙏 Credits

- Hand tracking powered by [MediaPipe Hands](https://google.github.io/mediapipe/solutions/hands.html)
- Inspired by [手势烟花](https://x.com/libukai/status/2012466547399487584)

## 📄 License

MIT License - feel free to use, modify, and share!

---

Made with ❤️ and 🎆
