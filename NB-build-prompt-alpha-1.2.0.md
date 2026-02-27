# Soccer Mamas -- Build Prompt: Alpha 1.2.0
Version: Alpha 1.2.0 (from 1.1.0)
Date: 2026-02-26
Source file: index.html
Location: F:\NicBall\

Estimated tokens: ~15-18k
Estimated time: ~15 minutes

---

## Pre-Build Checklist
1. Backup current file: `cp index.html index-1.1.0-backup.html`
2. Read the current index.html before making changes
3. Do NOT rewrite the whole file. These are targeted fixes to existing functions.
4. Use `function()` declarations, NOT arrow functions
5. Use `React.createElement()`, NOT JSX
6. If approaching the output limit, stop at a clean breakpoint, save the file, and note what remains

---

## Fix 1: Ghost Ball -- Interception Loop Reorder

**Problem:** In `updatePassBall` (around line 333), the `checkedBy` gate runs BEFORE the hard collision check. When a ball enters a defender's 3-unit range, the defender gets added to `checkedBy`. If the probabilistic roll fails, the ball keeps moving. On the next tick, even if the ball is within 1.0 units (hard collision range), `checkedBy.has(p.id)` is true and the entire check is skipped. The hard collision never fires.

**Fix:** Reorder the loop so hard collision evaluates BEFORE the `checkedBy` gate. Also bump the hard collision radius from 1.0 to 2.0 to better match the visual token size.

Replace the interception loop inside `updatePassBall` (lines ~333-369) with this exact structure:

```
  for (var i = 0; i < s.players.length; i++) {
    var p = s.players[i];
    if (p.team !== opposingTeam) continue;

    var pDist = getDist(p.x, p.y, pb.x, pb.y);

    // 1. Hard collision: ALWAYS evaluate first, even if previously checked
    if (pDist <= 2.0) {
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

    // 2. Out of range -- skip
    if (pDist > 3) continue;

    // 3. Probabilistic check: only evaluate once per pass
    if (pb.checkedBy.has(p.id)) continue;
    pb.checkedBy.add(p.id);

    if (Math.random() < getInterceptionChance(pDist)) {
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
  }
```

**Critical:** The hard collision check (pDist <= 2.0) runs BEFORE `checkedBy.has()`. This is the entire point of the fix. Do NOT put the `checkedBy` check first.

---

## Fix 2: Viewport Dynamic Sizing

**Problem:** The field is locked to 1.5:1 aspect ratio. Modern phones in landscape are ~2.1:1. The field ends up small with big dark margins on the sides.

**Fix:** Replace the `calcDims` function (around line 1055) with:

```
  var calcDims = useCallback(function() {
    var pad = 8;
    var vw = window.visualViewport ? window.visualViewport.width : window.innerWidth;
    var vh = window.visualViewport ? window.visualViewport.height : window.innerHeight;

    var maxW = vw - pad * 2;
    var maxH = vh - 44 - pad * 2;

    // Unlock aspect ratio: use all available space
    var w = maxW;
    var h = maxH;

    // Clamp ratio between 1.4:1 and 2.4:1
    if (w / h < 1.4) h = w / 1.4;
    if (w / h > 2.4) w = h * 2.4;

    pixelsPerUnitRef.current = w / 100;
    setFieldDims({ width: w, height: h });
  }, []);
```

**Note:** The `ypu` (Y pixels per unit) used in rendering is already calculated as `fieldHeight / 100` independently from `ppu` (which is `fieldWidth / 100`). Confirm this still works correctly -- player Y positions and pitch line Y positions should use `ypu`, not `ppu`, for vertical measurements. Check `renderPitchLines` and `renderPlayer` to confirm they already use both `ppu` and `ypu` separately. They do in the current code (line 1420+), so this should be fine.

---

## Fix 3: Post-Tackle Cooldown Applies to Tackles

**Problem:** In `checkTackles` (line 486), when a player tackles and wins the ball, the tackled player gets `cooldownUntil` set. But `checkTackles` never checks `cooldownUntil` on potential tacklers. So the player who just lost the ball can immediately tackle back. The dispossession cooldown only blocks loose ball pickup, not re-tackling.

