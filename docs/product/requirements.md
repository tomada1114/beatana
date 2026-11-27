# Beatana — Requirements

- **Status:** Signed off 2026-11-26 (values in §3 are starting points, tuned by the
  first playable build)
- **Platform and stack:** Web (desktop and mobile browsers). Cut from
  `tomada1114/web-game-template`, which fixes TypeScript, Node 24, pnpm 11, Vite 8,
  Phaser 4, a deterministic integer-only `packages/core` (seed + moves replay), a
  720×1280 portrait canvas, pointer and keyboard parity, and the Boot → Title → Play →
  Result scene flow.

## 1. Overview

Beatana is a side-view grid battle game for one player. Time moves only when the player
acts: every action is one beat, and every enemy advances one beat with it. Enemy attacks
are announced on the cells they will hit, so a fight is a string of small bets — stand
closer, charge longer, or stay in the announced cell and parry — where more risk pays
more. A play is a short run of battles ending in a boss, with one upgrade chosen after
each win, so each run grows into a different fighting style.

- **Target user:** the owner and friends; a non-commercial personal project.
- **Core interaction:** read the announced attacks, then spend one beat on move, strike,
  charge, parry or wait.
- **Originality rule:** every name, term, sprite, sound and screen layout is the game's
  own. No vocabulary, characters or look from an existing game.

## 2. Scope

### MVP

- **Beat-step battle** — the core interaction (§3.1).
- **Three risk axes: distance, charge, parry** — what replaces a chip or deck system
  (§3.2).
- **Enemies with announced attacks** — 4 kinds and 1 boss (§3.3).
- **Run of 5 battles** — 4 battles then the boss, health carried over (§3.4).
- **Build** — a main weapon chosen at the start, automatic sub-weapons, passive arts,
  levels and fusions, earned by taking risks (§3.5).
- **Pixel-art presentation** — provisional ninja/samurai theme from free assets (§3.6).

### Later

- **Beat timer** — an `apps/web` clock that plays "wait" when the player is slow, for a
  real-time feel; waits until the untimed game is fun. Core does not change.
- **More content** — 8-battle runs, a mid-boss, 20+ upgrades; waits until the 12-upgrade
  set shows which builds are fun.
- **Audio** — the template defers its sound helper to the first game that ships sound.

### Non-goals

- A chip, card or deck system — the owner ruled it out.
- Random hit, miss or damage rolls, and attacks that land unannounced — enemies choose
  at random, but what they chose is always shown before it lands.
- Real-time (clock-driven) rules in `packages/core`.
- Multiplayer, accounts, a server, monetization.
- A story mode or overworld; the game is battles and the choices between them.

## 3. Features

### 3.1 Beat-step battle

| Item         | Specification                                                                                   |
| ------------ | ----------------------------------------------------------------------------------------------- |
| Board        | 7 columns × 3 rows: player territory columns 1–3, neutral column 4, enemy territory columns 5–7 |
| Movement     | One cell up, down, forward or back per beat, inside the own territory only                      |
| Beat         | One player action (move, strike, charge, parry, wait), then every enemy advances one beat       |
| Announcement | A cell an enemy attack will hit shows the number of beats until it lands                        |
| End          | Win when every enemy is at 0 health; lose when the player is                                    |

- An action that cannot be done (moving off the territory) does nothing and spends no
  beat.

### 3.2 Risk axes

| Axis     | Reward                                                                 | Risk                                                         |
| -------- | ---------------------------------------------------------------------- | ------------------------------------------------------------ |
| Distance | Strike damage rises with the column: back < middle < front             | Front cells are inside more enemy attacks                    |
| Charge   | Each charged beat (up to 2) raises damage and reach                    | A hit while charging clears the charge                       |
| Parry    | A parry on the beat an attack lands negates it and counterattacks hard | A parry on any other beat does nothing and the beat is spent |

Proposed values, to be confirmed by the owner and then by a first playable build:

| Item          | Value                                                                                                                                 |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| Player health | 10                                                                                                                                    |
| Strike        | Hits the first enemy in the player's row; damage 1 back, 2 middle, 3 front                                                            |
| Charge        | +2 damage per charged beat, at most 2 beats; a 2-beat charge pierces every enemy in the row; moving keeps the charge, a hit clears it |
| Parry         | Negates the attack landing this beat and deals 4 to the attacker                                                                      |

