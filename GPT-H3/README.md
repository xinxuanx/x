# GPT-H3

MiniMax-H3 / ComfyUI V6.5 integration project.

## Current target

Integrate the uploaded **MiniMax-H3 V6.5** short-drama workflow with:

- **DaSiWa Hybrid Turbo V3 INT8**
- **Veda Sparse Attention**
- **V6.5 H3StreamedBlocks memory path**
- **V6.5 MotionContext continuity**
- **V6.5 FaceRefine / Stitch / audit path**
- **H3 X2 Stream** as an optional final-decode side branch

This project keeps the original V6.5 production path as the authoritative/rollback path.

## V6.5 node-level result

The uploaded V6.5 workflow contains 238 nodes and 318 links. The active model chain was:

`UNETLoader (DaSiWa REF2VA Hybrid V1)
→ ModelAttentionBackend
→ H3StreamedBlocks
→ old Turbo V4 compatibility LoRA
→ LMS LoRA
→ Set_Video_Mode
→ Get_Video_Mode
→ BasicGuider
→ Sampler`

The GPT-H3 revision changes this to:

`UNETLoader (DaSiWa Hybrid Turbo V3 INT8)
→ ModelAttentionBackend
→ H3StreamedBlocks
→ Veda Sparse Attention
→ Set_Video_Mode
→ Get_Video_Mode
→ BasicGuider
→ euler / simple / 8-step sampler`

### Disabled legacy layers

**Turbo V4 compatibility LoRA** is disabled. The V6.5 filename is explicitly tied to `DasiwaREF2VAHybridV1`; the selected V3 checkpoint is itself a Turbo variant, so the old V4 LoRA is not stacked.

**LMS LoRA** is disabled in the primary generation path. Its public documentation describes `minimax_h3_lms_v1.0_r64` as a guide-latent / aligned-source-video sharpening pass, while the uploaded V6.5 graph has no `MiniMaxH3AddGuide` node. It remains a backup component rather than being silently applied to native reference generation.

## Veda placement

Use Veda after all effective model/LoRA patches and immediately before the Guider/Sampler path.

Current project starting profile:

- predictor: `minimax_h3_t2va_veda_8nfe_600step_preview_fp8.safetensors`
- generated_sparsity: **90%**
- reference_sparsity: **0%**
- full_attention_layers: empty
- full_attention_steps: empty

The 0% reference setting is a conservative project choice for the reference-heavy V6.5 workflow. It is not Veda's default training value.

The Veda predictor is publicly documented as trained at 1344x768, 768x1344, 768x768 and 1024x768, at 5/10/14 s with an 8-step Turbo setup. Therefore the uploaded V6.5 1344x768 / 124-frame segment is a close size-duration match, while the exact DaSiWa hybrid task and any 4-step profile still require local A/B validation.

**Do not combine Veda with ComfyUI's native `Model Sparse Attention` / `BlockSparseAttention` on the same H3 model.**

## Sampling

The original V6.5 graph used a custom 6-step ManualSigmas node. GPT-H3 replaces that active schedule with:

- sampler: **euler**
- scheduler: **simple**
- steps: **8**
- denoise: **1.0**

This is an engineering baseline chosen to be closer to the Veda predictor's documented 8-step Turbo training distribution. It is not a claim that 8 steps is universally optimal for DaSiWa V3.

A separate **4-step Fast profile** should be benchmarked after the 8-step baseline is visually stable.

## V6.5 continuity and face pipeline

These remain in place:

`MotionContextLoadLatent
→ MiniMaxH3MotionContext_ContinuityGuard
→ sampler
→ H3FreeCache
→ normal H3 VAEDecode / VAEDecodeAudio
→ MotionContextTrim
→ FaceTrackCrop
→ FaceRefine
→ FaceStitch
→ H3V64TrackedSave
→ approval merge`

The Veda layer operates on MODEL attention and does not change the MotionContext latent-cache protocol.

The existing **normal H3 video VAE** stays in the conditioning and IMAGE-processing path:

`minimax_h3_video_vae_int8_convrot.safetensors`

