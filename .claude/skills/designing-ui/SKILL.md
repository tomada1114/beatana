---
name: designing-ui
description: >
  Use when choosing or changing a colour, font, type size, radius, spacing, or motion
  value; editing apps/web/src/theme/tokens.ts; finding a colour or size literal
  elsewhere in apps/web; checking text, cell, control, or focus-outline contrast (4.5:1,
  3:1); the accent, danger, or health colour roles; applying the design lock in
  references/design-lock.md to tokens.ts and docs/design/contrast-pairs.json; or
  changing the direction recorded in docs/design/design-direction.md.
---

# Designing UI

**Owns:** the visual design lock, `apps/web/src/theme/tokens.ts`, and the contrast rule.
**Does not own:** screens, flows, and where things sit on them
(`docs/product/ux-flows.md`, drawn by `building-scenes`); how a scene consumes a token
(`building-scenes`); the portal's page rules (`integrating-portals`).

## The lock

- [references/design-lock.md](references/design-lock.md) is binding: direction, role
  rules, token table, contrast table. `apps/web/src/theme/tokens.ts` holds the same
  values as code. When the two disagree, that is a bug to fix in one change, not a
  choice between them. `docs/design/design-direction.md` is the record the lock was
  written from.
- The lock describes Beatana; `tokens.ts` still holds the sample's values until the
  design-token issue applies it (the lock's own note says so). Until then the mismatch
  is expected, not a bug; no other change edits either side.
- Changing a value inside the direction is an edit. Changing the direction itself (a
  light theme, textures, image tiles) is a new design stage, not an edit.
- The role rules are what review catches most: `accent` (vermilion) is only what the
  player can press or gain right now, and never danger or health; `danger` (amber) is
  only announced cells and intent marks, never anything pressable; no token recolours a
  sprite; every shape is square (radius 0).
- One theme. Games run as full-screen canvases inside portal pages, so the lock does not
  follow the system colour scheme.

## Tokens

- `apps/web/src/theme/tokens.ts` exports `colors`, `type`, `radius`, `space`, `motion`,
  and `layout` as `as const` objects; `layout`'s keys and values are the lock's layout
  table. Each colour exists twice from one source: a `0xRRGGBB` number for Phaser and a
  `css` string for the DOM, so the canvas and the DOM elements (the Copy result
  fallback, the page background) cannot drift.
- Nothing else in `apps/web` writes a colour or size literal. A size that is not yet a
  token is added to `layout` in `tokens.ts`, not written inline.
- No lint rule enforces this. A reviewer looks for `0x`, `#`, and bare numeric sizes or
  durations outside `tokens.ts` in the diff.
- Sizes are design pixels of the 720×1280 frame; spacing is a multiple of the 8 px unit.

## Contrast

- Text is at least 4.5:1 against its background; tiles, controls, and the focus outline
  are at least 3:1. Colour is never the only signal: danger always carries a digit or
  hatching, and an intent mark is an icon with a digit.
- Ratios are WCAG 2.2 relative-luminance contrast (SC 1.4.3 and 1.4.11,
  https://www.w3.org/TR/WCAG22/, checked 2026-11-05). Changing any colour re-runs every
  row of the contrast table in the lock, and the new ratios land in the same change. A
  failing pair is not merged. `docs/design/contrast-pairs.json` lists every pair with
  its `kind`, and `apps/web/test/theme/contrast-pairs.test.ts` recomputes each from
  `tokens.ts` and fails below its threshold or when a colour token appears in no pair.
- Measure pairs as they are drawn. A new element on a new background adds a row; a row
  for a pair that no longer appears is removed.
- The focus ring is the pair most easily missed: it is measured on every background it
  is drawn on (`surface` and `surfaceRaised` in the lock), and it is drawn
  `layout.focusOffset` outside the control, never against a sprite.
- The font is DotGothic16 (file and licence: `adding-assets`).
