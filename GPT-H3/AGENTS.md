# GPT-H3 Agent Context

## Mission

Implement and maintain the MiniMax-H3 V6.5 pipeline:

Dasiwa Hybrid Turbo v3 INT8 + H3 FFN chunking + Veda Sparse Attention + H3 X2 Stream.

The agent must preserve the existing V6.5 SOP, provide rollback paths, and validate each optimization independently.

## Non-negotiable rules

### Veda
- Put Veda after the model and all LoRA/model patches, immediately before Guider/Sampler.
- Do not combine Veda with ComfyUI native Model Sparse Attention.
- Treat released Veda predictor training settings as the validated reference; Dasiwa 4-step and arbitrary V6.5 resolutions require local A/B validation.

### X2 Stream
- X2 Detail VAE is final video decode only.
- Ordinary H3 video VAE remains on the conditioning branch.
- Audio VAE remains independent.
- X2 conversion requires the official original FP16 Detail v1 source.
- Keep a conventional decode bypass for debugging and downstream IMAGE processing.

### FFN
- Start with chunks=0 and seq_threshold=4096.
- Apply FFN patch before Veda.

## Canonical graph

Model Loader
 -> Dasiwa H3 INT8
 -> existing LoRA/model patches
 -> H3 Chunk FeedForward
 -> Veda Sparse Attention
 -> Guider
 -> Sampler
 -> H3 latent
 -> H3 X2 Decode + Stream Save

Conditioning:
image/reference -> normal H3 video VAE -> native H3 conditioning

Audio:
existing H3 audio branch -> AUDIO -> X2 Stream Save

## Default preset

main_model: DasiwaMinimaxH3_dasiwaHybridTurboV3_int8.safetensors
steps: 4
ffn_chunks: 0
ffn_seq_threshold: 4096
veda_generated_sparsity: 90
veda_reference_sparsity_r2va: 0
veda_reference_sparsity_fl2va: 25-50
x2_output: MiniMax-H3-X2-Detail-v1-decoder-int8-convrot.safetensors
fps: 24
nvenc_gpu: 0
cq: 18
fp16_accumulation: true

## Failure handling

Quality issue:
1. bypass Veda
2. same-seed comparison
3. if fixed, reduce sparsity or restore full reference attention
4. for R2VA identity problems restore reference_sparsity=0

Decode issue:
1. bypass X2 Stream
2. use normal H3 video VAE
3. verify frame grid
4. verify X2 source and converted file
5. verify CUDA/PyTorch/comfy-kitchen/PyAV/NVENC

Never change model + sampler + sparse parameters + VAE at the same time.

## Source hierarchy

Implementation compatibility:
1. Veda-on-ComfyUI
2. ComfyUI-H3-X2-Stream

Benchmark evidence:
3. fourthplace43 Jev sparse comparison
4. fourthplace43 H3 VAE quality

Model guidance:
5. DaSiWa MiniMax H3 model page

Benchmark numbers must never be generalized across GPUs without validation.

## Required experiment record

For every optimization record:
- model filename
- source URL
- ComfyUI version/commit
- GPU
- resolution
- frame count
- steps
- seed
- sampling time
- decode time
- total time
- visual QA
- audio QA
