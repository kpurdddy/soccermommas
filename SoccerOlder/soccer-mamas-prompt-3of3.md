# Soccer Mamas — Alpha 1.0 Build Prompt (3 of 3)
## Game Structure, Clock, Halftime, Full Time, Polish

**Run this last. Open the existing soccer-mamas.html and add to it.**

The file already has: field, players, drift/burst movement, passing with trail and settling, interceptions, tackling, loose balls, CPU possession with AI, shooting, scoring, goalkeeper behavior, kickoff restarts, and the multiplayer-ready state architecture. Do not restructure — add to it and polish.

---

## Clock System

### Game Time
- Two halves, **3 real-time minutes per half** (6 total)
- Display as scaled game minutes: 0:00 → 45:00 (first half), 45:00 → 90:00 (second half)
- Conversion: 1 real second = 15 game seconds (0.25 game minutes)
- Clock counts UP
- Display format: "12:34" in game minutes and seconds

### Clock Behavior
- Clock runs during active play (phase === 'play')
- Clock **stops** during: goal celebrations, kickoff pause, out of bounds pause, halftime, full time
- Increment clock in the TICK action of updateGame

### Halftime
- When clock reaches 45:00 game time (3 real minutes elapsed in first half):
  - Freeze play immediately
  - Set phase to 'halftime'
  - Display "HALFTIME" overlay centered on pitch with current score below it
  - Tap anywhere or click → start second half

### Starting Second Half
- **MANDATORY STATE PURGE before flipping:** Before swapping positions, zero out ALL player velocities (`vx = 0, vy = 0`), clear ALL `burstTarget` values, clear ALL `burstCooldownUntil` timers, clear ALL `cooldownUntil` (dispossession) timers, and reset `ball.checkedBy`. Without this, players will attempt to run toward their old-side targets after the flip.
- **Switch sides:** Home team now attacks LEFT, away attacks RIGHT
  - Flip all formation home positions: every player's home x becomes (100 - x)
  - Ball at center (50, 50)
  - Possession to away team (they kick off second half)
- Reset all players to their new formation positions
- 1 second pause then play resumes, clock continues from 45:00

### Full Time
- When clock reaches 90:00 game time:
  - Freeze play immediately
  - Set phase to 'fulltime'
  - Display "FULL TIME" overlay centered on pitch
  - Show final score and result: "You Win!" / "You Lose!" / "Draw!"
  - "PLAY AGAIN" button → returns to title screen, resets all state

---

## Out of Bounds

### Sidelines (Y < 0 or Y > 100)
- Ball carrier runs past sideline → play stops
- Possession switches to other team
- Ball placed at boundary point (x stays, y clamped to 0 or 100)
- Nearest opponent to that point receives possession
- 0.5 second pause then play resumes

### Endlines (X < 0 or X > 100, not a goal)
- If ball goes past endline from a missed shot or a carry:
  - If the attacking team last touched it → goal kick: defending team gets ball at their 8 x-unit line, center y (8, 50) or (92, 50)
  - If the defending team last touched it → corner: attacking team gets ball at the corner (0 or 100 x, 0 or 100 y nearest to where ball went out). For simplicity in Alpha, just treat this the same as a throw-in — possession to attacking team from that corner point.
- 0.5 second pause then play resumes

### Ball Carrier Boundary Check
- Check on every tick: if ball carrier position is outside 0-100 range on either axis, trigger out of bounds

---

## Visual Polish

### Render Layer Order (verify from prompt 1)
- Ensure strict render order: Pitch lines → Ball trail → Player tokens → Ball → Gold carrier ring → UI overlays
- The ball must ALWAYS be visible on top of player tokens — verify this visually when players cluster together
- Use explicit z-index values or render order in JSX to guarantee layering

### Ball Trail (verify/improve)
- During passes and shots, render 3-4 trailing circles behind the ball
- Each trail circle: same color as ball (yellow), opacity decreasing (0.6, 0.4, 0.2, 0.1)
- Trail circles placed at ball's previous positions (store last 4 positions)
- Trail only appears when ball is in flight (not when attached to carrier)

### Gold Carrier Ring (verify/improve)
- Ball carrier has a pulsing gold ring — subtle opacity oscillation (0.6 to 1.0, ~1 second cycle)
- Ring is slightly larger than the player token

### Settling Animation (verify/improve)
- During the 500ms settling window, ball should visually jitter slightly at receiver's feet
- Small random offset (±0.3 units) each frame during settling, then snap to exact position

