# Beatana — UX flows

Screens and the flow between them. Product behaviour is in
`docs/product/requirements.md` (cited as §); app-wide UX policy is in
`docs/design/ux-guidelines.md` and is not restated here.

Design size 720×1280 (portrait), `Scale.FIT`. In landscape the canvas is centred with
the background colour on both sides. Every control is at least 88×88 design pixels.

## Screen inventory

| #   | Screen                   | Serves                             |
| --- | ------------------------ | ---------------------------------- |
| 1   | Boot                     | asset loading (§3.6)               |
| 2   | Title                    | modes (§3.7), credits entry (§3.6) |
| 3   | Weapon select            | main weapon (§3.5)                 |
| 4   | Battle                   | §3.1–§3.3                          |
| 5   | Pick (overlay on Battle) | §3.5                               |
| 6   | Interlude                | run (§3.4), build review (§3.5)    |
| 7   | Result                   | score (§3.7)                       |
| 8   | Credits                  | asset attribution (§3.6)           |

## Flow

```
 load ─▶ [Boot] ─▶ [Title] ──Credits──▶ [Credits] ──Back──▶ [Title]
                     │ Daily / Practice
                     ▼
              [Weapon select] ──Back──▶ [Title]
                     │ choose
                     ▼
        ┌────▶ [Battle n/5] ◀──────────────┐
        │        │      │ gauge reaches 10 │
        │        │      └──▶ [Pick] ───────┘ take 1 (or reroll, then take 1)
        │        │ all enemies down
        │        ├── n < 5 ──▶ [Interlude] ──Continue──┐
        │        │                                     │
        └────────┼─────────────────────────────────────┘
                 ├── n = 5 (boss down) ──▶ [Result: Cleared]
                 └── player at 0 health ─▶ [Result: Fallen]
   Battle / Interlude ──Pause──▶ Resume | Give up ──▶ [Title] (run discarded)
   [Result] ── Again (same seed, back to Weapon select) | Practice (new seed) | Title
```

Platform events follow the template: `gameplayStart` when Battle 1 becomes playable,
`gameplayStop` on entering Result or giving up; none between battles.

## 1. Boot

```
┌───────────────────────────┐
│                           │
│         BEATANA           │
│        ▓▓▓▓▓▓░░░░         │  progress 0–100 %
│                           │
└───────────────────────────┘
```

- Loading: the bar. Sprites are image files, so Boot is visible on a slow connection.
- Error: "Couldn't load. Tap to retry." — a tap, Enter or Space retries the loader.

## 2. Title

```
┌───────────────────────────┐
│         BEATANA           │
│   Daily · 2026-11-26      │  UTC day
│   Best today: 187         │  hidden when none or storage unavailable
│   ┌───────────────────┐   │
│   │    Play daily     │   │  primary, focused by default
│   └───────────────────┘   │
│   ┌───────────────────┐   │
│   │     Practice      │   │
│   └───────────────────┘   │
│          Credits          │
│ Each action is one beat.  │  one line of how to play
└───────────────────────────┘
```

- Keyboard: Up/Down moves focus across the three buttons; Enter/Space activates.

## 3. Weapon select

```
┌───────────────────────────┐
│ ← Back     Choose a weapon│
│ ┌───────────────────────┐ │
│ │ [sprite] Blade        │ │  focused by default
│ │ Stronger the closer   │ │
│ │ you stand: 1 / 2 / 3  │ │
│ └───────────────────────┘ │
│ ┌───────────────────────┐ │
│ │ [sprite] Spear        │ │
│ │ Pierces 2. Charge +3  │ │
│ └───────────────────────┘ │
│ ┌───────────────────────┐ │
│ │ [sprite] Short blade  │ │
│ │ Parry counter 6       │ │
│ └───────────────────────┘ │
│        [ Start run ]      │
└───────────────────────────┘
```

- Tapping a card selects it; Start run begins Battle 1. Keyboard: Up/Down selects, Enter
  starts, Esc goes back.
- In Daily, the weapon is the player's choice; the seed fixes line-ups and offers only.

## 4. Battle

```
┌─────────────────────────────────────┐
│ ⏸   Battle 2/5            Score 64  │  ⏸ = Esc
│                                     │
│        ⚔2        ↷1       …         │  intent above each enemy: attack /
│  ┌───┬───┬───┬───┬───┬───┬───┐      │  move / pause, with beats left
│  │   │   │ 2 │ ░ │   │ E │   │      │  7×3 cells of 96×96
│  ├───┼───┼───┼───┼───┼───┼───┤      │  digits = beats until an attack
│  │ 1 │ P │ 1 │ ░ │ E │   │   │      │  lands on that cell
│  ├───┼───┼───┼───┼───┼───┼───┤      │  ░ = neutral column
│  │   │   │   │ ░ │   │   │ E │      │  enemy health bar under each E
│  └───┴───┴───┴───┴───┴───┴───┘      │
│  Health ██████░░░░ 6/10             │
│  Spirit ███████░░░ 7/10   Charge ●●○│
│  [Blade 2] [Stars 1] [ — ] [ — ]    │  main weapon + 3 sub-weapon slots
│  [Lunge 1] [ — ] [ — ]              │  3 art slots, each with its level
│                                     │
│      ┌───┐            ┌─────┬─────┐ │
│  ┌───┼ ▲ ┼───┐        │Strike│Charge│
│  │ ◀ │   │ ▶ │        ├─────┼─────┤ │  each action button 120×120
│  └───┼ ▼ ┼───┘        │Parry │ Wait│ │  d-pad arms 96×96
│      └───┘            └─────┴─────┘ │
└─────────────────────────────────────┘
```

