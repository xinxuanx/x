# GPT-H3 Architecture

## 1. Layer model

The system has six layers.

### L0 — Agent Harness Adapter

Any agent runtime.

Responsibilities:

- chat / task intake
- tool calls
- file system access
- scheduling / retry orchestration
- human escalation

No model-specific production logic belongs here.

### L1 — Production Skills

Portable instructions that tell an agent how to perform a domain task.

Examples:

- story planning
- asset creation
- H3 prompt compilation
- ComfyUI workflow patching
- QA
- regeneration
- benchmark logging

Skills must be usable without assuming a specific agent framework.

### L2 — Project IR / Context-IR

The durable project memory.

It contains:

- production specification
- story bible
- character identity locks
- environment locks
- prop locks
- per-shot timeline
- reference bindings
- spatial anchors
- audio/dialogue cues
- render parameters
- artifact lineage
- QA state

This layer is the local equivalent of the **role** played by H3-Context-IR in the official H3 architecture, not a copy of MiniMax's private implementation.

MiniMax explicitly says its Context-IR performs instruction parsing, cross-modal association, temporal understanding and logical reasoning before serializing structured generation context; the official implementation is hosted and not open sourced. citeturn481705view0

### L3 — Asset Factory

Qwen Image 2.1.

Purpose:

- visual development
- identity references
- scene references
- prop references
- storyboard images when visualization mode is enabled
- revisions
- transparent asset extraction

### L4 — Shot Engine

MiniMax H3 / GPT-H3 V6.5.

Purpose:

- per-shot audiovisual generation
- native audio + video
- multimodal reference binding
- first/last frame generation
- reference-to-audio-video generation
- shot-level regeneration

Optimization stack:

```
DaSiWa V3 INT8
→ V6.5 H3StreamedBlocks
→ Veda
→ Guider
→ Sampler
```

### L5 — Delivery / QA

Two paths:

**Audited master**

```
latent
→ normal H3 video/audio decode
→ MotionContext trim
→ FaceTrackCrop
→ FaceRefine
→ FaceStitch
→ tracked save
→ audit
→ assembly
```

**Fast delivery**

```
latent
→ H3 X2 Stream
→ X2 INT8 VAE
→ streamed NVENC
```

The fast branch is never automatically considered the audited master.

## 2. Data plane

The production graph operates on four data classes:

| Data | Example |
|---|---|
| Creative | brief, outline, story beats |
| Visual | character card, scene card, prop card |
| Temporal | shot table, storyboard, continuity handoff |
| Runtime | model names, node settings, seed, timings, QA |

Every downstream artifact must reference the latest approved upstream artifact IDs.

## 3. Control plane

The supervisor operates a state machine:

```
INTAKE
→ BRIEF
→ STORY_LOCKED
→ ASSET_LOCKED
→ SHOTTABLE_READY
→ SHOTTABLE_PASSED
→ STORYBOARD_READY
→ RENDER_READY
→ RENDERING
→ SHOT_QA
→ ASSEMBLY
→ FINAL_QA
→ PUBLISHED
```

Failure can move a stage to:

`FAILED → REPAIR → RERENDER → SHOT_QA`

or:

`FAILED → SUPERSEDED`

Never jump directly from failed upstream artifacts to final assembly.

## 4. Modular ComfyUI subgraphs

The recommended future packaging is:

```
SG-01 Project Input
SG-02 Qwen Asset Factory
SG-03 Character / Scene Reference Pack
SG-04 H3 Prompt Packager
SG-05 GPT-H3 V6.5 Shot Engine
SG-06 MotionContext Continuity
SG-07 FaceRefine / Audit
SG-08 H3 X2 Fast Output
SG-09 Shot QA
SG-10 Assembly
```

Each subgraph should expose only the narrowest inputs and outputs required.

This follows the same reusable-subgraph / workflow-template direction used by current ComfyUI and its official workflow_templates repository. citeturn533589view0turn533589view1

## 5. Agent execution

Preferred:

```
Project IR
   ↓
template selector
   ↓
workflow patcher
   ↓
ComfyUI /prompt
   ↓
WebSocket progress
   ↓
history/output
   ↓
QA
   ↓
artifact registry
```

This lets the same project operate under different agent harnesses.

## 6. Model residency policy for a 10 GB GPU

For the current RTX 3080-class target:

```
Qwen stage
  → unload
H3 stage
  → unload
QA / lightweight analysis
  → continue
```

Do not design the system around simultaneous residency of Qwen Image 2.1, its prompt-enhancer model, and the H3 stack.

ComfyUI's current design provides model offloading and asynchronous weight streaming specifically to support constrained local hardware. citeturn533589view0
