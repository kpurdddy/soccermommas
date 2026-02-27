# Soccer Mamas -- Build Prompt: Alpha 1.0.1
Version: Alpha 1.0.1 (from 1.0)
Date: 2026-02-26
Source file: index.html
Location: F:\NicBall\index.html

## Estimates
Estimated tokens: ~8-10k
Estimated time: ~10 minutes

## Pre-Build Checklist
1. Backup current file -> index.html-backup-1.0.html
2. Read the current file before making changes
3. This is a TARGETED update -- do not rewrite systems, do not change game logic

## What This Build Does
Two things only:
1. Add project info section below the existing title screen
2. Remove the forced landscape rotation gate (let the game load in any orientation)
3. Show version number and game name during gameplay

## Change 1: Landing Page Content Below Title Screen

The existing title screen (Soccer Mamas / Nicole's a Total Baller / KICK OFF) stays exactly as it is, except update the version to "Alpha 1.0.1 test build".

Below the title screen, add scrollable content in this exact order:

### A. Feedback Section (first thing below title screen)
- Box/card with background slightly different from page background
- Text: "Played it? Broke it? Have thoughts? Let me know."
- Big green button: "Give Feedback"
- Links to: https://docs.google.com/forms/d/e/1FAIpQLScpBfQ2cYIZmZngcsAf6fN7LhZ4EnUeMTZ7wswL-0FovC4c0g/viewform?usp=header

### B. "About the Project" heading

### C. Intro paragraph
"This is a game design project, primarily for **teaching and learning** about soccer and football. Soccer Mamas is a very rough first-draft prototype. Everything is Alpha versioned -- not expected to completely work."

Note: "teaching and learning" should be visually bold/highlighted.

### D. Game Mode block (visually distinct -- left border accent or card)
Title: "Game Mode"
Text: "**Single and two-player** (on separate devices). Gets closer to realism in scoring and ball movement at higher tiers -- from Kickabout (training wheels, lots of goals) through Friendly, League Match, and Cup Final."

Note: "Single and two-player" should be highlighted (green/bold).

### E. Sandbox Mode block (same visual treatment as Game Mode)
Title: "Sandbox Mode"
Text: "Manual placement of all players for instruction purposes. Run the game from that point to show the importance of positioning and setting up plays. For coaches and players learning the game."

### F. Personality powers paragraph
"On the map: **individual player personalities** with corresponding special abilities -- grouchy (other players avoid), garrulous (teammates congregate around), scary (clears a lane), always on the phone (good at longer distance plays) -- and **specific names and avatars/photographs** for teaching or just for fun."

Note: "individual player personalities" and "specific names and avatars/photographs" should be bold/white (visually prominent).

### G. Landscape note
Brief, non-demanding: "Best played in landscape mode" -- small text, not a gate.

### H. Other alphas links
Caption: "Other testing alphas in the same project:"
- Soccer Mamas: https://kpurdddy.github.io/soccermommas/
- Hail Marys (tactical football): https://kpurdddy.github.io/Hailmarys/

### I. Experimental note (at bottom, separated)
"This is an experimental project. It could prove too ambitious, but that's the point of trying. Someone else might take it over -- we'll see."

Italicized or muted color. Separated from content above with a subtle divider.

### Style Notes
- Match the existing dark background (#1a1a2e or whatever the current file uses)
- Green accent (#2ecc71 or whatever the current KICK OFF button uses)
- Max-width ~700px centered for readability
- NEVER use em dashes anywhere. Use double hyphens (--) only.
- The title screen should NOT be full viewport height. Reduce so project info is partially visible without scrolling, inviting the user to scroll down.

## Change 2: Remove Portrait Rotation Gate

Find the code that detects portrait orientation and blocks the game with a "rotate your phone" message. Remove the blocking behavior entirely. The game should load and be playable regardless of phone orientation.

Do NOT add a new rotation message in the game itself -- the landing page already has "Best played in landscape mode" as a gentle note.

## Change 3: Version and Game Name During Gameplay

Show "Soccer Mamas" and "Alpha 1.0.1" somewhere visible during gameplay -- small text, corner of the screen, not obstructing play. Should be visible but not prominent.

## Post-Build Requirements
1. Update version number to Alpha 1.0.1 in the file (title screen and gameplay display)
2. Save the file
3. Git add, commit with message "Alpha 1.0.1: landing page, remove portrait gate, version display", push
4. Update NOTES.md
5. Print DONE summary

## Playtest Checklist
1. Load the page -- title screen appears with KICK OFF button
2. Scroll down -- feedback form link visible immediately below title area
3. About the Project section visible with all content
4. Game Mode and Sandbox Mode visually distinct blocks
5. "Single and two-player" is highlighted
6. "individual player personalities" and "specific names and avatars/photographs" are bold
7. Feedback button links to Google Form (click it, verify)
8. Hail Marys link works
9. "This is an experimental project..." line visible at bottom
10. Click KICK OFF -- game loads
11. During gameplay, "Soccer Mamas" and "Alpha 1.0.1" visible on screen
12. Open on phone in portrait -- game loads (no rotation gate blocking it)
13. No em dashes anywhere in visible text
14. "Best played in landscape mode" note visible on landing page
