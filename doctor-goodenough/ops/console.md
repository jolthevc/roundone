# Simple HTML Console

The Doctor Goodenough console should remain a thin control surface, not a product platform.

A static HTML/JS page is enough. It may be hosted with GitHub Pages or another simple static host.

It talks to n8n webhooks. n8n reads and writes the Google Sheet and Drive.

## Home

Show:
- IDEA_READY count
- APPROVAL_PENDING count
- HOLD count
- READY_TO_PUBLISH count
- recently published pieces
- whether creation and publishing are enabled

## Queue

A simple table:
- piece ID
- status
- title or seed summary
- territory
- register
- created time
- failed stage / hold reason

Filters by status are enough.

## Piece view

Show:
- seed
- draft 1
- draft 2 if one exists
- editor decision
- final copy
- audio player when available
- illustration preview
- final video preview
- caption
- publication state

Actions:
- approve copy
- edit final copy
- choose the other draft
- HOLD
- archive
- trigger salvage with an operator note
- retry failed stage
- approve final media
- publish now
- pause publication for this piece
- choose music track 1 / 2 / 3 manually

## Global controls

- creation on/off
- publishing on/off
- review mode
- trigger ideation
- create next piece

That is the entire initial console.

Do not add authentication systems, analytics dashboards, user roles, notifications, or complex settings until the console actually needs them. If the static page is public-hosted, protect action webhooks appropriately rather than exposing write URLs in client code.
