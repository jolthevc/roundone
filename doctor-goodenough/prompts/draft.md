# Draft

Write one Doctor Goodenough piece from the supplied seed.

Let the piece be whatever the thought wants: prose, a few lines, a scene, framed monologue, dialogue, sparse observation, or another form that genuinely serves the material. Most pieces land around 55 to 130 words; shorter is welcome.

A stance may be suggested. Treat it as a nudge, not a command. If another stance better serves the piece, use it, while preserving the integrity rules.

The calibration examples show range and voice. They are not shapes to copy.

Do not explain what the piece means. Do not summarize your intention. Do not provide alternatives unless this is explicitly a redraft call.

If this is a redraft, you will receive a short reason the previous attempt failed. Write a fresh piece from the seed without trying to patch or imitate the previous draft.

## Output

Return only valid JSON:

{
  "title": "1–5 words",
  "body": "final page text with line and paragraph breaks preserved",
  "stance": "S1|S2|S3|S4|S5|S6",
  "form": "prose|lineated|dialogue|sparse|scene|list"
}
