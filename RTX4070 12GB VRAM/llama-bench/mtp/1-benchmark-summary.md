# llama-bench Summary Comparison

**Date:** 2026-06-12
**Build:** 715b86a36 (9559)
**GPU:** NVIDIA GeForce RTX 4070 (11868 MiB)
**CPU:** AMD Ryzen 5 5600X 6-Core

## Configuration Overview

| Parameter | Run 1 | Run 2 | Run 3 | Run 4 | Run 5 |
|-----------|-------|-------|-------|-------|-------|
| Model | Qwopus3.6-35B-A3B-v1-MTP | Qwopus3.6-35B-A3B-v1-MTP | Qwopus3.6-27B-Coder-MTP | **Gemma-4-26B-A4B-it** | Gemma-4-26B-A4B-it |
| Quant | Q6_K | Q5_K_M | Q4_K_M | **UD-Q3_K_M** | UD-IQ4_XS |
| Size | 27.19 GiB | 23.60 GiB | 15.65 GiB | **11.84 GiB** | 12.65 GiB |
| Params | 35.51 B | 35.51 B | 27.32 B | **25.23 B (3.8B active)** | 25.23 B (3.8B active) |
| ngl | 99 | 99 | 25 | **99** | 99 |
| n_cpu_moe | 36 | 36 | — | **12** | 12 |
| Threads | 12 | 12 | 12 | **12** | 12 |
| ctk/ctv | q8_0 / q8_0 | q8_0 / q8_0 | q4_0 / q4_0 | **q8_0 / q8_0** | q8_0 / q8_0 |
| Flash attn | on | on | on | **on** | on |
| mmap | off | off | off | **off** | off |

## Benchmarks

| Test | Q6_K (35B) | Q5_K_M (35B) | Q4_K_M Coder (27B) | **Gemma-4 Q3_K_M** | Gemma-4 IQ4_XS |
|------|-----------:|-------------:|-------------------:|-------------------:|---------------:|
| pp2048 (t/s) | 373.02 | 424.97 | 322.51 | **1349.87** | 1269.75 |
| tg (t/s) | 18.39 (tg512) | 20.30 (tg512) | 2.13 (tg128) | **40.58** (tg512) | 37.09 (tg512) |

> **Note:** Run 3 (Coder) only offloaded 25 layers to GPU (no `--n-cpu-moe`). Runs 4-5 use `--n-cpu-moe 12` — expert weights stay in RAM, only 3.8B active params run on GPU. IQ4_XS is slower than Q3_K_M due to heavier dequantization in importance-matrix quants.

## Visual Summary

```
pp2048 (t/s):
Q6_K (35B)       ███████████████████████████░░░░░░░░░░░░░░░░ 373
Q5_K_M (35B)     ███████████████████████████████░░░░░░░░░░░░ 425
Q4_K_M           ███████████████████████████░░░░░░░░░░░░░░░░ 323
Gemma-4 Q3_K_M   ████████████████████████████████████████████ 1350  🏆
Gemma-4 IQ4_XS   ███████████████████████████████████████████░ 1270

tg (t/s):
Q6_K (35B)       █████████████████████░░░░░░░░░░░░░░░░░░░░░░ 18.4
Q5_K_M (35B)     ███████████████████████░░░░░░░░░░░░░░░░░░░░ 20.3
Q4_K_M           ███░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  2.1
Gemma-4 Q3_K_M   ████████████████████████████████████████████ 40.6  🏆
Gemma-4 IQ4_XS   ██████████████████████████████████████████░░ 37.1
```
