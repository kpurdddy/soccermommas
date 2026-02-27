# Hail Marys — Roadmap

**Current Version:** Alpha 2.2.1
**Status:** Playable, undergoing iterative improvement. Core gameplay loop works.
**Repo:** https://github.com/kpurdddy/Hailmarys
**Live:** https://kpurdddy.github.io/Hailmarys/
**Game name:** Hail Marys (no apostrophe)

---

## Current State — Alpha 2.2.1

### What Works
- Turn-based tactical football — choose play, watch execution, make decisions
- 18 plays (5 run, 2 special, 5 pass, 3 situational, 3 trick)
- 5 defensive formations
- 4 difficulty tiers: Practice, Preseason, Regular Season, Playoffs
- RPO (Run-Pass Option) system
- Contact resolution system
- Coach system with play recommendations
- Contextual QB buttons (Drop Back behavior changes based on game state)
- Field visualization with player tokens, yard lines, first-down marker
- Scoreboard, down-and-distance display
- Commentary system (Dan & Kiki announcer booth)
- RUNNER phase for post-catch/post-handoff gameplay
- Camera system (full field default, follows action during plays)
- Single-file React app (index.html)
- GitHub Pages deployment

### Architecture
- Single HTML file, React 18 + useReducer
- Inline JSX with Babel CDN
- Phase-based game state machine (SELECT_PLAY → SNAP → DECISION → THROW/RUNNER → CONTACT → RESULT)
- All state in reducer, actions dispatched through standard React pattern

### Known Issues
- **Popping zoom:** Camera mode switches from strategic (full field) to follow (zoomed) during RUNNER and CONTACT phases. Visually jarring. Root cause: `FOLLOW_PHASES = ['RUNNER', 'CONTACT']` triggers camera mode switch. Fix: `FOLLOW_PHASES = []` to maintain consistent field view.
- **RUNNER phase feels unrealistic:** Post-catch/post-handoff is an empty-field sprint rather than navigating through defenders. YAC (yards after catch) is too easy.
- **Field visually bland:** Flat green rectangle, no turf stripes, no hash marks, no goal posts, muddy end zones.
- **Arrow percentages not distance-weighted:** Sprint shows 75% even with nobody within 15 yards. Should be 90%+.
- **"TE Open" text:** Green on green, disappears too fast. Needs better contrast and slightly longer display.
- **Click sound:** Current chirp is annoying. Needs a dry button click.

---

## Build History Summary

The game has gone through extensive development across 15+ alpha versions:
- v1-v4: Initial concept, basic play calling and execution
- v5: Blocking, camera, runner mechanics
- v6: WYSIWYG defense (what you see is what you get)
- v7-v8: Wall blocking and pursuit
- v9: Player agency (arrow system for runner decisions)
- v10-v12: Receiver routes, interceptions, play rewrites, contextual buttons
- v13-v14: Coach system, practice mode, UI polish, commentary
- v15+: Overlay layout, camera refinement, field visuals
- 2.0+: Major restructure, overlay architecture, version numbering change
- 2.2.1: Current — overlay panel, action-camera framing, primary receiver fix

Full build history in NOTES.md in the game directory.

---

## Immediate Queue

### Alpha 2.2.2 (or 2.3.0)
- Fix popping zoom (set FOLLOW_PHASES = [])
- Arrow percentages distance-weighted (Sprint 90%+ with nobody close)
- Field visuals: alternating turf stripes, hash marks, end zone patterns, goal posts
- "TE Open" text: better contrast, slightly longer display
- Click sound: replace chirp with dry button click
- Difficulty label visible during gameplay

---

## Major Upcoming Phases

### Phase: RUNNER Overhaul (2.3.0)
The biggest gameplay change remaining. Post-catch/post-handoff needs to feel like navigating a real field:
- Slow creep tick (500-600ms) with sprint burst + cooldown rhythm
- Defender repositioning after catch — remaining defenders set up between runner and end zone
- Defenders fill lanes instead of chasing from behind
- Blockers: non-targeted receivers run alongside carrier, occupy nearest defender in path
- One evasive move per contact, max two contacts per play
- This requires design conversation before building — the numbers, blocker pathing, lane-filling all need discussion

### Phase: Teaching & Feedback (2.4.0)
The "bar test" pass on all player-facing text and guidance:
- Coach advice becomes matchup-aware: names 2 specific alternative plays with reasons
- Play names rewritten in plain English (Out Routes → Quick Sideline Throws, etc.)
- Decision screen gets primary target highlighting, read order, "this play targets W1 deep" guidance
- Post-play text pattern: what happened → why → what to try instead
- Commentary positioning as field overlay
- CPU drive play-by-play ticker (quick text feed, 3-4 seconds total)
- Defensive posture guidance with concrete descriptions

### Phase: Coach Personalities (Phase 3)
Multiple coach characters with different philosophies:
- Each coach recommends plays differently based on personality
- Mechanical tradeoff table: conservative coach vs aggressive coach vs balanced
- Coach unlock or selection system

### Phase: Player Notes + Feedback Form (Phase 4)
Let players submit feedback from within the game

### Phase: Two-Player (Phase 5)
Second player controls defense (or takes turns on offense)

### Phase: More Trick Plays (Phase 6)
Trick plays are a priority — they're fun, memorable, and distinctive:
- Expand beyond current 3 trick plays
- Each should feel surprising and rewarding when it works
- "The Meleficent": Melissa's 92-yard field goal Easter egg. 15+ random failure messages, pity mechanic, secret cheat code for Kip to trigger success.

### Phase: Game Clock + CPU Offense (Phase 7)
- Running game clock with time pressure
- CPU gets its own offensive drives (currently human is always on offense)
- CPU play calling AI

### Phases 8-10: Advanced/Season/Meleficent
- Season mode with progression
- Player stat upgrades
- Advanced features TBD

---

## Design Decisions Made

These have been discussed and decided across multiple conversations. Don't relitigate:

- **"Drop Back" is the button text** with "(Run the Play)" in parentheses. Not "Run the Play" as the primary label.
- **Preseason = heroic/easy, Playoffs = realistic.** The tiers are different games, not just difficulty sliders.
- **Trick plays are a priority feature**, not a nice-to-have.
- **The bar test audience is "girlfriends of football fans learning football at a bar."** Every design decision filters through this.
- **Commentary is Dan & Kiki**, not a generic system.
- **The overlay panel layout** is the default for all screens (decided in 2.2.2 design session).
- **Camera defaults to full field**, only tracks during active plays, returns to full field after.
- **The game teaches football.** It's not a football game that happens to be accessible — it's a teaching tool that happens to be a game.

---

## File Locations

- **Thumb drive:** F:\TacFootball\
- **Source file:** index.html (also tacfoot4.html historically)
- **Supporting files:** NOTES.md, FIXES.md, LICENSE
- **LICENSE:** "Copyright (c) 2026 SL Flanagan, All Rights Reserved"
- **Desktop copies:** TacFoot\index.html (username varies: obrie/domain/Lash2 — check or ask)
