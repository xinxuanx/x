# GPT-H3 Artifact Contract

## Artifact IDs

Use stable human-readable IDs:

`project`
`story`
`char:<name>:<version>`
`scene:<name>:<version>`
`prop:<name>:<version>`
`shot:<number>:plan:<version>`
`shot:<number>:prompt:<version>`
`shot:<number>:video:<version>`
`episode:assembly:<version>`
`episode:final:<version>`

## Required fields

```json
{
  "artifact_id": "shot:03:video:004",
  "parent_artifact_id": "shot:03:prompt:004",
  "stage": "shot_video",
  "version": 4,
  "status": "passed",
  "source": {
    "workflow": "GPT-H3/V6.5",
    "git_commit": "..."
  },
  "parameters": {
    "model": "...",
    "steps": 8,
    "seed": 42,
    "width": 1344,
    "height": 768,
    "frames": 124
  },
  "checks": [
    "identity",
    "spatial_anchor",
    "audio_sync"
  ],
  "checksum": "sha256:..."
}
```

## Latest-artifact rule

Each stage stores a `current` pointer.

Example:

```
character:hero:007 = current
character:hero:006 = superseded
```

Downstream prompts must resolve `current`, not a historical ID.

## Versioning rule

Never overwrite the meaning of an artifact ID.

Changes create new versions.

## Approval rule

Only `status=passed` artifacts can cross a gate.

## Regeneration rule

Regenerate at the smallest failed scope.

Examples:

- bad eyebrow → image edit of character card
- bad hand in S03 → S03 regeneration
- scene drift in S04 → S04 reference/prompt repair
- BGM problem → BGM artifact only

## Storage rule

Store metadata and lightweight previews in Git.

Do not store multi-GB weights or large generated-video datasets in Git.
