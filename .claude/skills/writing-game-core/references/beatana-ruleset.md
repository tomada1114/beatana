# Beatana's ruleset in core

The shape the game's rules take in `packages/core`. The values (health, damage, cycles,
weights) are in `docs/product/requirements.md` §3 and become `BEATANA_CONFIG`; this page
fixes the structure that is expensive to change once goldens exist.

Not applied yet: the code still runs the sample. The first game issue implements this
page, replaces the sample (`starting-an-app`), and removes this line.

## Three random streams

- The run has three xoshiro128** generators, each with its own four words in `State`:
  `lineup`, `offers`, and `enemy`. Each is seeded through splitmix32 from the run seed
  mixed with its stream index (0, 1, 2); the exact mix is frozen by the first golden
  values.
- `lineup` is drawn only inside `createGame`: all five battles' enemy kinds and start
  cells, up front. Two players on the same Daily seed meet the same line-ups however
  many beats they spend.
- `offers` is drawn only by a pick opening and by a reroll. The same seed, the same pick
  number, and the same build give the same three options.
- `enemy` is drawn by enemy move choices and every other in-battle random choice (the
  fire pot's target). It is the only stream the player's beat count moves.
- Why three: with one stream, every enemy draw shifts every later line-up and offer, and
  §3.7's "same line-ups and offers for everyone" could not hold. A fourth stream is a
  rules change like any other (golden values move).

## Phases and moves

`State.phase` is one of `"weapon"`, `"battle"`, `"pick"`, `"interlude"`, `"over"`.

| Move                                               | Phase       | Effect                                                     |
| -------------------------------------------------- | ----------- | ---------------------------------------------------------- |
| `{ t: "weapon", id }`                              | `weapon`    | sets the main weapon, starts battle 1                      |
| `{ t: "move", dir }`                               | `battle`    | one cell up / down / forward / back, one beat              |
| `{ t: "strike" }`, `"charge"`, `"parry"`, `"wait"` | `battle`    | one beat                                                   |
| `{ t: "take", slot }` (0–2)                        | `pick`      | takes an option; back to `battle`, or the next queued pick |
| `{ t: "reroll" }`                                  | `pick`      | redraws the options from `offers`; one of the run's 2      |
| `{ t: "continue" }`                                | `interlude` | starts the next battle                                     |

- Every choice that changes the state or draws randomness is a move, so the replay
  record stays `{ v, seed, config, moves }` and a run replays from it alone. Giving up
  is not a move: the run is discarded and nothing is recorded.
- `canApply(state, move)` is exported beside `applyMove` and is the only legality check:
  false for a move the phase does not take, a step off the player's territory, a reroll
  with none left, or a finished run. `applyMove` throws `ERR_INVALID_MOVE` exactly when
  `canApply` is false. Scenes call `canApply` first; false means nothing visible and no
  beat spent (§3.1), and the scene never re-derives the territory or the phase.

## One beat

In this order, inside one `applyMove` of a `battle` move:

1. The player's action resolves (a parry is armed for this beat).
2. Each sub-weapon due this beat fires.
3. Every enemy advances: an announced attack lands or counts down, then an enemy whose
   cycle comes round draws its next move from `enemy` and announces it.
4. Spirit gained this beat is added; each time the gauge reaches 10 it empties and one
   pick is queued. A queued pick moves the phase to `pick` after the beat.
5. Every enemy at 0 ends the battle (`interlude`, or `over` after battle 5); the player
   at 0 ends the run (`over`).

- `State.lastBeat` lists what happened in the beat just played — hits with their damage,
  spirit gained, a successful parry, a sub-weapon firing, an attack announced or landed
  — as plain objects of integers and short string tags. It is replaced, never appended,
  by each `applyMove`. Scenes animate from it and never compute an outcome themselves;
  because it is part of `State`, it is covered by the golden values and the JSON round
  trip.
- The run's health, spirit gauge, rerolls, and build carry from battle to battle; the
  gauge is not reset between battles.
- Picks: when every sub-weapon and art slot is full, options are only level-ups of owned
  items and fusions; when nothing is left, the single option is +1 health.

## Config, score, end

- `BEATANA_CONFIG` holds every tuning value as JSON integers: player health, damage per
  column, charge, parry, enemy tables (health, moves, weights, cycles), battle sizes,
  gauge values, item tables. `RULES_VERSION` fixes it, as `SAMPLE_CONFIG` is fixed
  today.
- Half damage is `Math.floor(x / 2)` with a floor of 1; nothing in the rules needs a
  float.
- `score(state)` is the spirit earned in the run, plus remaining health × 10 when the
  run was won. `isOver(state)` is `phase === "over"`.
