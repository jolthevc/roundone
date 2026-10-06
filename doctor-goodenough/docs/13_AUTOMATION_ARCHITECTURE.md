# 13 — Automation Architecture

## Purpose

This document defines the end-to-end production architecture for Doctor Goodenough in n8n.

The target state is full autopilot:

**idea -> finished writing -> performed narration -> music -> illustration -> rendered video -> quality checks -> scheduled publication**

The system should be reliable without becoming a giant agent graph.

The core principle is:

> **Separate creative generation from production orchestration. Keep every step inspectable, bounded, and recoverable.**

Doctor Goodenough should use a small number of workflows with clear ownership rather than one enormous canvas.

---

# 1. Recommended system shape

Use **three operating workflows plus one shared error workflow**.

## Workflow A — IDEATION

Generates a batch of independent seeds and puts them into the content queue.

This workflow is allowed to work in batches.

It does **not** write finished pieces.

## Workflow B — CREATION

Claims exactly **one** queued seed and turns it into a finished, publish-ready media asset.

One run = one Doctor Goodenough piece.

This is the creative and production factory.

## Workflow C — PUBLISH

Claims exactly **one** publish-ready piece when its scheduled time arrives and distributes it to configured channels.

Publishing is separate from creation so a social API outage never forces the piece to be regenerated.

## Workflow D — ERROR HANDLER

A small global n8n error workflow.

It records the failure, increments attempt counts, preserves the current state, and alerts only when intervention is required.

---

# 2. Source-of-truth architecture

Different information belongs in different systems.

## GitHub — creative canon

GitHub remains the source of truth for:

- brand documents
- literary standards
- prompts
- voice standard
- visual standard
- performance standard
- production rules
- approved visual references
- music manifest / approved music metadata

A production run should record the **Git commit SHA** of the canon it used.

This makes every piece reproducible.

## Content ledger — workflow state

Use one persistent content ledger for:

- ideas
- current status
- generated copy
- editorial diagnosis
- locked final copy
- performance script
- asset URLs
- scheduled publish time
- platform publication state
- failures and retries

For an MVP, this can be an n8n Data Table.

For durable production, a simple Postgres / Supabase table is preferable.

Do not use GitHub as the production queue.

## Object storage — media

Store generated binary media in object storage:

- voice audio
- illustrations
- mixed audio
- rendered videos
- thumbnails / stills if needed

S3-compatible storage, Cloudflare R2, or an equivalent service is appropriate.

The content ledger stores URLs and metadata, not binary media.

## n8n — orchestration

n8n owns:

- schedules
- API calls
- state transitions
- branching
- retries
- queue claiming
- publication
- alerts

n8n should not become the renderer.

## Render service — deterministic media assembly

Use a small deterministic rendering service built with Remotion and FFmpeg.

n8n sends it a structured packet.

The renderer returns the finished MP4.

The renderer owns:

- typography
- exact positions
- text layout rules
- background texture
- illustration placement
- subtle motion
- audio placement
- final encoding

This keeps the visual system consistent and prevents generative image models from rendering text.

---

# 3. Content ledger

A piece should move through explicit states.

Recommended statuses:

```
IDEA_READY
CREATING
COPY_LOCKED
ASSETS_BUILDING
READY_TO_PUBLISH
PUBLISHING
PUBLISHED
HOLD
ERROR
ARCHIVED
```

Recommended fields:

```json
{
  "piece_id": "dg_000123",
  "status": "IDEA_READY",
  "canon_commit_sha": null,
  "created_at": null,
  "scheduled_at": null,

  "seed": {},
  "theme_tags": [],
  "emotional_temperature": null,

  "draft": null,
  "editorial_review": null,
  "page_copy_final": null,
  "copy_hash": null,

  "performance_note": null,
  "performance_script": null,

  "illustration_brief": null,
  "illustration_url": null,

  "voice_audio_url": null,
  "voice_duration_seconds": null,

  "music_family": null,
  "music_track_id": null,
  "mixed_audio_url": null,

  "render_url": null,
  "render_duration_seconds": null,

  "caption": null,

  "publish": {
    "instagram": {},
    "tiktok": {},
    "youtube": {}
  },

  "attempt_count": 0,
  "error_stage": null,
  "error_message": null,
  "hold_reason": null,
  "do_not_publish": false
}
```

