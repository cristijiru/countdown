# New Year Countdown - Project Documentation

## Overview
Single-file HTML/CSS/JS countdown to January 1st of the next year, featuring a European city skyline silhouette with animated fireworks.

## Architecture

### Single File Structure
Everything is in `index.html` - no external dependencies, no build step. Sections:
1. CSS (lines 7-628)
2. HTML (lines 630-1065)
3. JavaScript (lines 1067-1418)

### Dynamic Year Targeting
```javascript
const targetYear = new Date().getFullYear() + 1;
const targetDate = new Date(targetYear, 0, 1, 0, 0, 0).getTime();
```
Always targets next January 1st at midnight. Year updates automatically.

## Docker Setup

### Production Mode
```bash
docker-compose up --build
```
- Uses `Dockerfile` with nginx
- Serves static files on port 8080
- Container name: `web`

### Development Mode (Live Reload)
```bash
docker-compose -f docker-compose.dev.yml up --build
```
- Uses `Dockerfile.dev` with Node.js + live-server
- Auto-reloads on file changes
- Volume mounts current directory to `/app`
- Container name: `web-dev`
- Access at http://localhost:8080

### Docker Files
- `Dockerfile` - Production nginx image
- `Dockerfile.dev` - Dev image with live-server
- `docker-compose.yml` - Production compose
- `docker-compose.dev.yml` - Dev compose with volume mount

## Key Components

### Theme System
- Uses `data-theme="dark"` attribute on `<html>`
- CSS variables in `:root` and `[data-theme="dark"]`
- Persisted to `localStorage.theme`
- Respects system preference on first load

### Language System
- English (en) and Romanian (ro)
- Persisted to `localStorage.lang`
- Translations object contains: subtitle, mainTitle, days, hours, minutes, seconds, happyNewYear, fireworks

### Controls Menu
All controls are grouped in a collapsible menu (top-right gear icon):
- **Desktop**: Expands on hover
- **Mobile/Touch**: Expands on tap, closes when tapping outside

**Menu Layout:**
```
┌─────────────────────┐
│     Fireworks       │  ← full width button
├───────────┬─────────┤
│  ☀️ 🌙   │   🔊    │  ← theme + mute toggle
├───────────┴─────────┤
│      EN | RO        │  ← language toggle
└─────────────────────┘
```

**CSS Classes:**
- `.controls-menu` - Container with `z-index: 20`
- `.menu-trigger` - Gear icon button (44x44px)
- `.menu-panel` - Dropdown panel with controls
- `.menu-row` - Horizontal row for theme + mute
- `.open` class toggles visibility on touch devices

### Mute Toggle
- Persisted to `localStorage.muted`
- Mutes/unmutes both `audioLoop` and `audioClimax`
- SVG speaker icons (with sound waves / with X)
- State managed via `isMuted` variable and `updateMuteState()` function

### Audio System
Two audio tracks:
- `hedwig_loop.mp3` - Background loop music
- `hedwig_climax.mp3` - Plays during fireworks

**Robustness features:**
- `audioStarted` flag only set to `true` after successful `play()` (in `.then()` callback)
- Event listeners retry on each user interaction until music actually starts
- Handles browser autoplay restrictions gracefully
- Loop restarts automatically after climax ends

**Audio Crossfade:**
When transitioning to climax music:
1. Loop fades out over 2 seconds
2. After 1 second delay, climax starts
3. Climax fades in over 2 seconds
4. Creates smooth crossfade with brief quiet moment in middle

### City Skyline (SVG)
- ViewBox: `0 0 1200 180`
- Uses `preserveAspectRatio="xMidYMax slice"` to anchor at bottom
- Buildings include: Romanian Athenaeum, Notre-Dame cathedral, Opera House, Clock Tower, Parliament
- Two bridges with arches and lights
- Fixed height: 180px (120px on mobile)

### Window Lights
SVG rect elements inside `.svg-windows` group with `windowGlow` filter.

