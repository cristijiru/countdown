# New Year Countdown

A cinematic countdown to the New Year featuring a European city skyline, synchronized fireworks, and an orchestral soundtrack.

---

## The Experience

As midnight approaches, the city transforms. Windows pulsate with warm colors, fireworks light up the sky, and Hedwig's Theme builds to its crescendo.

**City Skyline** — Romanian Athenaeum, Notre-Dame Cathedral, Opera House, Clock Tower, and Parliament buildings connected by stone bridges.

**Fireworks** — Four explosion styles: classic bursts, expanding rings, starbursts with glitter, and layered double explosions. Launches automatically at midnight or on demand.

**Controls** — Gear icon (top-right) reveals settings: fireworks, theme, sound, and language. Hover on desktop, tap on mobile.

**Audio** — Background loop and climax movement from Hedwig's Theme. Starts after first interaction. Mute toggle available.

---

## Quick Start

**With Docker:**
```bash
# Development (live reload)
docker-compose -f docker-compose.dev.yml up --build

# Production
docker-compose up --build
```

**Without Docker:** Open `index.html` in a browser.

Access at http://localhost:8080

---

## Project Structure

```
index.html              Single-file app (HTML, CSS, JS)
hedwig_loop.mp3         Background music
hedwig_climax.mp3       Fireworks music
docker-compose*.yml     Docker configurations
```

Settings persist to localStorage: theme, language, mute.

---

## Credits

Music: Hedwig's Theme from Harry Potter (John Williams)
