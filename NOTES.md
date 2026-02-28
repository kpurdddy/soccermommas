# Soccer Mamas -- Build Notes

## Alpha 2.0.0 (2026-02-28)
- Major control scheme rewrite: tap-only replaced with tap + drag
- Drag any home player to move her (works on offense and defense) -- replaces tap-to-run and two-tap defense system
- Tap teammate = pass (unchanged), tap empty field = through ball to that spot (nearest teammate auto-runs to collect), tap goal area = shoot
- Through ball: ball travels to tapped position as loose ball; nearest non-GK home teammate bursts toward the spot
- Defense: tap the CPU ball carrier to commit nearest home defender to tackle (replaces select-then-tap)
- Tap vs drag detection: >10px movement = drag, otherwise tap on pointer up
- Shot scatter reduced (0.5x to 0.15x multiplier) -- player aim now matters significantly
- Tighter goal tap zone: last 8 game units between posts (y 35-65) instead of last 15 units full width -- prevents accidental shots
- Shooting zone indicator updated to match tighter zone with higher opacity
- Post-tackle separation increased from 5 to 8 units for more convincing breakaway
- Landing page Coaching Sandbox text updated to mention drawing paths
- Old input functions (tapField, selectDefender, positionDefender) left as dead code for safety
- Version updated to Alpha 2.0.0

## Alpha 1.3.1 (2026-02-27)
- Fix save crash: declared `var targetX = goalX` in executeShot and changed shot object to use `targetX: targetX` so saves animate toward keeper position
- Fix mobile tap highlight: added `-webkit-tap-highlight-color: transparent` to universal reset
- Tap sounds via Web Audio API: tic (pass/run taps), kick (shooting), whistle (kickoff) -- synthesized tones, no external files
- AudioSys.init() on first interaction handles browser autoplay policy
- Version updated to Alpha 1.3.1

## Alpha 1.3.0 (2026-02-27)
- Full-screen field: HUD removed from document flow, field now fills entire viewport edge-to-edge (padding: 0, no height reservations)
- HUD overlay: version text and score/half/clock rendered as absolute-positioned overlays inside the field wrapper with semi-transparent dark pill backgrounds
- Version text hidden on narrow screens (portrait) via media query
- All game overlays (GOAL!, SAVE!, KICK OFF, HALFTIME, FULL TIME) set to z-index: 100 to render above HUD (z-index: 10)
- Feedback button: replaced dead Google Form link with mailto:remote-tank-halves@duck.com
- Landing page: added "This is the design idea:" to end of About the Project paragraph
- Version updated to Alpha 1.3.0

## Alpha 1.2.1 (2026-02-27)
- Prominent version HUD: "Alpha 1.2.1 -- test build" now centered, bold, white, 1.2em on its own top row; score/clock/half on second row below
- HUD height allowance increased from 44px to 64px for two-row layout
- Save animation fix: ball now targets keeper position instead of goal line on saves, so ball visibly stops at keeper rather than crossing the goal line
- Title screen version updated to Alpha 1.2.1

## Alpha 1.2.0 (2026-02-26)
- Ghost ball fix: interception loop reordered so hard collision (now 2.0 units) evaluates BEFORE checkedBy gate -- balls can no longer skip through defenders
- Viewport dynamic sizing: aspect ratio unlocked, clamped between 1.4:1 and 2.4:1 -- fills wide phones and tablets properly
- Tackle cooldown: tacklers now checked for cooldownUntil, preventing immediate re-tackles after losing the ball
- Tackle rework: successful tackles give ball directly to tackler (not loose ball), tackler auto-bursts 5 units away from contact point
- SAVE! overlay: new save_display phase shows "SAVE!" text for ~1 second after keeper saves
- Defensive two-tap system: tap defender to select (green dashed ring), then tap field to position or tap ball carrier to commit tackle; tap selected defender again to deselect
- selectedDefender cleared on possession changes (interceptions, tackles, out of bounds)
- Version moved from corner overlay to HUD bar; updated to Alpha 1.2.0
- Landing page: title screen reduced to 50vh, feedback button shrunk to be secondary to Kick Off

## Alpha 1.1 (2026-02-26)
- Viewport fix: switched to 100dvh + visualViewport API for proper mobile sizing, reduced HUD height allowance to 44px, 8px edge padding
- Shot aiming: tap coordinates now pass through to executeShot; shots go where you tap (clamped to goal mouth), with distance-based scatter
- Keeper/corner awareness: aiming at corners is harder to save but riskier wide; aiming away from keeper reduces save chance
- CPU shots use the same aim system with random goal-mouth targeting
- Ghost ball fix: hard collision at <= 1.0 units -- passes can no longer phase through defenders at point-blank range
- CPU ping-pong fix: cpuPass excludes lastCarrierId so the CPU won't immediately pass back to the same player
- Landing page: added TacFootball link, renamed Sandbox Mode to Coaching Sandbox with new description
- Version updated to Alpha 1.1

## Alpha 1.0.1 (2026-02-26)
- Added landing page content below title screen: feedback link, about section, game mode/sandbox mode descriptions, personality powers, other alpha links, experimental note
- Removed forced portrait rotation gate -- game loads in any orientation
- Added "Soccer Mamas -- Alpha 1.0.1" version display during gameplay (bottom-right corner)
- Title screen reduced from full viewport to 70vh so landing content peeks in
- "Best played in landscape mode" shown as gentle note on landing page (not a gate)

## Alpha 1.0 (2026-02-24)
- Foundation build -- initial game with title screen, single-player vs CPU, basic passing/shooting/defending
