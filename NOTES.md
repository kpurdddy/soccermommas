# Soccer Mamas -- Build Notes

## Alpha 2.4.4 (2026-03-05)
- Teleport zoom bug fix: MOVE tool now clears route, burstTarget, cooldownUntil, and settlingUntil when repositioning a player, preventing stale movement commands and frozen timers
- Ball follows carrier: if the moved player has possession, the ball teleports with them
- Ball drag strips possession: dragging the ball away from a carrier clears carrier, sets hasBall false, and cancels any active passBall or shot animation
- Ball drag always resets loose timer: dragging an already-loose ball gives a fresh LOOSE_BALL_TICKS window so nearby players don't instantly grab it
- Known issue: GK clamping in driftPlayers snaps goalkeepers back to the goal box after teleport -- fix deferred to next build
- Version updated to Alpha 2.4.4

## Alpha 2.4.3 (2026-03-05)
- Tactical pause in all modes: PAUSE/RESUME button now visible during play phase in solo and multiplayer (bottom-right corner), not just sandbox
- Pause uses dispatchAndSend so it works over multiplayer networking -- joiner can pause, host freezes tick loop, both players draw routes for their own team while paused
- Renamed sandboxPaused to tacticalPaused and SANDBOX_PAUSE/SANDBOX_RESUME to TACTICAL_PAUSE/TACTICAL_RESUME throughout codebase (8 replacements)
- "PAUSED -- Draw routes for your players" overlay text appears when paused in solo/multiplayer only (pointerEvents: none so it doesn't block touch)
- Sandbox mode unchanged -- keeps its full toolbar (PAUSE + MOVE + ROUTE), no PAUSED overlay shown
- Zero game logic changes, zero networking changes -- all infrastructure already existed from sandbox build
- Version updated to Alpha 2.4.3

## Alpha 2.4.2 (2026-03-05)
- Shot probability fix: saveChance was calculated but never used in the roll logic -- replaced entire probability block so keeperFactor now correctly modulates save rates; keeper distance uses tiered thresholds (>15=0, >8=0.15, else 0.4-1.0); un-saved on-target shots become goals
- Shot save animation: ball now stops at keeper's x-position instead of goal line on saves
- Route visibility in sandbox: opponent route lines and waypoint dots now visible in sandbox mode (still hidden in normal play)
- Sandbox MOVE/ROUTE toolbar: added coachTool state (route/move); toolbar shows PAUSE/RESUME + MOVE + ROUTE buttons; MOVE tool enables drag-to-teleport for any player or the ball; ROUTE tool preserves original drag-to-draw-route behavior; field cursor changes to grab when MOVE is active
- TELEPORT_ENTITY reducer action: moves player or ball to exact coordinates, updates homeX/homeY so players stay put after teleport
- Pointer handler rewrite: handleFieldPointerDown/Move/Up fully replaced to support mode-based branching (ball drag, player move, route draw); tap logic preserved verbatim
- Targeted CPU suppression: CPU carrier with an active route skips cpuPossession logic (no auto-pass/shoot override), but defensive behavior and tackles still fire normally
- Version updated to Alpha 2.4.2
- Token count: ~22k; Build time: ~12 min

## Alpha 2.4.1 (2026-03-04)
- Sandbox opponent drag fix: removed incorrect pause requirement for dragging opponent players -- in sandbox mode, opponent players can now be dragged at any time (paused or live), while non-sandbox modes still block opponent dragging
- Version updated to Alpha 2.4.1

## Alpha 2.4.0 (2026-03-04)
- Sandbox mode: new single-player mode selectable from title screen via green SANDBOX button
- Pause/Resume button appears in top-right corner during sandbox play phase
- When paused: game clock frozen, tick loop frozen, all player positions frozen
- While paused: drag ANY player (home or away) to reposition or draw routes for them
- While paused: tap actions (pass, shoot, through ball, tackle) are disabled -- only drag/route works
- On resume: players with drawn routes follow them, players without routes drift normally
- sandboxPaused flag in state controls freeze -- no new game phases added
- Reducer actions: SANDBOX_PAUSE and SANDBOX_RESUME toggle the flag
- sandboxMode module var ensures pause button only appears in sandbox sessions (not solo/multiplayer)
- startSoloGame resets sandboxMode to false so regular solo play is unaffected
- Version updated to Alpha 2.4.0

## Alpha 2.3.3 (2026-03-04)
- Shot direction fix: keeperFactor now checks keeper X position relative to shooter -- if keeper is behind the shooter (further from goal), save chance drops to 0.05 instead of using normal Y-distance calculation
- Shot direction fix: removed save result targetX override that sent the ball toward the keeper's X position -- ball now always flies toward the goal line on saves, keeper slides to intercept via updateShotAnimation
- Multiplayer crash prevention: added null check on netState.conn, try/catch around conn.send(), and 50ms throttle (lastSyncRef timestamp gate) to prevent WebRTC RTCDataChannel buffer overflow from crashing the game loop
- Multiplayer crash prevention: added netState dependencies to game tick useEffect for proper cleanup on connection state changes
- iPad joiner HUD fix: moved HUD divs (version text, score/half/clock) to render AFTER the SVG in DOM order so Safari compositor cannot layer the SVG on top of the HUD on joiner devices
- Version updated to Alpha 2.3.3

## Alpha 2.3.2 (2026-03-04)
- OOB fix: lowered carrier out-of-bounds threshold from 0/100 to 2.5/97.5 so sideline/endline triggers actually fire (route clamps and ARRIVAL_DIST prevented players from ever reaching 0/100)
- OOB fix: added boundary check for passes in flight -- ball crossing sideline/endline during a pass now triggers throw-in/goal kick/corner correctly
- OOB fix: added boundary check for loose balls as a safety net
- iPad HUD: added env(safe-area-inset-top) padding to .hud-overlay and .hud-version so score/clock/version are visible below the status bar/notch on iPad Safari
- Added viewport-fit=cover to meta viewport tag (required for safe-area-inset CSS to work)
- WiFi/VPN note: added helper text on Host and Join screens: "Best on the same WiFi. Works over the internet but may be laggy. Turn off VPNs."
- LICENSE file: added copyright notice (all rights reserved, no reuse/modification/distribution)
- Version updated to Alpha 2.3.2

## Alpha 2.3.1 (2026-03-01)
- Host authority sync: Host is now single source of truth for game state
- Joiner's tick loop disabled in multiplayer -- Joiner only renders state received from Host
- Host broadcasts full gameState to Joiner via __STATE_SYNC__ message every frame (~30fps)
- Joiner inputs (pass, shoot, route, tackle) sent to Host only, not dispatched locally
- Host dispatches Joiner actions into its reducer; result comes back in next state broadcast
- Fixes state divergence issue from 2.3.0 where independent tick loops caused mismatched positions/scores/clock
- Solo play completely unchanged -- all multiplayer routing dormant when netState.isMultiplayer is false
- Seeded PRNG kept in place for potential future deterministic sync optimization
- Version updated to Alpha 2.3.1

## Alpha 2.3.0 (2026-03-01)
- Two-player local WiFi via PeerJS WebRTC: Host Game shows 4-digit room code, Join Game connects with code
- Title screen now offers Play Solo, Host Game, and Join Game (replaces single Kick Off button)
- Team-aware input routing: localTeam variable controls which team the player interacts with (home for host/solo, away for joiner)
- Action transmission: player-initiated actions (pass, shoot, through ball, route, tackle) sent to remote peer via dispatchAndSend helper
- CPU possession disabled for away team in multiplayer mode so human player controls it
- Reducer functions (tapPlayer, tapField, throughBall, tapDefender, setRoute, dragPlayer) made team-agnostic to accept actions from either team
- Seeded PRNG reseed on connection: host generates shared seed, both phones call reseedRNG for deterministic simulation
- Disconnect handling: connection loss shows error and returns to menu
- Shoot indicator, route lines, and win/lose text all use localTeam instead of hardcoded 'home'
- Version updated to Alpha 2.3.0

## Alpha 2.2.2 (2026-02-28)
- Tackle burst distance adjusted from 5 to 6.5 game units -- tackler separates enough to avoid congestion but stays in the area for a natural takeover feel
- Role labels on player tokens: all 22 players now show two-letter abbreviations (GK, LB, CB, RB, LM, CM, RM, ST) centered inside their circles in white bold text
- Version updated to Alpha 2.2.2

## Alpha 2.2.1 (2026-02-28)
- Shooting range expanded from 25 to 45 game units -- players can now attempt long-range shots instead of silently getting through balls; distance factor still punishes long shots with high scatter and low goal chance
- Tackle burst distance reduced from 8 to 5 game units -- tackler no longer flies away after winning the ball, looks like a natural takeover
- Version updated to Alpha 2.2.1

## Alpha 2.2.0 (2026-02-28)
- Drift tether fix: increased HOME_PULL multiplier from 0.3 to 1.5 so players hold position near where you place them via route drawing instead of sliding toward the goal
- Seeded PRNG: replaced all 20 Math.random() calls with a deterministic mulberry32 PRNG (gameRNG). Seeded with Date.now() for normal play; reseedRNG(seed) function added for future two-player shared-seed mode
- Version updated to Alpha 2.2.0

## Alpha 2.1.2 (2026-02-28)
- Fix magnetic goalie: removed targetY override on saves so the ball travels where the player aimed, not directly at the keeper; keeper now visibly dives to intercept using existing interpolation logic
- Version updated to Alpha 2.1.2

## Alpha 2.1.1 (2026-02-28)
- Fix rubber-banding: route endpoint now rewrites homeX/homeY for outfield players so they stay where you put them instead of snapping back to formation
- Fix GK route: moved route/burstTarget checks above GK clamp block in driftPlayers so keepers can follow drawn routes; when route ends, keeper drifts back to goal naturally
- Version text now yellow (#FFD700) and 1.2em for visibility
- Version updated to Alpha 2.1.1

## Alpha 2.1.0 (2026-02-28)
- Waypoint route system: drag draws a path of waypoints, player follows them in sequence instead of beelining to finger position
- Visible route lines on the field: dashed gold for ball carrier, blue for other home players, with small dots at each waypoint
- isDown flag on dragRef prevents double-event (pointerUp + pointerLeave) from firing phantom through balls on drag release
- Shot zone widened back to 15 game units on X axis (was 8) -- fixes untappable zone on phones and prevents "through ball to goalkeeper" problem
- Route cleanup: through ball overrides route, tackle clears route, OOB clears route, shot clears route; pass intentionally does NOT clear passer's route (pass-and-move)
- Players on routes skip drift movement to prevent jittery path-following
- Landing page text: "Game Mode" renamed to "Live Play", "Coaching Sandbox" renamed to "Tactical Mode (Coaching)", descriptions updated
- Version updated to Alpha 2.1.0

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
