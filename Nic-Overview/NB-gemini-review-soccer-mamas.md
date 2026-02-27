# Gemini Review Contributions

Additions from Gemini's architectural review. These supplement the Design Bible and Workshop Manual — they don't override anything in those documents.

---

## Viewport Units Fix

Use `100dvh` and `100dvw` instead of `100vh`/`100vw` for the game container.

The `dv` (dynamic viewport) units account for mobile browser chrome that appears and disappears — address bar, bottom toolbar. Standard `vh`/`vw` units calculate against the full viewport including the space behind browser UI, which means the field can end up partially hidden on phones.

This applies to all games in this project folder.

---

## Architecture Language

Two cleaner descriptions of concepts already documented:

- The game is a **deterministic state machine.** Given the same initial state and the same sequence of actions, the game always produces the same result. This is what makes multiplayer possible without syncing the full game state.

- The multiplayer model is **lockstep peer-to-peer.** Both devices run the same math at 30fps. The network only transmits user inputs (actions), not positions or game state. Each device computes the same outcome independently.

---

## clearAllIntentions() — Named Purge Function

Instead of inlining the state purge logic at every phase transition (halftime, goal reset, kickoff), centralize it in one named function: `clearAllIntentions()`.

This function nullifies:
- All player velocity vectors (vx, vy)
- All burstTarget values
- All burstCooldownUntil timers
- All cooldownUntil (dispossession) timers
- All settlingUntil timers
- ball.lastCarrierId
- ball.checkedBy

Call it at every phase transition before resetting positions or flipping coordinates.

**Why this matters:** When the purge is inlined in multiple places, it's easy to forget a field in one of them. One function, called everywhere, means one place to update if new state fields are added later.
