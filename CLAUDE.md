# Apogee Zero

Single-file browser game (index.html). 2D top-down spaceship survival with endless waves.

## Tech

- Vanilla JS, HTML5 Canvas, no dependencies
- Everything in one HTML file
- Procedural Canvas drawing, no image assets
- localStorage for leaderboard + player name

## How to run

Open index.html in a browser. Click canvas to engage pointer lock.

## Controls

- WASD: thrust (Newtonian physics with damping)
- Mouse movement: rotate ship (pointer lock, relative)
- Left click: plasma bolt (fast, weak, drains plasma bar)
- Right click: missile (slow, powerful, drains missile bar)

## Code structure (inside `<script>`)

### State & setup (~lines 78-200)
- Global state variables, input handlers, pointer lock
- `initGame()`, `makeStars()`, `makePlanets()`, `nextWave()`
- `spawnEnemy()`, `spawnAlly()`, `safeSpawnPoint()`

### Shared helpers (~lines 262-370)
- `screenShake()`, `spawnParticles()`, `spawnExhaust()`
- `turnToward(current, target, rate, dt)` — smooth angle rotation
- `damagePlayer(amount)` — centralized player hp/death/respawn
- `handlePlanetCollision(obj, pl)` — shared bounce/damage for any entity vs planet
- `avoidPlanets(obj, dt)` — AI steering away from planets, returns force applied
- `fireProjectile()`, `leadShot()` — weapon and aim-ahead math

### Update sub-functions (~lines 370-870)
- `updatePhysics(dt)` — gravity, planet collisions for all entities
- `updatePlayer(dt)` — input, thrust, ammo recharge, shooting, health regen
- `updateEnemies(dt)` — enemy AI (strafing/orbit, shoot, planet avoidance, EMP disable)
- `updateAllies(dt)` — ally AI (target nearest enemy, orbit, shoot, EMP disable)
- `updateProjectiles(dt)` — movement, trails, all projectile-vs-entity hits
- `updateCollisions(dt)` — body-to-body: enemy-player, ally-player, ally-enemy
- `updatePowerups(dt)` — timers, pickup logic, powerup gravity toward player
- `updateParticles(dt)` — movement, cleanup
- `update(dt)` — orchestrator calling the above + wave spawning + camera + EMP shockwave

### Rendering (~lines 870-1210)
- `drawShip()` — procedural ship shapes (player/ally arrow, drone triangle, gunship diamond, bruiser hexagon)
- `drawBar()` — reusable neon HUD bar
- `draw()` — stars, planets, particles, projectiles, powerups, ships, shield, EMP shockwave, HUD
- `drawHUD()` — HP/plasma/missile bars, lives, score, wave, active powerups

### Game flow (~lines 1210-1280)
- `gameOver()` — releases pointer lock, hides save UI for non-qualifying scores
- `showLeaderboard()`
- Event listeners for start/restart/submit score
- `loop()` with requestAnimationFrame

## Game systems

### Enemies (4 types)
- **Drone**: fast, low hp, weak shots, red — strafing runs (charge/break/extend)
- **Gunship**: medium, orbits at preferred distance, orange
- **Bruiser**: slow, tanky, fires missiles, purple/hexagonal
- **Kamikaze**: very fast, fragile, yellow-green — charges and explodes on contact, slow turning (1 rad/s)

All ships rotate smoothly via `turnToward()`. Enemies aim at lead-shot direction and fire straight from their nose.

### Allies
- Same ship shape as player, green tint
- Target nearest enemy, orbit and shoot (straight from nose with lead aiming)
- Subject to friendly fire both ways
- 30% spawn chance per wave (starting wave 2, max 2)

### Planets & asteroids
- Asteroids: gray, radius 150-400, gravity affects everything
- Planets: colorful, radius 1500-4000, strong gravity
- Both cause impact damage on collision (speed-based)
- Projectiles destroyed on planet contact

### Powerups (drop from enemies, 20% chance)
- **Health** (+40 hp, instant, green "+")
- **Shield** (8s invulnerability, blue "S")
- **Spread Shot** (10s triple plasma + 6x recharge, magenta "*")
- **EMP** (6s disables all non-player ships — they drift and spin helplessly, yellow "E", expanding shockwave animation)

Powerups gravitate toward the player from any distance (inverse-square pull).

### Player
- HP slowly regenerates at 2/sec
- Pointer lock released on game over
- Leaderboard save UI only shown if score qualifies for top 5

### Ammo
- Plasma: 100 energy, costs 5/shot (15 for spread), recharges 15/sec (90/sec during spread)
- Missiles: 100 energy, costs 20/shot, recharges 8/sec
