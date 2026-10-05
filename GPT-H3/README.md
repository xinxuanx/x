# GPT-H3

## MiniMax-H3 + Qwen-Image-2.1 + ComfyUI automated 3D short-video system

This project is an agent-independent production architecture for creating stylized 3D animated shorts locally with ComfyUI.

The three upstream systems have deliberately different roles:

| Upstream | Role in GPT-H3 |
|---|---|
| MiniMax H3 | audiovisual shot generator; multimodal reference binding; native video + stereo audio; per-shot rendering |
| Qwen Image 2.1 | character / prop / environment / storyboard image factory; multi-reference editing; transparent asset generation; image revision |
| ComfyUI / workflow_templates | executable graph; model loading/offload; caching; API queueing; reusable subgraphs; templates; provenance |

The production system does **not** make any of these repositories the agent itself.

## Target architecture

```
USER IDEA
   |
   v
PROJECT IR / CONTEXT-IR CONTRACT
   |
   +-- story bible
   +-- character cards
   +-- scene cards
   +-- prop cards
   +-- shot table
   +-- text storyboards
   +-- continuity state
   |
   v
QWEN IMAGE 2.1 ASSET FACTORY
   |
   +-- T2I character / scene / prop concepts
   +-- multi-reference edits
   +-- 3D-style visual development
   +-- transparent RGBA assets
   +-- asset revision
   |
   v
H3 PROMPT COMPILER
   |
   v
GPT-H3 / V6.5 SHOT ENGINE
   |
   +-- DaSiWa Hybrid Turbo V3 INT8
   +-- H3StreamedBlocks
   +-- Veda Sparse Attention
   +-- MotionContext continuity
   +-- H3 video + audio latent
   +-- normal H3 INT8 VAE
   +-- V6.5 FaceRefine / Stitch / audit
   |
   +---------------- optional fast branch
   |
   v
H3 X2 Stream + X2 INT8 VAE + NVENC
   |
   v
SHOT QA
   |
   v
ASSEMBLY / BGM / FINAL QA
   |
   v
PUBLISHED SHORT
```

## Architecture principle

MiniMax's official H3 release defines a three-module system around H3-Context-IR, H3-Base and H3-Regenerate-2K. Context-IR handles instruction parsing, cross-modal association, temporal understanding and reasoning before generation; the hosted Context-IR implementation is not open sourced. GPT-H3 therefore defines a local **Project IR** that plays the same architectural role without claiming to reproduce MiniMax's private implementation.

Qwen Image 2.1 is the visual asset layer, not the video layer. Its official release combines text-to-image and image editing, supports multi-reference editing and native transparency, and is natively supported by current ComfyUI templates.

ComfyUI is the execution substrate. The current project uses its workflow JSON, reusable subgraph/blueprint model, local API, WebSocket monitoring, asynchronous queueing, partial graph execution, offloading and quantized model support.

## 3D definition

The current target is **stylized 3D animation**, not a requirement that every shot originate from a polygonal mesh.

Default visual target:
- stylized feature-film 3D
- C4D / Octane-like material language
- cinematic lighting
- readable silhouettes
- expressive squash-and-stretch animation
- persistent environment landmarks

### Optional geometry mode

A later extension may add:

```
Qwen Image 2.1 visual development
    ->
3D reconstruction / mesh generation
    ->
turntable / render references
    ->
H3 shot generation
```

This is deliberately optional. The current GPT-H3 production path does not claim that Qwen Image 2.1 or H3 itself is a polygonal 3D asset generator.

## Core production profiles

### QUALITY

- DaSiWa Hybrid Turbo V3 INT8
- H3StreamedBlocks
- Veda generated sparsity: 90%
- Veda reference sparsity: 0%
- 8 steps
- conventional H3 INT8 video VAE for audited master
- MotionContext + FaceRefine + Stitch + V6.5 audit

### BALANCED

