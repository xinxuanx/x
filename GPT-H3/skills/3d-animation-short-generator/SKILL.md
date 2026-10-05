---
name: 3d-animation-short-generator
description: Portable production ordering for a complete stylized 3D animated short.
---

# 3D Animation Short Generator

Mandatory order:

1. Intake / production spec lock
2. Project brief
3. Story outline and gate
4. Character cards
5. Scene cards
6. Standard six-column shot table
7. Shot-table self-check gate
8. One authoritative text storyboard
9. Optional pencil storyboard
10. Per-shot model selection
11. Per-shot video generation
12. Shot QA
13. Assembly
14. BGM / audio finishing
15. Final QA

Never skip the shot table or its self-check.

Authority:
- Character card = identity authority.
- Scene card = environment/landmark authority.
- Text storyboard = timing, camera, action, spatial anchors and continuity authority.
- Pencil storyboard = human visual review only.

Required shot-table fields:
- shot id / duration
- continuity handoff
- spatial + identity reference anchors
- hook type
- per-second shot description
- audio / dialogue track

Per-second instructions cover action/pose/expression, camera, spatial position, audio cue and handoff.

Hard gates:
- shot <= 15 s
- bounded important-character count
- spatial anchors inherited across shots
- complete timecode coverage
- no continuity contradiction
- every shot has a hook classification

Regenerate the smallest failed scope.

Source logic:
https://github.com/MiniMax-AI/MiniMax-H3/tree/main/skills/3d-animation-short-generator

MiniMax Hub-specific tool calls are intentionally excluded.
