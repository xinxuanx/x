---
name: artifact-lineage
description: Prevent stale assets from mixing with regenerated downstream outputs.
---

# Artifact Lineage

Every artifact records its direct parent.

Example:
char:hero:003
-> shot:04:plan:008
-> shot:04:prompt:008
-> shot:04:video:008

Each logical role has one current approved version.

Invalidation:
- new character -> dependent plans/prompts/videos stale
- new scene -> dependent plans/prompts/videos stale
- new shot prompt -> that shot video + assembly stale
- new shot video -> assembly stale
- new BGM -> final composite stale

Regenerate the smallest necessary scope.
