# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a highly interactive personal website for Ruby (age 2), built with vanilla HTML, CSS, and JavaScript. The site is deployed to GitHub Pages at `rubysisk.com` and features:

- **Zero frameworks or build tools** - Pure vanilla JS/CSS
- **12+ interactive features** - Games, easter eggs, AR camera, live effects
- **Mobile-first design** - Optimized for touch interactions
- **Performance-focused** - Object pooling, RAF animations, GPU acceleration
- **PicPerf image optimization** (https://picperf.io)
- **Accessibility features** - Keyboard navigation, reduced motion support

## Architecture

### Core Files

- `index.html` (1665 lines) - Single-page site with comprehensive inline JavaScript
- `styles.css` (1192 lines) - Complete styling with advanced animations and responsive breakpoints
- `assets/` - Local image files (production uses PicPerf CDN URLs)
- `assets/gallery/` - Secret gallery photos (auto-loaded into hidden gallery)

### Major Feature Systems

**1. Interactive Features** (`index.html:241-1662`)
- Particle burst system with object pooling (CONFIG.particlePoolSize = 50)
- Rainbow trail drawing mode (canvas-based, circular buffer)
- Pop-the-bubbles game (RAF animations, touch-optimized)
- Color splash mode (5 predefined cute color themes)
- Click counter game (localStorage leaderboard for family competition)
- Konami code easter egg (keyboard + swipe pattern detection)
- Shake-for-surprise (DeviceMotion API + desktop mouse detection)
- Time-based greetings (different by hour, special birthday mode)
- AR camera with face filters (getUserMedia, simple overlay positioning)
- Weather-reactive theming (OpenWeatherMap API with time/season fallback)
- Cursor friends (desktop-only, spring physics followers)

**2. Mobile Touch Optimizations** (`index.html:369-394`)
- `touch-action: manipulation` prevents double-tap zoom
- Passive event listeners for better scroll performance
- Custom double-tap prevention (allows zoom on photos, blocks on games)
- Touch-friendly button sizes (50px+ on mobile)

**3. Performance Architecture**
- **Object Pooling**: Particles and bubbles reuse DOM elements (no GC pressure)
- **RAF Loops**: All animations use requestAnimationFrame for 60fps
- **Throttle/Debounce**: Critical events throttled (shake: 100ms, mouse: 50ms)
- **GPU Acceleration**: `will-change`, `transform`, `opacity` for smooth animations
- **Canvas-based Drawing**: Rainbow trail uses offscreen rendering
- **Circular Buffers**: Trail points (maxTrailPoints: 50) prevent memory leaks

**4. Configuration System** (`index.html:247-271`)
All features tunable via CONFIG object:
```javascript
CONFIG = {
  birthday: { month: 1, day: 15 },        // Ruby's birthday for special effects
  weatherApiKey: '',                       // OpenWeatherMap API key
  clickGameTarget: 10,                     // Taps to win click game
  bubbleSpawnInterval: 1500,               // Bubble spawn rate (ms)
  particlePoolSize: 50,                    // Particle object pool size
  maxActiveParticles: 30,                  // Performance limit
  galleryPath: 'assets/gallery/',          // Secret gallery photos
}
```

**5. CSS Animation System** (`styles.css:8-408`)
- Multi-layered background gradients (6 layers) with `gradient-shift` animation
- Polaroid cards use CSS custom properties (`--tilt`) for rotation angles
- Staggered `float` animations with different delays per card
- `pulse-glow`, `disco-mode`, `birthday-mode` special effects
- Mobile: animations disabled on `prefers-reduced-motion` or `max-width: 800px`

**6. Feature Toggle System**
- Fixed position emoji buttons (🌈 🫧 🎨 📸) in top-right (desktop) or bottom-right (mobile)
- Active state with pulsing animation
- State stored in FEATURES object, easily persisted to localStorage

## Development Commands

**Local Development**
```bash
# Serve locally with Python
python3 -m http.server 8000

# Or with Node.js
npx serve .
```

**Deployment**
The site auto-deploys via GitHub Actions (`.github/workflows/pages.yml`) on push to `main` branch. No manual deployment needed.

**Testing Changes**
- Open `index.html` directly in browser for quick previews
- **Mobile testing is critical** - primary use case is mobile web app
- Test with reduced motion: Enable in OS accessibility settings
- Test touch interactions: Use browser dev tools device emulation
- Check console for easter egg hints and debug logs
- Test features: Tap Ruby's name 10x fast, shake device, Konami code, try camera filters

## Code Style & Patterns

**HTML/JS Comments**
Every major section has "WHY" comments explaining technical decisions:
- Why object pooling for particles (prevents GC pauses)
- Why canvas for rainbow trail (avoids reflows/repaints)
- Why throttle on shake detection (battery optimization)
- Why touch-action CSS (prevents double-tap zoom)

**CSS Organization**
1. CSS custom properties (`:root`)
2. Original animations (float, gradient-shift, pulse-glow, sparkle-float)
3. New feature animations (confetti-fall, disco-mode, birthday-sparkle)
4. Global/reset styles
5. Original component styles
6. New feature component styles
7. Mobile optimizations (@media)

**Performance Patterns**
- **RAF over setInterval**: `requestAnimationFrame(update)` for smooth 60fps
- **Object Pooling**: Create once, reuse forever
- **Passive Listeners**: `{ passive: true }` on touch events
- **Will-Change Hints**: Applied to animated elements
- **Debounced Resize**: `debounce(resizeCanvas, 250)`

**Mobile-First Considerations**
- Touch targets: 50px+ for easy tapping by kids
- Feature buttons: Repositioned to bottom for thumb reach on mobile
- Double-tap zoom: Prevented on interactive elements, allowed on photos
- Pinch zoom: Allowed on photos, blocked on game elements
- Swipe gestures: Gallery dismissal, Konami code, photo navigation

## Easter Eggs & Hidden Features

Users can discover these by exploring or checking the console:
1. **Click Game**: Tap "Ruby" 10 times in 3 seconds → Achievement + Leaderboard
2. **Konami Code**: ↑↑↓↓←→←→ (or swipe pattern) → Disco mode (10s)
3. **Shake Device**: Shake phone or wiggle mouse → Confetti + color shift
4. **Birthday Mode**: On Ruby's birthday (CONFIG.birthday) → Continuous celebration
5. **Time Greetings**: Different greeting based on hour (morning, evening, etc.)
6. **Console Hints**: Check console.log for easter egg instructions

## Configuration Tasks

**Setting Ruby's Birthday**
Update `CONFIG.birthday` in `index.html:249`:
```javascript
birthday: { month: 10, day: 15 }, // October 15th
```

**Adding Weather API**
1. Get free key at https://openweathermap.org/api
2. Update `CONFIG.weatherApiKey` in `index.html:253`:
```javascript
weatherApiKey: 'your-api-key-here',
```
3. Update location in `index.html:1437` if needed:
```javascript
const response = await fetch(
  `https://api.openweathermap.org/data/2.5/weather?q=Portland,OR&appid=${CONFIG.weatherApiKey}`
);
```

**Tuning Game Difficulty**
Adjust CONFIG values (`index.html:247-271`):
- `clickGameTarget`: Number of taps to win
- `clickGameTimeout`: Time limit (ms) for click game
- `bubbleSpawnInterval`: How often bubbles spawn
- `bubbleMaxCount`: Max simultaneous bubbles
- `particlePoolSize`: Particle effects quantity

## Important Notes

- **Mobile-First**: Primary use case is mobile web app for kids (age 2-6)
- **Performance**: Must stay 60fps even on older mobile devices
- **Images**: Local files in `assets/` for development, PicPerf CDN for production
- **No Build Step**: Pure static files, works offline
- **Branch**: GitHub Pages workflow targets `main` (repo currently uses `master`)
- **CNAME**: Custom domain `rubysisk.com` configured
- **localStorage**: Used for click game leaderboard, weather cache
- **Comprehensive Comments**: Every feature explains "WHY", not just "WHAT"

## Performance Budget

- **Target FPS**: 60fps (16.6ms per frame)
- **Particle Pool**: Max 50 particles, max 30 active
- **Bubble Limit**: Max 8 simultaneous bubbles
- **Canvas Updates**: RAF-based, only when active
- **API Calls**: Cached 5min (weather), no unnecessary fetches
- **Touch Events**: Throttled 50-100ms on high-frequency events

## Troubleshooting

**Features Not Working**
- Check console for errors
- Verify prefers-reduced-motion isn't enabled
- Ensure JavaScript is enabled
- Test in modern browser (Chrome/Safari/Firefox)

**Camera Not Working**
- Check HTTPS (required for getUserMedia)
- Grant camera permissions when prompted
- Test on device with camera

**Bubbles/Particles Lag**
- Reduce CONFIG.particlePoolSize and CONFIG.bubbleMaxCount
- Check if device is in low-power mode
- Verify no other heavy animations running

**Sparkles Causing Performance Issues**
- Reduce sparkleInterval in `index.html` (currently 1800ms)
- Disable sparkles on older devices via CONFIG
- Check prefers-reduced-motion is respected
