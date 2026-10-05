# GPT-H3 validation matrix

## Rules

Use the same:

- prompt
- width / height
- frame length
- reference assets
- seed
- ComfyUI build
- GPU

Only change one optimization layer at a time.

Record:

- model load time (separate, do not compare as generation time)
- sampling time
- VAE decode time
- video encode / save time
- total wall time
- VRAM peak
- CPU RAM peak
- output file size
- visible defects

## A0 — Dense baseline

DaSiWa V3 INT8  
+ V6.5 H3StreamedBlocks  
+ conventional H3 INT8 video VAE  
+ existing MotionContext / FaceRefine / audit

Veda: bypassed.

Purpose: establish the new main-model baseline.

## A1 — Veda reference-safe

A0 + Veda

- generated_sparsity = 90%
- reference_sparsity = 0%
- full_attention_layers = empty
- full_attention_steps = empty
- 8 steps

Purpose: verify sparse attention does not create visible identity/reference drift.

## A2 — Reference sparsity sweep

Keep everything from A1 and sweep:

- 25%
- 50%
- 75%
- 90%

Use the same seed set.

Selection rule: choose the highest reference sparsity that passes identity, motion and fine-detail inspection.

## A3 — X2 Stream

A2 + X2 INT8 VAE + H3X2StreamSave.

Keep the conventional VAE/FaceRefine branch active.

The X2 file is treated as a separate delivery/preview result, not as a replacement for the audited master until the exact downstream requirements are satisfied.

## A4 — 4-step Fast profile

Compare:

- 8-step baseline
- 4-step fast

Do not simultaneously change Veda sparsity and scheduler while making the step comparison.

DaSiWa V3 is a Turbo checkpoint, but the Veda predictor's documented training configuration is 8-step Turbo, so 4-step + Veda is an extrapolation that must be validated.

## Quality checklist

Inspect at minimum:

- face identity
- eyes and mouth
- hair strands
- hands / fingers
- thin props
- hard edges / text
- motion continuity
- reference-background consistency
- audio continuity
- first/last-frame transition
- MotionContext carry-over across shots

## Rollback order

When a defect appears, roll back the most recent optimization layer first:

1. X2 Stream
2. Veda
3. sparse reference attention
4. 4-step profile
5. model/LoRA change

Do not change five things at once.
