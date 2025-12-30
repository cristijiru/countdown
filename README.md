# New Year Countdown

A beautiful, interactive New Year countdown timer built with HTML, CSS, and JavaScript. Features particle animations, dark/light theme toggle, and audio playback using Hedwig's Theme from Harry Potter.

## Features

- Real-time countdown to New Year's Day (January 1st)
- Particle animation effects
- Dark and light theme support
- Audio playback with climax and loop tracks
- Responsive design
- Smooth animations and transitions
- Live reload during development

## Configurations

This project includes two Docker configurations:

### Production Configuration
- Uses nginx for efficient static file serving
- Optimized for performance and minimal image size
- Files are copied into the image at build time
- Suitable for deployment

### Development Configuration
- Uses live-server for automatic browser reloading
- Mounts source files as volumes for instant updates
- Includes live reload functionality
- Ideal for development and testing

## Prerequisites

- Docker
- Docker Compose

## Quick Start

### Development (with live reload)
```bash
docker-compose -f docker-compose.dev.yml up --build
```

Open your browser and go to `http://localhost:8080`

## Commands

### Development
- **Start**: `docker-compose -f docker-compose.dev.yml up`
- **Build and start**: `docker-compose -f docker-compose.dev.yml up --build`
- **Stop**: `docker-compose -f docker-compose.dev.yml down`
- **Logs**: `docker-compose -f docker-compose.dev.yml logs`

### Production
- **Start**: `docker-compose up`
- **Build and start**: `docker-compose up --build`
- **Stop**: `docker-compose down`
- **Logs**: `docker-compose logs`

## Files

- `index.html` - Main application file
- `hedwig_climax.mp3` - Audio file for countdown climax
- `hedwig_loop.mp3` - Audio file for background loop
- `Dockerfile` - Production Docker image (nginx)
- `Dockerfile.dev` - Development Docker image (live-server)
- `docker-compose.yml` - Production Docker Compose configuration
- `docker-compose.dev.yml` - Development Docker Compose configuration (with live reload)

## Customization

You can modify the countdown target date in the JavaScript code within `index.html` if needed.