# GPT-H3 Source Registry

This file separates source-derived facts from GPT-H3 engineering decisions.

## S1 — MiniMax H3 official

Repository:
https://github.com/MiniMax-AI/MiniMax-H3

Source-derived facts:

- H3 is multimodal across text, images, video and audio.
- H3 generates video with native stereo audio.
- H3 Base supports FL2VA and Ref2VA task families.
- Ref2VA supports up to 9 images, 3 videos and 3 audio clips, with 12 mixed files total.
- H3-Context-IR is a hosted preprocessing/orchestration layer and is described as critical to output quality.
- H3-Context-IR is not included in the open-source release.
- H3-Base produces 768p-class generation; official H3-Regenerate-2K is an in-context regeneration system and is not open sourced.
- H3 uses separate visual and audio VAEs.
- H3 exposes a 33B dense single-stream transformer architecture.
- H3 uses MM-RoPE with temporal + two spatial dimensions.

Citations:
- README lines 216-296 in the current public repository.
- H3 prompt skill and 3D animation skill.

GPT-H3 decisions derived from S1:

- build a local Project IR rather than depend on hosted Context-IR;
- split long-form stories into <=15s shots;
- use Ref2VA as the default reference-heavy local task;
- keep X2 Stream separate from official H3-Regenerate-2K semantics.

## S2 — Qwen Image 2.1 official

Repository:
https://github.com/QwenLM/Qwen-Image-2.1

Source-derived facts:

- Qwen Image 2.1 has a 7B visual generation component.
- It supports T2I and image editing.
- It supports multi-reference editing with up to 10 reference images.
- It supports native transparency / RGBA.
- It supports native 2K output.
- Official prompt enhancer checkpoints exist for T2I and image-editing tasks.
- The prompt enhancer edit profile takes text plus 1..N images and emits stable structured output.

GPT-H3 decisions:

- Qwen is the asset factory, not the video generator.
- Character / scene / prop cards are canonical visual references.
- Multi-reference editing is used to compose or repair locked assets.
- PE is optional, not mandatory on every asset.

## S3 — ComfyUI official

Repository:
https://github.com/Comfy-Org/ComfyUI

Source-derived facts:

- node graph architecture
- local API
- reusable subgraphs
- workflow templates / App Mode
- asynchronous queueing
- partial graph re-execution
- VRAM/RAM management and model offloading
- quantized model support
- native H3 support
- native Qwen Image 2.1 support

GPT-H3 decisions:

- automation uses API + WebSocket rather than browser clicking;
- production logic lives outside workflow-specific UI state;
- reusable subgraphs are the unit of workflow modularization.

## S4 — Comfy-Org workflow_templates

Repository:
https://github.com/Comfy-Org/workflow_templates

Source-derived facts:

- official standalone workflow templates live under `templates/`;
- reusable subgraph blueprints live under `blueprints/`;
- template repositories carry model metadata and validation / packaging conventions;
- current bundles include MiniMax H3 T2V/R2V, continuation and multiframe-reference workflows, and Qwen Image 2.1 templates.

GPT-H3 decisions:

- keep the main project workflow JSON versioned locally;
- later extract reusable GPT-H3 subgraphs into blueprint-compatible artifacts;
- include exact model metadata in reusable template variants.

## S5 — GPT-H3 local V6.5 fusion

Source:
uploaded `MiniMax-H3一键无限短剧V6.5(2).json`

Source-derived facts:

- existing 7-stage production layout
- existing MotionContext / FaceRefine / Stitch / audit chain
- existing H3StreamedBlocks
- existing conventional H3 VAE and audio VAE
- legacy Turbo V4 and LMS components in the source workflow

GPT-H3 decisions:

- switch primary checkpoint to DaSiWa Hybrid Turbo V3 INT8
- disable legacy Turbo V4 compatibility layer
- disable LMS on the primary native reference path
- insert Veda
- use 8-step Simple baseline
- keep existing audited V6.5 decode/postprocess chain
- add X2 Stream only as an optional side branch

## S6 — Veda-on-ComfyUI

Repository:
https://github.com/veda-sparse/Veda-on-ComfyUI

Source-derived facts:

- learned sparse attention for H3
- model-wire node
- default generated/reference sparsity fields
- released predictor
- 8-step Turbo training reference
- current public README states SM80–SM89 code path ready but RTX 30 not hardware-validated

GPT-H3 decisions:

- use Veda as the single sparse-attention layer
- start reference sparsity at 0% for identity safety
- test 25/50/75/90 before production selection

## S7 — H3 X2 Stream

Repository:
https://github.com/sepiablue-ai/ComfyUI-H3-X2-Stream

Source-derived facts:

- X2 INT8 VAE preparation node
- H3X2StreamSave
- H3X2ChunkFeedForward
- streamed decode
- asynchronous NVENC output
- current example workflow is built as an FL2VA 4-step pipeline

GPT-H3 decisions:

- use the X2 stack as a delivery/preview branch;
- do not replace the V6.5 audit path automatically;
- do not stack H3StreamedBlocks and X2 FFN chunking until a controlled benchmark proves a benefit.

## S8 — fourthplace43 H3 optimization reports

Sources:

- https://fourthplace43.com/labs/jev-sparse-comparison/en/
- https://fourthplace43.com/labs/h3-vae-quality/en/

Source-derived facts are kept as benchmark evidence.

GPT-H3 treats their measured GPU timings as **relative evidence only**, never as RTX 3080 predictions.

## Evidence levels

Use these labels in future notes:

- `SOURCE_FACT` — directly stated by upstream project
- `SOURCE_BENCHMARK` — upstream measured result
- `LOCAL_TEST` — tested on the target machine
- `ENGINEERING_DECISION` — selected for this project
- `HYPOTHESIS` — not yet tested
