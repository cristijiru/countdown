# New Year Countdown - Project Documentation

## Overview
Single-file HTML/CSS/JS countdown to January 1st of the next year, featuring a European city skyline silhouette with animated fireworks.

## Architecture

### Single File Structure
Everything is in `index.html` - no external dependencies, no build step. Sections:
1. CSS (lines 7-628)
2. HTML (lines 630-970)
3. JavaScript (lines 978-1490)

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

### Audio System
Two audio tracks:
- `hedwig_loop.mp3` - Background loop music
- `hedwig_climax.mp3` - Plays during fireworks

**Robustness features:**
- `audioStarted` flag only set to `true` after successful `play()` (in `.then()` callback)
- Event listeners retry on each user interaction until music actually starts
- Handles browser autoplay restrictions gracefully
- Loop restarts automatically after climax ends

### City Skyline (SVG)
- ViewBox: `0 0 1200 180`
- Uses `preserveAspectRatio="xMidYMax slice"` to anchor at bottom
- Buildings include: Romanian Athenaeum, Notre-Dame cathedral, Opera House, Clock Tower, Parliament
- Two bridges with arches
- Fixed height: 180px (120px on mobile)

### Window Lights
- Positioned using percentage (x) and pixels from bottom (y)
- **Critical**: Windows must be centered within building rectangular bodies, not in triangular roofs
- Conversion: SVG x-coordinate / 12 = percentage
- Animate with `svgTwinkle` in dark mode
- During fireworks: colorful pulsating animation (see Fireworks section)

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
const FIREWORKS_DELAY = 4000;      // 4 seconds delay before fireworks start
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
1. **Immediately**: Night sky (moon/stars) and city glow appear
2. **After 4s delay**:
   - `.lit` class added to city (triggers colorful window pulsating)
   - Continuous rockets every 200ms
   - Continuous explosions every 400ms
   - Initial burst of 15 rockets
3. **After 60s**: `stopFireworks()` called, city returns to normal

#### Colorful Window Pulsating
During fireworks (`.lit` state), windows pulsate with colors:
```css
.city-skyline.lit .svg-windows .window {
    animation: windowPulse 0.8s ease-in-out infinite;
}
```
- 7 different color pairs assigned via `nth-child(7n+x)`
- Staggered animation delays (0s to 0.6s)
- Colors include: pink/orange, green/cyan, blue/magenta, yellow/red, etc.
- Drop-shadow glow effect matches current color

### Fireworks Behavior
- **Auto-switches to dark mode** when launched
- Night sky and city glow appear immediately (sets mood)
- Windows start colorful pulsating when fireworks actually begin (after 4s delay)
- City skyline opacity increases (`.lit` class)
- All windows illuminate with animated colors
- Effects last 60 seconds, then city returns to normal (dark mode persists)

## Positioning Quirks

### Top-Right Controls Layout
```
[Fireworks] [EN|RO] [Light/Dark Toggle]
   13.5rem    7rem        2rem (from right)
```
All have `height: 36px` for alignment.

### Window Positioning
Windows use hybrid units:
- `x`: percentage of container width
- `y`: pixels from bottom of 180px container

To add windows to a building:
1. Find building rect in SVG (e.g., `x="245" width="100"`)
2. Calculate center: (245 + 100/2) / 12 = 24.6%
3. Position windows symmetrically around center
4. Keep y values within building's rectangular body height

### Z-Index Stack
```
0  - night-sky, city-glow
1  - city-skyline
2  - city-lights (windows, street lights)
10 - fireworks
15 - fireworks button
```

## Mobile Responsiveness
Breakpoint at 600px:
- Smaller title and numbers
- City skyline reduced to 120px
- Fireworks button moves to `top: 6rem; right: 1rem`
- Several buildings hidden via `.hide-mobile` class:
  - Left side buildings
  - Townhouse block
  - Bridge section 1
  - Residential block
  - Parliament style building
  - End buildings
- Corresponding window lights also hidden (marked with `.hide-mobile` class based on x% position ranges)

## State Persistence
- `localStorage.theme`: 'dark' or 'light'
- `localStorage.lang`: 'en' or 'ro'

## Animation Timings
- Time boxes float: 4s cycle, staggered by 0.15s
- Window twinkle: 3-4s cycle (normal), 0.8s cycle (fireworks pulsate)
- Star twinkle: 2s cycle
- Firework show duration: 60 seconds total (after 4s delay)
- Rocket travel: 0.6-1.0s
- Spark animations: 0.8-1.4s

## Common Pitfalls

1. **Window alignment**: Always check windows are within rectangular building bodies, not roofs
2. **Button spacing**: Three buttons at top-right need consistent gaps (~0.5rem visual spacing)
3. **Dark mode dependency**: Stars, moon, street lights, and window glow all depend on `[data-theme="dark"]`
4. **Fireworks cleanup**: Use `stopFireworks()` for proper cleanup; don't create orphan intervals
5. **Mobile layout**: Controls stack differently; test at 600px breakpoint
6. **Audio autoplay**: Browsers block autoplay; music only starts after user interaction
7. **Fireworks spam**: Button clicks while show is running will restart cleanly (not stack)
