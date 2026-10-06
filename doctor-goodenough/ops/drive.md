# Google Drive Asset Storage

Google Drive stores generated binary assets.

Suggested structure:

```
Doctor Goodenough/
  pieces/
    dg_000001/
      voice.wav
      illustration_raw.png
      illustration.png
      final.mp4
      cover.jpg
```

Use `piece_id` as the folder name and stable filenames inside it.

The Google Sheet stores Drive file IDs or usable URLs. n8n should resolve the files from those IDs rather than searching by filename when possible.

Do not store generated binary media in GitHub.

Music tracks may live in a shared `Doctor Goodenough/music/` Drive folder once approved. GitHub stores the creative description and the stable file IDs/links, not the audio binaries.
