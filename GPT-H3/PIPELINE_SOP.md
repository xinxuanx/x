# GPT-H3 Production SOP

## Stage 0 — Intake

Input:

- rough story idea
- target aspect ratio
- total duration
- dialogue requirement
- desired 3D style
- delivery format

Output:

`project_brief.json`

No image or video generation before the production specification is locked.

## Stage 1 — Story architecture

Produce:

- premise
- protagonist want / need / flaw
- world rules
- beat sheet
- emotional anchor
- dialogue beats
- continuity constraints

Pass gate:

- protagonist acts
- conflict escalates
- coincidence does not solve the story
- emotional anchor is paid off
- dialogue changes relationships rather than only explaining theme

## Stage 2 — Character / scene / prop bible

Create visual anchors.

Character card:

- exact name / ID
- 3/4 view
- front / side / back where useful
- expression set
- clothes
- signature props
- visual lock list
- non-changeable features

Scene card:

- environment only
- landmarks
- lighting baseline
- spatial coordinates / relative positions
- recurring props

Prop card:

- exact shape
- material
- color
- markings
- scale relative to characters

Qwen Image 2.1 is the default asset factory.

Use multi-reference editing when combining locked people, props and environments. The official model supports up to 10 reference images and is specifically designed for identity preservation in image editing. citeturn533589view2

## Stage 3 — Asset QA

Check:

- identity
- outfit consistency
- prop count
- scene layout
- lighting
- 3D style consistency
- unwanted text / logos
- transparent-background correctness where used

Only approved assets may enter the shot table.

## Stage 4 — Standard shot table

Mandatory six-column table:

| Shot & duration | Continuity handoff | Reference anchors | Hook | Per-second shot description | Audio / dialogue |
|---|---|---|---|---|---|

Per-second instructions must cover:

- body / pose / expression
- camera move
- spatial position
- audio cue
- handoff to the next second / shot

Maximum shot duration: 15 s.

This mirrors the ordering and hard gates in the official MiniMax H3 3D animation skill, which requires a standardized shot table before storyboard/video generation. citeturn791176view3

## Stage 5 — Shot-table self-check

Reject the table when:

- hook metadata is missing
- shot > 15 s
- too many important characters
- spatial anchors drift without explanation
- timecode coverage is incomplete
- continuity contradicts the previous shot

Only after passing this gate may the storyboard stage proceed.

## Stage 6 — Storyboards

Default:

- one authoritative text storyboard document
- one section per shot

Optional:

- multi-panel pencil storyboard for human visual checking

The text storyboard remains authoritative even when a pencil storyboard exists.

Before video rendering, remove storyboard-only labels such as:

`[char:...]`
`[scene:...]`
`[shot:...]`
`[dur:...]`
`[hook:...]`

The official H3 3D skill explicitly treats the text storyboard and locked character/scene cards as the authoritative sources for per-shot rendering and requires storyboard markers to be stripped before final rendering. citeturn791176view3

## Stage 7 — H3 prompt compilation

For T2VA / I2VA / FL2VA:

`integrated_multimodal_description`
→ `overall_soundscape`
→ `non_diegetic_music`

For Ref2VA:

`subject_definitions`
→ `summary`
→ `retention_analysis`
→ `detailed_description`
→ `overall_soundscape`
→ `non_diegetic_music`

Reference labels must be stable and ordered.

The official portable H3 prompt skill documents the two prompt structures and explicitly requires exact field ordering and reference-label consistency. citeturn791176view3

## Stage 8 — GPT-H3 V6.5 render

Primary chain:

```
DaSiWa Hybrid Turbo V3 INT8
→ H3StreamedBlocks
→ Veda Sparse Attention
→ Guider
→ Euler / Simple
→ 8 steps
```

Initial Veda settings:

- generated: 90%
- reference: 0%
- dense layers: none
- dense steps: none

Reason for 8-step starting point:

The released Veda predictor documented in its current ComfyUI project was trained around an 8-step Turbo setup. Applying it to 4-step DaSiWa is an extrapolation and must be experimentally validated.

## Stage 9 — Continuity

Use the existing V6.5 MotionContext system as the primary continuation state.

Keep:

- latent continuity
- spatial landmark continuity
- character exit state
- lighting baseline
- audio bridge
- camera handoff

Where a specific shot needs an exact keyframe, native H3 multi-frame / Add Guide style workflows may be used as an optional auxiliary mechanism. Current ComfyUI workflow templates include H3 continuation and multiframe-reference examples.

## Stage 10 — Shot QA

Evaluate:

- face identity
- eyes
- mouth
- hair
- hands
- props
- scene landmarks
- lighting
- camera
- motion continuity
- audio sync
- first/last-frame handoff

Failure ladder:

1. strengthen the exact reference-anchor block;
2. shorten / split the shot;
3. switch model only for the failing shot;
4. escalate if three retries fail.

The official H3 3D skill uses the same local-shot fallback philosophy: strengthen anchors, split long shots, then switch model only when necessary. citeturn791176view3

## Stage 11 — Fast X2 branch

Use only after the shot has a stable latent result.

```
latent
→ X2 INT8 VAE
→ H3X2StreamSave
→ NVENC
```

Do not remove:

``normal H3 decode → FaceRefine → Stitch → audit
```

when the audited master is required.

## Stage 12 — Assembly

Assembly input must be:

- only passed shots
- exact latest artifact IDs
- final approved BGM
- approved subtitles / SRT if enabled

No regeneration may silently reintroduce an older shot.

## Stage 13 — Final QA

Check:

- no storyboard artifacts
- no internal labels
- no missing shot
- audio continuity
- BGM level
- dialogue intelligibility
- no bad transitions
- correct frame rate
- expected resolution
- final filename / manifest

The official H3 3D skill includes the same final checks, especially removal of storyboard artifacts and double-binding labels. citeturn791176view3