The exact schema may evolve.

The important rule is that every major artifact is stored and every transition is explicit.

---

# 4. Workflow A — IDEATION

## Purpose

Create a healthy queue of distinct Doctor Goodenough seeds without writing the pieces themselves.

This workflow can run manually during calibration and later on a schedule.

## Recommended flow

### 1. Trigger

Manual Trigger while testing.

Later:

- Schedule Trigger
- or "replenish when queue < X" logic

### 2. Load current canon

Read the Doctor Goodenough creative canon and ideation prompt from GitHub.

Record the current commit SHA.

### 3. Load recent content context

Pull a compact view of recent ideas and published pieces from the content ledger.

The purpose is not to imitate past pieces.

The purpose is to avoid unconscious clustering.

Useful recent context:

- titles
- seed summaries
- broad subject tags
- emotional temperature
- whether past / present / future
- publish date

Do not feed full old pieces unless needed.

### 4. Ideation model call

Use `prompts/ideation.md`.

Generate a batch such as 10–20 seeds.

Each seed should return structured fields such as:

```json
{
  "working_seed": "...",
  "human_pressure": "...",
  "hidden_subject": "...",
  "images_or_behaviors": ["...", "..."],
  "why_it_may_resonate": "...",
  "emotional_temperature": "...",
  "broad_subject": "...",
  "temporal_orientation": "present"
}
```

The model should seek variety through judgment, not quotas.

### 5. Lightweight duplicate / clustering check

Compare new seeds against the recent ledger.

Reject only obvious repeats or near-repeats.

Do not over-engineer semantic uniqueness.

The goal is to prevent ten variations of the same father / childhood / career piece, not to make every idea mathematically unique.

### 6. Insert into queue

Write accepted seeds to the ledger with:

```
status = IDEA_READY
```

No drafting occurs in this workflow.

## Expected node count

Approximately **5–7 nodes**.

This should remain small.

---

# 5. Workflow B — CREATION

## Purpose

Turn exactly one queued seed into exactly one finished, publish-ready Doctor Goodenough video.

**One run = one piece.**

Never fan 15 pieces through this workflow at once.

The unit of production is the individual piece.

---

## Stage 1 — Claim one idea

### 1. Trigger

During testing:

- Manual Trigger

On autopilot:

- Schedule Trigger every few minutes
- or queue / webhook trigger

### 2. Atomically claim one piece

Find the oldest or highest-priority row with:

```
status = IDEA_READY
```

Immediately change it to:

```
status = CREATING
```

This lock prevents duplicate creation if two runs overlap.

If no item exists, end successfully.

### 3. Load and freeze canon

Fetch current relevant Doctor Goodenough docs and prompts from GitHub.

Store the Git commit SHA on the piece.

That SHA stays with the piece forever, even if the canon changes later.

---

## Stage 2 — Literary creation

### 4. Draft

Inputs:

- seed
- current canon
- `prompts/draft.md`

Outputs:

- title
- draft page copy
- internal discovery

Store the draft.

### 5. Editorial review

Use a separate model call with:

- canon
- draft
- `prompts/editorial_review.md`

The review diagnoses the piece.

It does not score it.

### 6. Revision

Inputs:

- original draft
- editorial diagnosis
- canon
- `prompts/revision.md`

Output:

- title
- revised page copy
- revision note

### 7. Narrative gate

Run one final editorial check.

Return only:

```json
{
  "decision": "PASS | REVISE | HOLD",
  "reason": "...",
  "revision_instruction": "..."
}
```

This is not a numeric rubric.

### Bounded revision rule

If the decision is `REVISE`, allow **one additional revision pass**.

Then re-check.

If it still does not pass:

```
status = HOLD
```

Stop.

Do not create an infinite critic / writer loop.

Autopilot quality comes from refusing to publish weak work, not from allowing agents to argue forever.

### 8. Lock literary copy

Once passed:

- write `page_copy_final`
- compute / store a hash
- set `status = COPY_LOCKED`

From this point forward, the literary text is immutable for this run.

Downstream nodes may adapt delivery but may not silently rewrite the piece.

---

# 6. Performance and voice

## Stage 3 — Performance adaptation

