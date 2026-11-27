# Design lock — Beatana

Binding values for Beatana. `apps/web/src/theme/tokens.ts` holds the same values as
code; `docs/design/design-direction.md` is the research record this was written from.

Not applied yet: `apps/web/src/theme/tokens.ts` and `docs/design/contrast-pairs.json`
still hold the sample's values. The design-token issue applies this lock to both in one
change and removes this line.

## Direction

- **Primary reference:** design-atlas `pulse-track` — a near-black ground, one
  vermilion, and one large number the eye goes to. Chosen because the announced-beat
  digit is this game's "one number", and black with vermilion reads as ink and seal.
- **Preserved:** the near-black ground; vermilion as the only action colour; large
  tabular digits; dense layout; no gradients; no evaluation chrome (badges, confetti).
- **Changed, on purpose:** every shape is square (radius 0, 4 px steps) instead of the
  system's capsules, so chrome sits with pixel sprites; the type is the pixel font
  DotGothic16 instead of Barlow Condensed, because the UI is Japanese.
- **Borrowed:** the system's `warning` amber as this game's danger colour, so danger is
  never the action colour; its `success` green for the player's health.
- **Rejected:** rounded or capsule controls; a second accent; colour-coded weapon kinds;
  smooth scaling of sprites; drop shadows.
- **Media:** pixel sprites drawn at integer scale with nearest-neighbour sampling.
  Chrome is flat fills and 4 px outlines.
- **Memorable move:** amber digits counting down on the cells, and a single vermilion
  thing on screen — whatever the player can press or gain right now.
- **One theme:** dark only (template rule).

## Role rules

- `bg` is the page, the canvas and the neutral column. `surface` is the board cells and
  HUD rows. `surfaceRaised` is the Pick cards and the pause panel. `control` fills
  secondary buttons, the d-pad and the action buttons.
- `controlBorder` outlines every cell, slot, card and secondary control (4 px).
- `text` for labels and numbers; `textMuted` only for secondary lines (dates, units,
  hints, empty slots).
- `accent` (vermilion) only for: the primary button fill, the keyboard focus ring, the
  spirit gauge fill and its "+n", the Parry button highlight on a "1" cell, and "New
  best". Never for danger, health or sprites' outlines.
- `accentSoft` with `onAccentSoft` only for the NEW / Lv / FUSION tag on a Pick card.
- `danger` (amber) only for announced cells — outline, hatching, digit — and enemy
  intent marks. Nothing the player can press is amber.
- `health` only for the player's health bar; `enemyBar` only for enemy health bars.
- The focus ring is `focusRing` of `accent`, drawn `focusOffset` outside the control.
- Sprites keep their own palette; no token recolours a sprite. The hit flash is a white
  tint for `flashMs`.

## Tokens

