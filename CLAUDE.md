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

### Desktop
- WASD: thrust (Newtonian physics with damping)
- Mouse movement: rotate ship (pointer lock, relative)
- Left click: plasma bolt (fast, weak, drains plasma bar)
- Right click: missile (slow, powerful, drains missile bar)

### Mobile (touch)
- Left-thumb drag in bottom-left zone (`x < 40% W, y > 50% H`): floating thrust joystick (appears at touch point, 55 px max deflection)
- Right-thumb drag in bottom-right zone (`x > 60% W, y > 50% H`): floating rotation joystick — horizontal deflection only, pill-shaped (90×190 px), rate-based at `RJOY_RATE = 0.1` rad/s/px (dt-correct)
- **Autofire**: fires automatically when an enemy is within `AUTOFIRE_ARC = 0.22 rad` (~12.5°) of the lead-shot heading. Sequence: plasma → plasma → plasma → missile → repeat (`mobileAutoSeq`). If the missile slot is unavailable (energy/cooldown), falls back to plasma and resets the sequence.

## Code structure (inside `<script>`)

### State & setup
- Global state variables, input handlers, pointer lock
- `isMobile` — detected via `matchMedia('(pointer: coarse) and (hover: none)')`. Reliable on touch-primary devices; correctly `false` on touchscreen laptops where the mouse is still the primary pointer. Controls `MAX_PARTICLES` cap (120 vs 300) and glow suppression.
- `initGame()`, `makeStars()`, `makePlanets()`, `nextWave()`
- `spawnEnemy()`, `spawnAlly()`, `safeSpawnPoint()`
- `mobileSetup()` — registers all touch handlers once at startup; no-ops on desktop. `showMobileControls(show)` hides both joystick bases on `show=false` (no-op on `true` — joysticks are floating and appear on demand). `updateJoystickVisual(dx, dy)` moves the left stick knob and is called from `resetInputState()` to snap it back to centre. `updateRJoyVisual(dx)` moves the right stick knob horizontally.

### Shared helpers
- `screenShake()`, `spawnParticles()`, `spawnExhaust()`
- `turnToward(current, target, rate, dt)` — smooth angle rotation
- `damagePlayer(amount)` — centralized player hp/death/respawn
- `handlePlanetCollision(obj, pl)` — shared bounce/damage for any entity vs planet
- `thrustLocal(obj, f, s, accel)` — canonical W/S/A/D thrust in ship's local frame (W ×1.0, S ×0.5, A/D ×0.7). Every ship uses this.
- `thrustToward(obj, heading, accel)` — AI helper; picks W/S from cos(delta), A/D from sin(delta), then calls `thrustLocal`.
- `avoidPlanets(obj, dt, navTarget)` — AI planet escape; rotates nose toward escape heading (radial away + 60% tangential bias) and calls `thrustToward`. Returns total urgency (0 if below threshold) so callers suppress attack steering.
- `fireProjectile(owner, x, y, angle, type, srcVx, srcVy, opts)`, `leadShot()` — weapon and aim-ahead math. Projectiles inherit the shooter's velocity on launch.
- `aimAndShoot(ship, target, dt)` — shared for enemies + allies. Reads `ship.weapon` (owner/projectile/bulletSpeed/range/muzzle/cdMin/cdSpan) attached at spawn. Caller picks target; mechanics are uniform.

### NPC behaviors
Each NPC has a `behavior(ship, dt)` function attached at spawn. The shared `updateNPCs` loop handles EMP drift, `applyMovement`, and exhaust uniformly; per-ship AI lives in its behavior.

- `kamikazeAI` — ram the player; no shooting
- `droneAI` — charge/break/extend state machine; calls `aimAndShoot` during charge/break
- `orbiterAI` — shared by gunship + bruiser (identical behavior, different weapon + orbit params). Calls `steerOrbit` then `aimAndShoot` — no separate player-facing turn, since two competing `turnToward` calls fight when orbital velocity is high.
- `allyAI` — find nearest enemy, orbit + `aimAndShoot`; drift toward player if no enemies
- `pickEnemyShootTarget(e, distToPlayer)` — helper; coin-flip between player and a closer ally

