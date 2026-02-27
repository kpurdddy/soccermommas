# Workshop Manual -- How We Build

This document covers the practical workflow, technical standards, and hard-won lessons for building games in this project folder. It applies to all games. It's a living document that grows as we learn new constraints.

---

## The Trigger Phrase Rule -- ABSOLUTE, NON-NEGOTIABLE

**Claude (in chat or in code) must NEVER write code, build prompts, create files, or execute builds without the explicit trigger phrase: "Go with the [X]"**

That exact pattern -- "Go with the" followed by whatever Kip is authorizing -- is the ONLY authorization to produce deliverables.

**What counts as authorization:**
- "Go with the prompt"
- "Go with the documents"  
- "Go with Alpha 1.1"
- "Go with that approach"

**What does NOT count as authorization:**
- "I'm thinking about building X" -- that's Kip thinking out loud
- "I want to text her an overview" -- that's Kip describing his intentions
- "What would the prompt look like?" -- that's a question, not a command
- "That sounds good" -- that's agreement with a concept, not authorization to build
- "Let's do it" -- ambiguous, ask for clarification
- "I think we're ready to go with X" -- that's assessing readiness, not authorizing
- "Go ahead" -- close but not the pattern, ask for the trigger phrase

**Why this exists:** Kip uses conversations to think collaboratively. He's working through ideas, testing concepts, exploring options. When Claude interprets "I'm thinking about X" as "build X right now" and produces 3,000 lines of code or a full message draft, it derails the conversation, wastes time, and takes the fun out of the collaborative process. This has been a persistent, repeated problem.

**When in doubt:** Ask. "Want me to write that up, or are we still talking?" is always better than producing something unauthorized.

---

## Context Truncation -- Hard Rule

Long conversations silently drop earlier content from Claude's context window. There's no warning when this happens -- earlier discussion simply disappears. Claude must monitor conversation length and proactively warn Kip when truncation is approaching. Suggest handing over to a new thread BEFORE it becomes a problem, not after key decisions have already been lost. Every handover to a new thread should include a summary of decisions made and open questions.

---

## The 32k Output Token Ceiling

Claude Code has a hard limit of 32,000 output tokens per response. A single HTML game file can easily exceed this.

**What this means in practice:**
- A single prompt that says "write the whole game" will hit the wall
- Even "write prompt 1 of 3" might hit it if prompt 1 is complex enough
- The error message is: `API Error: Claude's response exceeded the 32000 output token maximum`
- Claude Code sometimes recovers from this by continuing in a follow-up, but it's not reliable

**How to structure prompts around this:**
- Target 15-20k tokens of output per prompt, leaving headroom
- Structure builds as incremental additions: "open the existing file and add X" rather than "write a file that does X, Y, and Z"
- When a build is large, explicitly break it into sections: "Build the field and movement system. Stop. Then in the next pass, add interceptions and tackling."
- Include this note in build prompts: "If you're approaching the output limit, stop at a clean breakpoint, save the file, and note what remains to be done."

**The environment variable:** `CLAUDE_CODE_MAX_OUTPUT_TOKENS` can be set higher if the model supports it, but designing around the limit is more reliable than hoping it'll stretch.

---

## Architecture Pattern -- All Games

All games in this project use the same core architecture. This exists so that multiplayer can be added later without rewriting the game.

### The Pattern
1. **Single `gameState` object** holds everything: players, ball, score, clock, phase, timers. This is the complete state of the game at any point in time.
2. **A pure `updateGame(gameState, action)` function** processes all logic. Actions are simple objects: `{ type: 'TICK' }`, `{ type: 'TAP_PLAYER', playerId }`, `{ type: 'TAP_FIELD', x, y }`. Given the same state and the same action, the function always produces the same result.
3. **React only renders.** Components read gameState and draw the screen. They capture user input and dispatch actions. They NEVER compute game outcomes. No `setState` that determines whether a pass was intercepted. No game logic in event handlers.

