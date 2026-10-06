# Google Sheets State

Use one Google Sheet as the queue and production ledger.

## Tab 1 — Pieces

One row equals one piece from seed through publication.

Recommended columns:

### Identity
- `piece_id`
- `status`
- `created_at`
- `updated_at`
- `canon_sha`
- `hold_reason`

### Seed
- `seed_json`
- `territory`
- `register`
- `time_orientation`
- `stance_hint`

### Writing
- `draft_1_json`
- `draft_2_json`
- `editor_json`
- `title_final`
- `final_copy`
- `stance_final`
- `form_final`
- `copy_locked`

### Audio
- `performance_script`
- `voice_drive_url`
- `voice_duration_s`
- `music_track`

### Visual
- `illustration_subject`
- `illustration_drive_url`
- `caption_line`
- `render_drive_url`
- `cover_drive_url`

### Publication
- `publish_after`
- `instagram_status`
- `instagram_post_id`
- `instagram_url`
- `tiktok_status`
- `tiktok_post_id`
- `youtube_status`
- `youtube_post_id`

### Operations
- `failed_stage`
- `error_message`
- `retry_count`
- `operator_note`

Do not add columns merely because they might someday be useful.

## Tab 2 — Config

A tiny key/value table is enough:

- `creation_enabled`
- `publishing_enabled`
- `review_mode` = copy | final | off
- `idea_low_watermark`
- `max_ready_buffer`
- `music_next_track`
- `operator_note`

`publishing_enabled = false` is the kill switch.

## Claiming

Because Creation runs with concurrency = 1, claiming can stay simple:

1. Find the oldest or selected `IDEA_READY` row.
2. Immediately update it to `CREATING`.
3. Continue the run.

No database lock machinery is needed at this scale.

## Resume behavior

Before any expensive stage, check whether its output column already exists.

If `voice_drive_url` exists, do not rerun TTS unless the operator explicitly cleared or retried it.

If `illustration_drive_url` exists, do not regenerate the illustration.

If `render_drive_url` exists, do not rerender.

This gives us practical resumability without an event platform.
