# Prompt: Compile Hail Marys Design Bible

Paste this into a new conversation (or into the project folder once it exists):

---

**Search all my past conversations about Hail Marys / Tactical Football / TacFoot and compile a comprehensive design bible for the game.**

This document is called `design-bible-hail-marys.md`. It goes in a project folder alongside a shared Workshop Manual and a separate Soccer Mamas design bible. Don't duplicate general workflow stuff (that's in workshop-manual-all-games.md). Focus on what's philosophically and mechanically specific to Hail Marys.

**Cover all of the following:**

## 1. Core Philosophy (Hail Marys version)
- The bar test applied to football — "girlfriends of football fans learning football at a bar"
- Two-audience design: newcomer vs expert, and how the tier structure (Practice → Playoffs) serves both
- Teaching through failure: how play calling, coverage reading, and decision-making are learned through consequences, not tutorials
- Fixed-field architecture: full field visible, camera defaults to strategic view, follows action during plays
- Transparent mechanics: the player sees why a play worked or failed
- Difficulty through parameters, not complexity: same plays, same controls, different tuning per tier
- Scoring calibration per tier: Practice = touchdowns every drive, Playoffs = realistic but not boring

## 2. The Control Scheme and Phase System
- How the turn-based phase state machine works: SELECT_PLAY → SNAP → DECISION → THROW/RUNNER → CONTACT → RESULT
- What the player does in each phase (select play, read defense, choose QB action, pick target/runner direction)
- The QB button system: Drop Back, Roll Left/Right, contextual changes based on game state
- The RUNNER phase: post-catch/post-handoff movement, arrow system, sprint/juke/truck decisions
- The RPO system: how run-pass options work

## 3. All Plays and Defenses
- Full list of all 18 plays with descriptions and what they're good for
- The 5 defensive formations and what each one does
- How the coach recommends plays — the recommendation logic and matchup system
- Audibles — what they are, when they're available

## 4. The Tier System in Detail
- Practice: what's different, what's forgiving, what tips appear
- Preseason: heroic/easy feel, what changes from Practice
- Regular Season: the standard competitive game
- Playoffs: realistic, what the experienced player faces
- Specific tuning differences between tiers (defender speed, interception rates, CPU behavior)

## 5. People and Characters
- Dan & Kiki: the commentary system, their personalities, what they say and when
- Coach system: how recommendations work, what's planned for coach personalities
- The Meleficent: Melissa's 92-yard field goal Easter egg — the concept, failure messages, pity mechanic, Kip's secret cheat code
- Any other named characters, inside jokes, or personal references

## 6. Every Design Decision That's Been Settled
Search thoroughly. These are things we've debated and resolved — they should not be relitigated:
- Button labels (Drop Back with "(Run the Play)" in parentheses)
- Camera behavior (full field default, action tracking during plays)
- Overlay panel layout as default for all screens
- Play naming conventions
- Commentary system design
- Anything else that came up repeatedly and was finally decided

## 7. Architecture
- Phase state machine: how it works, what each phase does
- The reducer pattern (React 18 + useReducer)
- How plays are defined in code (the array structure, waypoints, route definitions)
- How the contact resolution system works
- How the field rendering works
- Single-file HTML with inline JSX and Babel CDN

## 8. Build History
- Narrative of how the game evolved from v1 through Alpha 2.2.1
- Key turning points (when it first felt fun, when major systems were added)
- What was learned at each stage that shaped future decisions

## 9. Known Issues and Pending Fixes
- Everything currently broken or unsatisfying, pulled from the most recent conversations
- The popping zoom, RUNNER phase, field visuals, arrow percentages, etc.

## 10. Full Roadmap
- Phases 1 through 10 as discussed
- What's been designed but not built
- What's been discussed but not designed
- What's been mentioned but not discussed

## 11. Kip's Signature (Hail Marys version)
- What makes this identifiably Kip's game, not a generic football game
- The bar test as primary filter
- Fixed-field architecture
- Turn-based tactical decisions over reflexes
- Trick plays as a priority feature
- Teaching the sport through the mechanic
- The personal/social elements (Meleficent, Dan & Kiki, named characters)

**Format as a single markdown document. Be comprehensive — this is the foundational reference for all future development. If you find contradictions between conversations, note them and use the most recent decision.**