### Why This Matters
- **Multiplayer:** Two phones can run the same `updateGame` function. Only the actions need to sync between devices. Both phones compute the same game state independently.
- **Debugging:** The game state is inspectable. You can pause, look at every value, and understand exactly what the game thinks is happening.
- **Testability:** You can feed a known state and a known action into `updateGame` and verify the output without rendering anything.

### What This Means for Build Prompts
- Never say "when the player taps, move the ball carrier." Say "when the player taps the field, dispatch `{ type: 'TAP_FIELD', x, y }`. The `updateGame` function handles the movement."
- Never put game logic in React components
- Never put rendering logic in the update function

---

## Tech Stack

All games are single HTML files. This is intentional and correct.

- **React 18** via CDN (unpkg)
- **Inline CSS** -- no external stylesheets
- **Inline JavaScript** -- no modules, no bundler, no build step
- **No external dependencies** beyond React and ReactDOM

### Why Single-File
- Deploy by copying one file
- No build process to break
- No dependency versioning issues
- GitHub Pages serves it directly
- Anyone can open it in a browser from a thumb drive
- Claude Code can read and edit the entire game in one file

### When This Breaks Down
If a game exceeds ~5000 lines, consider splitting into multiple files loaded in order. But don't do this prematurely -- a 3000-line file is fine.

---

## Viewport-Responsive Field Sizing -- MANDATORY

The playing field must fill the available screen space. Not a fixed-pixel rectangle. Not a small centered box.

### The Rule
```
fieldWidth = viewportWidth - (small padding, ~16px total)
fieldHeight = fieldWidth / aspectRatio
```

Soccer aspect ratio: ~1.5:1 (width is 1.5x height)
Football aspect ratio: ~2.25:1 (width is 2.25x height)

### The PIXELS_PER_UNIT Constant
On mount and on every window resize:
```
pixelsPerUnit = fieldPixelWidth / 100
```
ALL positions in game state are 0-100 percentages. This single constant converts game units to pixels. Movement speeds that look correct on any screen size because they're defined in game units, not pixels.

