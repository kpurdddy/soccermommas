# Soccer Mamas -- Build Notes

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