| Token                  | Value                                                                                                                                                                                                                                  |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `bg`                   | `#0C0C0E`                                                                                                                                                                                                                              |
| `surface`              | `#161619`                                                                                                                                                                                                                              |
| `surfaceRaised`        | `#1F1F23`                                                                                                                                                                                                                              |
| `control`              | `#26262B`                                                                                                                                                                                                                              |
| `controlBorder`        | `#6E6E77`                                                                                                                                                                                                                              |
| `text`                 | `#F2F1EF`                                                                                                                                                                                                                              |
| `textMuted`            | `#9C9BA3`                                                                                                                                                                                                                              |
| `accent`               | `#EE3F2C`                                                                                                                                                                                                                              |
| `onAccent`             | `#140504`                                                                                                                                                                                                                              |
| `accentSoft`           | `#3A1710`                                                                                                                                                                                                                              |
| `onAccentSoft`         | `#FFB4A3`                                                                                                                                                                                                                              |
| `danger`               | `#E8A93A`                                                                                                                                                                                                                              |
| `health`               | `#47C27F`                                                                                                                                                                                                                              |
| `enemyBar`             | `#E4E3E0`                                                                                                                                                                                                                              |
| Font                   | `"DotGothic16", "Hiragino Sans", "Noto Sans JP", sans-serif`; DotGothic16 is bundled (SIL Open Font License 1.1, https://fonts.google.com/specimen/DotGothic16)                                                                        |
| Type scale (design px) | title 64, heading 48, body 32, caption 24, cell digit 48                                                                                                                                                                               |
| Radius                 | 0 everywhere                                                                                                                                                                                                                           |
| Outline                | 4 design px                                                                                                                                                                                                                            |
| Spacing unit           | 8 design px                                                                                                                                                                                                                            |
| Motion                 | player action 120 ms ease-out; enemy and sub-weapon actions 120 ms starting 80 ms later; hit flash 60 ms; shake 4 px for 120 ms; damage number 400 ms; scene fade 150 ms; Pick input delay 250 ms; pause before the next screen 300 ms |

Layout tokens (`layout`, design px unless noted):

| Token           | Value                                          |
| --------------- | ---------------------------------------------- |
| `designWidth`   | 720                                            |
| `designHeight`  | 1280                                           |
| `tapMin`        | 88                                             |
| `focusRing`     | 4                                              |
| `focusOffset`   | 4                                              |
| `cellSize`      | 96 (7 columns = 672, 24 side margins)          |
| `spriteScale`   | integer only; 6 for 16 px sprites, 3 for 32 px |
| `dpadArm`       | 96                                             |
| `actionButton`  | 120                                            |
| `resultPauseMs` | 300                                            |

`focusRing` is 4, not the template's 2, to stay on the 4 px outline step.

## Contrast

WCAG 2.2 relative-luminance contrast (SC 1.4.3 and 1.4.11,
https://www.w3.org/TR/WCAG22/), measured 2026-11-27 from the token values above.

```
name                                               fg       bg       kind  ratio  required  result
-------------------------------------------------  -------  -------  ----  -----  --------  ------
text / bg                                          #F2F1EF  #0C0C0E  text  17.31  4.50      PASS
text / surface                                     #F2F1EF  #161619  text  16.00  4.50      PASS
text / surfaceRaised                               #F2F1EF  #1F1F23  text  14.55  4.50      PASS
text / control (button label)                      #F2F1EF  #26262B  text  13.34  4.50      PASS
textMuted / bg                                     #9C9BA3  #0C0C0E  text  7.10   4.50      PASS
textMuted / surface                                #9C9BA3  #161619  text  6.56   4.50      PASS
textMuted / surfaceRaised                          #9C9BA3  #1F1F23  text  5.97   4.50      PASS
accent / bg (primary button)                       #EE3F2C  #0C0C0E  ui    5.01   3.00      PASS
accent / surface (focus ring, spirit gauge)        #EE3F2C  #161619  ui    4.63   3.00      PASS
accent / surfaceRaised (focus ring on Pick)        #EE3F2C  #1F1F23  ui    4.21   3.00      PASS
onAccent / accent (button label)                   #140504  #EE3F2C  text  5.11   4.50      PASS
onAccentSoft / accentSoft (tag label)              #FFB4A3  #3A1710  text  9.40   4.50      PASS
danger / surface (announcement digit)              #E8A93A  #161619  text  8.74   4.50      PASS
danger / bg (neutral column)                       #E8A93A  #0C0C0E  text  9.46   4.50      PASS
health / surface (health bar)                      #47C27F  #161619  ui    7.99   3.00      PASS
enemyBar / surface (enemy health bar)              #E4E3E0  #161619  ui    14.07  3.00      PASS
controlBorder / bg (control outline)               #6E6E77  #0C0C0E  ui    3.87   3.00      PASS
controlBorder / surface (cell grid, slot outline)  #6E6E77  #161619  ui    3.58   3.00      PASS
controlBorder / surfaceRaised (card outline)       #6E6E77  #1F1F23  ui    3.25   3.00      PASS
```

`danger` and `accent` differ in hue and in lightness (8.74 against 4.63 on `surface`),
and danger always carries a digit or hatching, so the two are told apart without colour.
