# Soccer Mamas — Alpha 1.0 Build Prompt (2 of 3)
## Gameplay: Interceptions, CPU Possession, Shooting, Scoring

**Run this second. Open the existing soccer-mamas.html and add to it.**

The file already has: field rendering, 22 players with drift movement, tap-to-pass with ball trail and settling, tap-to-run with burst, title screen, and the multiplayer-ready state architecture (single gameState object, pure updateGame function, React as renderer only). Do not restructure the existing architecture — add to it.

---

## Interceptions (During Passes)

When a pass is in flight, the ball moves toward the receiver on each tick. **On every tick while the ball is in flight**, check the ball's current position against every opposing player:

- If the ball's position is within **3 units** of any opposing player, calculate interception chance:
- At 3 units distance: 5% chance
- At 2 units: 15%
- At 1 unit: 40%
- At 0.5 units: 70%

**CRITICAL — Roll-Once Rule:** Each pass must have a unique `passId` (increment a counter). Each opposing player gets **exactly one interception roll per pass**. Once a player has been checked against a specific `passId` (whether they succeeded or failed), they are not checked again for that same pass. Without this, a 5% chance checked 10+ times per pass becomes near-certain interception. Track checked players as a Set on the ball object: `ball.checkedBy = new Set()`.

On successful interception: intercepting player gains possession immediately. Ball snaps to them. Clear `checkedBy`.

**Do NOT try to calculate the "closest point on the passing lane" — just check ball position vs. opponent positions each tick as the ball moves. This is simpler and more reliable.**

---

## Tackling (During Runs)

When the ball carrier is burst-running, check proximity to opposing players each tick:

- Within 3 units: tackle check
- At 3 units: 10% chance per tick
- At 2 units: 25%
- At 1 unit: 50%
- At 0.5 units: 80%

On successful tackle: ball becomes **loose** at the contact point.

---

## Loose Ball

When a tackle succeeds or a pass deflects:
- Ball sits at the contact point for ~300ms (10 ticks)
- After delay, nearest player from either team picks it up — **BUT the player who just lost possession has a 1500ms (45 tick) cooldown and cannot regain the ball during that window** (this cooldown was defined in prompt 1). Skip cooldown players when finding nearest.
- If a CPU player picks it up → possession flips to away

---

## CPU Possession

When the away team has the ball, the same drift/burst system runs but controlled by simple AI:

### CPU Decision Making — Reactive Priority Queue

The CPU uses two layers of decision-making:

**Immediate reactions (checked every tick):**
- If any home defender is within **5 units** of the CPU ball carrier:
  - **Reaction delay:** The CPU does NOT react instantly. When a defender first enters the 5-unit zone, start a **300ms (9 tick) reaction timer**. Only after the timer expires does the CPU execute the response below. If the defender leaves the zone before the timer expires, reset it. This prevents robotic instant-passing that makes defense feel pointless.
  - After reaction delay: If within shooting range AND no defender between carrier and goal → shoot
  - Else → immediately pass to nearest open teammate (prefer forward, accept lateral/backward)
  - This prevents the CPU from running into defenders or out of bounds while "waiting" for a scheduled decision

**Scheduled strategy (every ~60 ticks / ~2 seconds, only if no immediate threat):**
- **70%:** Pass to the nearest teammate who is further toward the home goal (forward pass)
- **20%:** Pass to nearest teammate regardless of direction (safe pass)
- **10%:** Burst run toward home goal (dribble attempt)

CPU passes have **15% error chance** — the pass target is offset by 2-3 random units, making it slightly inaccurate. This creates organic near-misses and occasional turnovers.

### Human Defense During CPU Possession
- Home defenders continue drifting automatically (maintaining shape relative to ball)
- **Human can tap their own defenders** to sprint them toward the ball carrier
- This commits the defender — they burst toward the ball but leave space behind
- If human does nothing, the defense holds shape but is passive

### CPU Interception/Tackle Rules
The same interception and tackle rules apply in reverse — home defenders can intercept CPU passes and tackle CPU ball carriers using the same proximity checks.

---

## Shooting

When the ball carrier (either team) is within **25 x-units of the opponent's goal line**:

### Human Shooting
- Tapping the goal area (the endline near the opponent's goal) triggers a shot
- Add a visual indicator: when in shooting range, the goal area gets a subtle highlight

### CPU Shooting
- When the CPU ball carrier is within 25 x-units and has a relatively clear path (no defender within 3 units directly between them and goal), they shoot with 30% probability on each decision tick

### Shot Resolution
Calculate success probability:
- **Distance factor:** Linear from 50% at 5 x-units to 5% at 25 x-units
- **Angle factor:** Multiply by 0.5 (tight angle, Y far from 50) to 1.0 (central, Y near 50)
- **Defender factor:** If nearest opponent within 3 units, multiply by 0.7

Roll against this probability:
- **Goal:** Ball moves quickly toward goal, crosses line. Freeze play for 1.5 seconds. Display "GOAL!" large and centered. Update score. Then reset all players to formation positions for kickoff. Possession goes to the team that conceded.
- **Save:** Goalkeeper slides toward ball trajectory. Ball rebounds to a loose ball position ~10 units in front of goal.
- **Wide:** Ball goes past the endline. Possession to defending team, ball placed at (8, 50) or (92, 50) for a goal kick.

### Goalkeeper Behavior
- Keepers already drift to track ball Y position (from prompt 1)
- On a shot, the keeper moves quickly toward the ball's projected Y position
- Visual: keeper slides laterally during the shot sequence
- Save probability is the complement of shot success (if shot has 30% chance of scoring, keeper saves 65% of the time, 5% goes wide)

---

## Kickoff After Goals

- All 22 players reset to formation positions
- **Clear all velocities, burst targets, cooldown timers, `lockoutTimer`, `ball.lastCarrierId`, and `ball.checkedBy`** before resetting positions (same purge as halftime — prevents players carrying stale movement intent or lockout states into the restart)
- Ball at center spot (50, 50)
- Possession to the team that conceded
- 1 second pause, then play resumes automatically

---

## Updated Input Handling

Inputs now depend on game phase and possession:

**Home possession:**
- Tap home teammate → pass
- Tap field → run
- Tap goal area (when in range) → shoot
- Tap away player → nothing

**Away possession (CPU has ball):**
- Tap home defender → that defender bursts toward ball carrier
- Tap field → nothing
- Tap away player → nothing

**Loose ball / goal celebration / kickoff pause:**
- All taps ignored

---

## What This Prompt Adds

**Reminder: Maintain the render layer order from prompt 1:** Pitch lines → Ball trail → Player tokens → Ball → Gold carrier ring → UI overlays. The ball must always be visible on top of players.

The game now has:
- Passes can be intercepted
- Runners can be tackled
- CPU takes possession and plays soccer (passes, runs, shoots)
- Human can influence defense by committing defenders
- Both teams can shoot and score
- Goals update the score and trigger kickoff restarts
- The core gameplay loop is complete
