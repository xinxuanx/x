---
name: qwen-image-prompt-compiler
description: Optional Qwen Image 2.1 prompt-enhancement stage for T2I and image editing.
---

# Qwen Image Prompt Compiler

T2I:
input rough prompt + target ratio
output rewritten prompt + wh_ratio

Edit:
input edit instruction + 1..N images
output rewritten prompt + wh_ratio or ratio_follow + parse_ok

Rules:
- never silently drop images
- preserve reference order
- pass ratio metadata downstream
- reject parse_ok=false

Source:
https://github.com/QwenLM/Qwen-Image-2.1/tree/main/prompt_rewrite