### Update sub-functions
- `updatePhysics(dt)` — gravity, planet collisions for all entities
- `updatePlayer(dt)` — input, thrust, ammo recharge, shooting, health regen (player stays separate — mouse rotation, spread shot, homing missiles, ammo energy system, HP regen, blue engine particles don't fit the NPC mold)
- `updateNPCs(dt)` — unified enemy + ally loop; EMP drift, `ship.behavior(ship, dt)`, movement, exhaust
- `updateProjectiles(dt)` — movement, trails, all projectile-vs-entity hits
- `updateCollisions(dt)` — body-to-body: enemy-player, ally-player, ally-enemy
- `updatePowerups(dt)` — timers, pickup logic, powerup gravity toward player
- `updateParticles(dt)` — movement, cleanup
- `update(dt)` — orchestrator calling the above + wave spawning + camera + EMP shockwave

### Rendering
- `drawShip()` — procedural ship shapes (player/ally arrow, drone triangle, gunship diamond, bruiser hexagon)
- `drawBar()` — reusable neon HUD bar
- `draw()` — stars, planets, particles, projectiles, powerups, ships, shield, EMP shockwave, HUD
- `drawHUD()` — HP/plasma/missile bars + lives + active powerup timers (top-left); score + wave (top-centre); radar (top-right, radius 80, 20 px margin)

### Game flow
- `gameOver()` — releases pointer lock, hides save UI for non-qualifying scores
- `showLeaderboard()`
- Event listeners for start/restart/submit score
- `loop()` with requestAnimationFrame

## Game systems

### Physics (uniform across all ships)

**Design principle: AIs do not cheat physics.** Enemies and allies are bound by the same movement rules as the player. They have the same thrust vocabulary, the same per-frame damping application, and the same constraint that force can only be applied in the ship's local frame. Per-class parameters (thrust magnitude, turn speed, mass-equivalent drag coefficient) can differ — different ships have different engines — but the *rules* are identical. When adding new AI behavior, it must be expressible as "which buttons to press and when," not as direct velocity or position manipulation.

- All ships (player, enemies, allies) use the same thrust vocabulary: **W ×1.0 forward, S ×0.5 reverse, A/D ×0.7 strafe** — applied in the ship's local frame. AIs pick which axes to fire based on desired heading; the player presses keys.
- Damping: player retains velocity at `0.99^(dt*60)` per second (~55%/s); enemies and allies at `0.98^(dt*60)` (~30%/s). (Different coefficients per class are allowed; they represent different ship mass/aero, not a rule exception.)
- Projectiles inherit the shooter's velocity at launch. `leadShot()` already assumes this frame, so enemy aim is consistent with actual projectile motion.
- `avoidPlanets()` is fully physics-consistent (flip-and-burn). Slow-turning ships can occasionally eat planets — intentional.
- ⚠️ **Known exception to the principle:** `steerOrbit()` applies world-frame forces independent of ship facing. Used by gunships, bruisers, and orbiting allies. Kept for combat feel, but flagged as a cheat to address later if the principle tightens.

### Enemies (4 types)
- **Drone**: fast, low hp, weak shots, red — strafing runs (charge/break/extend). Break state uses full four-axis W/S/A/D thrust to flip-and-burn past the player without ramming.
- **Gunship**: medium, orbits at preferred distance, orange
- **Bruiser**: slow, tanky, fires missiles, purple/hexagonal
- **Kamikaze**: very fast, fragile, yellow-green — charges and explodes on contact, slow turning (1 rad/s)

All ships rotate smoothly via `turnToward()`. Enemies aim at lead-shot direction and fire straight from their nose. Kamikazes defer to `avoidPlanets` when near planet danger zones (can't ram through terrain).

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
- **Homing** (12s — player missiles steer toward nearest enemy + 2x missile recharge, orange "H", missiles render orange while homing)

Powerups gravitate toward the player from any distance (inverse-square pull).

### Player
- HP slowly regenerates at 2/sec
- Pointer lock released on game over
- Leaderboard save UI only shown if score qualifies for top 5

### Ammo
- Plasma: 100 energy, costs 5/shot (15 for spread), recharges 15/sec (90/sec during spread), cooldown 0.15s/shot
- Missiles: 100 energy, costs 20/shot, recharges 8/sec (16/sec during homing)

### Audio
All sounds are synthesized via Web Audio API — no audio files. `AudioContext` (`_ac`) is lazily initialized on the first START GAME / PLAY AGAIN click to comply with browser autoplay policy.

Helpers:
- `_tone(f0, f1, dur, vol, type)` — sweeping oscillator burst
- `_noise(ftype, freq, dur, vol, Q)` — filtered noise burst (reuses a cached 0.5s white-noise buffer `_nBuf`)
- `_fm(carFreq, modFreq, modDepth, dur, vol)` — FM synthesis (sine carrier modulated by sine)

Sound functions:
- `sndPlasma(isPlayer)` — FM buzz; player louder/higher than NPCs
- `sndMissileFire()` — low sine thump + bandpass whoosh sweeping up
- `sndHit(vol)` — lowpass thud + sine drop; called at 0.15 for NPC hits, 0.30 for player hits
- `sndExplosion(big)` — noise burst; big = player death (long, low), small = NPC/ally death
- `sndImpact()` — planet/asteroid collision thud
- `sndPickup(type)` — ascending 3-note arpeggio, unique notes per powerup type
- `sndEMP()` — bandpass crack + sawtooth sweep
- `sndEngineUpdate(on)` — ramps a continuous bandpass noise chain in/out; called every frame from `updatePlayer`
