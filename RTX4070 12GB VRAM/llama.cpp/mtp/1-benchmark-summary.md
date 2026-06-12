# llama-bench Summary Comparison

**Date:** 2026-06-12
**Build:** 715b86a36 (9559)
**GPU:** NVIDIA GeForce RTX 4070 (11868 MiB)
**CPU:** AMD Ryzen 5 5600X 6-Core

## Configuration Overview

| Parameter | Run 1 | Run 2 | Run 3 |
|-----------|-------|-------|-------|
| Model | Qwopus3.6-35B-A3B-v1-MTP | Qwopus3.6-35B-A3B-v1-MTP | Qwopus3.6-27B-Coder-MTP |
| Quant | Q6_K | Q5_K_M | Q4_K_M |
| Size | 27.19 GiB | 23.60 GiB | 15.65 GiB |
| Params | 35.51 B | 35.51 B | 27.32 B |
| ngl | 99 | 99 | 25 |
| n_cpu_moe | 36 | 36 | — |
| Threads | 12 | 12 | 12 |
| ctk/ctv | q8_0 / q8_0 | q8_0 / q8_0 | q4_0 / q4_0 |
| Flash attn | on | on | on |
| mmap | off | off | off |

## Benchmarks

| Test | Q6_K (35B) | Q5_K_M (35B) | Q4_K_M (27B) |
|------|-----------:|-------------:|-------------------:|
| pp2048 (t/s) | **373.02** | **424.97** | **322.51** |
| tg (t/s) | **18.39** (tg512) | **20.30** (tg512) | **2.13** (tg128) |

> **Note:** Run 3 (Coder) only offloaded 25 layers to GPU vs 99 for runs 1-2. Most layers run on CPU, making generation ~9× slower. The Coder model is also 27B vs 35B, with smaller KV cache types (q4_0 vs q8_0). Prompt processing is bottlenecked by GPU memory bandwidth for the offloaded layers.

## Visual Summary

```
pp2048 (t/s):      Q6_K       ████████████████████████████████████████░░░░ 373
                   Q5_K_M     ████████████████████████████████████████████ 425
                   Q4_K       █████████████████████████████████░░░░░░░░░░ 323

tg (t/s):          Q6_K       ████████████████████░░░░░░░░░░░░░░░░░░░░░░ 18.4
                   Q5_K_M     ████████████████████████░░░░░░░░░░░░░░░░░░ 20.3
                   Q4_K       ██░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  2.1
```
