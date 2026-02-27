# HAIL MARYS — CRITICAL REFERENCE
### This document is ALWAYS in the project. Every conversation reads it. No exceptions.

Last updated: 2026-02-25

---

## WHAT THIS IS

This is the "never lose this" document for Hail Marys, a turn-based football strategy game. It contains every design decision that has been settled, every feature that keeps regressing, and every mistake that keeps being repeated. If something is in this document, it is NOT up for debate — it was decided through extensive discussion and should be implemented exactly as described.

The full design bible (`design-bible-hail-marys.md`) has comprehensive detail. This document is the short version that prevents the most common failures.

---

## THE GAME

**Name:** Hail Marys (no apostrophe)
**Current baseline:** Alpha 2.2.4 (cleanest 2.2.x version)
**Tech:** Single-file HTML, React 18, useReducer, Babel CDN
**Code style:** React.createElement only (NO JSX). function() declarations (NO arrow functions).
**Repo:** github.com/kpurdddy/Hailmarys (public, main branch)

---

## THE BAR TEST

The primary design filter: "A girlfriend of a football fan, learning football at a bar, picks up a phone and understands in 10 seconds." If a feature fails this test, it doesn't ship. This is not a guideline — it is the foundational principle of the entire game.

**What this means in practice:**
- A glancing look at the screen should tell you what to do next
- No wall of text. No studying required. Key info pops visually.
- Buttons tell you what they DO, not what they're called in football jargon
- The game teaches football through play outcomes, not instruction manuals
- Preseason uses plain English: "The defender closed the running lane" not "LB filled gap"

---

## ITEMS THAT KEEP REGRESSING — DO NOT CHANGE THESE

These features have been requested, agreed upon, and implemented at least once. They keep getting lost in rebuilds and handovers. **Implement them exactly as described. Do not simplify, omit, or reinterpret.**

### 1. Dan & Kiki Commentary: ON THE FIELD
- Dan & Kiki render as a **semi-transparent overlay ON the football field itself**
- Broadcast-booth style graphic sitting on top of the game, **top area of the field**
- **NOT text below the field. NOT a line at the bottom of the screen. NOT in the control panel.**
- Appears **2.75 seconds** after play ends (delay so player sees the result first)
- Dismiss button **below the text**, not in a header bar
- Stays on screen until player dismisses it (NO auto-dismiss timer)
- Dan = factual play-by-play, white/blue text. Kiki = personality/color, gold text.
- Teaching mode (kikiTeach) active in Practice and Preseason
- **THIS HAS REGRESSED THREE TIMES. It was working. Then rebuilds moved it below the field. It must be on the field.**

### 2. Button Text: "Drop Back (Run the Play)"
- The main action button in DECISION phase 1 says **"Drop Back (Run the Play)"**
- "(Run the Play)" is in parentheses — it tells newcomers what to do
- Phase 2: button changes to **"Buy Time"** (subtitle: "Receivers getting open")
- Phase 3+: button changes to **"Hold On!"** (subtitle: "Pocket is shaky — make a decision soon")
- **This is NOT "Run the Play" by itself. It is NOT just "Drop Back" by itself. It is "Drop Back (Run the Play)" with the parenthetical.**
- This has been requested repeatedly and keeps not happening.

### 3. Escape Buttons: HIDDEN When Pocket Is Healthy
- Roll Left, Roll Right, Tuck & Run, Throw Away — these are **NOT visible** at phase start
- They **appear only when pocket integrity degrades** below a threshold
- Roll L/R show context: "Roll Left — move away from pressure. W1 is on this side."
- Roll L/R were renamed from "Scramble" — never use "Scramble"
- When they appear, a brief sequential flash animation (300ms per button, one cycle) draws the eye

### 4. No Popping Zoom — EVER
- `FOLLOW_PHASES` must be `[]` (empty array). This is the line that causes the camera pop.
- Full field is the default view. Camera tracks action during live play on small screens with CSS transform on the field container div.
- Camera panning is a single `transform: translateY()` on the field container. NOT baked into individual player coordinate calculations.
- The popping zoom was the single most hated visual bug. It must never return.

### 5. Overlay Panel ON the Field
- During DECISION, RUNNER, CONTACT: the control panel renders as an **absolutely positioned overlay at the bottom of the field viewport**
- Semi-transparent gradient background
- All other phases (MENU, PLAY_SELECT, PRESNAP, RESULT, FOURTH_DOWN): panel below field in normal document flow
- During overlay phases: LOS anchored at **top 20%** of viewport, end zone off-screen above
- Down & distance HUD moves to **top of field** during overlay phases (bottom otherwise)
- This applies to **ALL screen sizes**, not just mobile

### 6. Decluttered Control Panel
- The control panel in its current state has too much information competing for attention
- A newcomer glancing at the screen cannot tell what to do
- Primary receiver gets gold TARGET badge and gold border on their card
- Coach advice names the primary receiver by name on pass plays
- Contact buttons show percentages: "55% break free / 3% fumble (1 in 33)"
- The goal: at a glance, the player knows what their best option is