### 9. Performance adaptation

Use:

- locked page copy
- Doctor Goodenough performance canon
- voice canon
- `prompts/performance_adaptation.md`

Output:

```json
{
  "performance_note": "...",
  "performance_script": "..."
}
```

The performance script is a score for delivery.

The page copy remains the literary source of truth.

### 10. Text-to-speech

Send the performance script to the approved Doctor Goodenough voice in ElevenLabs or the chosen TTS provider.

Store:

- voice audio URL
- exact duration
- provider request / generation ID
- voice version / voice ID

Prefer WAV or another high-quality intermediate.

### 11. Audio sanity check

Programmatically verify:

- file exists
- duration is plausible
- audio stream is valid
- no zero-length render
- no obvious clipping / invalid level
- expected sample rate / format

Do not add a complicated AI audio critic unless real failures show it is needed.

---

# 7. Music

## Core recommendation

**Do not generate a brand-new soundtrack for every piece.**

Doctor Goodenough needs a sonic identity.

Create a small approved library of excellent ambient beds and let the workflow select among them.

A recommended initial library is roughly **6–12 tracks** spanning a few emotional families:

- quiet / reflective
- warm
- ache
- resolve
- tension
- nocturnal / mysterious
- lighter / gently amused

Every track should still live inside the same Doctor Goodenough sonic world.

This is more consistent, cheaper, faster, and safer than asking a generative music model to reinvent the brand every day.

Music generation can be a separate occasional library-maintenance process.

It is not part of the daily piece factory.

## Stage 4 — Music selection

### 12. Select music family and track

Use the final piece and its emotional temperature to select:

- music family
- one approved track ID

Selection may be deterministic or a small model call.

Avoid using the same track too many times in succession.

### 13. Mix voice + music

Use FFmpeg or the render service.

The mix should:

- keep voice clearly foregrounded
- keep music materially quieter
- fade music in / out cleanly
- trim or loop the bed to the required duration
- avoid an audible seam if looping
- optionally apply gentle ducking under voice
- normalize final output to a consistent target

Store the mixed audio master.

The music should feel like the room around the narration, not a score competing with it.

---

# 8. Illustration generation

## Stage 5 — Art direction

### 14. Generate illustration brief

After the literary copy is locked, generate a compact art brief.

Output:

```json
{
  "subject": "...",
  "symbolic_connection": "...",
  "illustration_prompt": "...",
  "placement_note": "bottom-right",
  "avoid": ["text", "logo", "photorealism"]
}
```

The illustration subject depends on the narrative.

The illustration style does not.

### 15. Generate illustration only

Use the image-generation API to create the small contextual illustration.

Do **not** ask the image model to create the full page.

Do **not** ask it to render the text.

The generated asset should ideally be:

- transparent background
- warm ivory linework
- etched / hand-drawn style
- no text
- no logo
- no border
- no photographic realism

If transparency from the provider is unreliable, use a known solid background and remove it programmatically before rendering.

### 16. Illustration QC

Use a lightweight vision check only for obvious failures:

- text accidentally generated
- wrong style
- unusable crop
- photorealistic output
- major subject mismatch

Allow **one regeneration** with a corrected prompt.

If still unusable:

```
status = HOLD
```

Do not silently publish a visually off-brand image.

---

# 9. Deterministic visual rendering

## Stage 6 — Render

### 17. Build render packet

Send the render service:

```json
{
  "piece_id": "dg_000123",
  "title": "...",
  "page_copy": "...",
  "signature": "— Doctor Goodenough",
  "illustration_url": "...",
  "audio_url": "...",
  "duration_seconds": 42.3,
  "template_version": "dg-page-v1"
}
```

### 18. Render with Remotion

The template owns:

- 1080 x 1920 canvas
- black / near-black textured paper
- exact font family
- exact title anchor
- exact title treatment
- exact body safe area
- exact signature anchor
- illustration safe area
- page margins
- line spacing
- subtle film grain
- very slow push / drift

The generated illustration is an ingredient.

The page itself is deterministic.

### Text fitting policy

Use only a narrow set of approved typography states.

Example:

- standard
- slightly denser
- sparse

Never shrink text indefinitely to force long writing onto the page.