### 3.3 Enemies

4 regular kinds and 1 boss. Enemy behaviour is random within rules the player can learn:

- Each kind has 2–3 moves (an attack shape, a step to another cell, a pause). When its
  cycle comes round it draws one from a weighted table, using the run's seed.
- A drawn attack is announced on its cells at least 1 beat before it lands; stronger
  attacks are announced 2–3 beats ahead. Nothing lands unannounced.
- The same seed and the same player actions give the same enemy choices, so a daily run
  is the same for everyone who plays it the same way.

Proposed set (names provisional; the Attack column is each kind's main move):

| Enemy          | Health | Attack                                                                                             |
| -------------- | ------ | -------------------------------------------------------------------------------------------------- |
| Spearman       | 6      | Every 3 beats, thrusts along its own row                                                           |
| Archer         | 6      | Every 2 beats, marks the player's current cell; the mark does not follow                           |
| Powder thrower | 8      | Every 4 beats, a plus-shaped blast of 5 cells                                                      |
| Charger        | 10     | When the player shares its row, rushes that row 2 beats later                                      |
| Boss (warlord) | 30     | Cycles the row thrust, the marked cell and the blast; at half health every cycle is 1 beat shorter |

### 3.4 Run

| Item   | Specification                                                 |
| ------ | ------------------------------------------------------------- |
| Length | 5 battles: 4 regular, then the boss                           |
| Health | Carried from battle to battle                                 |
| Order  | The enemy line-up of each battle is drawn from the run's seed |
| Loss   | The run ends; the player starts a new run                     |

### 3.5 Upgrades

The build is what makes one run differ from the next.

| Item         | Specification                                                                                                                                               |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Spirit gauge | Risky actions fill it: middle-column hit +1, front-column hit +2, full-charge hit +3, successful parry +5. At 10 it empties and a pick opens, between beats |
| Pick         | 3 options drawn from the seed; the player takes 1. Options are a new item or +1 level of an owned one                                                       |
| Reroll       | 2 per run; redraws the 3 options                                                                                                                            |
| Slots        | 1 main weapon, up to 3 sub-weapons, up to 3 arts                                                                                                            |
| Levels       | Every item has levels 1–3                                                                                                                                   |
| Fusion       | A level-3 weapon plus its paired art, both owned, becomes a fused weapon at the next pick                                                                   |

- Safe play fills the gauge slowly, so a careful player ends with a thin build and a
  bold one with a strong build.
- A pick with nothing left to offer gives 1 health instead.

Main weapon — chosen at the start of the run; shapes the manual strike:

| Weapon      | Strike                                                                     |
| ----------- | -------------------------------------------------------------------------- |
| Blade       | First enemy in the row; 1 / 2 / 3 by column (the §3.2 baseline)            |
| Spear       | Pierces 2 enemies in the row; 2 from any column; each charged beat adds +3 |
| Short blade | 1 / 2 / 3 by column; parry counter deals 6                                 |

Sub-weapons — fire by themselves every few beats, including beats spent waiting:

| Sub-weapon     | Level 1                                                                               | Per level                                              |
| -------------- | ------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| Throwing stars | Every 3 beats, 1 damage to the first enemy in the player's row                        | +1 star to an adjacent row                             |
| Fire pot       | Every 5 beats, a plus-shaped blast of 2 on a random enemy                             | +1 damage                                              |
| Weighted chain | Every 4 beats, delays the nearest enemy's next move by 1 beat                         | −1 beat between uses (floor 2)                         |
| Shadow double  | A manual strike repeats in one adjacent row for half damage, rounded down, at least 1 | Second adjacent row at level 2; full damage at level 3 |

Arts — passive:

| Art         | Per level                                                                              |
| ----------- | -------------------------------------------------------------------------------------- |
| Lunge       | Front-column strike +1                                                                 |
| Deep breath | Charge damage +1 per charged beat                                                      |
| Keen eye    | Counter damage +2                                                                      |
| Afterguard  | Damage taken on the beat after a risky action −1 (levels 2–3 extend it by a beat each) |
| Quick hands | Sub-weapons fire 1 beat sooner at level 2 and 2 beats sooner at level 3 (floor 2)      |
| Fervor      | Each risky action fills the gauge by +1 more                                           |

Fusions in the MVP:

| Weapon at level 3 | Art         | Result                                                                       |
| ----------------- | ----------- | ---------------------------------------------------------------------------- |
| Throwing stars    | Quick hands | Star storm — hits every enemy in all three rows                              |
| Blade             | Lunge       | Draw cut — a front-column strike pierces the row and counts as a full charge |

### 3.6 Presentation

- Pixel art with a provisional ninja/samurai theme, taken from free assets.
- Assets may be CC0 or require attribution (such as CC BY). Every asset is listed with
  its author, source and licence in a credits file in the repository and on a credits
  screen. Assets that forbid redistribution or modification are not used.
- What no pack covers is drawn as a placeholder sprite in code.
- The pack is chosen at the design stage.

### 3.7 Score and modes

| Item  | Specification                                                                                                                                                                |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Score | All spirit earned in the run (§3.5: middle-column hit +1, front-column hit +2, full-charge hit +3, successful parry +5); at the end of a won run, plus remaining health × 10 |
| Modes | Daily (one seed per UTC day, same line-ups and offers for everyone, share text) and Practice (a new seed each run)                                                           |
| Best  | The best daily score is kept on the device                                                                                                                                   |

- A lost run keeps the points earned so far and gets no health bonus.
- Closing the tab ends the run; resume is not in the MVP.

## 4. Cross-cutting rules

- Every action is available by touch and by keyboard.
- The same seed and the same list of actions always give the same run.

## 6. Open questions

- **Tuning (§3.2, §3.3, §3.5, §3.7)** — the owner accepted the values as a starting
  point; a first playable build tunes them.
- **Asset pack** — which pack, and whether one style covers the player, 4 enemies and a
  boss; settled at the design stage.
- **Where it is published** — played locally first; the target (own static page or a
  portal) is decided after playing. The template's portal packaging stays untouched.

## 7. Decision log

- 2026-11-26 Act-to-advance time (rejected: real-time on fixed ticks, because feel
  tuning needs playtesting the tests cannot cover and touch input is hard; beat-paced
  time kept as a Later timer; full turn-based puzzle, because it is furthest from the
  wanted feel).
- 2026-11-26 Three risk axes — distance, charge, parry (rejected: one charge attack, too
  narrow for varied strategies; three cooldown skills, closer to a chip system).
- 2026-11-26 Two 3×3 territories with a neutral column (rejected: an open 7×3 board,
  more rules; a single row, loses dodging between rows).
- 2026-11-26 Run of 5 battles with upgrade picks, 4 enemy kinds + boss, 12 upgrades
  (rejected: single battle, too thin; 8 battles and 20 upgrades, too much to tune
  first).
- 2026-11-26 Seeded randomness for line-ups, offers and enemy move choice; every chosen
  attack is announced before it lands (rejected: fixed enemy cycles, the owner wants
  enemies to vary; hit or damage rolls, because an announced attack must be reliable).
- 2026-11-26 Build modelled on survivor-style games: automatic sub-weapons, levels,
  weapon-plus-art fusions, picks earned through a gauge filled by risky actions
  (rejected: 12 flat upgrades with one pick per won battle, only 4 picks per run and no
  growth within an item). Size is 3 main weapons, 4 sub-weapons, 6 arts, 2 fusions —
  larger than the 12-upgrade size chosen earlier; the owner confirmed the larger size
  (rejected: starting with the blade only).
- 2026-11-26 Pixel art from free assets, ninja/samurai theme (rejected: abstract shapes;
  code-drawn sprites).
- 2026-11-26 Working title Beatana.
- 2026-11-26 Score rewards risk taken (rejected: beat count, gives no reason to take
  risks; no score, leaves only builds as a reason to replay).
- 2026-11-26 Daily mode stays (rejected: removing it, which is work for less game).
- 2026-11-26 Local play first; publishing target decided later (rejected for now: own
  static page, CrazyGames).
- 2026-11-26 Assets may require attribution (rejected: CC0 only, too few packs; a
  private repository with any licence, the problem returns on publishing).
