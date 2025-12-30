# New Year Countdown - Project Documentation

## Overview
Single-file HTML/CSS/JS countdown to January 1st of the next year, featuring a European city skyline silhouette with animated fireworks.

## Architecture

### Single File Structure
Everything is in `countdown.html` - no external dependencies, no build step. Sections:
1. CSS (lines 7-628)
2. HTML (lines 630-797)
3. JavaScript (lines 799-1265)

### Dynamic Year Targeting
```javascript
const targetYear = new Date().getFullYear() + 1;
const targetDate = new Date(targetYear, 0, 1, 0, 0, 0).getTime();
```
Always targets next January 1st at midnight. Year updates automatically.

## Key Components

### Theme System
- Uses `data-theme="dark"` attribute on `<html>`
- CSS variables in `:root` and `[data-theme="dark"]`
- Persisted to `localStorage.theme`
- Respects system preference on first load

### Language System
- English (en) and Romanian (ro)
- Persisted to `localStorage.lang`
- Translations object contains: subtitle, mainTitle, days, hours, minutes, seconds, happyNewYear

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
- Animate with `twinkle` (light mode) and `twinkleDark` (dark mode)
- More visible in dark mode

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
Four explosion types (randomly selected):
1. **Classic burst** - 20 sparks radiating outward with trails
2. **Ring explosion** - Expanding ring with inner sparks
3. **Starburst** - 12 rays with falling glitter
4. **Double burst** - Two layers of sparks at different velocities

Sequence:
- 35 rockets launched from bottom over ~5 seconds
- 20 random sky explosions
- Grand finale at 7 seconds: 15 rapid explosions

### Fireworks Behavior
- **Auto-switches to dark mode** when launched
- Lights up city skyline (`.lit` class increases opacity)
- All windows fully illuminate
- Night sky becomes visible
- City glow appears at bottom
- Effects last 12 seconds, then city returns to normal (dark mode persists)

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
- Window twinkle: 3-4s cycle
- Star twinkle: 2s cycle
- Firework show duration: ~12 seconds total
- Rocket travel: 0.6-1.0s
- Spark animations: 0.8-1.4s

## Common Pitfalls

1. **Window alignment**: Always check windows are within rectangular building bodies, not roofs
2. **Button spacing**: Three buttons at top-right need consistent gaps (~0.5rem visual spacing)
3. **Dark mode dependency**: Stars, moon, street lights, and window glow all depend on `[data-theme="dark"]`
4. **Fireworks cleanup**: All created elements use `setTimeout` for removal to prevent DOM bloat
5. **Mobile layout**: Controls stack differently; test at 600px breakpoint
