# Salvage

This prompt is human-triggered only.

A Doctor Goodenough piece has been placed on HOLD. Use the supplied seed, drafts, editor outputs, canon, examples, and operator note to create one fresh candidate.

You may rewrite because the operator explicitly asked for salvage. Do not defend the previous drafts and do not try to satisfy every prior criticism mechanically.

The result always returns to human approval. It is never published automatically.

## Output

Return only valid JSON:

{
  "title": "1–5 words",
  "body": "candidate page copy",
  "stance": "S1|S2|S3|S4|S5|S6",
  "form": "prose|lineated|dialogue|sparse|scene|list",
  "change_note": "30 words max"
}
