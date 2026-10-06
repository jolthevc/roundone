# Visual Standard

Doctor Goodenough should look like a page from a dark literary book that somehow appeared inside a social feed.

The page is an editorial object, not a quote card.

## Fixed world

- 9:16 vertical canvas
- near-black textured paper
- warm ivory serif typography
- small understated title near the top
- literary body copy with generous margins
- `— Doctor Goodenough` anchored near the lower left
- one small hand-drawn / etched illustration near the lower right
- restrained grain and almost imperceptible motion
- no kinetic captions
- no large ROUND ONE branding
- full text visible from the first frame

The exact font, measurements, coordinates, and safe zones should be deterministic in the renderer once the approved visual-reference assets are locked. They are never decided by an image model.

## Two layouts

Use two layouts only:

**Prose** — paragraph-driven copy, ragged right, no justification.

**Lineated** — intentional source line breaks are preserved; long lines may wrap conservatively.

The renderer may use a small number of typographic density states to fit copy, but it should never shrink indefinitely. If a piece cannot fit within the approved range, return a fit failure before copy lock.

## Illustration

Image generation creates the illustration only. It never renders the page, typography, title, signature, or branding.

The illustration is:
- contextual to the piece
- secondary to the text
- small
- warm ivory linework after processing
- etched, drawn, or editorial rather than photographic
- free of text, logos, and faces

Prefer an **adjacent object** to a literal illustration of the main noun in the piece. The image should feel as though it belongs in the same world without explaining the writing.

A practical generation method is black ink on a white background, followed by deterministic conversion of luminance to alpha and tinting to the canonical ivory. This is more reproducible than asking the image model to create final transparent ivory linework.

## Motion

The page is effectively still. A very slow unified push or drift, slight texture movement, and subtle grain are enough. Text and illustration do not animate independently.

Stillness is part of the premium feeling.

## Platform safety

The final renderer must use conservative safe zones for Reels, TikTok, and Shorts so title, text, signature, and illustration remain clear of platform overlays and common profile-grid crops.

Verify these measurements against current platform templates when the renderer is built. Do not hard-code unverified social safe-zone assumptions into the canon.

## Muted test

With audio muted, the post should still feel complete as a literary page.

With the image removed, the page should still feel intentionally designed.

With the branding removed, the writing should still be worth reading.