### 7. Practice Difficulty on Title Screen
- All four difficulties visible on the title screen: Practice, Preseason, Regular Season, Playoffs
- Difficulty label shows on the scoreboard during gameplay ("PRESEASON" etc.)

### 8. Primary Receiver Highlight
- The play's designed primary target gets a gold border and "INTENDED" badge on their receiver card
- Fix is: `play.primary === rk` (NOT `rk === bestKey` which compares against the most-open receiver)
- Coach advice at PRESNAP references primary by name: "This play targets W1"

---

## ARCHITECTURE — NON-NEGOTIABLE

- **One system positions players.** React state is the single source of truth. CSS transitions handle smooth movement. NO requestAnimationFrame for player positions. NO visualPos refs. NO parallel DOM manipulation.
- **useReducer, not useState.** One dispatch, one state update, no partial-flush bugs.
- **Camera via CSS transform on field container.** Players positioned relative to field with NO camY in their coordinates. Camera is one div's transform, not 22 recalculated positions.
- **Turn-based DECISION phase.** ANIMATION_TICK does NOT move defenders during DECISION. Defenders move one step per QB action (Drop Back, Roll, Step Up, Look). Between actions, field is frozen. RUNNER phase stays real-time.

---

## DO NOT DO — LEARNED FROM BITTER EXPERIENCE

1. **Do not rewrite the whole file to fix one bug.** Every rewrite has produced worse results than targeted fixes. One change, test, commit or revert.
2. **Do not use rAF for player positions.** This caused the unfixable teleporting bug across Alphas 12-15.5. Two systems fighting over positions = players snapping between locations.
3. **Do not put Dan & Kiki below the field.** (See item 1 above.)
4. **Do not let Claude Code improvise numbers.** If a data file exists with the correct values, Claude Code must read it and use those values. Not make up new ones.
5. **Do not add features during bug fix passes.** Fix what's broken. Nothing else.
6. **Do not summarize design decisions generically in handovers.** "Commentary system" is not enough. "Dan & Kiki render as overlay ON the field, not text below" is what survives.
7. **Three-strike rule:** If a bug fix fails twice, stop patching. Structural rewrite of that specific system. No arguing.
8. **Do not fight Kip on requested changes.** If he says "put Run the Play on the button," put Run the Play on the button. Don't debate it, don't suggest alternatives, don't note it for later. Do it.
9. **Do not use JSX or arrow functions.** The codebase uses React.createElement and function declarations throughout. Mixing styles breaks Babel transpilation.
10. **Do not have two systems controlling the same thing.** One source of truth per feature. If React state controls positions, nothing else touches positions.

---

## COACHES

**Baby Boy Joey — The Gunslinger**
"Young, cocksure, and always scheming. Calls plays that make defensive coordinators swear. When they work, they're spectacular — and when they don't, his grit and hustle already have the next one drawn up."
Visual: Visor, cool sunglasses, headset. Slim-fit polo. Hotshot energy.

**Grizzled Jim — The Old Fox**
"Forty years on the sideline. Players run through walls for him. Knows the game cold, never panics, and a steadying presence when things get crazy."
Visual: Old-school. Big white mustache/beard. Weathered face. Windbreaker. Seen-it-all energy.

---

## THE MELEFICENT

Melissa's 92-yard field goal Easter egg. NOT Melissafent. NOT Maleficent. **Meleficent.**

- Any FG attempt over 60 yards triggers the Meleficent event
- All FG attempts need ball flight animation (arc, hang time, visual result)
- 15+ randomized failure messages (pool exists, fully written)
- Pity mechanic: after multiple attempts, opposing team "gives the down back"
- Secret cheat code so Kip can trigger success when Melissa is playing
- Natural success chance: 0.01%
- Success sequence: dramatic yard-by-yard countdown, massive celebration, fireworks, confetti
- Fully designed. Not yet built.

---

## CURRENT KNOWN BUGS (Alpha 2.2.x)

1. Overlay positioning broken on some phases (renders below field instead of on it)
2. Escape buttons always visible (should be hidden until pocket degrades)
3. Look L/R restricted to deep plays only (spec says ALL passing plays)
4. Sprint-to-Run label swap broken (Sprint fades but Run label doesn't appear)
5. Dive doesn't end play (can be used repeatedly)
6. Dan & Kiki below field instead of on-field overlay
7. Button text doesn't say "Drop Back (Run the Play)"

---

## WORKFLOW QUICK REFERENCE

(Full workflow details in workshop-manual-all-games.md)

- **Design conversations:** Claude.ai chat. Talk through mechanics. No code.
- **Build execution:** Claude Code in PowerShell. Paste prompts.
- **Authorization:** Kip must say "Go with the [X]" before any code execution. That exact phrase is the ONLY authorization.
- **File location:** F:\HM2\ on thumb drive (may vary by PC)
- **Every build:** Backup first (auto-increment). Update NOTES.md. DONE summary at end.
