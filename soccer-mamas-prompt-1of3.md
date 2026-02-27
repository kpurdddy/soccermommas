# Soccer Mamas — Alpha 1.0 Build Prompt (1 of 3)
## Foundation: Field, Players, Title Screen, Movement

**Run this first. Output: a single HTML file at soccer-mamas.html**

---

## What You're Building

A tactical soccer game as a single HTML file. React 18 via CDN, inline CSS, inline JS. No external dependencies. This prompt builds the field, players, title screen, and core movement. Later prompts add gameplay and polish.

---

## ARCHITECTURE — NON-NEGOTIABLE

All game logic must use this pattern:

1. **Single `gameState` object** holds everything: ball, players, score, clock, phase.
2. **A pure `updateGame(gameState, action)` function** processes all logic. Actions are objects like `{ type: 'TICK' }`, `{ type: 'TAP_PLAYER', playerId }`, `{ type: 'TAP_FIELD', x, y }`.
3. **React only renders.** It reads gameState, draws the pitch, captures taps, dispatches actions. No game logic in components. No setState that computes game outcomes.

This architecture exists for future multiplayer. Do not deviate.

---

## Title Screen

On load, show a title screen, vertically centered:

- **"Soccer Mamas"** — large, bold
- **"a *Nicole's a Total Baller!* Production"** — medium size, "Nicole's a Total Baller!" in italics
- **"Alpha 1.0 test build"** — small, understated

Each line visually separated. "KICK OFF" button below.

---

## Field

- Fixed rectangle, ratio ~1.5:1 (e.g., width = viewport width minus padding, height = width / 1.5)
- Responsive: recalculate on window resize
- **PIXELS_PER_UNIT constant:** On mount and on every resize, calculate `pixelsPerUnit = fieldPixelWidth / 100`. Use this single constant to convert all game-unit positions and speeds to pixel values. This ensures movement speed looks visually consistent regardless of screen size. Store this in a React ref, not in gameState (it's a rendering concern, not game logic).
- Green pitch with white lines: sidelines, endlines, halfway line, center circle, both penalty boxes, both goal areas, penalty spots, center spot, corner arcs
- Goals visible at each end (white rectangles on the endlines)
- No scrolling, no camera. The whole pitch is always visible.

---

## Players

22 player tokens — circles with position labels.

- **Home (human): blue tokens**, attacking right
- **Away (CPU): red tokens**, attacking left
- Ball carrier gets a **gold ring highlight**
- Tokens sized for tappability on mobile (minimum ~36px diameter, larger tap target radius ~48px)

### Formation (4-4-2, positions as % of field):

**Home (attacking right):**
GK(5,50) LB(20,15) CB(20,38) CB(20,62) RB(20,85) LM(45,15) CM(45,38) CM(45,62) RM(45,85) ST(70,35) ST(70,65)

**Away (attacking left, mirrored):**
GK(95,50) LB(80,85) CB(80,62) CB(80,38) RB(80,15) LM(55,85) CM(55,62) CM(55,38) RM(55,15) ST(30,65) ST(30,35)

These are "home" positions players loosely gravitate toward.

---

## Ball

- Small yellow circle, visually distinct
- Attached to carrier's position when dribbling
- During passes: travels from passer to receiver with a **faint trail** (3-4 afterimage circles with decreasing opacity)

---

## Movement System

All positions as percentages (0-100) for both x and y.

### Drift Layer (every tick, ~30fps via setInterval at 33ms)
- All 20 outfield players move continuously at **0.15 units per tick**
- Attackers drift loosely toward opponent's goal
- Defenders drift to maintain shape relative to ball position
- Goalkeepers drift laterally to track ball's Y position, staying near their goal line
- **Goalkeeper X-clamp (MANDATORY):** Home GK's X must stay between 2 and 8. Away GK's X must stay between 92 and 98. Clamp on every tick. This prevents drift logic from pulling keepers out of their box over time.
- Players gently gravitate back toward formation home positions (apply a weak pull of ~0.02 units/tick toward home pos) to prevent chaos
- Nobody ever stands still

### Burst Layer (on player input)
- **Tap field → Run:** Ball carrier moves at **0.45 units/tick (3x drift)** toward tap point. Others continue drifting. On arrival (within 2 units), carrier reverts to drift. New tap overrides previous destination.
- **Burst cooldown:** After a burst run completes (carrier arrives at destination), the carrier **cannot burst again for 2000ms (60 ticks)**. They revert to drift speed during cooldown. This prevents "jitter-tapping" to maintain permanent 3x speed, which would break the risk-distance tradeoff. Track as a `burstCooldownUntil` tick counter on the player. Tapping the field during cooldown is ignored.
- **Tap teammate → Pass:** Ball travels at **0.75 units/tick (5x drift)** to teammate. On arrival, **500ms settling window** — receiver can't burst. Show a subtle bobble animation. After settling, receiver has ball.

### Dispossession Cooldown
- When a player loses the ball (via tackle or interception), that specific player **cannot regain possession for 1500ms (45 ticks)**. Track this as a `cooldownUntil` timestamp or tick counter on the player object. During cooldown, that player is skipped when determining nearest player for loose ball pickup. This prevents "ping-pong" where the tackled player instantly sucks the ball back.

### Rendering
- Use `requestAnimationFrame` for smooth rendering
- Game tick at ~30fps (setInterval 33ms) dispatches TICK actions
- React re-renders from gameState on each animation frame
- **Render layer order (MANDATORY):** Pitch lines → Ball trail → Player tokens → Ball → Gold carrier ring → UI overlays. The ball must always render ON TOP of player tokens, never underneath. Use explicit z-index or rendering order to guarantee this.

---

## In-Game UI

Outside the pitch:
- Score: "0 — 0" (just numbers for now, scoring comes in prompt 2)
- Clock: "0:00" (clock logic comes in prompt 3, just display placeholder)
- Half: "1st Half"
- Corner text: "Soccer Mamas — Alpha 1.0 test build"

---

## Input Handling

Use pointer events (onPointerDown) for both touch and mouse.

**Mobile touch safety:** Apply `touch-action: none;` CSS to the main game container element. This prevents browser back-gestures, address bar toggles, and pinch-zoom from stealing taps on mobile.

When home team has possession:
- Tap a home player token → dispatch `{ type: 'TAP_PLAYER', playerId }`
- Tap the field → dispatch `{ type: 'TAP_FIELD', x, y }` (convert pixel coords to field %)
- Taps during settling window → ignored

For this prompt, possession is always home. CPU possession comes in prompt 2.

---

## What This Prompt Produces

A playable field where you can:
- See the title screen and tap KICK OFF
- See 22 players drifting in formation
- Tap the field to make the ball carrier run
- Tap teammates to pass (with ball trail and settling)
- See the pitch alive with continuous movement

No interceptions, no tackling, no CPU possession, no shooting, no scoring, no clock. Just the core feel of moving the ball around a living pitch.
