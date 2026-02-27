# Design Bible — Soccer Mamas

This document is the philosophical and mechanical foundation for Soccer Mamas. If a design choice contradicts something in here, the choice is wrong unless this document has been explicitly updated.

---

## The Bar Test

The bar test is the single most important design filter. Every feature, every mechanic, every UI element must pass it.

**The scenario:** Someone is sitting at a bar. They've never played a soccer video game before. Maybe they've never watched soccer. A friend hands them a phone with the game running. Within 10 seconds — with zero instruction, zero tutorial text, zero onboarding — they understand what to do, and they're doing it.

That's the bar test. If a feature requires explanation, it fails. If a mechanic can only be understood by reading a tooltip, it fails. If the interface has buttons whose purpose isn't immediately obvious from context, it fails.

This doesn't mean the game is simple. It means the *entry point* is simple. Depth lives underneath.

**Who this person is:** The canonical bar test user is someone who's there socially, curious enough to try, but not invested enough to read instructions. She's the person every design decision optimizes for at the lowest tier. If she can play it, anyone can. If she's having fun, the mechanic is working.

This is not a condescending persona. This is the hardest audience to design for. Making something accessible to a complete newcomer without dumbing it down for an expert is orders of magnitude harder than designing for the audience that already cares.

---

## Two-Audience Design

Soccer Mamas serves two audiences simultaneously:

### Audience 1: The Newcomer
Someone who doesn't know soccer, doesn't play video games much, and picked up the phone because it was there. For this person, the game needs to:
- Be immediately playable with no instruction
- Produce fun within 30 seconds
- Teach soccer's core principles *through play*, not through text
- Generate lots of goals even if they're playing badly
- Never make them feel stupid

### Audience 2: The Expert / Coach
Someone who played soccer at a high level (like Nicole, D1), coaches it, or deeply understands its tactics. For this person, the game needs to:
- Reward real tactical thinking at higher difficulty levels — creating overloads, switching play, manipulating defensive shape
- Produce moments they recognize from real competition
- Allow them to execute concepts they already know (positional play, through balls, pulling defenders)
- Have enough depth that they can't just brute-force it with speed or reflexes
- Serve as a coaching tool — something they could use to show a student *why* positioning matters

**The critical insight:** These two audiences don't require two different games. They require a *tier structure* where the lowest tier is pure bar-test accessibility and the highest tier rewards genuine expertise. The mechanics are the same at every level — what changes is how punishing the game is when you make bad decisions.

---

## The Control Scheme — Two Inputs

The entire game is controlled by two taps:

- **Tap a teammate** → pass to them
- **Tap the field** → ball carrier runs toward that point

Everything else — positioning, defense, pressure, interceptions, tactical depth — is emergent from the movement rules. There are no buttons, no menus, no mode switches. The field IS the interface.

This is the most important design decision in the game. It's what makes the bar test possible. A newcomer sees players, taps one, the ball goes there. A veteran taps a midfielder to run wide, dragging a fullback, opening a channel for a through ball. Same input, fundamentally different level of play.

---

## Teaching Through Failure

The game teaches soccer by letting the player fail, not by telling them what to do.

**The risk-distance tradeoff — the core learning loop:**
- Tap far downfield → your player runs a long way → defenders close in → you lose the ball
- Tap nearby → short run → safe but no progress
- The player naturally learns: short runs, then pass. That's actual soccer.
- "If I run this direction, that defender commits, which opens a pass to the winger" — real tactical thinking, discovered not taught.

You discover "move the ball faster than you move your body" by failing the other way first. No tooltip needed. The mechanic teaches itself.

**What this means for design:**
- No tutorial popups that say "try short passes" — the mechanic itself teaches that
- No arrows pointing to where you should pass — that removes the learning
- Tip text at the lowest tier (Kickabout) is acceptable as training wheels, but the mechanic must teach the lesson even without the tips
- The feedback loop must be immediate and visual — the player sees *why* they failed

---

## The Jack Hughes Principle

