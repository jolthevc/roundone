# Doctor Goodenough — Workflow Architecture

## Principle

The creative object should be sophisticated. The machinery should be simple.

Doctor Goodenough does not need an internal SaaS platform, event-sourced database, agent swarm, or elaborate recommendation engine.

The practical stack is:

- **GitHub** — canon, prompts, examples, production standards
- **Google Sheets** — content queue and state
- **Google Drive** — generated audio, illustrations, renders, and covers
- **n8n** — orchestration
- **Claude / model APIs** — ideation, drafting, editing, packaging
- **TTS provider** — fixed Doctor Goodenough narrator
- **image API** — illustration only
- **Remotion + FFmpeg** — deterministic page, audio mix, and final MP4
- **simple static HTML console** — queue, preview, approve, hold, retry, and publish controls

That is enough.

## Why Google instead of a database

Creation is intentionally one piece at a time.

The n8n Creation workflow should run with concurrency limited to one. With serialized creation, Google Sheets is a perfectly reasonable state store for the current scale and avoids building infrastructure before it is needed.

If the project later reaches a point where concurrent workers, large volumes, or stronger transactional guarantees are genuinely necessary, the state layer can be replaced without changing the creative system.

Do not build for that hypothetical now.

## Workflow A — Ideation

Purpose: keep a healthy queue of seeds.

Flow:

`Schedule/Manual -> Load manifest + ideation context -> Read recent Sheet rows -> Sample moments -> Ideation model -> Basic duplicate/format checks -> Append IDEA_READY rows`

Ideation may create a batch. It does not draft finished pieces.

The model sees recent summaries and simple distribution counts only to notice obvious repetition. There is no recommender or balancing engine.

## Workflow B — Create One

Purpose: take exactly one seed through finished media.

**One run = one piece.**

Flow:

`Claim one IDEA_READY row -> Draft -> Checks -> Editor -> optional one Redraft -> Editor -> Final copy -> [review stop if enabled] -> Performance -> TTS -> Packaging -> Illustration -> choose one of 3 music tracks -> Render -> READY_TO_PUBLISH`

Important rules:

- Creation concurrency is one.
- The row is marked `CREATING` immediately when claimed.
- Original drafts are preserved in their own columns.
- The editor can cut and lightly edit; it does not rewrite.
- At most one fresh redraft is allowed.
- Weak creative work goes to `HOLD`.
- Once `final_copy` is approved, downstream stages never change it.
- If an artifact URL already exists, that stage normally skips rather than regenerating it.
- A failed illustration does not rerun TTS. A failed render does not rerun the writing.
- Music is one of three fixed canonical beds, chosen by simple rotation with optional manual override.

## Workflow C — Publish

Purpose: publish finished pieces without coupling social API reliability to creation.

Flow:

`Schedule -> Find next READY_TO_PUBLISH row -> mark PUBLISHING -> publish enabled channels -> write returned IDs/URLs -> PUBLISHED`

Platform uploads are tracked separately in Sheet columns. If Instagram succeeds and another platform fails, only the failed platform is retried.

Start with one platform. Add others after the first one works reliably.

## Errors

Do not build a complex Error workflow at the beginning.

Use n8n's normal Retry on Fail for transient API errors. If a stage still fails, write the error into the Sheet and set the piece to `HOLD`.

The console exposes a **Retry failed stage** action. Because each major artifact has its own URL/status column, retrying simply resumes at the first missing or failed output.

A shared error workflow can be added later if real operations justify it.

## Statuses

Keep the state machine small:

- `IDEA_READY`
- `CREATING`
- `APPROVAL_PENDING`
- `COPY_LOCKED`
- `READY_TO_PUBLISH`
- `PUBLISHING`
- `PUBLISHED`
- `HOLD`
- `ARCHIVED`

No separate ASSETS_BUILDING state. The presence of asset URLs tells us what has been built.

No generic ERROR state. A terminal problem is a HOLD with a reason.

## Review modes

During calibration, stop after final literary copy at `APPROVAL_PENDING`.

Later, optionally stop after the final MP4.

Autopilot uses the same workflow with review stops disabled.

Do not maintain separate creative pipelines for review and autopilot.

## What remains human-approved

Automation creates inside the world. It does not redesign the world.

Keep these stable:
- creative canon
- calibration examples
- narrator voice
- three music tracks
- visual template
- fonts and colors
- illustration style references
- publishing cadence

## Build order

1. Prove `Ideation -> Draft -> Editor -> Final copy`.
2. Add performance and TTS.
3. Add illustration, three-track music, and deterministic render.
4. Add one-platform publishing.
5. Add the simple console.
6. Turn on autopilot only after the output has earned it.

Autopilot is the last switch, not the first feature.
