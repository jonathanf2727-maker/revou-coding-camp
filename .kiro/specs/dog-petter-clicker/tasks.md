# Implementation Plan: Dog Petter Clicker

## Overview

Implement the entire Dog Petter Clicker app as a single self-contained HTML file. Each task builds on the previous one, moving from data and core logic up through rendering and audio, then wiring everything together into the final interactive page. The design document is the source of truth for all interfaces and pseudocode.

## Tasks

- [x] 1. Scaffold the HTML file with layout, Tailwind CDN, and static structure
  - Create `dog-petter-clicker.html` in the workspace root
  - Add the Tailwind CSS CDN `<script>` tag in `<head>`
  - Write the static DOM skeleton: `#dog` container, `#bubble` speech bubble, `#score` and `#streak` HUD elements, mute toggle button `#mute-btn`
  - Add a `<style>` block with the six CSS animation keyframes (`bounce`, `shake`, `spin`, `wiggle`, `zoom`, `float`) using only `transform` and `opacity`
  - Apply Tailwind utility classes for centering and basic visual polish
  - _Requirements: 8.1, 8.6, 9.1, 9.4, 9.5_

- [x] 2. Implement the `DogRenderer` module
  - [x] 2.1 Build the SVG dog character and expression map
    - Write the inline SVG markup for the minimalist dog face inside `#dog`
    - Define a `EXPRESSIONS` map of the seven named expressions (`happy`, `love`, `surprised`, `sleepy`, `dizzy`, `tongue`, `wink`) as SVG/HTML snippets that are swapped into the face area
    - Implement `DogRenderer.setExpression(expression)` to swap the active expression using `textContent`/`innerHTML` on the face sub-element only
    - _Requirements: 8.1, 8.2, 2.1_

  - [ ]* 2.2 Write property test for expression fidelity (Property 9)
    - **Property 9: Reaction expression fidelity**
    - **Validates: Requirements 2.1, 8.2**
    - For every expression name, assert that after `setExpression(name)` exactly that expression class/snippet is active and no other expression is simultaneously present

  - [x] 2.3 Implement `DogRenderer.playAnimation()` with guard class
    - Define `ALL_ANIMATION_CLASSES` constant array
    - Implement `playAnimation(name, duration = 600)`: remove all existing animation classes, add `name` and `animating`, then `setTimeout` to remove both after `duration` ms
    - Implement `DogRenderer.reset()` to remove all animation and expression classes
    - _Requirements: 2.2, 2.6, 8.3, 8.4, 8.5_

  - [ ]* 2.4 Write property test for animation guard (Property 5)
    - **Property 5: Animation guard prevents class stacking**
    - **Validates: Requirements 2.2, 2.6, 8.4, 8.5**
    - Assert that calling `playAnimation()` while `animating` class is present does not add a new animation class; assert all animation classes are absent after the timer fires

- [x] 3. Implement the `ScoreTracker` module
  - [x] 3.1 Implement core score and streak logic
    - Define the `STREAK_WINDOW_MS = 800` constant and the `MILESTONES` array matching Requirement 5.6
    - Implement `ScoreTracker.init()` — restore `GameState` from `localStorage` if present and valid; fall back to in-memory defaults silently
    - Implement `ScoreTracker.increment()` following the pseudocode in the design: increment score, evaluate streak window, update `lastClickTime`, check milestones in order, persist to `localStorage`, return `PetResult`
    - Implement `ScoreTracker.getScore()` and `ScoreTracker.getStreak()`
    - Wrap all `localStorage` calls in try/catch to handle unavailability
    - _Requirements: 4.1, 4.2, 4.3, 4.5, 5.1, 5.2, 5.5, 5.6, 6.1, 6.2, 6.3, 6.4_

  - [ ]* 3.2 Write property test for score monotonicity (Property 1)
    - **Property 1: Score strict monotonicity**
    - **Validates: Requirements 1.2, 4.1**
    - For any sequence of `n` `increment()` calls, assert `score === n`

  - [ ]* 3.3 Write property test for streak reset (Property 2)
    - **Property 2: Streak resets on idle gap**
    - **Validates: Requirements 4.2, 4.3, 4.5**
    - Parameterize gap over values ≥ 800 ms and < 800 ms; assert streak === 1 for cold clicks and streak increments for hot clicks

  - [ ]* 3.4 Write property test for milestone firing once (Property 4)
    - **Property 4: Milestone fires at most once per threshold**
    - **Validates: Requirements 5.1, 5.5**
    - Drive `increment()` through and past each threshold; assert milestone is returned exactly once per threshold and never again

  - [ ]* 3.5 Write property test for localStorage round-trip (Property 8)
    - **Property 8: LocalStorage round-trip**
    - **Validates: Requirements 6.1, 6.2**
    - After `increment()`, create a fresh `ScoreTracker` from the same `localStorage` entry and assert restored score and unlockedTiers match

- [x] 4. Checkpoint — Core logic complete
  - Ensure all tests pass, ask the user if questions arise.