In the 2026 Men's Olympic gold medal hockey game, Jack Hughes skated *away* from the scramble for the puck. His teammate gained control, passed it across the ice to where Hughes was standing open, and he fired it past the goaltender. Hughes wasn't playing the puck — he was playing the space where the puck should go next.

This is the principle that separates advanced play from beginner play in *every* sport. A beginner chases the ball. An expert creates the passing lane by moving to where the ball needs to be.

In Soccer Mamas: at the lowest tier, you chase the ball and still have fun. At the highest tier, you tap your striker to run *away* from the ball, pulling a defender with them, and opening the lane for a pass to your other attacker. Same two-tap input, fundamentally different level of play.

---

## The Movement System

### Drift Layer (always active, ~30fps)
- All 20 outfield players move continuously at drift speed (~0.15 units/tick)
- Attackers drift loosely toward the opponent's goal
- Defenders drift to maintain shape relative to ball position
- Goalkeepers drift laterally to track the ball's Y position, clamped to their box
- Players gently gravitate back toward formation home positions
- Nobody is ever standing still — the pitch is always alive

This is one of the best design decisions in the game. Having all players always moving means:
- The game is visually alive from frame one
- The tactical situation is always evolving, even when the player does nothing
- Someone who watches for 5 seconds before touching anything sees the flow before committing
- It's an implicit tutorial — the movement itself teaches what the game is

### Burst Layer (triggered by player input)
- **Run:** Ball carrier moves at ~3x drift speed toward the tapped field position. Others continue drifting. On arrival, carrier reverts to drift.
- **Pass:** Ball travels at ~5x drift speed to the tapped teammate.
- **Burst cooldown (2000ms):** After a burst run completes, the carrier can't burst again for 2 seconds. Prevents jitter-tapping to maintain permanent 3x speed.

### The Settling Window — "The Matrix Moment"
When a pass arrives, the receiver has a 500ms settling window where they can't immediately burst. This simulates first touch. But more importantly, it's the **decision moment**. The ball arrives, the field is readable, and the player chooses: pass, run, or shoot.

This rhythm — Read → Decide → Execute → Read → Decide → Execute — is the core experience. It's nothing like FIFA or any real-time soccer game where it's all reflexes. It's closer to chess with a clock. You have time to think, but not forever.

At higher tiers, this window effectively shrinks because defenders close faster. But it's always there.

---

## Fixed-Field Architecture

All 22 players are visible on screen at all times. No scrolling. No camera. No zooming.

**Why:**
- Spatial awareness IS the game. You can't create an overload if you can't see the whole pitch.
- It's the same perspective as a coach's tactics board. Nicole would recognize it immediately from drawing up set pieces.
- It eliminates an entire category of UI problems that would fail the bar test
- It makes the game feel like a board game come to life

**Non-negotiable:** The field must fill the available viewport. Not a tiny rectangle in the center of the screen. On a phone in landscape, the field should be nearly edge-to-edge.

---

## Transparent Mechanics

The player should always understand why something happened.

- If a pass gets intercepted, the player can see they passed through a defender
- If a tackle happens, the defender was visibly close to the ball carrier
- If a shot is saved, the keeper was visibly in position
- On higher difficulty, defenders are *visibly* faster, not just statistically better behind the scenes

The visual tells the story. No black boxes. No hidden dice rolls.

---

## Difficulty Through Parameters, Not Complexity

Difficulty levels never add new mechanics or new controls. They adjust tunable values.

**The tier structure (soccer-specific names):**

| Tier | What It Teaches | Target Scoreline | Feel |
|------|----------------|-----------------|------|
| Kickabout | Controls — "I can tap to pass, tap to run" | 6-4, 5-3 | Park soccer with mates |
| Friendly | Sequences — "Short pass chains work" | 4-3, 3-2 | Exhibition match |
| League Match | Reading the field — "That defender shifted, this lane is open" | 3-2, 2-1 | Competitive, entertaining |
| Cup Final | Tactical manipulation — "Move this player here, create space there" | 2-1, 3-2 | High-stakes knockout |

Each tier isn't just harder. It's a different *game* in terms of what's required to succeed.

**Tunable parameters by tier:**