**Positioning:**
- Use SVG coordinates directly (viewBox 0-1200 for x, 0-180 for y)
- Windows must be centered within building rectangular bodies, not in triangular roofs
- Extend windows close to building edges for realistic coverage

**Size Variations:**
- Small (3x4): Tower tops, upper floors, apartments
- Medium (4x5): Standard windows
- Large (5x6, 5x7): Lower floors of grand buildings
- Grand (6x8): Ground floor entrances (Athenaeum, Opera, Parliament)

**Special Windows:**
- Athenaeum cupola: Large 8x10 window
- Opera House dome: Window at y=62
- Notre-Dame central spire: Window at y=50
- Bridge lights: Small 3x4 lights along bridge spans

**Mobile Visibility:**
Windows for hidden buildings are wrapped in `<g class="hide-mobile">`:
- Left side buildings, Townhouse blocks, Bridge section 1
- Residential blocks, Parliament, End buildings

**Animation:**
- Normal mode: `svgTwinkle` animation (3-4s cycle) in dark mode
- Fireworks mode: `windowPulse` animation (0.8s cycle) with warm colors

### Street Lights
- 5 poles positioned between buildings at: 13%, 44.5%, 60.5%, 77.5%, 90.5%
- Use `::before` for pole (18px height) and `::after` for light pool effect
- Only visible in dark mode

### Stars and Moon
- 120 stars dynamically created, positioned in upper 60% of screen
- Stars only visible in dark mode via CSS (`[data-theme="dark"] .star`)
- Moon positioned top-left (8%, 8%), uses radial gradient with crater details
- Moon also only visible in dark mode

### Fireworks System

#### Timing Constants
```javascript
const FIREWORKS_DELAY = 5000;      // 5 seconds (syncs with second 4 of climax, accounting for 1s audio delay)
const FIREWORKS_DURATION = 60000;  // 60 seconds of fireworks
```

#### Explosion Types (randomly selected)
1. **Classic burst** - 20 sparks radiating outward with trails
2. **Ring explosion** - Expanding ring with inner sparks
3. **Starburst** - 12 rays with falling glitter
4. **Double burst** - Two layers of sparks at different velocities

#### State Management (prevents spam crashes)
Global state variables track fireworks status:
```javascript
let fireworksActive = false;
let fireworksRocketInterval = null;
let fireworksExplosionInterval = null;
let fireworksStartTimeout = null;
let fireworksEndTimeout = null;
```

`stopFireworks()` function cleanly terminates:
- Clears all intervals and timeouts
- Removes `.lit` class from city elements
- Clears all firework DOM elements via `innerHTML = ''`
- Resets `fireworksActive` flag

Clicking fireworks button while show is running will stop and restart cleanly.

#### Fireworks Sequence
1. **Immediately**: Night sky (moon/stars) and city glow appear, loop music starts fading out
2. **After 1s**: Climax music starts fading in
3. **After 5s delay**:
   - `.lit` class added to city (triggers warm window pulsating)
   - Continuous rockets every 200ms
   - Continuous explosions every 400ms
   - Initial burst of 15 rockets
   - Syncs with second 4 of climax music
4. **After 60s**: `stopFireworks()` called, city returns to normal

#### Warm Window Pulsating
During fireworks (`.lit` state), windows pulsate with warm colors:
```css
.city-skyline.lit .svg-windows .window {
    animation: windowPulse 0.8s ease-in-out infinite;
}
```
- 7 different color pairs assigned via `nth-child(7n+x)`
- Staggered animation delays (0s to 0.6s)
- **Warm palette**: yellows, oranges, reds, warm whites
- **Occasional cool accents**: 2 of 7 combinations include purple or blue
- Drop-shadow glow effect matches current color

