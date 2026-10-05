# GPT-H3 Hardware Policy

## Target machine

Primary development target:

- RTX 3080 10 GB
- Ryzen 9 9900X
- 32 GB RAM
- Windows/Linux compatible deployment

The architecture must also work on larger NVIDIA GPUs.

## GPU lease

Only one heavyweight model family owns the GPU at a time.

### Lease A — Qwen Image

```
Qwen Image 2.1
+ Qwen Image 2.1 text encoder
+ optional PE model
```

### Lease B — H3

```
DaSiWa V3
+ H3 text encoder
+ H3 VAE(s)
+ Veda
+ continuity patches
```

Release all possible Lease-A residency before starting Lease-B.

## Memory strategy

Prefer:

- ComfyUI offloading
- quantized model variants
- CPU/RAM cache where supported
- sequential execution
- low-resolution validation before production
- separate output/decode timings

Do not design around simultaneous model residency.

ComfyUI explicitly supports model offloading, asynchronous execution and quantized models for constrained local hardware. citeturn533589view0

## RTX 3080 Veda rule

The current Veda project documents an SM80–SM89 code path but states that RTX 30 hardware was not yet validated by that project. Therefore:

**Veda on RTX 3080 is a validation target, not a transferred benchmark.**

Record:

- actual kernel selected
- compile status
- actual attention fallback
- timing
- visual A/B result

## Qwen Image note

The current official Qwen Image 2.1 repository describes a 7B visual-generation component and native 2K support, while current ComfyUI examples also provide INT8-compatible weights and a Qwen Image 2.1 cache node. Use the smallest tested local configuration that preserves the required asset quality.

## H3 note

Official H3 is a 33B dense single-stream transformer with a multimodal text/video/audio architecture and separate visual/audio VAEs. Local quantized ComfyUI variants should be treated as engineering deployments of the same task family, not as identical numerical implementations.

## 10 GB operating mode

Recommended workflow:

```
1. Load project IR
2. Run Qwen asset stage
3. Save assets
4. Fully release Qwen residency
5. Run H3 shots one at a time
6. Save latent/video
7. Run decode / QA
8. Release H3
9. Assembly
```

## Runtime flags

The exact ComfyUI launch flags depend on the installed build.

Do not hard-code a flag merely because it worked on another machine.

The fourthplace43 H3 VAE report and the GPT-H3 validation docs may recommend a flag such as `--fast fp16_accumulation`; the agent must still verify it against the local build before applying it.