| Parameter | Kickabout | Friendly | League | Cup Final |
|-----------|-----------|----------|--------|-----------|
| Defender drift speed | 0.2 | 0.35 | 0.5 | 0.65 |
| GK Save % | 20% | 35% | 50% | 65% |
| Interception radius | 1.0 | 1.5 | 2.0 | 2.3 |
| CPU pass accuracy | - | - | 80% | 90% |
| Avg goals/game | 8-10 | 5-7 | 4-5 | 3-4 |

---

## Scoring and Player Agency

Scoring must always involve player agency. A goal that comes from tapping the goal area and watching a dice roll isn't fun even if it goes in. The player needs to feel like they *did something* to score.

**The shooting problem (Alpha 1.0):** Currently there's no aiming — you tap the goal area, the game rolls dice, the keeper saves almost everything. In several minutes of play by multiple people, only one goal was scored.

**The solution:** Shot selection. At minimum: tap left/right/center of goal to aim away from where the keeper is positioned. Possible two-step: direction first, then height. Corner shots harder to place but harder to save. The skill is reading the keeper's position and choosing where to aim.

---

## The Personal/Social Layer

### Personality-Based Special Powers
Each player token has a special ability based on a real person's personality trait. The trait maps to a tactical effect:

- Grouchy person → defenders avoid them for a few seconds (zone of exclusion, creates open space)
- Always on her phone → perfect long-range passing to a distant teammate (great at distant communication)
- Really aggressive → runs at defenders and they flinch out of the way briefly
- Super chatty → nearby players cluster around her (natural shield/overload creator)
- Great hair → ??? (for Nicole and friends to decide)

**Why this works as game design, not just flavor:**
- Every power is *situational* — none are just "this player is better"
- Using them well requires reading the game state and knowing *when* to use *which* teammate
- The knowledge of who has what power lives outside the game, in the friendship group ("that's Susie, use her for the cross-field pass!")
- People would be yelling at each other at the bar

**Cooldown visualization:** Color drains off the token when a power is used, slowly fills back up when recharged. No separate UI elements — the information lives on the token itself.

**Implementation:** Not hard. Every power is a modification to values the system already tracks — proximity thresholds, drift vectors, interception probabilities. The special powers layer sits on top of the core simulation without changing it.

---

## The Coaching Application

Nicole played D1 soccer and coaches high school. The game can serve as a teaching tool because:

- The top-down fixed-field view IS the tactics board her students already know
- Kids playing single-player on their own phones learn positioning because the game punishes ball-chasing
- In schoolyard games, all the kids pile toward the ball. This game naturally trains them out of that.
- At higher tiers, success requires concepts she's trying to teach: maintaining shape, creating overloads, switching play
- Her students as characters with unique abilities makes it personal and motivating

**Nicole's contributions:**
- What tactical concepts the higher tiers should reward
- Whether the two-tap system captures real decision-making
- Defending with minimal controls — is "commit a defender and open space" enough
- Formation and positional behavior that makes drift movement feel authentic
- Her students' personalities and quirks as game powers
- The coaching tool angle — how to integrate into her program

---

## What This Game Is

- A tactical decision game wearing soccer's clothes
- A teaching tool that happens to be a game
- A social object — something friends play together and yell about at a bar
- Optimized for the audience that FIFA excludes
- The field is the interface. Two taps. Everything else is emergent.

## What This Game Is Not

- Not a simulation of broadcast soccer
- Not a reflex test — reading the field matters, not thumb speed
- Not aimed at FIFA/eFootball/FM players
- Not multiplayer-first — single player is the core, multiplayer layers on top
- Not monetized through dark patterns

---

## Kip's Signature

This game is identifiable as Kip's work through:
- The bar test as the primary design filter
- Fixed-field, no-camera architecture
- Two-input control scheme generating emergent complexity
- Difficulty through parameter tuning, not added mechanics
- Personal/social layer with real people as characters
- Teaching-through-failure philosophy
- The mechanic teaches the sport rather than the sport dictating the mechanic
- Scoring calibrated for entertainment, not realism
- A coaching application that emerges naturally from the design

The combination is unique. Any one of these choices might appear in another game. Together, they couldn't come from a studio.
