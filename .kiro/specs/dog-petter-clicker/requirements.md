# Requirements Document

## Introduction

Dog Petter Clicker is a single-page clicker web application delivered as one self-contained HTML file. The user pets a minimalist dog character by clicking or tapping it, triggering random, delightful reactions (expressions, speech bubbles, animations, and sounds). A happiness score grows with every pet, a streak counter rewards fast clicking, and milestone thresholds unlock rarer reactions. The app requires no build step and no backend — only a Tailwind CSS CDN link and browser-native APIs.

## Glossary

- **App**: The single-page Dog Petter Clicker HTML application.
- **Dog**: The interactive SVG-based minimalist dog character rendered on screen.
- **DogRenderer**: The component responsible for the Dog's visual representation, expressions, and animations.
- **ReactionEngine**: The component that selects a weighted-random Reaction from the active reaction pool.
- **ScoreTracker**: The component that maintains game state — score, streak, and milestone detection.
- **MessageBubble**: The speech-bubble UI element that displays reaction text above the Dog.
- **SoundPlayer**: The component that generates audio cues via the Web Audio API.
- **Reaction**: A data record combining an expression, a speech-bubble message, an animation name, an optional sound cue, a weight, and a tier.
- **Score**: The cumulative count of pets (clicks/taps) the user has given the Dog.
- **Streak**: The count of consecutive clicks where each gap between clicks is under `STREAK_WINDOW_MS` (800 ms).
- **Milestone**: A score threshold that triggers an unlock event and displays a toast notification.
- **Expression**: One of the named Dog face states: `happy`, `love`, `surprised`, `sleepy`, `dizzy`, `tongue`, `wink`.
- **AnimationName**: One of the named CSS animations: `bounce`, `shake`, `spin`, `wiggle`, `zoom`, `float`.
- **Tier**: A rarity level for Reactions — `common`, `rare`, or `legendary`.
- **GameState**: The persisted object stored in `localStorage` containing score, streak, lastClickTime, and unlockedTiers.
- **HUD**: The on-screen heads-up display showing the current Score and Streak.

---

## Requirements

### Requirement 1: Pet Interaction

**User Story:** As a user, I want to click or tap the Dog to trigger a reaction, so that I can enjoy a satisfying and responsive feedback loop.

#### Acceptance Criteria

1. WHEN a user clicks or taps the Dog element, THE App SHALL invoke the `handlePet` event handler exactly once per physical interaction.
2. WHEN `handlePet` is invoked, THE ScoreTracker SHALL increment the Score by exactly 1.
3. WHEN `handlePet` is invoked on a touch device, THE App SHALL call `event.preventDefault()` on the `touchend` event to suppress the synthetic click and prevent double-firing.
4. WHEN `handlePet` is invoked and the Dog element does not have the `animating` guard class, THE App SHALL apply a Reaction to the Dog and update the HUD.
5. WHEN `handlePet` is invoked and the Dog element has the `animating` guard class, THE ScoreTracker SHALL still increment the Score but THE App SHALL skip the visual and audio Reaction for that click.

---

### Requirement 2: Reaction Display

**User Story:** As a user, I want the Dog to show a randomly chosen expression, animation, and speech bubble on every pet, so that the interaction feels varied and entertaining.

#### Acceptance Criteria

1. WHEN a Reaction is triggered, THE DogRenderer SHALL set the Dog's face to the Expression specified in the Reaction.
2. WHEN a Reaction is triggered, THE DogRenderer SHALL play the AnimationName specified in the Reaction for the configured duration, then automatically remove the animation class.
3. WHEN a Reaction is triggered, THE MessageBubble SHALL display the Reaction's message text for a configured duration, then fade it out.
4. WHEN a Reaction is triggered and the Reaction's sound field is not null, THE SoundPlayer SHALL play the specified sound cue.
5. WHEN a Reaction is triggered and the Reaction's sound field is null, THE SoundPlayer SHALL not produce any audio output.
6. WHEN `DogRenderer.playAnimation()` is called, THE DogRenderer SHALL remove all existing animation classes before applying the new one, so that animation classes do not stack.

---

### Requirement 3: Weighted Reaction Selection

**User Story:** As a user, I want reactions to be selected randomly with rarer reactions being less frequent, so that the app feels progressively more surprising as I unlock new content.

#### Acceptance Criteria

1. WHEN `ReactionEngine.pick()` is called, THE ReactionEngine SHALL return exactly one Reaction from the currently active (unlocked) pool.
2. WHEN `ReactionEngine.pick()` is called, THE ReactionEngine SHALL never return a Reaction whose Tier has not been unlocked.
3. WHEN `ReactionEngine.pick()` selects a Reaction, THE ReactionEngine SHALL use weighted random selection proportional to each Reaction's `weight` value.
4. THE ReactionEngine SHALL only include Reactions with a positive integer `weight` value in the selection pool.

---

### Requirement 4: Score and Streak Tracking

**User Story:** As a user, I want to see my happiness score grow and a streak counter reward my fast clicking, so that I feel encouraged to keep petting the Dog.

#### Acceptance Criteria

1. THE ScoreTracker SHALL maintain a Score that starts at 0 and increases by exactly 1 on each `increment()` call.
2. WHEN `ScoreTracker.increment()` is called and the time elapsed since the last click is less than 800 ms, THE ScoreTracker SHALL increment the Streak by 1.
3. WHEN `ScoreTracker.increment()` is called and the time elapsed since the last click is 800 ms or more, THE ScoreTracker SHALL reset the Streak to 1.
4. WHEN the Score or Streak changes, THE App SHALL update the HUD to reflect the current values.
5. THE ScoreTracker SHALL ensure the Streak is always a positive integer of at least 1 after any `increment()` call.

