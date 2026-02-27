# Soccer Mamas — Roadmap

**Current Version:** Alpha 1.0
**Status:** First playable build. Core loop works but has significant issues.
**Repo:** https://github.com/kpurdddy/soccermommas
**Live:** https://kpurdddy.github.io/soccermommas/
**Working Title:** Soccer Mamas — "a Nicole's a Total Baller! Production"

---

## Current State — Alpha 1.0

### What Works
- Title screen with proper formatting
- Fixed-field with pitch markings, all 22 players visible
- Drift movement keeps the pitch alive — nobody stands still
- Tap teammate to pass (with ball trail and settling window)
- Tap field to make ball carrier run (with burst cooldown)
- Interceptions on passes (roll-once-per-defender system)
- Tackles on ball carriers
- Loose ball pickup with dispossession cooldown (no ping-pong)
- CPU possession with AI (passes, runs, occasional shots)
- CPU reaction delay (doesn't instantly pass under pressure)
- Human defense: tap your defenders to commit them toward ball carrier
- Shooting with goal area highlight when in range
- Goalkeeper behavior (slides toward shots)
- Goal celebrations, score tracking, kickoff restarts
- Game clock (scaled: 3 real minutes = 45 game minutes per half)
- Halftime with side switch and state purge
- Full time with result and play again
- Out of bounds handling
- Landscape-only with rotate message for portrait
- Git/GitHub Pages deployment working

### Known Issues — Priority Order

**1. Viewport sizing (CRITICAL)**
The field renders as a small rectangle instead of filling the screen. Likely hardcoded pixel dimensions instead of viewport-relative sizing. Must use `viewportWidth - padding` for field width, derive height from aspect ratio.

**2. Shot system has zero player agency (CRITICAL)**
Tapping the goal area triggers a dice roll. No aiming. The keeper saves almost everything because there's only one target. In several minutes of play by multiple people, only one goal was scored.

**Needed:** Shot selection — at minimum, tap left/right/center of goal to aim. The keeper is positioned somewhere; the skill is shooting where the keeper isn't. Possible two-step selection: direction (left/center/right) then height (low/mid/high). Corner shots harder to place but harder to save. This is where player agency lives in the scoring moment.

**3. Offsides drift / formation collapse**
Attackers drift forward past the defensive line and stack up near the goal. Nicole (D1 player, current coach) specifically noted players were in positions they'd never be in real soccer. The home-pull toward formation positions is too weak (HOME_PULL * 0.3) relative to the forward drift.

**Needed:** Stronger formation gravity on attackers. Possibly a positional ceiling — forwards can't drift past a certain X threshold relative to the opponent's defensive line. Or a simple offside clamp: attackers can't be further forward than the second-to-last defender.

**4. Positional lane constraints**
Nicole noted a specific position was moving somewhere it would never go. Defenders track toward the ball regardless of lane — a LB at y=15 will drift toward a ball at y=85 given enough time. Positions need lateral constraints so defenders stay in their zone.

**5. Post-tackle behavior is static**
After a tackle, the ball sits at the contact point for 300ms and the nearest player picks it up. No momentum, no ball squirting away, no realistic scrum. 

**Needed:** Ball should deflect a few units in a random direction from the tackle point. Gives nearby players time to react. Creates more dynamic possession changes.

**6. Defense needs more depth**
Nicole's note: "once your player has the ball, she should take a couple steps away or quickly pass to a teammate." Currently after a tackle/interception, the new ball carrier just has the ball with no urgency. Could add: brief auto-movement away from the nearest opponent after gaining possession, or a faster settling time when winning the ball defensively.

---

## Planned Features — Rough Priority

### Alpha 1.1 — Critical Fixes
- Viewport-responsive field sizing
- Shot selection system (aim left/right/center of goal, at minimum)
- Stronger formation discipline / offsides prevention
- Positional lane constraints for defenders

### Alpha 1.2 — Feel and Polish
- Post-tackle ball physics (deflection, not static drop)
- Post-possession auto-step (brief movement away from tackler after winning ball)
- Improved visual hierarchy (ball always on top, clearer who has possession)
- Movement smoothing (lerp between ticks for buttery rendering)
- Better goalkeeper behavior

### Alpha 2.0 — Tier Structure
- Kickabout tier: Training wheels, tip text, very forgiving defenders, lots of goals (6-4 scorelines)
- Friendly tier: Tips reduced, real defenders but slow, 4-3 / 3-2 scorelines
- League Match tier: No tips, defenders read space, 2-1 / 3-2 scorelines
- Cup Final tier: Smart defense, tactical play required, 2-1 target scorelines
- Scoring calibration for each tier
- Tier-specific tuning tables

### Alpha 3.0 — Player Personality Powers
- Character ability system: each player has a special power based on a real person's personality
- Cooldown visualization (color drains off token, fills back up)
- Initial powers:
  - Grouchy: defenders avoid them briefly (zone of exclusion)
  - Phone addict: perfect long-range pass to distant teammate
  - Aggressive: defenders flinch when charged at
  - Chatty: nearby players cluster around her (shield/overload)
  - More to come from Nicole's input on her students
- Powers are situational — using them well requires reading the game state

### Alpha 4.0 — The Decision Moment (Matrix Slow-Down)
- When a player receives the ball, time slows dramatically (visual slow-motion effect)
- The settling window becomes the tactical decision moment
- Player reads the field: where are defenders, who's open, whose power is charged
- Tap and everything accelerates — pass flies, defenders react
- Read → Decide → Execute rhythm becomes the core experience
- At higher tiers, decision window shrinks (defenders close faster)

### Future — Not Yet Prioritized
- **Multiplayer (same-location):** Two phones on same wifi, WebRTC peer-to-peer. One creates game code/QR, other joins. Tap events sync directly phone-to-phone. Only viable for same-location play. This is a v2/v3 feature.
- **Control panel mode for multiplayer:** If two-player on one device, switch from spatial taps to sequential controls (select player → choose direction → choose height). Different input layer, same underlying simulation.
- **Formation selection:** Pre-match choice of 4-4-2, 4-3-3, 3-5-2, etc. Changes home positions and drift behavior.
- **Commentary/flavor text:** Brief contextual comments on plays ("great through ball!", "the keeper had no chance")
- **Sound:** Crowd noise, whistle, kick sounds, goal celebration
- **Offsides rule:** Optional for higher tiers. Creates interesting tactical depth — forces attackers to time runs.
- **Set pieces:** Corner kicks, free kicks as special mini-phases
- **Season mode / league structure**

---

## Nicole's Involvement

Nicole played D1 soccer and currently coaches high school. Her contributions:

**Confirmed:**
- Design consultation — what tactical concepts the game should reward
- Playtesting feedback (Alpha 1.0 — offsides drift, positional accuracy)
- Real-people personality powers using her students' traits
- Coaching application input — how the game could be a teaching tool

**Planned for Thursday meeting:**
- Full design doc review
- What her students' personalities and quirks translate to as powers
- What tactical decisions separate smart players from ball-chasers
- Whether the two-tap system captures real decision-making
- Defensive depth with minimal controls
- Formation and positional behavior authenticity

**Open question:** Game name. "Soccer Mamas" is the working title. Nicole may have opinions.

---

## Design Decisions Made

These have been discussed and decided. Don't relitigate:

- **Two-input control scheme:** Tap teammate to pass, tap field to run. That's it. Everything else is emergent.
- **No offsides in v1:** Adds complexity without teaching value for newcomers. May add as optional rule at higher tiers later.
- **No fouls or cards:** Same reasoning. May add later if it creates interesting decisions.
- **Out of bounds = simple possession change:** No throw-in animations, no corner kick mechanics for now.
- **CPU plays the same system:** Same drift/burst mechanics, controlled by AI. Not a separate simulation.
- **Landscape only:** The field aspect ratio requires landscape. Portrait gets a rotate message.
- **Architecture: gameState/updateGame/React-as-renderer:** Non-negotiable. Exists for future multiplayer.
