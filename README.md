# Scriblit Carousel Assets — Canva Editing Guide

Source assets for the Instagram carousel ideation work (started 2026-08-02), hosted here
publicly so [Canva](https://canva.com) can fetch them by raw URL. This repo is a mirror of
`campaign/assets/social/ideation/mock/` in the private `marketing-scriblit` repo — **the HTML/CSS
files here are the actual source of truth** for copy, color, and spacing; the PNGs are rendered
output, and the Canva versions are an editable/collaborative surface, not canonical. If a Canva
design and the HTML/CSS ever disagree, the HTML/CSS wins.

## What's in this repo

| Path | What it is |
|---|---|
| `card-1-hook.png` … `card-7-cta.png` | Composed reference cards, rendered from the HTML/CSS. Flat, not editable — use as a visual target. |
| `card-N-*.html`, `carousel-base.css` | The real source of truth: exact copy, layout, colors, spacing constants. |
| `components/ring-accent.png` | Hand-drawn purple emphasis ring, transparent PNG, 1600×480 (3.33:1). Used on cards 1 and 7 only — the ring motif is reserved for the first/last card. |
| `components/swipe-arrow-accent.png` | Hand-drawn purple swipe-cue arrow, transparent PNG, 1152×448 (2.57:1). |
| `brand/scriblit-logo.png` | Brand logo, 2000×2000. Appears in the top brand row of every card and the oversized panel on card 7. |
| `ref-title-page.png`, `ref-word-count.png`, `ref-illo-note.png` | Product screenshots actually used inside cards 5, 4, and 3 respectively. |
| `card-2-spreads.mp4`, `card-2-plate.png` | Card 2 is a video slide (Instagram carousels support mixed photo/video). The plate is the still frame with a transparent `.video-slot`; the mp4 is composited into that slot via ffmpeg. |
| `phone-sizes/` | Each card downscaled to 390/375/360px-wide PNGs, for checking true on-device legibility (cards deliver at 1080×1350 but display ~390px wide — everything scales by ~0.36). |

## Brand tokens

| Token | Value |
|---|---|
| Paper (background) | `#F9F4EB` |
| Ink (body text) | `#1A1613` |
| Accent (purple emphasis / ring / arrow) | `#6B2FE3` |
| Display/headline font | Fraunces |
| Body font | Inter |
| Handwriting (swipe cues only) | Architects Daughter |

All three fonts are standard Google Fonts and already built into Canva — **nothing to upload**,
just search for them by name in Canva's font picker.

## Live example

Card 1 was rebuilt as a fully editable Canva design (separate text/image layers, not a flat
import) as a proof of concept:
- Design: https://www.canva.com/d/c3ZnekniLKAocZl
- Project folder (all uploaded components + this card): https://www.canva.com/folder/FAHRMMVYGoo

## Building or editing a card in Canva

### Option A — by hand, in the Canva UI
1. Open the project folder above (or your own — pull assets in from this repo's raw URLs, e.g.
   `https://raw.githubusercontent.com/rmorgan15/scriblit-carousel-assets/main/components/ring-accent.png`).
2. Start a blank **Instagram Post** design — Canva's built-in size is 1080×1350, matching these cards exactly.
3. Optionally drop in the matching `card-N-*.png` as a low-opacity background reference to trace over, then delete it once you're done.
4. Drag in `component-scriblit-logo`, `component-ring-accent`, `component-swipe-arrow-accent`, and whichever screenshot the card needs, as separate image layers.
5. Add text boxes for the copy. Set the font family to Fraunces/Inter/Architects Daughter by name in the font picker, and set fill colors using the hex codes above.
6. Position and resize by eye against the reference PNG or the phone-size preview.

### Option B — AI-assisted (Claude + the Canva MCP tools)
This is how card 1 was built. Roughly:
1. `generate-design` with `design_type: "instagram_post"` (1080×1350 is the built-in default),
   passing the relevant component/screenshot asset IDs via `asset_ids`, and a detailed `query`
   describing the exact copy, colors, and style — don't undersell the detail here, vague queries
   produce vague layouts.
2. `create-design-from-candidate` on whichever candidate looks closest.
3. `read-design` with `open_transaction: true` to get a `transaction_id` and the locator ID of every element.
4. **Budget for a cleanup pass.** The AI generator reliably adds off-brand decoration (sparkles,
   stars) and drops your `asset_ids` in with arbitrary crops rather than showing the full image.
   Expect to `delete_element` the extras and rebuild text as separate `add_text` runs — one run
   per differently-styled word/phrase, since `format_text` styles a whole text box, not a substring.
5. Check the after-thumbnail on every `edit-design` call before continuing — edits come back
   `status: "edits_unverified"` and stay that way until you actually look.
6. `finalize: "commit"` only once the thumbnail looks right — this step is irreversible.

### Two gotchas worth knowing before you start

1. **`format_text` cannot change font family** — only size, color, weight, style, and line-height.
   If the AI-generated font isn't close enough to Fraunces/Inter, you have to reassign it by hand
   in the Canva UI after the design is committed; there's no API path for it.
2. **`update_fill` bakes the image-to-box crop mapping at the instant you call it.** If you
   `resize_element` an image *after* calling `update_fill`, the crop mapping goes stale and the
   image snaps back to looking cropped or distorted, even though the box itself resized correctly.
   The fix: set the element's final size and position **first**, then call `update_fill` **last**.
   For the ring and arrow specifically, resize the box to the component's true source aspect ratio
   before that final `update_fill` call (ring 1600:480 = 3.33:1, arrow 1152:448 = 2.57:1) —
   mismatched aspect ratios make the crop math behave unpredictably even after a refresh.
