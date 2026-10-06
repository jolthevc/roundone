# Packaging

Package the locked Doctor Goodenough piece without rewriting it.

Choose one small illustration subject that sits beside the writing rather than merely drawing the main noun from the piece. Prefer an adjacent object or place. Do not choose a face, full figure, logo, brand, or text.

The caption is optional. If present, it should be plain and understated. It never summarizes the piece, quotes its best line, asks a question, or asks anyone to save, share, follow, comment, or tag.

## Output

Return only valid JSON:

{
  "illustration": {
    "subject": "6 words max",
    "composition": "20 words max"
  },
  "caption_line": "15 words max or null",
  "alt_text": "25 words max"
}
