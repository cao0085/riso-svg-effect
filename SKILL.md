---
name: seamless-recolorable-riso-tile
description: >-
  Produce a SEAMLESSLY TILING, single-color, fully recolorable risograph-style
  SVG tile — pure geometry, no baked filters — meant to be tiled (CSS
  background-image / SVG pattern) and re-inked from the outside via
  `currentColor`. Use this skill WHENEVER the user wants a riso / zine / screen-
  print BACKGROUND or底紋 / repeating texture that must (a) tile without visible
  seams AND (b) be recolored to a theme color or tone-on-tone, ESPECIALLY when
  some EXTERNAL pipeline (a "filter render", a post-processing step, a grain/
  noise pass) will add the riso grain and ink wobble afterward — so the SVG
  itself must stay clean. Trigger even if the user just says "seamless riso
  tile", "recolorable riso pattern", "riso底紋 I can change the color of", or
  describes the constraints ("must tile", "must recolor", "no embedded filters")
  without naming this skill. This is the CLEAN-MONO counterpart to the
  `digital-risograph` skill: reach for `digital-risograph` when the user wants a
  finished, self-contained multi-ink overprint piece with baked grain; reach for
  THIS skill when seamlessness + recolorability win and texture is applied
  elsewhere.
---

# Seamless Recolorable Riso Tile

A riso底紋 that has to do two jobs at once — **tile seamlessly** and **recolor
cleanly** — cannot be built the classic way. The two requirements actively
fight the riso recipe, so this skill is about resolving that conflict, not about
reproducing overprint. Read this whole file before writing the SVG; the rules
are short but each one prevents a specific failure you will otherwise ship.

## The core conflict (state it to the user)

Classic risograph's signature is **multi-color off-register overprint**: the
same artwork printed once per ink, offset a few pixels, blended with `multiply`
so the overlap darkens into a third color (pink × cyan → purple). That look is
inherently (1) multi-color and (2) built from per-layer translate offsets.

Both of those break the two things this skill exists to deliver:

- **Recolor to one `currentColor`** is incompatible with multi-ink overprint.
  One color cannot be two inks. If the user truly wants tone-on-tone single-
  color, the two-ink overprint is gone — accept it, don't fake it.
- **Seamless tiling** is incompatible with a global `translate` offset. The
  moment you translate a whole layer, the tile's edge no longer matches the
  opposite edge and a seam appears. The riso "registration drift" trick is off
  the table for tiles.

