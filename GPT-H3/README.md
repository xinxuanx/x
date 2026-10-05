
# GPT-H3

MiniMax-H3 / ComfyUI V6.5 integration project.

Goal:
Dasiwa Hybrid Turbo v3 INT8 + H3 FFN chunking + Veda Sparse Attention + H3 X2 Stream,
while keeping the existing V6.5 SOP, rollback path and A/B validation.

## 1. Recommended architecture

Prompt / image / reference
    |
    v
H3 Text Encoder
    |
    v
Dasiwa Hybrid Turbo v3 INT8
    |
    v
H3 Chunk FeedForward
    chunks = 0
    seq_threshold = 4096
    |
    v
Veda Sparse Attention (MiniMax H3)
    generated_sparsity = 90%
    reference_sparsity = 0% for first R2VA quality baseline
    |
    v
BasicGuider / Sampler
    |
    v
H3 video latent
    |
    +--> existing H3 audio branch
    |
    v
H3 X2 Detail v1 INT8 VAE
    |
    v
H3 X2 Decode + Stream Save
    |
    v
H.264 NVENC MP4 + optional AAC

Conditioning side:
image / first-frame / reference
    -> normal H3 video VAE
    -> native H3 conditioning node

Important:
The ordinary H3 video VAE remains on the conditioning branch.
The X2 VAE is only the final video decode VAE.

## 2. Primary model

Main checkpoint:
DasiwaMinimaxH3_dasiwaHybridTurboV3_int8.safetensors

The DaSiWa author describes this family as Hybrid, compatible with REF2VA and FL2VA, and the Turbo variant as a 4- or 8-step distilled model.

Recommended starting point:
4 steps for production speed.
8 steps as the quality / complex-motion fallback.

Author-published Turbo guidance:
sampler family: euler/simple or LCM/simple
video shift: 6-12
audio shift: 3-5
steps: 4 or 8

Do not assume those ranges are universally optimal for every V6.5 workflow. Keep the V6.5 sampler path as the baseline and change one variable at a time.

Source:
https://civitai.com/models/2877206/dasiwa-minimax-h3

## 3. Veda Sparse Attention

Veda is a learned sparse-attention implementation shipped as a ComfyUI MODEL-wire node for native MiniMax-H3 T2VA, FL2VA and R2VA.

Placement:
after the model and all LoRA/model patches,
last before Guider/Sampler.

Recommended node chain:
Dasiwa model
 -> existing LoRA(s)
 -> H3 Chunk FeedForward
 -> Veda Sparse Attention
 -> Guider
 -> Sampler

Do NOT combine Veda with ComfyUI's native Model Sparse Attention node.
Veda's README explicitly says that the native H3 sparse node replaces the attention blocks, preventing Veda from being called.

Released predictor:
models/veda/minimax_h3_t2va_veda_8nfe_600step_preview_fp8.safetensors

Released predictor training reference:
1344x768, 768x1344, 768x768, 1024x768
5 / 10 / 14 s
8-step Turbo LoRA

Therefore:
Veda on Dasiwa 4-step, R2VA/FL2VA and arbitrary V6.5 resolutions is an extrapolation case.
It should be validated locally rather than treated as benchmark-proven.

Recommended Veda profiles:

Speed profile:
generated_sparsity = 90%
reference_sparsity = 75-90%
full_attention_layers = empty
full_attention_steps = empty

Identity-safe profile:
generated_sparsity = 90%
reference_sparsity = 0%
full_attention_layers = empty
full_attention_steps = empty

For R2VA, start with reference_sparsity 0%.
Then test 25%, 50%, 75%, 90% using the same seed before selecting a production value.

For FL2VA, start with reference_sparsity 25-50% and test upward.

If visual corruption appears, first bypass Veda or restore full reference attention.
Do not immediately replace the main model.

Source:
https://github.com/veda-sparse/Veda-on-ComfyUI

## 4. Jev sparse benchmark interpretation

The Jev comparison establishes the following useful engineering conclusions:

1. The major speed gain comes from reducing 20 steps to 4 with Turbo.
2. Sparse attention still helps at 4 steps.
3. In the reported 4-step setup, sparsification was about 1.96x on top of the already-distilled pipeline.
4. Reducing keep rate from roughly 10% to 5% produced only about 3-5% additional sampling improvement.
5. VAE decode and output work remain a substantial fixed cost.
6. Their later comparison found that sparse allocation itself did not produce a reliable speed difference versus an average-compute-matched uniform allocation.

Therefore this project does NOT stack another SLA implementation on top of Veda.
Jev is retained as the measurement baseline and design evidence.
Veda becomes the long-term sparse-attention layer.

Source:
https://fourthplace43.com/labs/jev-sparse-comparison/en/

## 5. H3 VAE speed layer

ComfyUI 0.36+ introduced a faster H3 VAE path.
The fourthplace43 benchmark measured:

about 5.2 s clip, 124 frames, 1152x640:
old VAE decode 14.2 s
new lighter INT8 decode 5.0 s

about 10.1 s clip, 243 frames, 1152x640:
old 28.5 s
new lighter INT8 10.0 s

about 10.1 s clip, 243 frames, 1408x800:
old 44.2 s
new lighter INT8 15.5 s

The tested lighter VAE was:
minimax_h3_video_vae_int8_convrot.safetensors

This is the ordinary H3 video VAE used for the video conditioning/generation path.
It is NOT the X2 output VAE.

Also use the ComfyUI launch option:
--fast fp16_accumulation

Only add the flag after checking the actual current ComfyUI/PyTorch environment.