Store `pixelsPerUnit` in a React ref, not in gameState (it's a rendering concern).

### Mobile
- Landscape orientation preferred for both games but do NOT gate the game behind a forced rotation message
- Apply `touch-action: none` to prevent browser gestures stealing taps
- Tap targets minimum 48px even if visual tokens are smaller
- Test that the field is edge-to-edge on a phone screen, not a tiny rectangle

---

## Build Prompt Format

Every build prompt must include:

### Header
```
# [Game Name] -- Build Prompt: Alpha X.Y.Z
Version: Alpha X.Y.Z (from X.Y.W)
Date: YYYY-MM-DD
Source file: [filename]
Location: [path]
```

### Estimates
```
Estimated tokens: ~Xk
Estimated time: ~X minutes
```

### Pre-Build Checklist
```
1. Backup current file -> [filename]-backup.html
2. Read the current file before making changes
3. [Any other setup steps]
```

### The Fixes/Additions
Numbered, specific, with context explaining *why* each change is needed. Not just "fix the scoring" but "the shot probability system has no player agency -- the player taps the goal area and watches a dice roll."

### Post-Build Requirements
```
1. Update version number in the file
2. Save the file
3. Git add, commit with descriptive message, push
4. Update NOTES.md
5. Print DONE summary: what changed, file locations, anything that needs manual attention
```

### Playtest Checklist
Specific, numbered items to verify after the build. These catch the most common failures (stale state after halftime, phantom tackles, viewport sizing, etc.)

---

## File Locations and Deployment

### Soccer Mamas
- **GitHub repo:** https://github.com/kpurdddy/soccermommas
- **GitHub Pages:** https://kpurdddy.github.io/soccermommas/
- **Source file:** `index.html` in repo root
- **Local:** F:\nicball\ (thumb drive)

### Hail Marys
- **GitHub repo:** https://github.com/kpurdddy/Hailmarys
- **GitHub Pages:** https://kpurdddy.github.io/Hailmarys/
- **Source file:** `index.html` in repo root
- **Local:** F:\TacFootball\ (thumb drive)

### Deployment Process
1. Claude Code commits and pushes to GitHub
2. GitHub Pages auto-deploys from the main branch, root folder
3. If Pages hasn't been set up yet: Settings -> Pages -> Deploy from branch -> main -> / (root) -> Save
4. First deployment requires a commit *after* enabling Pages to trigger the build
5. If push is rejected ("remote contains work you don't have locally"), pull --rebase first, then push

### PC Paths
Username varies between machines: `obrie`, `domain`, `Lash2`. Don't assume -- check or ask.

---

## NOTES.md Convention

Each game has a NOTES.md file in its directory. This is the running build history.

### Format
```
# [Game Name] -- Build Notes

## Alpha X.Y.Z -- [Date]
- What was built/changed
- What was fixed
- Known issues discovered
- Token count and build time

## Alpha X.Y.W -- [Date]
...
```

### Rules
- Update after every build
- Include token count and time spent
- Note known bugs even if they're not being fixed yet
- Note design decisions made during the build (not just code changes)
- Most recent build at the top

---

## Common Failure Modes

Things that have gone wrong before and will go wrong again:

### Stale State After Phase Transitions
When the game transitions between phases (halftime, kickoff, goal celebration), ALL player state must be purged: velocities, burst targets, cooldown timers, lockout timers. If any of these carry across a transition, you get ghosts -- players running to old positions, tackled players starting locked out, burst targets pointing at the wrong side of the field.

**The fix:** Every phase transition gets an explicit state purge. List every field that needs clearing. Don't assume "resetting positions" clears velocities.

### Tackle/Interception Probability Stacking
A 10% chance per tick at 30fps means ~3 checks per second. Over a 1-second run through a defender's zone, that's a cumulative ~27% chance. Over 2 seconds it's ~47%. What feels like "10% tackle chance" to the designer feels like "I get tackled every time" to the player.

**The fix:** Either use roll-once-per-encounter (like the pass interception system), or dramatically reduce per-tick probabilities and compensate with proximity scaling.

### The Ping-Pong Problem
After a tackle, the tackled player is often the nearest player to the loose ball. Without a cooldown, they instantly regain possession. The tackle accomplished nothing.

**The fix:** 1500ms dispossession cooldown on the player who lost the ball. They're skipped in nearest-player checks during that window.

### CPU Robotic Reactions
If the CPU instantly passes the moment a defender gets close, defense feels pointless -- the human can never win the ball. 

**The fix:** Reaction delay (300ms minimum) before the CPU responds to threats. The human sees their defender closing in and feels like they have a chance before the CPU reacts.

### Viewport Sizing
If the field is sized in fixed pixels, it'll be tiny on high-DPI screens and huge on low-res ones. 

**The fix:** Always use viewport-relative sizing. Field width = viewport width minus padding. Everything else derives from that.

### Git Push Rejection
If Kip created a README or made any edit on GitHub directly, the local repo doesn't have that commit. Push will be rejected.

**The fix:** `git pull --rebase origin main` then push again. Claude Code usually handles this automatically.

---

## Communication Style

When working in this project:
- **Never end replies with follow-up questions**
- **Don't give directives or orders** -- suggest rather than command
- **Never use the phrase "scratches the itch"** or variations
- **Never use em dashes.** Use double hyphens (--) instead.
- **Be direct about what you don't know** -- don't fabricate examples or add false specificity
- **When discussing ideas, DISCUSS.** Don't immediately start building. The conversation IS the work until Kip says otherwise.
- **Build prompts must be downloadable .md files** with time/token estimates
- **Always include specific playtest instructions** in DONE notes after builds

---

## Version Numbering

### Alpha Phase
- Major builds: Alpha 1.0, 2.0, 3.0
- Feature additions within a major: Alpha 1.1, 1.2, 1.3
- Bug fixes: Alpha 1.1.1, 1.1.2
- The game is in Alpha until it's genuinely playable and fun for the bar test audience

### Beta Phase
- Beta 1.0: First version that passes the bar test end-to-end
- Beta increments for polish, balance, additional content
- The game is in Beta until it's ready for strangers to play without caveats

### Release
- v1.0: First public release
- v1.1, v1.2 for updates
- v2.0 for major feature additions
