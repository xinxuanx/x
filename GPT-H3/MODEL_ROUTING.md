# GPT-H3 Model Routing

## Model roles

### Qwen Image 2.1

Use for:

- visual development
- character cards
- scene cards
- prop cards
- 3D style frames
- multi-reference composition
- image repair

Do not use as the shot-video generator.

### Qwen Image 2.1 Prompt Enhancer

Use as an optional prompt compiler before asset generation/editing.

The official prompt-enhancer package has separate T2I and edit profiles and stable JSONL output records. citeturn791176view0

### MiniMax H3

Use for:

- T2VA
- I2VA
- FL2VA
- Ref2VA

Default for the project:

**Ref2VA for reference-heavy shots**.

Use FL2VA for shots where exact first/last frame interpolation is the better constraint.

H3 officially supports up to 9 reference images, 3 reference videos and 3 audio clips, with a 12-file total mixed-input ceiling for Ref2VA. citeturn481705view0

### DaSiWa Hybrid Turbo V3 INT8

Use as the main local V6.5 H3 diffusion checkpoint.

Legacy V1/Turbo V4 compatibility assets stay disabled unless a controlled experiment specifically targets them.

### Veda

Use as the primary learned sparse-attention layer.

Starting:

`generated=90%`
`reference=0%`

Sweep reference sparsity only after the identity-safe baseline passes.

### H3 X2 Stream

Use as:

- fast preview
- fast delivery
- streaming decode / encode

Do not assume it replaces the V6.5 audited IMAGE-space post-processing branch.

## Model selection rules

```
Needs image asset?
→ Qwen Image 2.1

Needs audiovisual shot?
→ MiniMax H3 / GPT-H3

Needs reference-heavy shot?
→ H3 Ref2VA

Needs exact first/last keyframes?
→ H3 FL2VA

Needs fast latent delivery?
→ X2 Stream branch

Needs audit-quality face correction?
→ conventional V6.5 decode + FaceRefine
```

## Model-switch escalation

Model switching is shot-local whenever possible.

Do not change the global model because one shot failed.

The official 3D short-video skill uses shot-local fallback rather than forcing a global model replacement. citeturn791176view3