Source:
https://fourthplace43.com/labs/h3-vae-quality/en/

## 6. H3 X2 Stream output layer

Repository:
https://github.com/sepiablue-ai/ComfyUI-H3-X2-Stream

This project provides:
H3 X2 Prepare INT8 VAE
H3 X2 Decode + Stream Save
H3 Chunk FeedForward (X2 Stream)

The tested public workflow uses a specific MATLOW fused model, but this does NOT mean the MATLOW model should replace Dasiwa in V6.5.
The X2 Stream functions are an output/decode layer and can be integrated after the Dasiwa generation graph, subject to model/version compatibility.

X2 VAE source:
MiniMax-H3-X2-Detail-v1.safetensors

Converted target:
MiniMax-H3-X2-Detail-v1-decoder-int8-convrot.safetensors

The public converter:
- verifies the official X2 Detail v1 source hash
- quantizes 144 decoder Linear weights with per-channel INT8 ConvRot, groupsize 256
- preserves encoder, norms, biases and final FP16 X2 projection
- writes the converted file separately
- reuses an already-compatible converted file

X2 Stream decode characteristics:
- native H3 spatial/temporal blending
- 256-pixel tile
- 64-pixel overlap
- output_buffer streaming
- asynchronous H.264 NVENC encoding
- optional AAC audio
- one video latent per execution
- H3 frame grid 5 + 17*k
- no full IMAGE batch output

Consequently:
Any node that requires IMAGE frames for post-processing must sit before the X2 Stream output branch or use a conventional decode bypass.

## 7. Final V6.5 wiring

Main MODEL path:

Model Loader
 -> Dasiwa Hybrid Turbo v3 INT8
 -> existing LoRA(s)
 -> H3 Chunk FeedForward
 -> Veda Sparse Attention
 -> BasicGuider
 -> KSampler / H3 sampler

Conditioning:
reference/first-frame image
 -> normal H3 video VAE
 -> native H3 conditioning

Output:
KSampler latent
 -> H3 X2 Decode + Stream Save
 -> NVENC MP4
 + audio from existing audio branch

Keep a bypass:
Veda bypass = dense attention
X2 bypass = conventional H3 video VAE decode

## 8. Recommended V6.5 production preset

Main model:
DasiwaMinimaxH3_dasiwaHybridTurboV3_int8.safetensors

Steps:
4

Sampler:
keep the validated V6.5 sampler first; if rebuilding from author guidance use euler/simple

Video shift:
6-12

Audio shift:
3-5

FFN:
chunks = 0
seq_threshold = 4096

Veda:
generated_sparsity = 90%
R2VA reference_sparsity = 0% initially
FL2VA reference_sparsity = 25-50% initially
full_attention_layers = empty
full_attention_steps = empty

Output VAE:
MiniMax-H3-X2-Detail-v1-decoder-int8-convrot.safetensors

Output:
H3 X2 Decode + Stream Save
fps = 24
nvenc_gpu = 0
cq = 18
fp16_accumulation = true

## 9. Validation matrix

A0 Dense:
Dasiwa + conventional H3 video VAE

A1 Veda:
Dasiwa + Veda 90% + conventional H3 video VAE

A2 Veda + FFN:
Dasiwa + FFN chunks=0 + Veda 90%

A3 Fast decode:
A2 + ordinary H3 INT8 video VAE + fp16 accumulation

A4 Final:
A3 + X2 INT8 VAE + H3 X2 Stream + async NVENC

Each test:
- same prompt
- same resolution
- same frame count
- same steps
- three different seeds
- discard first timing due model loading
- record sampling, decode and total time
- inspect face, hair, hands, thin lines, motion, reference identity and audio

## 10. RTX 3080 10 GB deployment note

The current public Veda README states the SM80-SM89 Triton code path is ready but not yet verified on RTX 30 hardware.
Do not transfer RTX 5070/5080/4070 timings directly to RTX 3080.

For a 10 GB VRAM machine, prioritize:
- one H3 task at a time
- ComfyUI offload
- no simultaneous large LLM/image model residency during H3
- one-time X2 VAE conversion
- topology test at low resolution before production resolution

## 11. Agent operating rules

Never:
- combine Veda with native H3 Model Sparse Attention
- use X2 VAE as the conditioning VAE
- replace Dasiwa with the MATLOW measurement checkpoint merely because X2 Stream README uses it
- change model, sampler, sparse settings and VAE simultaneously
- store an unverified benchmark claim as a production fact
- fabricate ComfyUI node IDs when the actual V6.5 JSON is unavailable

Always:
- preserve the existing V6.5 SOP
- make every optimization independently bypassable
- use same-seed A/B comparison
- record exact filenames, versions and parameters
- separate sampling time from VAE/output time
- rollback the most recently changed layer first

## 12. Current project limitation

The exact V6.5 workflow JSON was not available in this repository at project creation time.
Therefore this repository contains the integration architecture and agent context, not a fabricated node-by-node V6.5 JSON.

Once the real V6.5 JSON is available, the next implementation step is to map its exact node IDs and edges to the architecture above and produce a directly importable final workflow.

## Sources

Veda:
https://github.com/veda-sparse/Veda-on-ComfyUI

Jev sparse comparison:
https://fourthplace43.com/labs/jev-sparse-comparison/en/

H3 VAE benchmark:
https://fourthplace43.com/labs/h3-vae-quality/en/

H3 X2 Stream:
https://github.com/sepiablue-ai/ComfyUI-H3-X2-Stream

DaSiWa:
https://civitai.com/models/2877206/dasiwa-minimax-h3
