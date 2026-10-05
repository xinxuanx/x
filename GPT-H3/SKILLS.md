# GPT-H3 Skills Catalog

Each skill is an instruction contract, not a specific agent product.

## Skill matrix

| Skill | Source | Portable | Purpose |
|---|---|---:|---|
| h3-prompt-writing | MiniMax H3 official | Yes | Convert structured shot context into H3-native prompts |
| 3d-animation-short-generator | MiniMax H3 official | Logic: yes; original runtime: no | Story-first 3D short production ordering |
| qwen-image-asset-factory | GPT-H3 local | Yes | Generate/edit/lock visual assets |
| qwen-image-prompt-compiler | Qwen Image 2.1 official | Yes | Optional T2I / edit prompt rewriting |
| comfyui-graph-engineering | GPT-H3 local + ComfyUI | Yes | Build, patch and validate workflow graphs |
| h3-v6.5-engineering | GPT-H3 local | Yes | Apply DaSiWa / StreamedBlocks / Veda / X2 rules |
| continuity | GPT-H3 local | Yes | Maintain temporal, spatial and identity state |
| qa-regression | GPT-H3 local | Yes | A/B testing, defect taxonomy, rollback |
| resource-mutex | GPT-H3 local | Yes | Protect a 10 GB GPU from incompatible model residency |
| artifact-lineage | GPT-H3 local | Yes | Prevent stale asset mixing |

## 1. h3-prompt-writing

Read:

- H3 mode selection
- base prompt guide
- Ref2VA prompt guide

Inputs:

- shot table row
- character IDs
- scene ID
- reference assets
- duration
- audio cues

Outputs:

- H3 prompt package

Hard rules:

- exact reference labels
- exact field order
- no storyboard-only labels in the render prompt
- timeline must fit requested duration

## 2. 3d-animation-short-generator

Port the following logic from the official skill:

```
idea
→ brief
→ story
→ character cards
→ scene cards
→ six-column shot table
→ self-check
→ text storyboard
→ optional pencil board
→ model selection
→ per-shot video
→ assembly
→ BGM
→ final review
```

Do not port Hub-only tool calls as requirements.

## 3. qwen-image-asset-factory

Use Qwen Image 2.1 for:

- character design
- scene design
- prop design
- visual style tests
- multi-reference compositions
- editing / repair
- transparent isolated assets

Qwen Image 2.1 officially supports text-to-image, image editing, multi-reference editing up to 10 images and native transparency. citeturn533589view2

Preferred asset sequence:

```
T2I concept
→ select
→ edit against reference
→ identity lock
→ save canonical
```

Never use a regenerated character image without updating downstream bindings.

## 4. qwen-image-prompt-compiler

The official Qwen Image 2.1 repository ships two prompt-enhancer checkpoints:

- PE-T2I
- PE-I2I

The code returns structured records containing rewritten prompts and canvas-ratio metadata. The edit task supports 1..N source images. citeturn791176view0

GPT-H3 usage:

```
raw creative request
→ optional PE
→ structured prompt
→ Qwen Image 2.1
```

Use it when prompt expansion is worth the extra model invocation. Do not require it for every image.

## 5. comfyui-graph-engineering

Rules:

- prefer native nodes
- isolate custom nodes
- create reusable subgraphs
- make every optimization bypassable
- use model metadata when packaging templates
- record custom-node minimum versions
- use API execution for automation

ComfyUI officially supports reusable subgraphs, workflow templates, local API, asynchronous queueing, partial graph re-execution, model offloading and quantized models. citeturn533589view0

The official workflow_templates repository treats workflow JSON and subgraph blueprints as separately reusable artifacts. citeturn533589view1

## 6. h3-v6.5-engineering

Primary production topology:

```
DaSiWa V3 INT8
→ H3StreamedBlocks
→ Veda
→ Guider
→ Sampler
```

Do not:

- stack Veda + native SLA
- stack legacy Turbo V4 with V3 unless explicitly tested
- use X2 VAE for conditioning
- replace the audited master branch with X2 output automatically

## 7. continuity

Canonical state:

- current shot
- previous shot terminal pose
- next shot opening target
- landmark positions
- character presence
- object state
- light direction
- audio bridge
- camera orientation

The state is stored in the project IR, not hidden only inside one agent conversation.

## 8. qa-regression

Every optimization is an A/B layer.

A/B record:

- build
- workflow commit
- model checksum
- prompt checksum
- seed
- resolution
- frames
- steps
- sampling time
- VAE time
- encode time
- peak VRAM
- defects

## 9. resource-mutex

Rule for 3080 10 GB:

```
Qwen asset job
  exclusive GPU lease
      ↓ release
H3 job
  exclusive GPU lease
      ↓ release
QA / assembly
```

Parallel workers must not both assume ownership of the same GPU.

## 10. artifact-lineage

A regenerated character invalidates:

```
future shot-table rows
→ storyboard
→ video
→ assembly
```

unless they explicitly use a new approved asset.

A regenerated shot invalidates:

```
assembly
→ final composite
```

and nothing else.

This prevents stale references from silently reappearing.
