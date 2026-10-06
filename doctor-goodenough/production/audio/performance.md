# Performance

The literature is the source of truth. The performance script is its score.

Performance adaptation changes delivery, never meaning and never wording.

## What may change

The performance script may add:
- breath and pause notation
- speech-only punctuation
- line or paragraph spacing for cadence
- at most one or two light temperature cues

The title is not spoken by default.

## Delivery

Speak intimately and quietly. The pace is deliberate but conversational, not sleepy. Important lines do not automatically receive dramatic pauses. A simple line can remain simple.

Useful temperature cues:
- `[quiet]`
- `[warm]`
- `[dry]`
- `[slower]`
- `[firmer]`

Useful pause cues:
- `[short pause]`
- `[pause]`
- `[long pause]`

The eventual TTS implementation must test which cues the pinned model actually supports. Any cue the model reads aloud is removed or translated into provider-specific syntax before rendering.

Most pieces should use very few cues.

## Register

Light pieces should sound amused without selling the joke. Warm pieces should feel close and unhurried. Still pieces should be level enough to let the image sit. Heavy pieces should become steadier, not sadder. Mixed pieces can move slightly in temperature without becoming theatrical.

## Hard production rule

After adaptation, code strips delivery notation and normalizes punctuation. The spoken word sequence must match the locked literary copy exactly.

If words changed, the performance adaptation fails and is retried once. The literary copy is never silently altered downstream.
