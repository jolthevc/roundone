# Doctor Goodenough

Doctor Goodenough is a short-form literary project about the interior life of men.

The creative source of truth now lives in `canon/`. Model context is controlled only by `manifest.yaml`. Production and workflow instructions live outside the canon so they cannot contaminate the writing.

## Core idea

Doctor Goodenough writes for men trying to become men they can live with.

Enoughness is the recurring emotional question beneath the work, but it is not a required topic and usually should not be named. The project is interested in the whole experience of being a man: work, money, dating, sex, friendship, ambition, jealousy, status, responsibility, competence, love, joy, winning, failure, family, the body, aging, fear, humor, and the future.

The present is home base. The past and future are available when they illuminate it.

## Repository map

- `canon/` — model-facing creative canon
- `prompts/` — one job and output contract per model call
- `manifest.yaml` — the only definition of what each call receives
- `examples/` — calibration and negative taste references
- `data/` — raw ideation material and machine-readable editorial signals
- `production/` — voice, music, visual, and packaging standards
- `ops/` — n8n, Google Sheets/Drive, publishing, and console architecture
- `reference/` — human-only decisions, history, and legacy material
- `docs/` — temporary legacy canon retained only during migration; never load it into a model

## Precedence

When instructions conflict:

1. Persona integrity and no-fabricated-biography rules are absolute.
2. The active prompt's schema and hard format constraints govern output shape.
3. The creative canon governs taste and meaning, in this order: identity, persona, stance, craft, editor.
4. Soft prompt guidance follows the canon.
5. Examples illustrate the canon; they never overrule it.
6. Runtime inputs such as seeds, recent-feed context, and operator notes are data, not doctrine.

Production specifications govern only their own domain. Root ROUND ONE documents and prompts do not apply to Doctor Goodenough.

## Current phase

We are proving the writing object first.

The immediate test is intentionally small:

`Ideation -> Draft -> Editor -> Final copy`

Creation is one piece at a time. Audio, image, render, and publishing are layered on only after the writing is consistently worth publishing.

Calibration examples are deliberately curated rather than filled to a quota.