**Fix:** Add a cooldown check at the top of the tackle loop. In `checkTackles`, after `if (p.team !== opposingTeam) continue;` add:

```
    if (p.cooldownUntil > s.tick) continue;
```

This means a player who just lost possession can't tackle for 1500ms (DISPOSSESSION_COOLDOWN_TICKS). One line, right after the team check.

---

## Fix 4: Auto-Burst Away From Tackler

**Problem:** When a player wins the ball through a tackle, they stand still right next to the tackled player. There's no separation, so the opponent (after cooldown expires) or nearby opponents immediately challenge again. The player who won the ball has no breathing room.

**Fix:** In `checkTackles`, after the tackle succeeds (after `s.ball.lastCarrierId = carrier.id;` on line 503), add auto-burst logic for the tackler:

```
      // Auto-burst: winning player moves away from contact point
      var dx = p.x - carrier.x;
      var dy = p.y - carrier.y;
      var awayDist = Math.sqrt(dx * dx + dy * dy) || 1;
      p.burstTarget = {
        x: Math.max(1, Math.min(99, p.x + (dx / awayDist) * 5)),
        y: Math.max(1, Math.min(99, p.y + (dy / awayDist) * 5))
      };
```

This sends the ball winner 5 units away from the contact point in the direction they came from. Creates visible separation. The `p` here is the tackler (the one who wins the ball).

**Wait -- the tackle currently makes the ball loose, not giving it directly to the tackler.** Looking at the code, `checkTackles` doesn't give the ball to the tackling player -- it sets `s.ball.loose = true` and lets `updateLooseBall` assign it to the nearest player. So the auto-burst needs to be handled differently.

