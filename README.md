# Apogee Zero

A 2D top-down spaceship survival game with endless waves, built entirely in a single HTML file. No dependencies, no build step — just open and play.

## Play

Open `index.html` in any modern browser. Click the canvas to start.

**[Play on GitHub Pages](https://hatchr.github.io/ApogeeZero/)**

## Screenshots

<!-- Add screenshots here -->

## Controls

| Input | Action |
|-------|--------|
| WASD | Thrust (Newtonian physics with damping) |
| Mouse | Aim (pointer lock) |
| Left Click | Fire plasma bolt (fast, weak) |
| Right Click | Fire missile (slow, powerful) |

## Features

- **Single-file game** — everything in one `index.html`, no dependencies
- **Procedural rendering** — all graphics drawn with HTML5 Canvas, no image assets
- **Newtonian physics** — momentum, gravity, planetary collisions
- **4 enemy types** — Drones, Gunships, Bruisers, and Kamikazes with distinct AI behaviors
- **Allied ships** — AI wingmen that fight alongside you
- **Powerups** — Health, Shield, Spread Shot, and EMP with visual effects
- **Planetary gravity** — asteroids and planets pull everything in range
- **Local leaderboard** — top 5 scores saved to localStorage
- **Minimap radar** — track enemies, allies, and powerups

## Enemies

| Type | Color | Behavior |
|------|-------|----------|
| Drone | Red | Fast strafing runs — charges in, breaks past, loops back |
| Gunship | Orange | Orbits at distance, steady fire |
| Bruiser | Purple | Slow and tanky, fires homing missiles |
| Kamikaze | Yellow-Green | High speed charge, explodes on contact, poor turning |

## Powerups

| Powerup | Color | Effect |
|---------|-------|--------|
| Health | Green | Instant +40 HP |
| Shield | Blue | 8s invulnerability |
| Spread Shot | Magenta | 10s triple plasma + 6x recharge rate |
| EMP | Yellow | 6s — disables all non-player ships, shockwave animation |
| Homing Missiles | Orange | 12s — missiles steer toward nearest enemy, 2x missile recharge |

## Tech

- Vanilla JavaScript
- HTML5 Canvas API
- No frameworks, libraries, or build tools
- localStorage for persistence

## License

MIT
