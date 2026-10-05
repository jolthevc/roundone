# 11 - Metadata Standard

Metadata exists to help future systems understand creative intent.

This document defines vocabulary, not database architecture.

## Principles

Metadata should:
- describe the finished Round
- support programming
- support search
- support personalization
- support QC
- remain interpretable by humans

Avoid metadata that exists only because it might be useful someday.

## Core fields

### Identity
- title
- slug
- version
- status

### Narrative
- core_realization
- archetype
- story_vehicle
- factuality
- source_requirements
- callback_object

### Themes
Potential tags:
- discipline
- persistence
- failure
- time
- regret
- mortality
- courage
- fear
- ambition
- identity
- relationships
- patience
- craft
- uncertainty
- rejection
- recovery
- gratitude
- responsibility
- second_chances
- invisible_progress

### Emotional
- emotional_register
- starting_state
- ending_state
- intensity
- darkness_level
- activation_level

### Language
- profanity_level
- directness
- aggression
- introspection

### Performance
- primary_persona
- secondary_persona
- voice_id
- performance_intensity
- performance_notes

### Music
- music_family
- music_master_id
- music_intensity
- mix_notes

### Visual
- visual_object
- visual_accent
- visual_prompt_version
- cover_asset_id

### Programming
- repeat_cooldown
- theme_cooldown
- intensity_band
- sequence_notes

### Master
- runtime_seconds
- master_asset_id
- approved_at
- approver
- qc_version

## Working scales

### Intensity
1 = almost contemplative
5 = steady pressure
10 = maximum ROUND ONE intensity

### Profanity
0 = none
1 = light
2 = moderate
3 = prominent but controlled

### Darkness
1 = warm / open
5 = serious / reflective
10 = emotionally very dark

### Activation
1 = primarily reflective
5 = renewed resolve
10 = immediate physical urge to act

## Example metadata

~~~json
{
  "title": "The Ninety-Ninth Hit",
  "archetype": "parable",
  "themes": ["persistence", "invisible_progress"],
  "emotional_register": "pressure",
  "intensity": 8,
  "darkness_level": 6,
  "activation_level": 9,
  "profanity_level": 2,
  "primary_persona": "corner",
  "music_family": "pressure",
  "visual_object": "split_stone",
  "ending_type": "callback"
}
~~~

This example is illustrative, not a final schema.

## Controlled vocabulary

Prefer defined tags over endless free-text synonyms.

If persistence exists, do not also create:
- perseverance
- keep_going
- never_quit
- staying_power

unless there is a real programming distinction.

## Versioning

Creative metadata will evolve.

When taxonomy changes materially:
- document the change
- preserve old meaning where possible
- avoid silently changing tag definitions

The future programming engine will be only as good as the consistency of the metadata.