So before building, make the user choose the axis. If they want seamless +
recolorable (the reason they're here), build the **clean mono tile** below. If
they instead want authentic two-ink overprint and will accept a fixed two-color
palette and a non-tiling (centered) placement, send them to the
`digital-risograph` skill instead.

## Two gotchas that justify going clean (tell the user why)

1. **`mix-blend-mode: multiply` often does NOT render when an SVG is tiled as a
   CSS `background-image` or used as a `<pattern>` fill.** Many engines flatten
   or ignore blend modes between internal SVG layers in those contexts. So a
   duotone-overprint tile can fail *precisely* in the tiling use case it was
   made for. A mono tile depends on no blend mode and is immune.
2. **An external "filter render" / grain pass may re-rasterize or strip
   `<filter>` elements.** If the user's downstream pipeline adds the riso grain,
   ink wobble, and mottle itself, any `<filter>` you bake in either collides
   with it or gets thrown away (and can muddy the shapes when re-rastered).
   **When an external pipeline supplies the texture, bake NOTHING — ship pure
   shapes.** Only bake `feTurbulence`/`feDisplacementMap` yourself if you have
   confirmed nothing downstream is adding texture.

Always confirm with the user: *does your render add the grain/filter itself, or
just display the SVG?* The answer decides whether the file is pure geometry or
carries its own (stitched, tileable) filters.

## Required structure

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="80" height="80" viewBox="0 0 80 80">
  <!-- DISCARDABLE paper: delete for transparent tone-on-tone, or recolor. -->
  <rect id="bg" width="80" height="80" fill="#f3eccf"/>

  <!-- MOTIF: everything inherits currentColor. Set `color:` on the <svg> or
       any ancestor to re-ink the whole tile. Unset = black (fine for preview). -->
  <g fill="currentColor">
    <!-- ...shapes... -->
  </g>
</svg>
```

Three non-negotiable parts:

- **One discardable background `<rect id="bg">`.** Give it an id and a comment.
  The user deletes it for transparent tone-on-tone, or swaps its `fill` for a
  paper color. It is never `currentColor` — it must stay independently
  controllable from the ink.
- **A single motif group `fill="currentColor"`.** Never hard-code the ink
  color on the shapes. Recoloring is then "set `color:` from the outside",
  which works tone-on-tone and from any ancestor. Do NOT set `color` on the
  `<svg>` itself unless asked — that blocks inheritance and defeats external
  recoloring. Unset `currentColor` falls back to black, which is a perfectly
  good standalone preview on cream.
- **No baked `<filter>`, no `mix-blend-mode`, no gradients, no embedded raster**
  when texture is applied downstream. Pure flat paths/circles/polygons only.
  (Gradients and embedded bitmaps also kill recolorability outright.)

## Seamlessness rules (this is where tiles actually fail)

The tile must wrap perfectly on all four edges. Enforce all of these:

- **No global `translate`/`rotate`/`scale` on the motif layer.** Bake any
  intended rhythm into coordinates, not transforms.
- **Duplicate every edge-crossing element onto the opposite edge.** A shape
  centered on the left edge must be re-drawn at `x + W` on the right; one on the
  top edge re-drawn at `y + H` on the bottom; a corner element appears at all
  four corners. This is what makes a half-cut dot on one side complete itself on
  the abutting tile.
- **Keep every NON-edge element fully inside,** at least its own radius/half-
  extent away from each edge, so it never silently clips.
- **Choose a period that divides the tile.** If the lattice step is `s`, make
  `W` and `H` integer multiples of `s`. Pick a tile larger than one repeat unit
  (e.g. 2× the step) when you want room to vary element sizes without breaking
  the wrap.
- **If you DO bake your own grain** (only when nothing downstream does), the
  `feTurbulence` must use `stitchTiles="stitch"` and a `baseFrequency` whose
  period fits the tile, or the noise itself will seam.

**Example — duplication:**
Input: a dot at the top-left corner `(0,0)` of an 80×80 tile.
Output: also draw it at `(80,0)`, `(0,80)`, `(80,80)`.

**Example — edge midpoint:**
Input: a dot at top-edge midpoint `(40,0)`.
Output: also draw it at `(40,80)`.

## Reference tile (drop-in dot底紋, 80×80, seamless, mono)

A safe, good-looking default: two-size halftone dots on a half-drop (brick)
lattice, with a sparse intentional scatter. All edge dots are duplicated; all
scatter dots sit inside.

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="80" height="80" viewBox="0 0 80 80">
  <rect id="bg" width="80" height="80" fill="#f3eccf"/>
  <g fill="currentColor">
    <!-- big half-drop lattice, r=6, edges duplicated -->
    <circle cx="0"  cy="0"  r="6"/><circle cx="40" cy="0"  r="6"/><circle cx="80" cy="0"  r="6"/>
    <circle cx="20" cy="40" r="6"/><circle cx="60" cy="40" r="6"/>
    <circle cx="0"  cy="80" r="6"/><circle cx="40" cy="80" r="6"/><circle cx="80" cy="80" r="6"/>
    <!-- mid dots fill the holes; the (0,40)/(80,40) pair is the duplicate -->
    <circle cx="40" cy="40" r="3.6"/><circle cx="0" cy="40" r="3.2"/><circle cx="80" cy="40" r="3.2"/>
    <!-- intentional halftone scatter, all interior -->
    <circle cx="12" cy="18" r="1.5"/><circle cx="68" cy="18" r="1.5"/>
    <circle cx="12" cy="62" r="1.5"/><circle cx="68" cy="62" r="1.5"/>
    <circle cx="34" cy="58" r="1.5"/><circle cx="46" cy="22" r="1.5"/>
  </g>
</svg>
```

## Motif vocabulary (all must obey the seamless rules)

Swap the dots for any of these; they all tile mono and recolor cleanly:

- **Halftone dots** — two or three sizes; the default above. Densest, most
  forgiving.
- **Wavy lines** — a horizontal sine `path` that completes an integer number of
  cycles across `W` (so phase matches at left/right) and repeats every `H`
  vertically. Keep amplitude inside the band so it never crosses top/bottom.
- **Scallops / fish-scale** — overlapping arcs on a half-drop grid; very "print"
  feeling. Duplicate the edge arcs.
- **Diagonal hatch** — short parallel strokes; offset rows; ensure a stroke
  exiting one edge re-enters on the opposite edge (duplicate it).
- **Crosses / sparkles** — small `+` or 4-point stars scattered interior, with a
  faint dot lattice behind for rhythm.

Keep the motif geometric and flat — the riso character (rough edges, grain,
density mottle) is supplied by texture, here either downstream or, only if
needed, by your own stitched filters. Do not try to express grain as hundreds of
hand-placed micro-shapes; that fights a real grain pass and bloats the file.

## Tuning knobs

| Want… | Change |
|---|---|
| denser / sparser | smaller / larger lattice step `s` (keep `W,H` multiples of it) |
| bigger / smaller motif | the `r` / stroke width on the shapes |
| more halftone depth | add a third dot size between the two existing ones |
| less mechanical grid | switch square lattice → half-drop; vary interior sizes over the full tile (not over one step) |
| different paper | the `fill` on `#bg` (or delete `#bg` for transparent) |
| re-ink | set `color:` on the `<svg>` / container — never touch the shape fills |

## Output

- Deliver a single `.svg` file (the tile), commented so the user can find the
  discardable `#bg` line and the `currentColor` group at a glance.
- Default the tile to 80×80 unless the user specifies; mention how density
  scales with tile size (40×40 denser, 120×120 looser).
- Do not wrap the tile in a large pattern-filled canvas — ship the tile itself;
  the user's pipeline does the tiling.

## Pitfalls

- Hard-coding the ink color on shapes (kills recoloring). Use `currentColor`.
- Setting `color` on the root `<svg>` (blocks external re-inking).
- Any gradient, embedded bitmap, or per-layer blend mode (kills recolor and/or
  tiling).
- A global transform on the motif layer (instant seam).
- Forgetting to duplicate an edge element (a dot that's a half-circle on one
  side and missing on the abutting side).
- Baking `<filter>` grain when a downstream pipeline already adds texture
  (collision, or stripped + muddied on re-raster).