### Goal Celebration
- "GOAL!" text: large, bold, white with dark shadow, centered on pitch
- Fade in quickly, hold 1 second, fade out, then kickoff setup

### Movement Smoothing
- Player positions should interpolate smoothly between ticks for rendering
- Don't just snap to new positions on each tick — lerp for visual smoothness
- Game logic uses tick positions; rendering interpolates between current and next

### Shooting Range Indicator
- When home team has ball and carrier is within 25 x-units of opponent's goal, show a subtle highlight on the goal area (faint glow or slightly brighter goal rectangle) to indicate shooting is available

### Mobile Tap Targets
- Each player token's tap detection radius should be ~48px even if the visual circle is ~36px
- This invisible padding prevents frustrating missed taps on small screens
- When tapping, find the nearest player within tap radius, not just exact hits

---

## Final UI Cleanup

### Score Display
- Clean, readable: "0 — 0" or "HOME 0 — 0 AWAY"
- Update immediately on goals

### Clock Display
- "1st Half — 23:15" or "2nd Half — 67:42"
- Font should be monospace or tabular-nums so digits don't shift width

### Version Text
- "Soccer Mamas — Alpha 1.0 test build" — small, low opacity, corner position
- Should not interfere with gameplay

### Title Screen
- Verify the three lines display correctly:
  - **Soccer Mamas** (large bold)
  - a *Nicole's a Total Baller!* Production (medium, italics on the phrase)
  - Alpha 1.0 test build (small)
- KICK OFF button should be obvious and tappable

---

## Playtest Checklist

After completing the build, verify each of these:

1. **Title screen:** Three lines display correctly with proper formatting. KICK OFF works.
2. **Drift:** All 20 outfield players visibly moving at all times. Pitch feels alive.
3. **Tap to pass:** Tap a blue teammate → ball goes to them with visible trail. Settling prevents immediate action for ~500ms.
4. **Tap to run:** Tap the field → ball carrier runs there at burst speed, slows on arrival.
5. **Burst cooldown:** After a run completes, immediately tap the field again. The carrier should NOT burst — they should drift for ~2 seconds before bursting is available again. If they burst instantly every time, the cooldown is broken.
6. **Interception:** Pass near a red player → they sometimes intercept. Frequency feels fair, not constant. Passing through the same defender repeatedly should NOT result in interception every time (roll-once rule).
7. **Tackle:** Run carrier close to red player → sometimes tackled. Loose ball appears briefly, nearest player picks up.
8. **No ping-pong:** After a tackle, the player who lost the ball does NOT immediately regain it. A different player should pick up the loose ball. If the tackled player keeps getting it back, the 1500ms dispossession cooldown is broken.
9. **CPU possession:** When CPU gets ball, they pass and occasionally run. They make forward progress. They sometimes shoot. They react quickly when a defender closes in (don't run into defenders or off the pitch). They do NOT react *instantly* — there should be a brief visible delay before they pass under pressure.
10. **Human defense:** During CPU possession, tap a blue defender → they sprint toward ball carrier.
11. **Shooting:** Get within range of goal (look for highlight). Tap goal → shot happens. Keeper reacts.
12. **Scoring:** Score a goal → "GOAL!" appears, score updates, kickoff restart with correct possession.
13. **Clock:** Game time advances during play. Displays as scaled minutes (not real time). Stops during goals/pauses.
14. **Halftime:** At 45:00 → "HALFTIME" overlay with score. Tap to continue. Teams switch sides.
15. **Halftime state purge:** After halftime, NO player should be running toward an old-side position. All velocities and burst targets should be clean.
16. **Full time:** At 90:00 → "FULL TIME" overlay with result. "PLAY AGAIN" returns to title screen.
17. **Out of bounds:** Carrier runs off edge → possession switches, brief pause, play resumes.
18. **Side switch:** After halftime, home team attacks left instead of right. Formations flip.
19. **Render order:** When players cluster together or near the ball, the ball is always visible ON TOP of player tokens. It never disappears behind them.
20. **Mobile:** On a phone-sized viewport — tokens tappable, pitch readable, no overflow or scroll.
21. **Version text:** "Soccer Mamas — Alpha 1.0 test build" visible during gameplay, unobtrusive.

---

## What Alpha 1.0 Does NOT Include

- No character abilities or special moves
- No difficulty selection (Medium only)
- No formation selection (4-4-2 only)
- No multiplayer networking (architecture is ready, not connected)
- No commentary or flavor text
- No sound
- No offsides, fouls, or cards
- No throw-in or corner kick animations

This is the core loop. If it feels like soccer in 10 seconds, everything else layers on.
