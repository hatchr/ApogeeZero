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
- [ ] Touch / screen controls for mobile remain an open design topic
  - Mobile is detected (`isMobile` flag, reduced particle cap) but no touch event handlers are registered yet.

### Platform
- [x] Continue as browser‑based
- [ ] Or consider a desktop build later
