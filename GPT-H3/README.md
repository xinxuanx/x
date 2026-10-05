# GPT-H3

## MiniMax-H3 + Qwen-Image-2.1 + ComfyUI automated 3D short-video system

This project is an agent-independent production architecture for creating stylized 3D animated shorts locally with ComfyUI.

The three upstream systems have deliberately different roles:

| Upstream | Role in GPT-H3 |
|---|---|
| MiniMax H3 | audiovisual shot generator; multimodal reference binding; native video + stereo audio; continuity-aware per-shot rendering |
| Qwen Image 2.1 | character / prop / environment / storyboard image factory; multi-reference editing; transparent asset generation; image-edit revision |
| ComfyUI / workflow_templates | execution graph, model loading/offload, caching, API queueing, reusable subgraphs, templates, provenance, and workflow packaging |

The production system does **not** make any of these three repositories the agent itself.

## Target architecture

```
USER IDEA
   │
   ▼
PROJECT IR / CONTEXT-IR
   │
   ├── story bible
   ├── character cards
   ├── scene cards
   ├── props
   ├── shot table
   ├── text storyboards
   └── continuity state
   │
   ▼
QWEN IMAGE 2.1 ASSET FACTORY
   │
   ├── T2I character / scene / prop concepts
   ├── multi-reference edits
   ├── 3D-style visual development
   ├── transparent RGBA assets
   └── asset revision
   │
   ▼
H3 PROMPT COMPILER
   │
   ▼
GPT-H3 / V6.5 SHOT ENGINE
   │
   ├── DaSiWa Hybrid Turbo V3 INT8
   ├── H3StreamedBlocks
   ├── Veda Sparse Attention
   ├── MotionContext continuity
   ├── H3 Video + Audio latent
   ├── normal H3 INT8 VAE
   └── V6.5 FaceRefine / Stitch / audit
   │
   ├─────────────── optional fast branch
   │
   ▼
H3 X2 Stream + X2 INT8 VAE + NVENC
   │
   ▼
SHOT QA
   │
   ▼
ASSEMBLY / BGM / FINAL QA
   │
   ▼
PUBLISHED SHORT
```

## Why this architecture

MiniMax's official H3 documentation describes H3 as a unified text/image/video/audio generation system, with H3-Base FL2VA and Ref2VA task families, native audiovisual latents, and a hosted H3-Context-IR layer. The official release says Context-IR is critical to final quality but is not part of the open-source release. GPT-H3 therefore implements a **local project IR / Context-IR-like contract** instead of depending on a proprietary agent runtime. citeturn481705view0

Qwen-Image-2.1 is used before video generation because the official model is a 7B visual-generation component designed for text-to-image and image editing, with multi-reference editing and native transparency. It is not treated as a video model. citeturn533589view2

ComfyUI is used as the execution substrate because its current architecture exposes reusable subgraphs, workflow templates, a local API, asynchronous queueing, partial graph re-execution, model offloading and quantized model support. The official workflow_templates repository separately maintains full templates and reusable subgraph blueprints. citeturn533589view0turn533589view1

## 3D definition

The primary project target is **stylized 3D animation**: C4D/Octane-like materials, cinematic lighting, strong silhouettes, expressive stylized character animation, and physically readable environments.

An actual mesh / geometry production mode may be added later as an optional branch. It is not required for the primary H3 video path.

## Core production profiles

### QUALITY

- DaSiWa Hybrid Turbo V3 INT8
- H3StreamedBlocks
- Veda generated sparsity: 90%
- Veda reference sparsity: 0%
- 8 steps
- conventional H3 INT8 video VAE for the audited master
- MotionContext + FaceRefine + Stitch + V6.5 audit

### BALANCED

- same as QUALITY
- reference sparsity swept at 25 / 50 / 75 / 90%
- choose the highest value that passes identity / motion / spatial-anchor QA

### FAST

- DaSiWa V3 INT8
- Veda
- 4 steps
- optional H3 X2 Stream + X2 INT8 VAE + asynchronous NVENC
- treated as a delivery/preview branch until it passes the same QA

## Upstream-source rules

### MiniMax H3

Use the official portable `h3-prompt-writing` skill where possible. It is explicitly documented as portable to agents that can read Markdown/local files. The separate official `3d-animation-short-generator` skill contains a strong production ordering, but its runtime binding is MiniMax Hub-specific; GPT-H3 ports its production logic into the local SOP rather than depending on Hub-only tools. citeturn791176view3

### Qwen Image 2.1

Use:

- T2I for initial asset creation
- image edit for revisions
- multi-reference edit for composing locked subjects/props/scenes
- transparent RGBA output for isolated props when useful
- official prompt enhancer only as an optional prompt-compilation stage

The official repository documents up to 10 reference images for multi-subject composition and native 2K image generation. citeturn791176view0

### ComfyUI

Use native workflows / blueprints as the implementation vocabulary. The current ComfyUI source exposes native H3 and Qwen Image 2.1 nodes, and the official workflow-template repository already carries H3 continuation/multiframe/reference workflows and Qwen Image 2.1 workflows.

## Repository map

```
GPT-H3/
├── AGENTS.md
├── README.md
├── ARCHITECTURE.md
├── PIPELINE_SOP.md
├── SKILLS.md
├── ARTIFACT_CONTRACT.md
├── MODEL_ROUTING.md
├── HARDWARE.md
├── SOURCE_REGISTRY.md
├── VALIDATION.md
├── V6.5_NODE_PATCH.json
├── MODEL_MANIFEST.md
├── pipeline_manifest.json
├── skills/
│   ├── h3-prompt-writing/
│   ├── qwen-image-asset-factory/
│   ├── comfyui-graph-engineering/
│   ├── h3-v6.5-engineering/
│   ├── continuity/
│   └── qa-regression/
└── workflows/
    └── MiniMax-H3-GPT-H3-V6.5-integrated.json
```

The workflow JSON is a concrete implementation snapshot. The documents and manifests are the durable contract used by replacement agent frameworks.

## Current status

Source research: completed against current official repositories.

Workflow design: completed.

V6.5 integration JSON: generated from the uploaded workflow.

Runtime execution: **not performed in this environment**.

Hardware-specific claims for RTX 3080: must come from local validation, not transferred from other GPUs.

## Sources

- MiniMax H3: https://github.com/MiniMax-AI/MiniMax-H3
- Qwen Image 2.1: https://github.com/QwenLM/Qwen-Image-2.1
- ComfyUI: https://github.com/Comfy-Org/ComfyUI
- ComfyUI workflow templates: https://github.com/Comfy-Org/workflow_templates
