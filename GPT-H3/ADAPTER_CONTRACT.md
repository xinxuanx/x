# GPT-H3 Adapter Contract

## Agent adapter

Conceptual operations:
load_project()
plan()
create_or_update_asset()
compile_h3_prompt()
patch_workflow()
queue_comfy_job()
monitor_comfy_job()
inspect_output()
run_qa()
regenerate_scope()
assemble()
record_artifact()

Names may differ across frameworks; semantics may not.

## ComfyUI adapter

Input: workflow JSON, parameter override map, runtime endpoint.
Output: prompt_id, execution state, output artifact paths/metadata, errors.

Preferred transport:
HTTP /prompt + WebSocket /ws + history/output inspection.

## Separation

Agent decides WHAT.
Skills decide HOW the production task should be performed.
Project IR decides WHAT is authoritative now.
ComfyUI adapter decides HOW to submit/monitor the graph.
Artifact registry decides WHICH version is current.

## Replacement test

A new agent harness is compatible when it can read the skills, read/write Project IR, generate valid ComfyUI workflow JSON, call the ComfyUI adapter, and obey artifact lineage + QA gates.

No agent-specific conversational memory should be required.