---

### Requirement 5: Milestone Unlocks

**User Story:** As a user, I want to reach score milestones that unlock new, rarer reactions and show me a celebration toast, so that I have a sense of progression and reward.

#### Acceptance Criteria

1. WHEN `ScoreTracker.increment()` causes the Score to equal a Milestone threshold for the first time, THE ScoreTracker SHALL return the triggered Milestone in the `PetResult`.
2. WHEN a Milestone is unlocked, THE App SHALL call `ReactionEngine.expandPool()` with the Milestone's Tier exactly once.
3. WHEN a Milestone is unlocked, THE MessageBubble SHALL display the Milestone's label as a toast notification.
4. WHEN a Milestone is unlocked, THE SoundPlayer SHALL play the `tada` sound cue.
5. WHEN `ScoreTracker.increment()` is called after the Score has already passed a Milestone threshold, THE ScoreTracker SHALL not return that Milestone again.
6. THE App SHALL define the following Milestones in ascending threshold order: Score 10 → "Good boy energy!" unlocks `rare` Tier; Score 50 → "Absolute legend!" unlocks `legendary` Tier; Score 100 → "Professional petter!" cosmetic badge only.

---

### Requirement 6: State Persistence

**User Story:** As a user, I want my score to be saved across page reloads, so that I do not lose my progress when I revisit the app.

#### Acceptance Criteria

1. WHEN `ScoreTracker.increment()` is called, THE ScoreTracker SHALL write the current GameState to `localStorage` immediately after updating state.
2. WHEN the App initialises, THE ScoreTracker SHALL read the GameState from `localStorage` and restore the Score and unlockedTiers if a valid saved state exists.
3. IF `localStorage` is unavailable, THEN THE ScoreTracker SHALL operate using in-memory state only and SHALL NOT display an error to the user.
4. IF `localStorage` is unavailable, THEN THE App SHALL continue to function normally for the current session, with the Score resetting on page reload.

---

### Requirement 7: Audio Playback

**User Story:** As a user, I want cheerful sound effects on each pet and a mute toggle, so that I can enjoy the audio feedback or silence it when needed.

#### Acceptance Criteria

1. THE SoundPlayer SHALL generate all audio programmatically using the Web Audio API, without loading any external audio files.
2. WHEN `SoundPlayer.play()` is called, THE SoundPlayer SHALL resume a suspended `AudioContext` before attempting to produce audio, to satisfy browser autoplay policy.
3. WHEN `SoundPlayer.setMuted(true)` is called, THE SoundPlayer SHALL not create or start any audio nodes for any subsequent `play()` calls.
4. WHEN `SoundPlayer.setMuted(false)` is called, THE SoundPlayer SHALL resume producing audio output for subsequent `play()` calls.
5. IF the browser does not support the Web Audio API, THEN THE SoundPlayer SHALL silently no-op on all `play()` calls and SHALL emit a single `console.warn` at initialisation.
6. IF the browser does not support the Web Audio API, THEN THE App SHALL hide the mute toggle control.

---

### Requirement 8: Dog Visual Representation

**User Story:** As a user, I want to see a cute, expressive dog character that responds visually to my pets, so that the interaction is charming and satisfying.

#### Acceptance Criteria

1. THE DogRenderer SHALL render the Dog as an SVG-based element that scales without quality loss on all screen densities.
2. THE DogRenderer SHALL support the following named Expressions: `happy`, `love`, `surprised`, `sleepy`, `dizzy`, `tongue`, `wink`.
3. THE DogRenderer SHALL support the following named AnimationNames: `bounce`, `shake`, `spin`, `wiggle`, `zoom`, `float`.
4. WHEN `DogRenderer.playAnimation()` is called, THE DogRenderer SHALL add the `animating` guard class to the Dog element for the duration of the animation, then remove it automatically.
5. WHEN `DogRenderer.playAnimation()` completes, THE DogRenderer SHALL remove all animation-related CSS classes from the Dog element, leaving it in a clean idle state.
6. ALL CSS animations applied by THE DogRenderer SHALL use only `transform` and `opacity` properties, so that no layout reflow is triggered.

---

### Requirement 9: Single-File Delivery and Initialisation

**User Story:** As a user, I want to open the app by simply opening one HTML file in a browser, so that there is zero setup or installation friction.

#### Acceptance Criteria

1. THE App SHALL be delivered as a single self-contained HTML file with no local dependencies beyond a Tailwind CSS CDN link.
2. WHEN the page's `DOMContentLoaded` event fires, THE App SHALL initialise ScoreTracker, ReactionEngine, DogRenderer, MessageBubble, and SoundPlayer in that order.
3. WHEN the App initialises, THE App SHALL attach `click` and `touchend` event listeners to the Dog element.
4. THE App SHALL make no outbound network requests other than loading the Tailwind CSS CDN stylesheet.
5. THE App SHALL not use any npm packages, build tooling, or backend services.

---

### Requirement 10: Content Safety and Security

**User Story:** As a user, I want the app to be safe and reliable, so that I can use it without risk of data leakage or unexpected behaviour.

#### Acceptance Criteria

1. WHEN THE App renders any dynamic text in the DOM, THE App SHALL set text using `textContent`, not `innerHTML`, to prevent cross-site scripting.
2. THE App SHALL not make any network requests that transmit user data.
3. THE App SHALL store only numeric game state values in `localStorage` and SHALL NOT store any personally identifiable information.
