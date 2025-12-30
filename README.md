# New Year Countdown

A cinematic countdown experience to welcome the New Year. Watch as a European city skyline comes alive with synchronized fireworks, colorful building lights, and an orchestral soundtrack.

---

## The Experience

As midnight approaches, the city transforms. The night sky fills with stars and a glowing moon. Windows across the skyline begin pulsating with vibrant colors. Fireworks illuminate the darkness while Hedwig's Theme builds to its crescendo.

### Visual Elements

**City Skyline** — A detailed silhouette featuring iconic European architecture: the Romanian Athenaeum, Notre-Dame Cathedral, an Opera House, a medieval Clock Tower, and Parliament buildings connected by stone bridges.

**Dynamic Lighting** — Street lamps cast warm pools of light. Building windows twinkle softly in the evening, then burst into colorful synchronized animations during the fireworks show.

**Fireworks Display** — Four distinct explosion styles create variety: classic bursts with trailing sparks, expanding rings, starbursts with falling glitter, and layered double explosions. A 60-second show launches automatically at midnight or on demand.

**Day and Night** — Toggle between light and dark themes. The dark theme reveals the full night atmosphere with moon, stars, and glowing city lights.

### Audio

The experience is accompanied by two movements from Hedwig's Theme:
- A gentle loop plays in the background
- The climax movement synchronizes with the fireworks

Audio begins after your first interaction with the page (browser requirement).

### Languages

Available in English and Romanian.

---

## Quick Start

### Run with Docker

**Development** (with live reload):
```bash
docker-compose -f docker-compose.dev.yml up --build
```

**Production**:
```bash
docker-compose up --build
```

Open http://localhost:8080

### Run without Docker

Simply open `index.html` in a browser. No build step required.

---

## Development

### Prerequisites

- Docker and Docker Compose (for containerized development)
- Or any modern web browser (for direct file access)

### Commands

| Action | Development | Production |
|--------|-------------|------------|
| Start | `docker-compose -f docker-compose.dev.yml up` | `docker-compose up` |
| Build & Start | `docker-compose -f docker-compose.dev.yml up --build` | `docker-compose up --build` |
| Stop | `docker-compose -f docker-compose.dev.yml down` | `docker-compose down` |

### Project Structure

```
index.html              Main application (HTML, CSS, JS in one file)
hedwig_loop.mp3         Background music
hedwig_climax.mp3       Fireworks music
Dockerfile              Production image (nginx)
Dockerfile.dev          Development image (live-server)
docker-compose.yml      Production configuration
docker-compose.dev.yml  Development configuration
```

### Architecture

The entire application lives in a single HTML file with no external dependencies:
- CSS handles theming via custom properties and `data-theme` attribute
- SVG graphics for the city skyline scale responsively
- Vanilla JavaScript manages state, animations, and audio

State is persisted to localStorage:
- Theme preference (light/dark)
- Language selection (en/ro)

---

## Customization

The countdown automatically targets January 1st of the next year. To test with a custom date, find the `targetDate` variable in the JavaScript section of `index.html`.

---

## Credits

Music: Hedwig's Theme from Harry Potter (John Williams)
