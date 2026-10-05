---
name: resource-mutex
description: Serialize heavyweight local model stages on constrained GPUs.
---

# Resource Mutex

RTX 3080 10 GB target:

Lease A = Qwen Image 2.1 + optional prompt enhancer.
Lease B = MiniMax H3 / DaSiWa + H3 encoder + VAE + Veda.

Schedule:
acquire(QWEN) -> assets -> release(QWEN)
-> acquire(H3) -> shots -> release(H3)

Never assume simultaneous residency is safe.
