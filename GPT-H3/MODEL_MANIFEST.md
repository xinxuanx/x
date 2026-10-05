# GPT-H3 model manifest

| Component | Filename | V6.5 role | Status |
|---|---|---|---|
| Main diffusion | `DasiwaMinimaxH3_dasiwaHybridTurboV3_int8.safetensors` | Primary H3 model | selected; runtime not tested here |
| Legacy Turbo | `minimax_h3_turbo_v4_step600_ema_DasiwaREF2VAHybridV1_0_curveproj1025_compat_v001.safetensors` | Old V1 compatibility Turbo | disabled |
| LMS | `minimax_h3_lms_v1.0_r64.safetensors` | Guide-latent sharpening | disabled in primary path |
| Veda predictor | `minimax_h3_t2va_veda_8nfe_600step_preview_fp8.safetensors` | Sparse attention predictor | selected |
| H3 video VAE | `minimax_h3_video_vae_int8_convrot.safetensors` | Conditioning + normal IMAGE decode | retained |
| H3 audio VAE | `minimax_h3_audio_vae_fp32.safetensors` | Audio decode | retained |
| X2 source VAE | `MiniMax-H3-X2-Detail-v1.safetensors` | X2 conversion source | required for preparation |
| X2 prepared VAE | `MiniMax-H3-X2-Detail-v1-decoder-int8-convrot.safetensors` | Optional final X2 decode | optional |

## Suggested local directories

```
ComfyUI/
├── models/
│   ├── diffusion_models/
│   │   └── DasiwaMinimaxH3_dasiwaHybridTurboV3_int8.safetensors
│   ├── loras/
│   │   └── H3/minimax_h3_lms_v1.0_r64.safetensors
│   ├── text_encoders/
│   │   └── qwen3vl_32b_minimax_h3_int8_convrot.safetensors
│   ├── vae/
│   │   ├── minimax_h3_video_vae_int8_convrot.safetensors
│   │   ├── minimax_h3_audio_vae_fp32.safetensors
│   │   └── h3_x2_stream/
│   │       └── MiniMax-H3-X2-Detail-v1-decoder-int8-convrot.safetensors
│   └── veda/
│       └── minimax_h3_t2va_veda_8nfe_600step_preview_fp8.safetensors
```

The exact path for the DaSiWa checkpoint can be changed according to the local ComfyUI model configuration.

## Source references

Veda predictor:
https://huggingface.co/Veda-Sparse/Minimax-H3-T2VA-Veda-8NFE-600Step-Preview/resolve/9a1fd3a41b4a754a7886e64e82edbddf599fd1bd/minimax_h3_t2va_veda_8nfe_600step_preview_fp8.safetensors

H3 X2 Stream:
https://github.com/sepiablue-ai/ComfyUI-H3-X2-Stream

DaSiWa reference:
https://civitai.com/models/2877206/dasiwa-minimax-h3

No multi-GB weights are stored in GitHub.