If the locked copy cannot fit within approved bounds, fail the render and return the piece to `HOLD` or a controlled editorial adjustment.

Visual consistency is more important than preserving an oversized piece.

### 19. Final mux / encode

Remotion may handle audio directly, or FFmpeg may mux the rendered visual with the mixed audio.

Create one platform-safe MP4 master.

---

# 10. Final quality control

## Stage 7 — Technical gate

### 20. Validate master

Programmatically check:

- file exists
- H.264 / supported codec
- 1080 x 1920
- correct frame rate
- valid audio stream
- duration matches expected audio duration
- no zero-byte asset
- acceptable file size
- no clipped ending
- correct template version

Because the page is rendered from the exact locked string, OCR verification should not be necessary.

The renderer already knows what text it placed.

### 21. Metadata / caption

Generate the platform caption only after the piece is locked.

Keep metadata minimal and on-brand.

Store:

- title
- caption
- any channel-specific copy
- scheduled publish time

### 22. Save master and mark ready

Store final assets in object storage.

Set:

```
status = READY_TO_PUBLISH
```

Workflow B ends here.

It does not publish.

---

# 11. Workflow C — PUBLISH

## Purpose

Publish one ready piece without coupling social API reliability to creative production.

## Recommended flow

### 1. Schedule Trigger

Run frequently enough to catch scheduled publication windows.

### 2. Claim one due piece

Query:

```
status = READY_TO_PUBLISH
scheduled_at <= now
do_not_publish = false
```

Lock one row and set:

```
status = PUBLISHING
```

### 3. Publish per channel

Each channel should have its own publication state.

Potential channels:

- Instagram Reels
- TikTok
- YouTube Shorts
- X / other channels later

Do not assume every platform uses the same payload.

Record for each channel:

- attempt time
- status
- returned post ID
- returned URL when available
- error
- retry count

### 4. Idempotency

Every platform upload needs a stable idempotency identity based on:

```
piece_id + platform
```

A retry must never create a duplicate post after a timeout or partial success.

Before retrying, check whether the platform post ID was already recorded.

### 5. Partial failure behavior

If Instagram succeeds and TikTok fails:

- preserve Instagram success
- retry TikTok only
- do not regenerate the video
- do not repost Instagram

### 6. Finish

When required channels succeed:

```
status = PUBLISHED
```

Store all final platform IDs and URLs.

---

# 12. Workflow D — ERROR HANDLER

Use the standard n8n Error Trigger.

On failure:

1. identify `piece_id`
2. identify workflow and stage
3. persist the error
4. increment attempt count
5. preserve existing assets
6. decide whether error is retryable
7. retry only the failed stage when safe
8. move to `HOLD` after the bounded retry limit
9. alert the operator only when intervention is actually required

Never restart an entire piece because one downstream API failed.

Examples:

- TTS timeout -> retry TTS
- illustration API failure -> retry illustration
- render failure -> retry render
- Instagram failure -> retry Instagram
- literary gate failure -> HOLD, do not repeatedly regenerate forever

---

# 13. Autopilot modes

The same architecture should support two operating modes.

## REVIEW mode

Useful during calibration.

The workflow automatically creates the piece and assets, then stops at a chosen checkpoint for human inspection.

Recommended early checkpoints:

- after final copy
- or after final MP4

## AUTOPILOT mode

Once the system proves itself:

```
IDEA_READY
-> CREATING
-> COPY_LOCKED
-> ASSETS_BUILDING
-> READY_TO_PUBLISH
-> PUBLISHING
-> PUBLISHED
```

No human action is required unless a piece enters `HOLD` or repeated technical errors occur.

Do not build separate creative logic for REVIEW and AUTOPILOT.

Use the same pipeline with one configuration flag.

---

# 14. What should not be autonomous

"Autonomous" does not mean every creative asset should be invented from scratch every day.

Keep these stable and human-approved:

- Doctor Goodenough canon
- prompts
- narrator / voice
- visual template
- fonts
- colors
- typography geometry
- background texture
- music library
- publishing account configuration
- platform cadence rules

The autonomous system should create within the world.

It should not redesign the world every run.

---

# 15. Suggested production packet

Once the piece is complete, the canonical record should resemble:

