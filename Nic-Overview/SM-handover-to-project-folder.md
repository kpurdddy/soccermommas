# Handover — Transition to Project Folder

**Date:** 2026-02-25
**From conversation:** Soccer game design, Nicole collaboration, project folder setup
**Next step:** Set up Claude project folder, then start Alpha 1.1 build inside it

---

## What Was Accomplished This Session

### Soccer Mamas — Design and Playtesting
- Reviewed the design handover doc and all three build prompts (1of3, 2of3, 3of3)
- Built and deployed Alpha 1.0 via Claude Code → GitHub Pages
- **Live at:** https://kpurdddy.github.io/soccermommas/
- **Repo:** https://github.com/kpurdddy/soccermommas
- Multiple people playtested including Nicole (D1 soccer, current HS coach)
- Identified six priority issues (see roadmap-soccer-mamas.md for full details)

### Key Playtest Findings
1. **Viewport too small** — field renders as tiny rectangle, not filling screen
2. **Shooting has zero player agency** — tap goal, watch dice roll, keeper saves everything. Only 1 goal scored in several minutes by multiple players.
3. **Offsides drift** — Nicole specifically noted players in unrealistic positions, attackers stacking past defensive line
4. **Positional lane violations** — Nicole noted a specific position moving somewhere it never would in real soccer
5. **Post-tackle behavior is static** — ball just sits there
6. **Defense needs more depth** — Nicole: "once your player has the ball, she should take a couple steps away or quickly pass"

### Nicole Collaboration
- Nicole is involved as D1/coaching subject matter expert
- Thursday meeting planned for full design review
- She'll provide: tactical concepts for higher tiers, student personalities for powers, coaching tool input
- Kip is texting her tonight with overview of concepts — focus on:
  - Two-input control scheme and tactical settling moment
  - Player personality powers with real people's traits mapped to mechanics
  - Coaching angle — students on their own phones learning positioning
  - The Hughes principle — advanced play is about where the ball goes next
  - What she should be thinking about: what does a player who understands soccer DO differently (decisions, not physical ability)

### New Ideas Developed This Session

**Player Personality Powers — Specific Examples:**
- Grouchy person → zone of exclusion, defenders avoid for a few seconds
- Phone addict → perfect long-range pass to distant teammate
- Aggressive → defenders flinch when charged at
- Chatty → nearby players cluster (natural shield/overload)
- Great hair → TBD
- Each power: situational (not stat boost), visible cooldown (color drains off token), easy to code (modifications to existing values)

**Tier Structure — Soccer-Specific Names:**
- Kickabout → Friendly → League Match → Cup Final
- Each teaches progressively: controls → sequences → reading the field → tactical manipulation
- Scoring calibrated per tier: 8-10 goals at Kickabout down to 3-4 at Cup Final

**The Matrix Moment:**
- Settling window after pass reception IS the decision moment
- Time effectively slows down — read the field, decide, then execute
- Rhythm: Read → Decide → Execute, repeating
- At higher tiers, window shrinks because defenders close faster

**Coaching Sandbox Mode:**
- Same app, mode switch on title screen: "Play" and "Coach"
- Coach mode: drag players to positions, hit play, watch simulation
- Players named after real team members with their actual photos as avatars
- Coach demonstrates in sandbox → student practices in game mode
- Same engine, same simulation, just different input layer

**Multiplayer Assessment:**
- Same-location (same wifi) is feasible and within agentic-coding-with-guidance range
- WebRTC peer-to-peer, no server needed, single-digit ms latency on local network
- Maybe a week to prototype once single-player is solid
- Cross-network multiplayer is a different class of problem — deferred

### Two Other Soccer Coaches
- Another coach (not Nicole) saw the game and was asking about it
- Interest in the coaching/teaching tool application
- Validates the coaching sandbox direction

---

## Project Folder Setup

### Files to Upload to Project Knowledge Base:
1. **design-bible-soccer-mamas.md** — SM-specific philosophy, mechanics, tiers, powers, Nicole, coaching
2. **workshop-manual-all-games.md** — shared workflow, trigger phrase rule, 32k ceiling, architecture, deployment, failure modes
3. **gemini-review-soccer-mamas.md** — dvh/dvw fix, deterministic state machine language, clearAllIntentions()
4. **roadmap-soccer-mamas.md** — current state, all 6 known issues, planned features through Alpha 4.0+
5. **roadmap-hail-marys.md** — current state, known issues, phases 1-10