**Revised approach:** Instead of auto-bursting the tackler (who doesn't have the ball yet), change `checkTackles` so that successful tackles give the ball directly to the tackling player instead of making it loose:

Replace the tackle success block (lines 497-504) with:

```
      // Tackle successful -- ball goes directly to tackler
      carrier.hasBall = false;
      carrier.burstTarget = null;
      carrier.cooldownUntil = s.tick + DISPOSSESSION_COOLDOWN_TICKS;

      p.hasBall = true;
      p.settlingUntil = s.tick + SETTLING_TICKS;
      s.ball.carrier = p.id;
      s.ball.x = p.x;
      s.ball.y = p.y;
      s.ball.loose = false;
      s.ball.lastCarrierId = carrier.id;
      s.possession = p.team;
      s.cpuThreatDetectedTick = null;
      s.cpuNextStrategyTick = s.tick + 30;

      // Auto-burst: winning player moves away from contact
      var dx = p.x - carrier.x;
      var dy = p.y - carrier.y;
      var awayDist = Math.sqrt(dx * dx + dy * dy) || 1;
      p.burstTarget = {
        x: Math.max(1, Math.min(99, p.x + (dx / awayDist) * 5)),
        y: Math.max(1, Math.min(99, p.y + (dy / awayDist) * 5))
      };

      return s;
```

This is a behavior change: tackles are now clean dispossessions (ball goes to tackler) rather than loose ball events. The tackled player can't get it back because (a) they have a cooldown, and (b) the tackler auto-bursts away. This feels more like a real tackle.

---

## Fix 5: "SAVE" Text Overlay

**Problem:** When the keeper saves a shot, there's no visual feedback. The ball animates toward the goal, the keeper slides over, but no text appears. The player thinks they scored and is confused when no goal registers.

**Fix -- two parts:**

### Part A: Add a save_display phase
In `updateShotAnimation` (around line 795), where the save result is handled, instead of immediately returning to play:

Change the save block from:
```
    } else if (s.shot.result === 'save') {
      const reboundX = s.shot.targetX < 50 ? 10 : 90;
      s.ball.x = reboundX;
      s.ball.y = 40 + Math.random() * 20;
      s.ball.loose = true;
      s.ball.looseTimer = s.tick + LOOSE_BALL_TICKS;
      s.ball.lastCarrierId = null;
      s.phase = 'play';
      s.shot = null;
```

to:
```
    } else if (s.shot.result === 'save') {
      var reboundX = s.shot.targetX < 50 ? 10 : 90;
      s.ball.x = reboundX;
      s.ball.y = 40 + Math.random() * 20;
      s.ball.loose = true;
      s.ball.looseTimer = s.tick + LOOSE_BALL_TICKS;
      s.ball.lastCarrierId = null;
      s.phase = 'save_display';
      s.saveDisplayTimer = s.tick + 30; // ~1 second display
      s.shot = null;
```

### Part B: Handle save_display phase in gameTick
Add a handler in `gameTick` (after the goal_celebration handler, around line 230):

```
  if (state.phase === 'save_display') {
    var s2 = { ...state, tick: state.tick + 1 };
    if (s2.tick >= s2.saveDisplayTimer) {
      return { ...s2, phase: 'play', clock: { ...s2.clock, running: true } };
    }
    return s2;
  }
```

### Part C: Add save_display to initial state
In `createInitialState` (around line 172), add `saveDisplayTimer: 0` to the returned state object.

### Part D: Render the SAVE overlay
In the rendering section, after the GOAL! overlay (around line 1345), add:

```
      // SAVE overlay
      gameState.phase === 'save_display' ? React.createElement('div', {
        key: 'save-overlay',
        style: {
          position: 'absolute', top: 0, left: 0, width: '100%', height: '100%',
          display: 'flex', alignItems: 'center', justifyContent: 'center',
          pointerEvents: 'none',
        }
      }, React.createElement('div', {
        style: {
          fontSize: Math.max(36, fw * 0.08) + 'px', fontWeight: 'bold',
          color: 'white', textShadow: '0 0 15px rgba(255,255,255,0.6), 0 4px 8px rgba(0,0,0,0.5)',
          letterSpacing: '6px', fontFamily: 'Arial, sans-serif',
        }
      }, 'SAVE!')) : null,
```

---

## Fix 6: Defensive Controls -- Two-Tap System

**Problem:** When the CPU has the ball, tapping a defender sends them charging at the ball carrier. There's no way to position defenders -- block a passing lane, drop back, cover space. Defense is "send guys at the ball" which is exactly what bad defenders do.

**Fix -- four parts:**

### Part A: Add selectedDefender to game state
In `createInitialState` (line 172), add to the returned object:
```
    selectedDefender: null,
```

Also add `selectedDefender: null` in `resetForKickoff` and `startSecondHalf` return objects to prevent stale selections carrying across phase transitions.

### Part B: Add new action types
In `updateGame` (line 194), add two new cases:

```
    case 'SELECT_DEFENDER': return selectDefender(state, action.playerId);
    case 'POSITION_DEFENDER': return positionDefender(state, action.x, action.y);
```

### Part C: New functions
Add these two functions after `tapDefender`:

```
function selectDefender(state, playerId) {
  if (state.phase !== 'play') return state;
  if (state.possession !== 'away') return state;

  var defender = state.players.find(function(p) { return p.id === playerId; });
  if (!defender || defender.team !== 'home') return state;

  // If tapping the already-selected defender, deselect
  if (state.selectedDefender === playerId) {
    return { ...state, selectedDefender: null };
  }

  return { ...state, selectedDefender: playerId };
}

function positionDefender(state, x, y) {
  if (state.phase !== 'play') return state;
  if (state.possession !== 'away') return state;
  if (!state.selectedDefender) return state;

  var defender = state.players.find(function(p) { return p.id === state.selectedDefender; });
  if (!defender || defender.team !== 'home') return { ...state, selectedDefender: null };
  if (defender.burstCooldownUntil > state.tick) return { ...state, selectedDefender: null };

  var s = { ...state, players: state.players.map(function(p) { return { ...p }; }) };
  var d = s.players.find(function(p) { return p.id === state.selectedDefender; });
  d.burstTarget = { x: x, y: y };
  s.selectedDefender = null;
  return s;
}
```

### Part D: Update handleFieldPointerDown
Replace the defensive input section (around lines 1142-1159) -- the `else if (state.possession === 'away')` block -- with:

```
    } else if (state.possession === 'away') {
      // Check if tap is on a home player
      var tappedPlayer = null;
      var minDist = Infinity;
      for (var i = 0; i < state.players.length; i++) {
        var p = state.players[i];
        if (p.team !== 'home') continue;
        var ppx = p.x * ppu;
        var ppy = p.y * (fieldDims.height / 100);
        var dist = Math.sqrt((px - ppx) * (px - ppx) + (py - ppy) * (py - ppy));
        if (dist < tapRadius && dist < minDist) {
          minDist = dist;
          tappedPlayer = p;
        }
      }

      if (state.selectedDefender) {
        // A defender is already selected
        if (tappedPlayer) {
          // Tapped another player
          var carrier = state.players.find(function(p) { return p.id === state.ball.carrier; });
          if (carrier && tappedPlayer.id === carrier.id) {
            // Tapped the ball carrier -- commit tackle
            // First, burst the selected defender at the carrier
            dispatch({ type: 'TAP_DEFENDER', playerId: state.selectedDefender });
          } else if (tappedPlayer.team === 'home') {
            // Tapped a different home player -- switch selection
            dispatch({ type: 'SELECT_DEFENDER', playerId: tappedPlayer.id });
          }
        } else {
          // Tapped empty field -- position the selected defender there
          dispatch({ type: 'POSITION_DEFENDER', x: gx, y: gy });
        }
      } else {
        // No defender selected yet
        if (tappedPlayer && tappedPlayer.team === 'home') {
          dispatch({ type: 'SELECT_DEFENDER', playerId: tappedPlayer.id });
        }
      }
    }
```

**The flow:**
1. Tap a blue defender -- they get selected (gold highlight ring)
2. Tap empty field -- selected defender moves to that position (cover space, block lane)
3. Tap the red ball carrier -- selected defender commits to tackle (charges at carrier)
4. Tap a different blue defender -- selection switches to that defender
5. Tap the selected defender again -- deselects

### Part E: Visual indicator for selected defender
In the rendering section, after the carrier ring (around line 1318), add a selected defender ring:

```
      // Selected defender ring
      gameState.selectedDefender ? (function() {
        var sd = gameState.players.find(function(p) { return p.id === gameState.selectedDefender; });
        if (!sd) return null;
        return React.createElement('circle', {
          key: 'selected-defender-ring',
          cx: sd.x * ppu, cy: sd.y * ypu,
          r: Math.max(20, ppu * 2.5),
          fill: 'none', stroke: '#00ff88',
          strokeWidth: 3,
          opacity: 0.7 + 0.3 * Math.sin(gameState.tick * 0.3),
          strokeDasharray: '6 3',
        });
      })() : null,
```

A pulsing green dashed ring around the selected defender. Visually distinct from the gold carrier ring.

### Part F: Clear selected defender on possession change
In `updateLooseBall` (line 510), when possession changes (line 531: `s.possession = nearestPlayer.team`), also add:
```
      s.selectedDefender = null;
```

Same in `updatePassBall` when an interception happens (around line 349/367 -- both interception blocks):
```
      s.selectedDefender = null;
```

In `checkTackles` when a tackle succeeds, add `s.selectedDefender = null;` to the tackle success block.

In `checkOutOfBounds` -- both the sideline block (around line 871) and the endline block (around line 908), wherever possession changes, add `s.selectedDefender = null;`. If the CPU runs out of bounds and possession flips to human, a lingering selected defender ring would cause input confusion.

---

## Fix 7: Version Text Prominent in HUD

**Problem:** Version text is at 30% opacity in the bottom-right corner. Invisible on the green field.

**Fix -- two parts:**

### Part A: Remove the old version text
Delete the version-text div at the end of the game rendering (line 1416):
```
React.createElement('div', { className: 'version-text' }, 'Soccer Mamas -- Alpha 1.1')
```

### Part B: Add version to HUD
In the HUD bar (around line 1253), add a version span. Change the HUD from three spans to include a fourth:

After the clock span (line 1259), add:
```
      React.createElement('span', {
        style: { color: 'rgba(255,255,255,0.6)', fontSize: '0.7em' }
      }, 'Alpha 1.2.0 -- test build')
```

This puts the version right in the HUD bar where it's always visible.

### Part C: Update title screen version
On the title screen (around line 1170), change the version text from `'Alpha 1.1 test build'` to `'Alpha 1.2.0 -- test build'`.

---

## Fix 8: Landing Page Spacing and Button Sizing

**Problem:** Too much empty space between Kick Off button and the feedback section. Feedback button is the same visual weight as Kick Off.

**Fix:**

### Part A: Reduce title screen height
In CSS (line 16), change `.title-screen` from `min-height: 70vh` to `min-height: 50vh`. This pulls the content below closer to Kick Off.

### Part B: Shrink feedback button
In CSS (line 54), change `.feedback-btn` from:
```
padding: 14px 40px; ... font-size: 1.2em;
```
to:
```
padding: 10px 28px; ... font-size: 1.0em;
```

The feedback button should clearly be secondary to Kick Off.

---

## Code Style Reminders
- Use `function()` declarations, NOT arrow functions
- Use `React.createElement()`, NOT JSX
- Use `var` for variable declarations inside the new code (consistent with the function-style approach)
- Do NOT rewrite the whole file -- make targeted edits to existing functions
- If approaching the output limit, stop at a clean breakpoint, save the file, and note what remains

---

## Post-Build Requirements
1. Save the file
2. `git add -A`
3. `git commit -m "Alpha 1.2.0: ghost ball fix, viewport dynamic sizing, tackle rework, defensive controls, save overlay, HUD version, landing page polish"`
4. `git push origin main` (if rejected: `git pull --rebase origin main` then push again)
5. Update NOTES.md with version 1.2.0 entry
6. Print DONE summary

---

## Playtest Checklist

### Ghost Ball (Fix 1)
1. Position a blue defender directly between two red players. When red passes, the ball should be intercepted -- NOT phase through.
2. Test multiple times. At 2.0 unit radius, a defender standing in the passing lane should catch virtually every pass through their space.
3. Test that the human team's passes can also be intercepted by red defenders (same system applies to both teams).

### Viewport (Fix 2)
4. Open on a phone in landscape. The field should fill nearly the entire screen with minimal dark margins.
5. Open on desktop in a wide browser window. Field should fill the height with the ratio clamped at 2.4:1 max.
6. Open on an iPad-shaped window. Field should fill with ratio clamped at 1.4:1 min.
7. Resize the browser window. Field should resize dynamically.

### Tackle Rework (Fix 3 + 4)
8. When a blue player tackles a red player, the blue player should auto-burst away from the contact point with the ball. Clear visible separation.
9. The red player who was tackled should NOT immediately re-tackle. They should be locked out for ~1.5 seconds.
10. When a red player tackles a blue player, same behavior -- red gets the ball and bursts away.

### Save Overlay (Fix 5)
11. Take a shot that gets saved. "SAVE!" text should appear on the field for ~1 second.
12. After the save text clears, play resumes with a loose ball.
13. Take a shot that scores. "GOAL!" should still appear (regression check).

### Defensive Controls (Fix 6)
14. When red has the ball, tap a blue defender. A green dashed ring should appear around them.
15. Tap empty field. The selected defender should move to that position.
16. Select a defender, then tap the red ball carrier. The defender should charge at the carrier (tackle commit).
17. Select a defender, then tap a different blue player. Selection should switch.
18. Tap the already-selected defender. Selection should clear (ring disappears).
19. When possession changes (interception, tackle), the selected defender ring should clear.

### Version Text (Fix 7)
20. During gameplay, "Alpha 1.2.0 -- test build" should be visible in the HUD bar at the top of the screen.
21. On the title screen, version should say "Alpha 1.2.0 -- test build".

### Landing Page (Fix 8)
22. Less gap between Kick Off button and the feedback section.
23. Feedback button should be visually smaller than Kick Off button.

### Regression Checks
24. Score a goal. Kickoff restarts correctly.
25. Play to halftime. Tap to continue. Sides switch. Second half works.
26. Ball trail visible on passes.
27. Gold carrier ring visible on ball carrier.
28. Game clock counts correctly.
29. Out of bounds handling still works.
30. Three links on landing page: Soccer Mamas, Hail Marys, TacFootball.