| Action | Touch  | Keyboard   |
| ------ | ------ | ---------- |
| Move   | d-pad  | arrow keys |
| Strike | Strike | Z          |
| Charge | Charge | X          |
| Parry  | Parry  | C          |
| Wait   | Wait   | Space      |
| Pause  | ⏸      | Esc        |

- One input is one beat: the player's action resolves, sub-weapons due this beat fire,
  then every enemy advances, in that order, and the next input is accepted when the
  beat's animation ends.
- A cell with an attack landing on the next beat is drawn in the danger colour with "1";
  standing on it, Parry is highlighted.
- A sub-weapon slot shows the beats until it next fires.
- A move that would leave the territory does nothing and spends no beat (§3.1).
- First battle of the first run on a device: three one-line hints appear in turn above
  the controls ("Numbers show when an attack lands", "Stand closer to hit harder",
  "Parry on 1 to counter"); each disappears after the action it describes is used.
- Pause panel: Resume (focused), Give up. Give up asks "Give up this run?" — Yes / No.
- Enemies all down → 300 ms pause → Interlude, or Result after battle 5. Player at 0
  health → 300 ms pause → Result.

## 5. Pick (overlay)

```
┌─────────────────────────────────────┐
│   (battle dimmed behind)            │
│            Choose one               │
│ ┌─────────────────────────────────┐ │
│ │ NEW  Throwing stars             │ │  1
│ │ Every 3 beats, 1 damage in row  │ │
│ └─────────────────────────────────┘ │
│ ┌─────────────────────────────────┐ │
│ │ Lv 2 → 3  Lunge                 │ │  2
│ │ Front-column strike +1          │ │
│ └─────────────────────────────────┘ │
│ ┌─────────────────────────────────┐ │
│ │ FUSION  Draw cut                │ │  3
│ │ Blade 3 + Lunge                 │ │
│ └─────────────────────────────────┘ │
│        [ Reroll · 2 left ]          │
└─────────────────────────────────────┘
```

- Opens between beats when the gauge reaches 10; no beat passes while it is open.
- Tapping a card takes it. Keyboard: 1/2/3 or Up/Down + Enter; R rerolls.
- Each card is tagged NEW, Lv a → b, or FUSION, and a fusion names its two parts.
- Reroll at 0 left is shown disabled. Esc does nothing here: a pick cannot be skipped.
- Nothing left to offer: a single card "+1 health" (§3.5).
- If the gauge passes 10 twice in one beat, the picks open one after the other.

## 6. Interlude

```
┌───────────────────────────┐
│ ⏸     Battle 2 cleared    │
│   Health 6/10   Score 64  │
│   Next: battle 3/5        │
│   [E] Archer  [E] Charger │  line-up of the next battle
│                           │
│   Your build              │
│   Blade 2 · Stars 1       │
│   Lunge 1                 │
│ ┌───────────────────────┐ │
│ │       Continue        │ │  focused; Enter/Space
│ └───────────────────────┘ │
└───────────────────────────┘
```

- Before battle 5 the line-up reads "Boss".
- Tapping a build item shows its description in a one-line strip under the list.

## 7. Result

```
┌───────────────────────────┐
│     Cleared! / Fallen     │
│     Score 187             │
│     Spirit 127 + Health 60│  health bonus on a clear only
│     New best!             │  Daily only, when the stored best rose
│     Reached battle 4/5    │  on Fallen only
│   Blade 3 · Stars 3 · …   │  final build
│ ┌───────────────────────┐ │
│ │     Copy result       │ │  Daily only
│ └───────────────────────┘ │
│   Again   Practice  Title │
└───────────────────────────┘
```

- Share text (Daily): `Beatana 2026-11-26 — 187 — Blade / Stars / Lunge`. Clipboard
  failure: the text is shown in a selectable box.
- Again returns to Weapon select with the same seed; Practice with a new seed.
- Practice shows no best line, no "New best!" and no Copy result.
- Focus order: Copy result (Daily only), Again, Practice, Title.

## 8. Credits

```
┌───────────────────────────┐
│ ← Back         Credits    │
│ Sprites                   │
│  <asset> — <author>       │
│  <licence> · <source>     │  one entry per asset pack; scrolls
│ Code                      │
│  Beatana, <year>          │
└───────────────────────────┘
```

- Up/Down or drag scrolls; Esc or Back returns to Title.
- The list is generated from the repository's credits file, so the two cannot differ.
- No assets needing credit: the Sprites heading is omitted.