### Files NOT to Upload:
- The game HTML files (index.html) — too large, not reference material. Upload to specific conversations when building.
- Build prompts — these are per-conversation, not standing reference

### Still Needed:
- **design-bible-hail-marys.md** — use prompt-compile-hailmarys-bible.md to generate this from past conversations, then add to project folder
- **Landing page content** — Kip writing this himself, covers the two-mode concept (game + coaching sandbox), development context, links to both games

### Project Instructions (set in Claude project settings):
Something like: "Read all project knowledge documents before responding. These contain the design philosophy, technical constraints, workflow rules, and current state of two games: Soccer Mamas (tactical soccer) and Hail Marys (tactical football). The workshop manual contains absolute rules including the trigger phrase requirement. Follow them."

---

## Immediate Next Steps

### 1. Set Up Project Folder
- Create Claude project
- Upload the 5 documents listed above
- Set project instructions
- Start all future game conversations inside this project

### 2. Generate Hail Marys Design Bible
- Open new conversation (inside or outside project)
- Paste the prompt from prompt-compile-hailmarys-bible.md
- Review output, then add design-bible-hail-marys.md to project folder

### 3. Soccer Mamas Alpha 1.1 — First Build in Project
Priority fixes for the next build:
- **Viewport-responsive sizing** — field must fill screen using dvw/dvh units
- **Shot selection system** — tap left/right/center of goal to aim, skill = shooting where keeper isn't
- **Stronger formation discipline** — prevent offsides drift, stronger home-pull on attackers
- **Positional lane constraints** — defenders stay in their zone, LB doesn't drift to opposite side

Upload index.html to the conversation when ready to build. The project docs provide all the context.

### 4. Landing Page
- Kip writes this
- Covers: two modes (game + coaching sandbox), what it teaches, avatar/photo feature for coaches, development context, links to both games and the TacFootball prototype
- Deploy at the GitHub Pages URL

### 5. Thursday Meeting with Nicole
- She plays the game beforehand (link already works)
- Full design doc discussion
- Personality powers for her students
- Coaching sandbox concept — drag to position, run simulation, student photos as avatars
- Formation and positional authenticity review

---

## File Locations

### Soccer Mamas
- **GitHub repo:** https://github.com/kpurdddy/soccermommas
- **GitHub Pages:** https://kpurdddy.github.io/soccermommas/
- **Local:** F:\nicball\ (thumb drive)

### Hail Marys
- **GitHub repo:** https://github.com/kpurdddy/Hailmarys
- **GitHub Pages:** https://kpurdddy.github.io/Hailmarys/
- **Also at:** https://kpurdddy.github.io/TacFootball/ (older version, Kip shares this too)
- **Local:** F:\TacFootball\ (thumb drive)

### PC
- Username varies: `obrie`, `domain`, `Lash2` — check or ask
- Claude Code launch: `cd F:\nicball` (or F:\TacFootball), `claude --dangerously-skip-permissions`, paste prompt

---

## Workflow Reminders

- **NEVER build without "Go with the [X]"** — this is the #1 source of frustration, documented extensively in workshop manual
- **Target 15-20k tokens per build prompt** — 32k ceiling is real
- **Backup before every build**
- **Git push after every build** — if rejected, pull --rebase first
- **Update NOTES.md after every build**
- **Playtest checklist in every DONE summary**

---

## What Claude Should Know Going Forward

- Kip is building these games collaboratively with Claude and Gemini — there's no competition, it's collaboration. Gemini reviews contribute to the project.
- Nicole is a key collaborator on Soccer Mamas — D1 player, current HS coach, providing tactical expertise and student personalities for game powers
- Another soccer coach has expressed interest in the teaching tool application
- The coaching sandbox (drag-to-position, student photos, scenario builder) is now a confirmed direction, not just an idea
- Both games share the same philosophy (bar test, teaching through failure, fixed field, transparent mechanics) but are mechanically distinct (real-time two-tap vs. turn-based play calling)
- Kip values the collaborative discussion process. The conversation IS the work. Don't jump to building.
