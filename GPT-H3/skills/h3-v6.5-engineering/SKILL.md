---
name: h3-v6.5-engineering
description: Apply the GPT-H3 fusion policy to the MiniMax-H3 V6.5 workflow.
---

# H3 V6.5 Engineering

Primary:
DaSiWa Hybrid Turbo V3 INT8
-> H3StreamedBlocks
-> Veda Sparse Attention
-> Guider
-> sampler

Initial profile:
- generated sparsity: 90%
- reference sparsity: 0%
- 8 steps
- euler / simple
- legacy Turbo V4 compatibility LoRA: off
- LMS primary path: off

Preserve MotionContext, normal H3 video VAE, H3 audio VAE, FaceRefine, FaceStitch, tracked save/audit/merge.

Optional fast path:
latent -> X2 INT8 VAE -> H3X2StreamSave -> NVENC

Do not stack Veda with native H3 Model Sparse Attention / BlockSparseAttention.
Do not stack X2 FFN chunking on H3StreamedBlocks before controlled benchmarking.

Validation:
A0 dense -> A1 Veda 90/0 -> A2 reference sweep -> A3 X2 -> A4 4-step
