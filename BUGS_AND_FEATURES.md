# My Thoughts (For Discussion)

**Ground rules**
- This is for discussion only — do not act on anything yet.
- Let's focus on **one problem at a time**.
- Identify **high‑impact fixes first**, then refine behavior and features.

---

## 1. Performance: High‑Confidence, High‑ROI Improvements

These are ideas I believe are promising, listed in priority order.

### ⚠️ 1. Add simple object pooling for projectiles
- Reuse projectile objects instead of allocating new ones
- Mainly helps with GC spikes during long sessions
- ✅ Useful if stutter remains after the above
- ⚠️ Moderate refactor; not first‑line

---

## 2. Performance: Remaining Unknowns Worth Discussing

These are areas where we **don't yet know** the primary cause or best solution.

### Gravity model & world complexity
- Gravity is calculated from multiple planets/asteroids — is that still too expensive after plasma gravity is removed?
- Would reducing the **number of planets/asteroids** materially help performance?
- Would a simplified gravity model help?
  - Aggregate / approximate gravity
  - Spatial partitioning
  - Favor efficiency over precision

### Entity & projectile counts
- Is slowdown driven more by:
  - Enemy count?
  - Projectile count?
- Would gameplay changes help performance?
  - Bosses replacing raw enemy count escalation

### General diagnosis
- Are there other likely root causes of slowdown starting around wave 7?
- Which remaining fixes are most likely to produce **order‑of‑magnitude** improvements?

---

## 3. Enemy Navigation & Combat Behavior

### Planet avoidance ✅ IMPLEMENTED
- ~~Enemies often fly directly into planets when the planet is between them and the player.~~
- `avoidPlanets()` is implemented with quadratic urgency scaling, radial repulsion, and tangential steering to curve around planets toward a navigation target. Force magnitude is returned to suppress attack steering when avoidance is critical.
- Applies to both enemies and allies.

### Combat effectiveness ("Stormtrooper effect") ✅ IMPLEMENTED
- ~~A stationary player is often very safe.~~
- ~~Enemies orbit and spray projectiles but rarely hit.~~
- `leadShot()` is implemented: enemies (and allies) solve a quadratic intercept equation using relative velocities to aim ahead of the player's position.

---

## 4. Feature Ideas (Explicitly Low Priority)

### Powerups
- Temporary heat‑seeking missiles ✅ IMPLEMENTED
  - 12s powerup ("H", orange) — player missiles steer toward nearest enemy and missile recharge doubles while active.
- EMP that disables all ships temporarily (allies included) ✅ IMPLEMENTED
  - EMP disables all non-player ships (both enemies and allies) for 6s — they drift and spin helplessly.
- Other ideas TBD

### World & structure
- Space stations
- Missions

### Multiplayer (speculative)
- Peer‑to‑peer multiplayer for 2 players
- More players likely require a server

---

## 5. Platform & Controls

### Controls
- Current control scheme is standard for this genre
- Touch / screen controls for mobile remain an open design topic
  - Mobile is detected (`isMobile` flag, reduced particle cap) but no touch event handlers are registered yet.

### Platform
- Continue as browser‑based
- Or consider a desktop build later
