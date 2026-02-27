# Soccer Mamas -- Build Prompt: Alpha 1.1
Version: Alpha 1.1 (from 1.0.1)
Date: 2026-02-26
Source file: index.html
Location: F:\NicBall\

Estimated tokens: ~12-15k
Estimated time: ~12 minutes

---

## Pre-Build Checklist
1. Backup current file: `cp index.html index-1.0.1-backup.html`
2. Read the current index.html before making changes
3. Do NOT rewrite the whole file. These are targeted fixes to existing functions.

---

## Fix 1: Viewport-Responsive Field Sizing

**Problem:** The field renders as a small rectangle on phones instead of filling the screen. The `calcDims()` function (around line 998) uses `window.innerWidth` and `window.innerHeight` which don't account for mobile browser chrome (address bar, bottom toolbar).

**Fix:**
- Change the game container CSS from `height: 100vh` to `height: 100dvh` (dynamic viewport height)
- In `calcDims()`, use `window.visualViewport?.width || window.innerWidth` and `window.visualViewport?.height || window.innerHeight` for more accurate mobile sizing
- Reduce the HUD height allowance from 60px to 44px (the HUD doesn't need that much)
- The field aspect ratio stays 1.5:1
- `pixelsPerUnit = fieldPixelWidth / 100` stays the same
- Make sure the field is edge-to-edge minus minimal padding (8px each side, 16px total)

**Do NOT change:**
- The coordinate system (0-100 game units)
- The SVG rendering approach
- The aspect ratio (1.5:1)

---

## Fix 2: Shot Aiming with Tap Coordinates

**Problem:** When the player taps the goal area to shoot, `handleFieldPointerDown` (around line 1059) dispatches `{ type: 'SHOOT' }` with NO coordinates. The `executeShot` function (line 655) then randomly generates `targetY` for the shot. The player has zero control over where the ball goes.

**Fix -- three parts:**

### Part A: Pass tap coordinates through
In `handleFieldPointerDown`, change the SHOOT dispatch from:
```
dispatch({ type: 'SHOOT' });
```
to:
```
dispatch({ type: 'SHOOT', aimX: gx, aimY: gy });
```

### Part B: Update the action handler
In `updateGame` (line 200), change:
```
case 'SHOOT': return executeShot(state, state.ball.carrier);
```
to:
```
case 'SHOOT': return executeShot(state, state.ball.carrier, action.aimX, action.aimY);
```

### Part C: Use aim coordinates in executeShot
Change `executeShot(state, carrierId)` to `executeShot(state, carrierId, aimX, aimY)`.

Replace the random `targetY` generation (around lines 694-697) with logic that uses the player's tap:
- `targetY` should be based on `aimY` clamped to the goal mouth range (roughly y=38 to y=62 -- the goal posts)
- Add scatter/error based on distance and defensive pressure. The further the shot, the more the ball deviates from where the player aimed. Close shots are accurate; long shots scatter more.
- Scatter formula -- use this exact math:
```
var distToGoal = Math.abs(goalX - carrier.x);
var scatterRange = Math.max(0, (distToGoal - 5) * 0.5);
targetY = aimY + (Math.random() - 0.5) * scatterRange;
```
This gives 0 scatter at 5 units (point blank is precise), scaling up to ~10 scatter at 25 units (long shots deviate). Do NOT invent a different formula.
- The result determination (goal/save/wide) should factor in WHERE the player aimed: shots aimed at corners are harder to save but more likely to go wide. Shots aimed at center are easier for the keeper but less likely to miss.
- Keeper behavior: the keeper's starting Y position matters. If the player aims away from the keeper, the save chance drops. If they aim right at the keeper, save chance increases.

For CPU shots (called from `cpuPossession` and `canCpuShoot`), generate random aimX/aimY values targeting the goal area so the CPU uses the same system. In `cpuPossession` where it calls `executeShot(s, carrier.id)`, change to pass random aim coordinates within the goal mouth.

---

## Fix 3: Ghost Ball -- Hard Collision on Passes

**Problem:** In `updatePassBall` (line 279), when a pass travels near a defender, the interception check is probabilistic. Even at 0.5 units distance, there's a 30% chance the ball phases right through the defender. Visually this looks absurd -- your player is standing in the passing lane and the ball goes through them.

**Fix:**
In the interception check loop inside `updatePassBall` (around lines 333-354), add a hard collision override BEFORE the random roll:

```
// Hard collision: if ball is within inner radius, automatic interception
if (pDist <= 1.0) {
  // Ball is ON the defender -- automatic interception, no dice roll
  p.hasBall = true;
  p.settlingUntil = s.tick + SETTLING_TICKS;
  s.ball.carrier = p.id;
  s.ball.x = p.x;
  s.ball.y = p.y;
  s.ball.loose = false;
  s.possession = p.team;
  s.passBall = null;
  s.cpuThreatDetectedTick = null;
  s.cpuNextStrategyTick = s.tick + 30;
  return s;
}
```

Keep the existing probabilistic check for distances 1.0 to 3.0. Only the very close range (ball literally passing through the player's space) becomes automatic.

The `checkedBy` set still applies -- each defender only gets checked once per pass. But if the ball enters their inner radius, it's an automatic interception regardless.

**Important:** This applies to ALL passes -- human and CPU. The same `updatePassBall` function handles both, so this fix is automatic for both sides.

---

## Fix 4: CPU Ping-Pong Passing

**Problem:** In `cpuPass` (line 591), the 'safe' pass type picks the closest teammate. After Player A passes to Player B, Player A is still the closest teammate to Player B. So the CPU passes right back. This loops visibly and looks robotic.

**Fix:**
In `cpuPass`, when building the `teammates` array (line 592), exclude `s.ball.lastCarrierId`:

Change:
```
const teammates = s.players.filter(p => p.team === 'away' && p.id !== carrier.id);
```
to:
```
const teammates = s.players.filter(function(p) {
  if (p.team !== 'away') return false;
  if (p.id === carrier.id) return false;
  if (p.id === s.ball.lastCarrierId) return false;
  return true;
});
```

Also, when the CPU initiates a pass (around line 621 where `carrier.hasBall = false`), set `s.ball.lastCarrierId = carrier.id` so the exclusion works:

After `carrier.hasBall = false;` add:
```
s.ball.lastCarrierId = carrier.id;
```

**Fallback:** If excluding `lastCarrierId` leaves zero teammates, fall back to the full list (allow the pass-back as a last resort). Add after the filter:
```
if (teammates.length === 0) {
  teammates = s.players.filter(function(p) { return p.team === 'away' && p.id !== carrier.id; });
}
```

**Note:** `s.ball.lastCarrierId` already exists in the initial state object (in `createInitialState`, initialized as `null`) and is already set during tackles in `checkTackles`. Confirm this before editing -- do NOT create a duplicate field. The only new assignment you're adding is in `cpuPass` when the CPU initiates a pass.

Note: use `function()` declarations, not arrow functions. The codebase uses `function()` throughout.

---

## Fix 5: Add TacFootball Link to Landing Page

**Problem:** The "Other testing alphas" section (around line 1156) has two links. A third link was requested and is missing.

**Fix:**
In the `alpha-links` div (around line 1158), add a third link after the existing two:

```
React.createElement('a', { href: 'https://kpurdddy.github.io/TacFootball/', target: '_blank', rel: 'noopener noreferrer' }, 'TacFootball (tactical football -- earlier build)')
```

The three links should be:
1. Soccer Mamas (existing)
2. Hail Marys (existing)
3. TacFootball (new)

---

## Fix 6: Coaching Sandbox Text Update

**Problem:** The Sandbox Mode card on the landing page (around line 1141) says "Sandbox Mode" and has a generic description. It needs to be renamed and rewritten.

**Fix:**
Change the h3 from `'Sandbox Mode'` to `'Coaching Sandbox'`.

Change the paragraph text to:
`'Drag players into any formation. Use it as a digital whiteboard to show positioning, or hit play and run the game from that setup -- single player against the CPU or two-player. Show your players where to be, then show them why.'`

---

## Fix 7: Version Number Update

Update all version references from 1.0.1 to 1.1:
- Title screen version text (line 1111): change `'Alpha 1.0.1 test build'` to `'Alpha 1.1 test build'`
- In-game version display (line 1356): change `'Soccer Mamas -- Alpha 1.0.1'` to `'Soccer Mamas -- Alpha 1.1'`

---

## Code Style Reminders
- Use `function()` declarations, NOT arrow functions
- Use `React.createElement()`, NOT JSX
- Do NOT rewrite the whole file -- make targeted edits to existing functions
- If approaching the output limit, stop at a clean breakpoint, save the file, and note what remains

---

## Post-Build Requirements
1. Save the file
2. `git add -A`
3. `git commit -m "Alpha 1.1: viewport fix, shot aiming, ghost ball fix, CPU passing fix, landing page updates"`
4. `git push origin main` (if rejected: `git pull --rebase origin main` then push again)
5. Update NOTES.md
6. Print DONE summary

---

## Playtest Checklist

After build, verify ALL of these:

### Viewport
1. Open on a phone (or use Chrome DevTools mobile view). The field should fill most of the screen, not render as a small rectangle.
2. Resize the browser window. The field should resize responsively.
3. The HUD (score, clock, half indicator) should be visible above the field.

### Shot Aiming
4. Get the ball into shooting range (the gold highlight zone appears).
5. Tap LEFT side of the goal area. The shot should go toward the left.
6. Tap RIGHT side of the goal area. The shot should go toward the right.
7. Tap CENTER of the goal area. The shot should go roughly center.
8. Take several shots -- they should NOT all go to the same spot. Scatter should be visible, especially from distance.

### Ghost Ball
9. Position a defender (tap to commit) directly between a CPU passer and their target. The pass should be intercepted, not phase through.
10. Multiple interception tests -- balls that travel THROUGH a defender's position should be caught every time at very close range.

### CPU Passing
11. Watch the CPU with the ball for 30+ seconds. They should NOT pass back and forth between the same two players repeatedly.
12. The CPU should cycle the ball to different teammates and generally move forward.

### Landing Page
13. Scroll down on the title screen. Three links should appear under "Other testing alphas": Soccer Mamas, Hail Marys, TacFootball.
14. The Sandbox section should say "Coaching Sandbox" with the updated description.
15. Version shows "Alpha 1.1 test build" on title screen.

### Regression Checks
16. Start a game, play through to halftime. Tap to continue. Second half starts correctly with sides switched.
17. Score a goal (or let CPU score). Kickoff restarts correctly.
18. Ball trail still visible on passes.
19. Gold carrier ring still visible on ball carrier.
20. Game clock still counts correctly.
