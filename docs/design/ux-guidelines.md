# Beatana — UX guidelines

|           |                                                                                                                                           |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| Product   | Beatana                                                                                                                                   |
| Platforms | Web: desktop and mobile browsers, Phaser 4 canvas, 720×1280 portrait                                                                      |
| App type  | Single-player turn-stepped game                                                                                                           |
| Related   | Requirements: `docs/product/requirements.md` · UX flows: `docs/product/ux-flows.md` · Design direction: `docs/design/design-direction.md` |

Rules the template already fixes and this game keeps unchanged are marked (template).

## Principles

- **The board never lies** — everything that will hurt the player is on screen before it
  lands. Decides: an enemy effect with no announcement is a bug, not a difficulty knob.
  Gives up: surprise attacks.
- **Speed belongs to the player** — nothing advances without an input, and the game
  never makes a fast player wait. Decides: animations are short and one input is
  buffered. Gives up: long, showy attack animations.
- **Risk is felt** — strong feedback is reserved for risky actions. Decides: only a
  full-charge hit, a parry counter and taking damage shake the screen. Gives up: impact
  on every ordinary hit.

## First minute

- Battle is reachable in two taps from Title: a mode, then Start run with Blade
  preselected.
- No blocking tutorial or modal (template). The first battle's hints sit above the
  controls and never stop input.

## Input

- Every action works with pointer and keyboard; tap targets ≥ 88×88 design pixels; no
  drag (except scrolling Credits, which also scrolls with Up/Down), long-press,
  double-tap or hover-only affordance (template).
- One input is one beat. While a beat animates, at most one further input is buffered
  and played when the animation ends; later presses in the same animation replace the
  buffered one. Why: mashing stays responsive without queuing moves the player no longer
  wants.
- The buffer is cleared when a Pick opens, a battle ends or the pause panel opens, so a
  stray press never takes an upgrade or an action in the next battle.
- Holding a key or button does not repeat. Why: every beat is a decision.
- A Pick accepts input 250 ms after it opens. Why: it can open in the middle of fast
  play.

## Feedback

| Event                                                    | Feedback                                                                                               |
| -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| Player action                                            | 120 ms, ease-out                                                                                       |
| Enemy and sub-weapon actions                             | 120 ms, starting 80 ms after the player's; a beat lasts about 200 ms                                   |
| Any hit                                                  | Target flashes white for 60 ms; a damage number rises 24 px and fades in 400 ms without blocking input |
| Full-charge hit, parry counter, player hit               | Camera shake 4 px for 120 ms, on top of the flash                                                      |
| Spirit gained                                            | "+n" beside the gauge; the gauge fills in the same beat                                                |
| Invalid input (off the territory, during a result pause) | Nothing visible, no beat spent (template)                                                              |
| Scene change                                             | 150 ms fade, never longer than 300 ms (template)                                                       |
| Battle end                                               | 300 ms pause on the final board, then the fade (template)                                              |

- Health, score, gauge and sub-weapon counters update in the same beat as their cause.

## Announced attacks

| Beats until landing | Cell shows                                                                                                    |
| ------------------- | ------------------------------------------------------------------------------------------------------------- |
| 2 or more           | Outline in the danger colour and the digit                                                                    |
| 1                   | Diagonal hatching over the cell, the digit "1", and the Parry button highlighted when the player stands there |

- Urgency is carried by the digit and the pattern; colour only reinforces it.
- An enemy's intent mark (attack, move, pause) is an icon with a digit, never colour
  alone.

## Motion

- With `prefers-reduced-motion: reduce`, the camera shake and the white flash are
  omitted and scene fades become cuts; damage numbers, digits and patterns stay. Why:
  they carry information, the shake and flash carry only impact.
- No element flashes more than 3 times per second.
- There is no speed setting. Why: one tuned speed with a buffer was chosen over a
  setting that needs storage and a screen.

## State and errors

- Storage unavailable: the best line is hidden and play continues; no error (template).
- Asset load failure: Boot's retry prompt; no other screen can hit it.
- Leaving and returning to the tab keeps the run exactly where it was (template).
  Reloading or closing the tab ends the run (requirements §3.7).
- Giving up a run is the only destructive action and the only confirmation.

## Audio

- No sound in the MVP. When sound is added, the template's mute and `AudioContext` rules
  apply.

## Accessibility

- Text contrast ≥ 4.5:1; board cells, intent marks, gauges and controls ≥ 3:1. Measured
  values are in the design direction.
- Keyboard focus is always visible: a 2 px ring, 4 px outside the control (template).
- Sprites are identified by silhouette and a name label on Weapon select, Pick and
  Interlude, never by colour alone.

## Copy

- UI text is Japanese only; no language switch. Why: the players are the owner and
  friends. Code, comments and documents stay English.
- The labels in the UX flows give each string's meaning in English; the shipped strings
  are their Japanese equivalents, kept in one strings module so a second language can be
  added later without touching scenes.
- Text is drawn with the device's system Japanese font, not a pixel font. Open: a
  redistributable Japanese pixel font — settled with the design direction.
- Short labels, no sentence-ending punctuation on buttons; numbers as digits; dates as
  `YYYY-MM-DD` (UTC).
- An item description fits two lines of the Pick card at the body size; longer effects
  are split across levels rather than abbreviated.

## Open items

- Japanese pixel font — the design direction decides whether one with a compatible
  licence is used.

## Decision log

| Date       | Decided                                                                                           | Rejected                                      | Why                                                                   |
| ---------- | ------------------------------------------------------------------------------------------------- | --------------------------------------------- | --------------------------------------------------------------------- |
| 2026-11-27 | Japanese-only UI                                                                                  | English only; both with a switch              | Players read Japanese; two languages double every string from day one |
| 2026-11-27 | ~200 ms beat with a one-input buffer                                                              | ~400 ms sequential beat; a speed setting      | Keeps fast play fast; no settings storage or screen                   |
| 2026-11-27 | Flash and number on every hit, shake on risky hits and damage taken; omitted under reduced motion | Numbers only; shake and hit-stop on every hit | Rewards risk without constant shaking once sub-weapons fire           |
| 2026-11-27 | Danger shown by digit plus pattern                                                                | Digit and colour depth only                   | Urgency readable without colour                                       |
| 2026-11-27 | Touch: d-pad plus four action buttons                                                             | Tap a cell to move; swipe                     | Thumbs stay off the board; the template rules out swipe-only input    |
| 2026-11-27 | Enemy intent marks above enemies                                                                  | Cell announcements only                       | Lets the player plan a charge or parry                                |
