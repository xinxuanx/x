---
name: qwen-image-asset-factory
description: Build and revise canonical character, scene, prop and storyboard visual assets.
---

# Qwen Image Asset Factory

Use for character references, scene/environment cards, prop cards, 3D visual development, multi-reference composition, identity-preserving edits, transparent RGBA props and storyboard visualization.

Pipeline:
raw brief -> optional prompt enhancement -> Qwen Image 2.1 -> asset QA -> canonical version

For multi-image edits, preserve input order and explicitly identify each image's role.

Record artifact_id, role, name, source images, prompt, model, seed, resolution, status and checksum when practical.

Source:
https://github.com/QwenLM/Qwen-Image-2.1
