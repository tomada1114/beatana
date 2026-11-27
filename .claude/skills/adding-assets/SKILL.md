---
name: adding-assets
description: >
  Use when adding, replacing, or removing a sprite sheet, image, or font file; choosing
  an asset pack or checking its licence (CC0, CC BY, OFL); editing the credits file or
  the Credits screen's list; loading assets in BootScene (this.load.spritesheet,
  this.load.font); DotGothic16; the Ninja Adventure Asset Pack; a placeholder sprite
  drawn in code; or a pnpm budget failure whose largest files are assets.
---

# Adding Assets

**Owns:** which image and font files ship, where they live, how Boot loads them, their
licences, and the credits file. **Does not own:** colours, sizes, and the type scale
(`designing-ui`); how a scene draws a sprite (`building-scenes`); the budget script and
its limits (`integrating-portals`).

Not applied yet: no asset ships. The first asset issue adds the files, the credits file,
and the loader calls, and removes this line.

## Where files live

- Every shipped asset sits under apps/web/public/assets/ (created by that issue) and is
  loaded by a relative URL such as `assets/sprites/player.png`. Vite copies `public/` as
  is, and `base: "./"` keeps every URL relative, which the portal requires.
- Boot loads all of them in `preload()`, behind its progress bar: sprites with
  `this.load.spritesheet`, the font with
  `this.load.font("DotGothic16", url, "truetype")` (both in the installed Phaser 4.2.1's
  types, checked 2026-11-27). Text that uses the font is created only after Boot
  completes, so no scene draws with a fallback face.
- A sprite is drawn at an integer scale only (`layout.spriteScale`); the game config's
  `render.pixelArt` keeps it sharp. A sheet whose frames are not a whole number of
  pixels at that scale is rejected, not resampled.

## Licences and credits

- An asset is used only when its licence allows redistribution and modification: CC0, CC
  BY, or the SIL Open Font License. "Personal use only", "no redistribution", or a
  licence stated differently in two places is not used until the author's own page
  settles it.
- Every asset — CC0 included — has an entry in one credits file,
  apps/web/src/credits/credits.json (created by the first asset issue): name, author,
  source URL, licence, and the date the licence was read. The Credits screen is drawn
  from that file, so the two cannot differ. The licence text that must travel with the
  asset (OFL.txt for the font) ships beside it under the assets folder.
- What no pack covers is a placeholder drawn in code at the pack's sprite size. A
  placeholder needs no credit; replacing it with an asset adds the entry in the same
  change.

## The font

- DotGothic16 Regular ships as its TTF from Google Fonts, unmodified: 2,069,236 bytes,
  SIL Open Font License 1.1, copyright The DotGothic16 Project Authors, no Reserved Font
  Name (https://github.com/google/fonts/tree/main/ofl/dotgothic16, checked 2026-11-27).
- It is not subset. With the sample's build at 1,393,067 bytes, the font leaves about
  1.7 MB of the bundle budget for sprites, and every Japanese string renders without a
  build step or a new dependency. If sprites outgrow the remainder, subsetting the font
  (allowed by the OFL, since no name is reserved) is the next step — the owner's
  decision, since the tool is a new dependency.

## The sprite pack

- First candidate, not yet settled: Ninja Adventure Asset Pack by Pixel-Boy and AAA,
  CC0, 16×16 sprites, 50+ characters and 30+ monsters with four-direction animations, 9
  bosses (https://pixel-boy.itch.io/ninja-adventure-asset-pack, checked 2026-11-27). The
  download is behind a pay-what-you-want page, so a human fetches it.
- The pack is top-down. Before it is settled, the first asset issue checks that its
  side-facing frames read as a side view, that four enemy kinds and a boss are distinct
  silhouettes, and that the selected sheets fit the budget's file count (one sheet per
  character, not one file per frame).