Color pairs:
1. Yellow (#ffee55) to Orange (#ffaa00)
2. Orange (#ff8833) to Red-orange (#ff5522)
3. Warm white (#ffffcc) to Yellow (#ffdd44)
4. Red-orange (#ff6644) to Orange (#ffaa33)
5. Yellow (#ffcc22) to Orange-red (#ff7744)
6. Purple (#aa88ff) to Warm orange (#ffbb55) - cool accent
7. Orange (#ffbb44) to Blue (#6699ff) - cool accent

### Fireworks Behavior
- **Auto-switches to dark mode** when launched
- Night sky and city glow appear immediately (sets mood)
- Audio crossfades smoothly (loop out, climax in)
- Windows start warm pulsating when fireworks begin (after 5s delay, synced with music)
- City skyline opacity increases (`.lit` class)
- All windows illuminate with animated warm colors
- Effects last 60 seconds, then city returns to normal (dark mode persists)

## Positioning Quirks

### Controls Menu Position
- Fixed position: `top: 2rem; right: 2rem`
- Mobile: `top: 1rem; right: 1rem`
- Gear icon trigger: 44x44px
- Menu panel appears below trigger on hover/tap

### Window Positioning
Windows use SVG coordinates directly:
- `x`: 0-1200 (SVG viewBox width)
- `y`: 0-180 (SVG viewBox height, 0 at top)

To add windows to a building:
1. Find building rect in SVG (e.g., `x="245" width="100"`)
2. Calculate center: 245 + 100/2 = 295
3. Position windows symmetrically around center
4. Keep y values within building's rectangular body height
5. Use appropriate size based on floor (smaller up top, larger at bottom)

### Z-Index Stack
```
0  - night-sky, city-glow
1  - city-skyline
2  - city-lights (windows, street lights)
10 - fireworks
20 - controls-menu
```

## Mobile Responsiveness
Breakpoint at 600px:
- Smaller title (2.5rem) and numbers (2.5rem)
- **Countdown cards closer together**: gap reduced to 0.75rem, padding to 1rem/1.25rem, min-width to 70px
- City skyline reduced to 120px
- Controls menu moves to `top: 1rem; right: 1rem`
- Touch devices: hover disabled, tap-to-toggle via `.open` class
- Several buildings hidden via `.hide-mobile` class:
  - Left side buildings
  - Townhouse blocks
  - Bridge section 1
  - Residential blocks
  - Parliament style building
  - End buildings
- Corresponding window lights wrapped in `<g class="hide-mobile">` groups

## State Persistence
- `localStorage.theme`: 'dark' or 'light'
- `localStorage.lang`: 'en' or 'ro'
- `localStorage.muted`: 'true' or 'false'

## Animation Timings
- Time boxes float: 4s cycle, staggered by 0.15s
- Window twinkle: 3-4s cycle (normal), 0.8s cycle (fireworks pulsate)
- Star twinkle: 2s cycle
- Audio crossfade: 2s fade out, 1s delay, 2s fade in
- Firework show duration: 60 seconds total (after 5s delay)
- Rocket travel: 0.6-1.0s
- Spark animations: 0.8-1.4s

## Common Pitfalls

1. **Window alignment**: Always check windows are within rectangular building bodies, not roofs
2. **Window sizing**: Use varied sizes - small at top, large at bottom for realism
3. **Window coverage**: Extend windows close to building edges
4. **Mobile windows**: Wrap windows for hidden buildings in `<g class="hide-mobile">`
5. **Controls menu**: All controls are in collapsible menu; hover on desktop, tap on mobile
6. **Dark mode dependency**: Stars, moon, street lights, and window glow all depend on `[data-theme="dark"]`
7. **Fireworks cleanup**: Use `stopFireworks()` for proper cleanup; don't create orphan intervals
8. **Mobile layout**: Menu uses tap-to-toggle; test at 600px breakpoint
9. **Audio autoplay**: Browsers block autoplay; music only starts after user interaction
10. **Fireworks spam**: Button clicks while show is running will restart cleanly (not stack)
11. **Audio sync**: FIREWORKS_DELAY accounts for 1s audio delay to sync with climax second 4
12. **Mute state**: Persists across page reloads; affects both loop and climax audio