The **audio VAE** stays:

`minimax_h3_audio_vae_fp32.safetensors`

## H3 X2 Stream integration

H3 X2 Stream is included as a **separate side branch**, disabled by default.

Branch:

`H3FreeCache samples
→ H3X2StreamSave(samples, X2 INT8 VAE, existing audio)`

The X2 INT8 VAE is prepared once with:

- source: `MiniMax-H3-X2-Detail-v1.safetensors`
- prepared: `MiniMax-H3-X2-Detail-v1-decoder-int8-convrot.safetensors`

### Why it is not the main V6.5 decode

`H3X2StreamSave` is a streaming LATENT→VAE→video-save node and does not provide the full IMAGE batch used by the V6.5 face-processing chain.

Therefore the safe topology is:

**Production master:** conventional decode + FaceRefine + audit.

**Fast X2:** optional latent-side stream output.

Do not replace the only V6.5 `VAEDecode` with X2 Stream.

This project does **not** claim that the X2-streamed file contains the IMAGE-space face refinement result. Achieving that would require a different post-face latent/re-encode design and must be engineered separately.

## H3StreamedBlocks vs H3 X2 FFN chunking

The uploaded V6.5 graph already uses `H3StreamedBlocks` with:

- q_chunk 16384
- kv_chunk 16384
- mlp_chunk 16384
- min_tokens 32768
- kv_store bf16 (exact)
- trim_forward true

GPT-H3 keeps this memory path.

The X2 repository also exposes `H3X2ChunkFeedForward`. It is **not stacked automatically** in the merged V6.5 path because both mechanisms chunk H3 feed-forward execution. The safe rule is to add the X2-specific FFN wrapper only after a controlled benchmark proves a benefit without conflicting with the existing stream-block patch.

## Compatibility notes

Veda currently documents ComfyUI >= 0.38.0 and a Triton sparse path covering SM80+ NVIDIA cards, but its RTX 30 path is documented as code-ready and not hardware-validated by the project.

The public H3 X2 Stream example was tested on a 0.37.0 ComfyUI revision. The two projects therefore do not share a single documented compatibility point.

**Recommended runtime policy:** use a current ComfyUI 0.38+ environment required by Veda, then explicitly validate H3 X2 Stream on that same environment. Keep the conventional VAE branch as the rollback path.

## Launch-level optimization

The fourthplace43 H3 VAE test documents the optional ComfyUI launch flag:

`--fast fp16_accumulation`

This is a runtime setting, not a workflow node. Only add it after checking the actual PyTorch / CUDA / ComfyUI environment.

## Validation policy

Do not change all optimization layers at once.

Recommended sequence:

1. **A0 Dense** — DaSiWa V3 + H3StreamedBlocks + conventional H3 VAE.
2. **A1 Veda** — A0 + Veda 90% generated / 0% reference.
3. **A2 Reference sweep** — compare reference sparsity 25/50/75/90%.
4. **A3 X2 Stream** — A2 + X2 INT8 decode side branch.
5. **A4 Fast** — compare 4 steps vs 8 steps with the same seed set.

For every A/B run keep prompt, resolution, frame count and seed fixed. Record sampling time, VAE time, output/save time and wall time separately.

## Source material

- Veda: https://github.com/veda-sparse/Veda-on-ComfyUI
- Jev sparse comparison: https://fourthplace43.com/labs/jev-sparse-comparison/en/
- H3 VAE quality report: https://fourthplace43.com/labs/h3-vae-quality/en/
- H3 X2 Stream: https://github.com/sepiablue-ai/ComfyUI-H3-X2-Stream
- DaSiWa reference page: https://civitai.com/models/2877206/dasiwa-minimax-h3

## Repository policy

Do not commit MiniMax-H3, DaSiWa, Veda predictor, X2 VAE or other multi-GB model weights into this repository.

Keep:

- workflow JSON / patch
- model manifest
- runtime requirements
- benchmark logs
- A/B decisions
- rollback rules

Runtime-tested facts must be clearly separated from model-card claims and engineering hypotheses.
