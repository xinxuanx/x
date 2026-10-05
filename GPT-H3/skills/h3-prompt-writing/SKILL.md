---
name: h3-prompt-writing
description: Compile Project IR and shot context into a MiniMax H3-native prompt package.
---

# H3 Prompt Writing

Modes:
T2VA, I2VA, FL2VA, L2VA, Ref2VA.

Base structure:
1. integrated_multimodal_description
2. overall_soundscape
3. non_diegetic_music

Ref2VA structure:
1. subject_definitions
2. summary
3. retention_analysis
4. detailed_description
5. overall_soundscape
6. non_diegetic_music

Reference rules:
- Keep <Picture N>, <Video N>, <Audio N> labels stable and ordered.
- State where each reference enters the timeline.
- Do not invent labels absent from the reference manifest.
- Strip storyboard-only labels before rendering.

Prompt timing must match shot duration.

Source:
https://github.com/MiniMax-AI/MiniMax-H3/tree/main/skills/h3-prompt-writing