Same as QUALITY, then sweep reference sparsity at 25 / 50 / 75 / 90% and choose the highest value that passes identity, motion and spatial-anchor QA.

### FAST

- DaSiWa V3 INT8
- Veda
- 4 steps
- optional H3 X2 Stream + X2 INT8 VAE + asynchronous NVENC
- remains a delivery/preview branch until it passes the same QA

## Upstream rules

### MiniMax H3

Use the official portable `h3-prompt-writing` skill where possible.

Port the logic of the official `3d-animation-short-generator` skill:

story -> assets -> shot table -> self-check -> storyboard -> video -> assembly -> final QA

Do not port MiniMax Hub-only runtime assumptions into the core system.

### Qwen Image 2.1

Use:
- T2I for initial character / scene / prop development
- image edit for repair and controlled revisions
- multi-reference edit for combining locked visual sources
- RGBA output for isolated assets when useful
- official PE-T2I / PE-I2I models as an optional prompt compiler

### ComfyUI

Use:
- native H3 nodes
- native Qwen Image 2.1 nodes
- workflow templates as full application graphs
- subgraphs/blueprints as reusable production modules
- API + WebSocket for automation

## Repository map

```
GPT-H3/
|-- AGENTS.md
|-- README.md
|-- ARCHITECTURE.md
|-- PIPELINE_SOP.md
|-- SKILLS.md
|-- ARTIFACT_CONTRACT.md
|-- ADAPTER_CONTRACT.md
|-- PROJECT_IR.schema.json
|-- MODEL_ROUTING.md
|-- HARDWARE.md
|-- SOURCE_REGISTRY.md
|-- VALIDATION.md
|-- V6.5_NODE_PATCH.json
|-- MODEL_MANIFEST.md
|-- pipeline_manifest.json
|-- skills/
|   |-- 3d-animation-short-generator/SKILL.md
|   |-- h3-prompt-writing/SKILL.md
|   |-- qwen-image-asset-factory/SKILL.md
|   |-- qwen-image-prompt-compiler/SKILL.md
|   |-- comfyui-graph-engineering/SKILL.md
|   |-- h3-v6.5-engineering/SKILL.md
|   |-- continuity/SKILL.md
|   |-- qa-regression/SKILL.md
|   |-- resource-mutex/SKILL.md
|   |-- artifact-lineage/SKILL.md
|-- workflows/
|   |-- MiniMax-H3-GPT-H3-V6.5-integrated.json
|   +-- ...
```

## How a replacement agent uses the repository

The new agent does not need prior conversation memory.

Read:

1. `AGENTS.md`
2. `ARCHITECTURE.md`
3. `PIPELINE_SOP.md`
4. `SKILLS.md`
5. `ADAPTER_CONTRACT.md`
6. `PROJECT_IR.schema.json`
7. `MODEL_ROUTING.md`
8. `HARDWARE.md`
9. current workflow + `V6.5_NODE_PATCH.json`

Then implement the same contracts using the new agent's own tool syntax.

## Current status

Source research: complete against the current official repositories.

Workflow design: complete.

V6.5 integration workflow: generated from the uploaded V6.5 workflow.

Portable skill contracts: added.

Project IR schema: added.

Runtime execution: **not performed in this environment**.

RTX 3080 / Veda timings: must come from local validation rather than transferred benchmarks.

No multi-GB model weights are stored in Git.

## Sources

- MiniMax H3: https://github.com/MiniMax-AI/MiniMax-H3
- Qwen Image 2.1: https://github.com/QwenLM/Qwen-Image-2.1
- ComfyUI: https://github.com/Comfy-Org/ComfyUI
- ComfyUI workflow templates: https://github.com/Comfy-Org/workflow_templates
- Veda Sparse Attention: https://github.com/veda-sparse/Veda-on-ComfyUI
- H3 X2 Stream: https://github.com/sepiablue-ai/ComfyUI-H3-X2-Stream
