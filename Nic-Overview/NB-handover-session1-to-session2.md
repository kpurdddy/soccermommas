# NB-Handover -- Project Folder Session 1 to Session 2

**Date:** 2026-02-26
**From conversation:** First conversation inside the project folder. Landing page design, workshop manual updates, workflow enforcement.
**Next step:** Build Alpha 1.0.1 in Claude Code (build prompt already written), then move to Alpha 1.1 gameplay fixes.

---

## What Was Accomplished This Session

### Project Folder Established
- This was the first conversation inside the Claude project folder
- Confirmed which documents belong in the knowledge base vs. per-conversation uploads
- Established file naming convention: NB prefix for Soccer Mamas, HM prefix for Hail Marys, no prefix for shared docs (stored in Claude memory)

### Landing Page Designed (Alpha 1.0.1)
The existing Soccer Mamas title screen stays as-is (Soccer Mamas / Nicole's a Total Baller / KICK OFF). Below it, scrollable content is added:

1. **Feedback section** (immediately below title screen, prominent green button)
   - Text: "Played it? Broke it? Have thoughts? Let me know."
   - Links to: https://docs.google.com/forms/d/e/1FAIpQLScpBfQ2cYIZmZngcsAf6fN7LhZ4EnUeMTZ7wswL-0FovC4c0g/viewform?usp=header

2. **"About the Project" section** with:
   - Game design project, primarily for teaching and learning
   - Alpha caveat -- not expected to completely work
   - **Game Mode** block (visually distinct): Single and two-player (highlighted), realism tiers from Kickabout through Cup Final
   - **Sandbox Mode** block (visually distinct): Manual placement for instruction, run simulation from that point, for coaches and players
   - **Player personalities** paragraph: individual player personalities (bold) with abilities (grouchy, garrulous, scary, phone addict) and specific names and avatars/photographs (bold) for teaching or fun
   - "Best played in landscape mode" -- gentle note, NOT a gate
   - Links to both games, captioned "Other testing alphas in the same project"
   - "This is an experimental project. It could prove too ambitious, but that's the point of trying. Someone else might take it over -- we'll see."

3. **Version and game name visible during gameplay** -- small text, corner of screen

4. **Portrait rotation gate removed** -- game loads in any orientation

### Build Prompt Written
- File: `NB-build-prompt-alpha-1.0.1.md` (already downloaded by Kip)
- Covers all three changes: landing page content, portrait gate removal, version/game name display
- Estimated ~8-10k tokens, ~10 minutes
- Includes full playtest checklist (14 items)
- Ready to paste into Claude Code

### Workshop Manual Updated
- File: `workshop-manual-all-games.md` (already downloaded by Kip)
- **Added:** Context Truncation -- Hard Rule section (its own ## heading, right after Trigger Phrase Rule)
- **Added** to trigger phrase "does NOT count" examples: "I think we're ready to go with X" and "Go ahead"
- **Added** to Communication Style: "Never use em dashes. Use double hyphens (--) instead."
- **Changed** Mobile section: landscape preferred but do NOT gate the game behind forced rotation
- **Changed** all em dashes to double hyphens throughout entire document
- **Changed** arrows to ->
- Kip needs to upload this to the knowledge base, replacing the current version

### Trigger Phrase Enforcement
- Three trigger phrase violations occurred this session (two by Claude, one caught in time)
- The rule was stress-tested and held. New failure modes ("I think we're ready to go with X" and "Go ahead") have been added to the workshop manual's explicit "does NOT count" list
- The rule works. Follow it exactly.

---

## Decisions Made This Session

These are settled. Don't relitigate.

- **File naming:** NB prefix for Soccer Mamas docs, HM prefix for Hail Marys docs, no prefix for shared docs
- **No em dashes anywhere.** Double hyphens (--) only. This applies to all documents, all code comments, all build prompts, all conversation output.
- **No forced portrait rotation gate.** Gentle "best played in landscape" note on landing page instead.
- **Landing page lives on the existing title screen** -- scrollable content below KICK OFF, not a separate page or redirect
- **Feedback form link:** https://docs.google.com/forms/d/e/1FAIpQLScpBfQ2cYIZmZngcsAf6fN7LhZ4EnUeMTZ7wswL-0FovC4c0g/viewform?usp=header
- **"About the Project"** not "About This Project"
- **"Let me know"** not "Let us know"
- **"ambitious"** not "audacious"
- **Version number and game name visible during gameplay**
- **Alpha 1.0.1** is the landing page build. Alpha 1.1 is the gameplay fixes (viewport, shot selection, formation, lanes).

---

## Knowledge Base Status

### Currently in knowledge base:
- workshop-manual-all-games.md (NEEDS REPLACEMENT with updated version downloaded this session)
- NB-roadmap-soccer-mamas.md
- NB-gemini-review-soccer-mamas.md
- HM-roadmap-hail-marys.md
- HM-critical-reference-hail-marys.md

### Needs to be added:
- Updated workshop-manual-all-games.md (replace current version)

### Not in knowledge base (by design):
- Build prompts (per-conversation)
- index.html game files (too large, upload per-conversation when building)
- Handover docs (one-time bridge documents)

### Still needed (not yet created):
- HM-design-bible-hail-marys.md (generate from prompt-compile-hailmarys-bible.md)
- NB-design-bible-soccer-mamas.md (if not already created)

---

## Immediate Next Steps

### 1. Update Knowledge Base
- Replace workshop-manual-all-games.md with updated version (already downloaded)

### 2. Build Alpha 1.0.1 in Claude Code
- Directory: `F:\NicBall\`
- File: index.html
- Build prompt: NB-build-prompt-alpha-1.0.1.md (already downloaded)
- `cd F:\NicBall`, launch Claude Code, paste prompt
- Git push after build to deploy to GitHub Pages

### 3. After 1.0.1 -- Plan Alpha 1.1 (Gameplay Fixes)
These are the critical gameplay issues from playtesting (documented in NB-roadmap-soccer-mamas.md):
- Viewport-responsive field sizing (field fills screen, not tiny rectangle)
- Shot selection system (aim left/right/center, not just dice roll)
- Stronger formation discipline (prevent offsides drift)
- Positional lane constraints (defenders stay in zones)

### 4. Thursday Meeting with Nicole
- She plays the updated game beforehand
- Full design doc review
- Student personalities for powers
- Coaching sandbox concept review
- Formation and positional authenticity

---

## File Locations

### Soccer Mamas
- **GitHub repo:** https://github.com/kpurdddy/soccermommas
- **GitHub Pages:** https://kpurdddy.github.io/soccermommas/
- **Local:** F:\NicBall\ (thumb drive)
- **Overview docs:** F:\NicBall\Nic-Overview\

### Hail Marys
- **GitHub repo:** https://github.com/kpurdddy/Hailmarys
- **GitHub Pages:** https://kpurdddy.github.io/Hailmarys/
- **Local:** F:\TacFootball\ (thumb drive)

### PC
- Username varies: `obrie`, `domain`, `Lash2` -- check or ask
- Claude Code launch: `cd F:\NicBall`, `claude --dangerously-skip-permissions`, paste prompt

---

## Workflow Reminders

- **NEVER build without "Go with the [X]"** -- three violations this session, all caught. The rule is absolute.
- **No em dashes.** Double hyphens (--) only. Everywhere. Always.
- **Target 15-20k tokens per build prompt** -- 32k ceiling is real
- **Backup before every build**
- **Git push after every build** -- if rejected, pull --rebase first
- **Update NOTES.md after every build**
- **Playtest checklist in every DONE summary**
- **Warn about context truncation** before it becomes a problem, suggest new thread with handover summary