```json
{
  "piece_id": "dg_000123",
  "canon_commit_sha": "abc123",
  "template_version": "dg-page-v1",

  "seed": {
    "working_seed": "...",
    "human_pressure": "...",
    "hidden_subject": "..."
  },

  "literary": {
    "title": "...",
    "draft": "...",
    "editorial_review": "...",
    "page_copy_final": "...",
    "copy_hash": "..."
  },

  "performance": {
    "note": "...",
    "script": "...",
    "voice_id": "...",
    "voice_audio_url": "...",
    "duration_seconds": 42.3
  },

  "music": {
    "family": "warm",
    "track_id": "dg_music_04",
    "mixed_audio_url": "..."
  },

  "visual": {
    "illustration_brief": "...",
    "illustration_url": "...",
    "render_url": "...",
    "template_version": "dg-page-v1"
  },

  "distribution": {
    "caption": "...",
    "scheduled_at": "...",
    "instagram": {},
    "tiktok": {},
    "youtube": {}
  }
}
```

This packet becomes the single audit trail for the piece.

---

# 16. Workflow map

```
                         DOCTOR GOODENOUGH CANON
                                  |
                                  v
+------------------+       +-----------------------+
| WORKFLOW A       |       | RECENT CONTENT LEDGER |
| IDEATION         |<----->| + QUEUE               |
+------------------+       +-----------------------+
        |                            |
        | seeds                      | claim one
        v                            v
  IDEA_READY               +------------------------+
                           | WORKFLOW B             |
                           | CREATION               |
                           +------------------------+
                                      |
                                      v
                              Draft
                                      |
                                      v
                              Editorial Review
                                      |
                                      v
                              Revision
                                      |
                                      v
                              Narrative Gate
                                 |         |
                              HOLD      PASS
                                            |
                                            v
                                      COPY_LOCKED
                                            |
                                            v
                                Performance Adaptation
                                            |
                                            v
                                           TTS
                                            |
                                            +------------------+
                                            |                  |
                                            v                  v
                                      Music Select      Illustration Brief
                                            |                  |
                                            |                  v
                                            |            Image Generation
                                            |                  |
                                            |                  v
                                            |              Image QC
                                            |                  |
                                            +---------+--------+
                                                      |
                                                      v
                                                Audio Mix
                                                      |
                                                      v
                                                Remotion Render
                                                      |
                                                      v
                                                Technical QC
                                                      |
                                                      v
                                             READY_TO_PUBLISH
                                                      |
                                                      v
                                           +--------------------+
                                           | WORKFLOW C         |
                                           | PUBLISH            |
                                           +--------------------+
                                                      |
                                +---------------------+---------------------+
                                |                     |                     |
                                v                     v                     v
                           Instagram               TikTok              YouTube
                                |                     |                     |
                                +---------------------+---------------------+
                                                      |
                                                      v
                                                  PUBLISHED
```

---

# 17. Build order

Do not build the whole thing at once.

## Phase 1 — prove writing

Build:

```
Trigger
-> Load Canon
-> Claim One Seed
-> Draft
-> Editorial Review
-> Revision
-> Narrative Gate
-> Save
```

Inspect real output.

## Phase 2 — prove performance

Add:

```
Performance Adaptation
-> TTS
-> Audio Sanity Check
```

## Phase 3 — prove the visual object

Add:

```
Illustration Brief
-> Illustration Generation
-> Music Selection
-> Audio Mix
-> Remotion Render
-> Technical QC
```

## Phase 4 — prove distribution

Add the independent publish workflow.

Start with one platform.

Then add channels one by one.

## Phase 5 — turn on autopilot

Only after the system has produced a meaningful run of pieces that would genuinely have been worth publishing.

Autopilot is the last switch, not the first feature.

---

# 18. Simplicity standard

The system should remain understandable by looking at the n8n canvas.

Avoid:

- multi-agent debate systems
- open-ended revision loops
- a separate agent for every micro-decision
- vector databases before there is a demonstrated need
- dynamic music generation for every post
- image models rendering typography
- regeneration of already-approved upstream assets after downstream failures
- one giant workflow that owns ideation, production, publication, analytics, and recovery

The architecture is intentionally boring.

That is a feature.

The creative object should be interesting.

The machinery should be dependable.
