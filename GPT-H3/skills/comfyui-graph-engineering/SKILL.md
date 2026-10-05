---
name: comfyui-graph-engineering
description: Build and patch ComfyUI graphs with reproducible modular execution.
---

# ComfyUI Graph Engineering

Preferred units:
1. Native nodes
2. Reusable subgraphs / blueprints
3. Custom nodes when native support is insufficient
4. Explicit external-service adapters

Rules:
- Find exact node IDs in the current workflow.
- Change one optimization layer per experiment.
- Keep bypass paths.
- Preserve data types and links.
- Record custom-node versions and model metadata.
- Never invent node IDs.

Automation:
POST /prompt -> WebSocket /ws -> execution state -> history/output inspection

Browser clicking is not the primary automation protocol.

Sources:
https://github.com/Comfy-Org/ComfyUI
https://github.com/Comfy-Org/workflow_templates
