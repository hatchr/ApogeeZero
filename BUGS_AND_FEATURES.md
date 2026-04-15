# My Thoughts (For Discussion)

**Ground rules**
- This is for discussion only — do not act on anything yet.
- Let's focus on **one problem at a time**.
- Identify **high‑impact fixes first**, then refine behavior and features.

---

## 1. Feature Ideas

### Powerups
- [x] Temporary heat‑seeking missiles
  - 12s powerup ("H", orange) — player missiles steer toward nearest enemy and missile recharge doubles while active.
- [x] EMP that disables all ships temporarily (allies included)
  - EMP disables all non-player ships (both enemies and allies) for 6s — they drift and spin helplessly.
- [ ] Other ideas TBD

### Sound
- [ ] Sound effects (weapons fire, hits, explosions, powerup pickups, EMP shockwave, engine exhaust)
- [ ] Music (ambient background loop, possibly intensity-scaling with wave)
- [ ] Mute / volume controls

### World & structure
- [ ] Space stations
- [ ] Missions

### Multiplayer (speculative)
- [ ] Peer‑to‑peer multiplayer for 2 players
- [ ] More players likely require a server

---

## 2. Platform & Controls

### Controls
- [x] Current control scheme is standard for this genre
- [x] Touch / screen controls for mobile
  - Floating joystick (bottom-left zone, appears at touch point) → WASD thrust; right-side drag → ship rotation; **FIRE** / **MSL** buttons bottom-right.
  - `isMobile` uses `matchMedia('(pointer: coarse) and (hover: none)')` — correctly excludes touchscreen laptops and Windows PCs with touch drivers.
  - Controls shown/hidden via `showMobileControls()` at game start/over; invisible on desktop regardless.

### Platform
- [x] Continue as browser‑based
- [ ] Or consider a desktop build later