- [x] 5. Implement the `ReactionEngine` module
  - [x] 5.1 Define the reaction catalogue and implement weighted pick
    - Define `REACTION_CATALOGUE` — at least 5 `common`, 4 `rare`, and 3 `legendary` reactions, each with `expression`, `message`, `animation`, `sound|null`, `weight` (1–10), and `tier`
    - Implement `ReactionEngine.init()` — start `activePools` with the `common` tier only
    - Implement `ReactionEngine.expandPool(tier)` — add reactions of that tier to `activePools`
    - Implement `ReactionEngine.pick(currentScore)` using the weighted random pseudocode from the design
    - Only include reactions with positive integer `weight` in the pool
    - _Requirements: 3.1, 3.2, 3.3, 3.4, 5.2_

  - [ ]* 5.2 Write property test for weighted pick pool membership (Property 3)
    - **Property 3: Weighted pick stays within the active pool**
    - **Validates: Requirements 3.1, 3.2, 3.4**
    - For any unlocked tier configuration, assert `pick()` never returns a reaction from a locked tier and always returns a reaction with `weight > 0`

- [x] 6. Implement the `MessageBubble` module
  - Implement `MessageBubble.init()` — attach to `#bubble`
  - Implement `MessageBubble.show(text, durationMs = 2000)` — set bubble text via `textContent`, make bubble visible, then fade/hide after `durationMs` using CSS transition
  - Implement `MessageBubble.showMilestone(label)` — display milestone label as a toast (distinct style); set text via `textContent`
  - Implement `MessageBubble.hide()` — immediately hide the bubble
  - _Requirements: 2.3, 5.3, 10.1_

- [x] 7. Implement the `SoundPlayer` module
  - [x] 7.1 Implement programmatic tone generation
    - Define `SOUND_MAP` with configs for `boop`, `squeak`, `sparkle`, `tada`, `woof` (oscillator type, frequency, duration)
    - Implement `SoundPlayer.init()` — detect `AudioContext` support; emit one `console.warn` and set `supported = false` if unavailable; hide `#mute-btn` if unsupported
    - Implement `getAudioContext()` — lazily create and resume `AudioContext`
    - Implement `SoundPlayer.play(soundName)` following the formal spec in the design: guard on muted/unsupported, resume context, create oscillator and gain, schedule envelope, start and stop
    - Implement `SoundPlayer.setMuted(muted)` and wire it to the `#mute-btn` click handler
    - _Requirements: 7.1, 7.2, 7.3, 7.4, 7.5, 7.6_

  - [ ]* 7.2 Write property test for muted state (Property 6)
    - **Property 6: Muted state produces no audio output**
    - **Validates: Requirements 7.3**
    - After `setMuted(true)`, spy on `AudioContext.createOscillator`; assert it is never called across any sequence of `play()` calls

- [ ] 8. Implement the `handlePet` event handler and wire everything together
  - [x] 8.1 Implement `handlePet` following the design pseudocode
    - Write `handlePet(event)`: call `event.preventDefault()` on `touchend`; check `animating` guard (score-only path); call `ScoreTracker.increment()`; call `ReactionEngine.pick()`; call `DogRenderer.setExpression()`, `DogRenderer.playAnimation()`, `MessageBubble.show()`; conditionally call `SoundPlayer.play()`; update HUD via `renderScore` and `renderStreak` using `textContent`
    - Handle the milestone branch: call `ReactionEngine.expandPool()`, `MessageBubble.showMilestone()`, `SoundPlayer.play('tada')`
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 2.4, 2.5, 4.4, 5.2, 5.3, 5.4, 10.1_

  - [ ]* 8.2 Write property test for sound cue conditionality (Property 7)
    - **Property 7: Sound cue conditionality**
    - **Validates: Requirements 2.4, 2.5**
    - For reactions with `sound !== null`, assert `SoundPlayer.play` is called with the correct key; for reactions with `sound === null`, assert `SoundPlayer.play` is never called

  - [-] 8.3 Bootstrap initialisation in `DOMContentLoaded`
    - Write the `DOMContentLoaded` listener that calls `ScoreTracker.init()`, `ReactionEngine.init()`, `DogRenderer.init()`, `MessageBubble.init()`, `SoundPlayer.init()` in that exact order
    - Attach `click` and `touchend` listeners to `#dog`
    - _Requirements: 9.2, 9.3_

- [~] 9. Final checkpoint — Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Property tests use the **fast-check** library as specified in the design's Testing Strategy section
- Each task references specific requirements for traceability
- Checkpoints ensure incremental validation at logical boundaries
- The entire app lives in one `.html` file — all JS goes in a `<script>` block at the bottom of `<body>`
- Use `textContent` exclusively for all dynamic text to satisfy XSS safety (Requirement 10.1)

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1"] },
    { "id": 1, "tasks": ["2.1"] },
    { "id": 2, "tasks": ["2.2", "2.3"] },
    { "id": 3, "tasks": ["2.4", "3.1"] },
    { "id": 4, "tasks": ["3.2", "3.3", "3.4", "3.5"] },
    { "id": 5, "tasks": ["4"] },
    { "id": 6, "tasks": ["5.1"] },
    { "id": 7, "tasks": ["5.2", "6"] },
    { "id": 8, "tasks": ["7.1"] },
    { "id": 9, "tasks": ["7.2", "8.1"] },
    { "id": 10, "tasks": ["8.2", "8.3"] },
    { "id": 11, "tasks": ["9"] }
  ]
}
```
