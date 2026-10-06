# Runbook

## A piece is creatively weak

Set `status = HOLD` and record a short reason.

Default action is archive. Alternatives are:
- manually edit and approve
- choose the other preserved draft
- trigger `salvage` once with an operator note
- return the seed to the queue once

Do not automatically keep regenerating until a model approves itself.

## TTS fails

Retry the TTS stage only. If the voice file already exists and is acceptable, preserve it.

If the performance script changed words from the locked copy, fix the performance stage; never edit the copy downstream.

## Illustration fails

Retry illustration only. Do not regenerate writing or audio.

## Render fails

Retry render only. Preserve locked copy, voice, music choice, and illustration.

## Publishing fails

Retry only the failed platform. Preserve any successful platform post IDs so a retry cannot intentionally repost there.

## Global pause

Set `publishing_enabled = false` in the Config sheet.

If needed, also set `creation_enabled = false`.

## Canon change

Merge the GitHub change, then run a small writing smoke test in review mode before trusting the new canon on autopilot.

Every piece should record the Git commit SHA used for its writing run.

## Takedown

Manually remove the live post from enabled platforms, record the reason in the row, and set `ARCHIVED`.

## When to add more infrastructure

Only consider a real database, event log, background queue, or richer admin application when the simple Google/n8n system is creating a demonstrated operational problem.
