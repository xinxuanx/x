# GPT-H3 Agent Contract

This directory is the portable execution contract for the MiniMax-H3 / Qwen-Image / ComfyUI automated 3D short-video system.

## Non-negotiable rule

The agent framework is replaceable. The production contract is not.

An implementation may use Claude Code, Codex, Gemini, Grok, Hermes, Muse, a custom Python agent, LangGraph, AutoGen, or another harness. The implementation must consume and produce the artifacts and states defined here.

## Read order

1. `README.md`
2. `ARCHITECTURE.md`
3. `PIPELINE_SOP.md`
4. `SKILLS.md`
5. `ARTIFACT_CONTRACT.md`
6. `MODEL_ROUTING.md`
7. `HARDWARE.md`
8. `workflows/MiniMax-H3-GPT-H3-V6.5-integrated.json`

For H3 prompt construction also read the H3 prompt-writing source references listed in `SOURCE_REGISTRY.md`.

## System invariant

The system must always preserve:

- story-first planning
- locked character and scene references
- a mandatory standardized shot table
- a shot-table self-check gate
- one authoritative text storyboard document
- per-shot generation
- latest-approved-artifact discipline
- single-shot regeneration
- QA before assembly
- reversible optimization layers
- reproducible workflow configuration
- model/version/file-name provenance

## Portable agent responsibilities

The supervising agent must:

1. interpret the user's rough idea;
2. build/update the project state;
3. produce or route the required skills;
4. materialize artifacts on disk / in the project's artifact store;
5. generate or patch ComfyUI workflow JSON;
6. queue execution through the ComfyUI API;
7. monitor execution;
8. run QA gates;
9. regenerate only the failing shot/artifact;
10. assemble only from the latest approved assets;
11. write benchmark and provenance records back to the repository.

## Never

- change model + LoRA + sampler + sparse settings + VAE in one experiment;
- silently mix old and regenerated assets;
- use the X2 Stream output as the audited V6.5 face-refined master without explicit validation;
- stack Veda with ComfyUI native Model Sparse Attention / BlockSparseAttention on H3;
- use X2 Detail VAE on H3 conditioning;
- treat MiniMax's hosted Context-IR as if it were open-source local code;
- assume a 4-step Veda configuration is trained/validated merely because an 8-step predictor exists;
- treat public benchmark timings as RTX 3080 timings;
- save model weights into this Git repository;
- overwrite a working workflow without a git checkpoint;
- infer a node ID that was not found in the current workflow.

## Required artifact lineage

Every expensive artifact must have:

`artifact_id`
`parent_artifact_id`
`stage`
`version`
`status`
`source`
`created_by`
`parameters`
`checksum` when practical

The latest approved artifact is the only input allowed to the next stage.

## ComfyUI execution contract

Preferred automation path:

`agent -> project IR -> workflow JSON/API payload -> POST /prompt -> WebSocket /ws -> history/output inspection`

Do not use browser clicking as the primary automation mechanism.

ComfyUI's local API and WebSocket execution path are part of the runtime adapter; the agent logic above them is framework-independent.

## Change protocol

A workflow change is accepted only when:

1. JSON remains syntactically valid;
2. referenced nodes exist;
3. model and custom-node requirements are recorded;
4. the workflow can be opened by the target ComfyUI build;
5. the intended optimization can be bypassed;
6. the validation record identifies what changed.

## Status vocabulary

`draft`
`locked`
`rendering`
`passed`
`failed`
`superseded`
`archived`

Do not use vague statuses such as "done-ish", "probably good", or "latest" without an explicit artifact ID.
