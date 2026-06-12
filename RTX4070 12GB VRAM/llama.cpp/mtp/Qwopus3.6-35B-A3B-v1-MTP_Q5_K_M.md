# llama-bench Results — Q5_K_M

**Date:** 2026-06-12
**Build:** 715b86a36 (9559)

## System

| Component | Details |
|-----------|---------|
| GPU | NVIDIA GeForce RTX 4070 (11868 MiB) |
| CPU | AMD Ryzen 5 5600X 6-Core Processor |
| RAM | 31989 MiB |

## Model

| Property | Value |
|----------|-------|
| Model | Qwopus3.6-35B-A3B-v1-MTP-GGUF |
| Quantization | Q5_K_M |
| Size | 23.60 GiB |
| Params | 35.51 B |

## Configuration

| Flag | Value |
|------|-------|
| GPU layers | 99 |
| CPU MoE threads | 36 |
| Threads | 12 |
| Cache type K | q8_0 |
| Cache type V | q8_0 |
| Flash attention | on |
| mmap | off |

## Benchmarks

| Test | Tokens/sec |
|------|-----------:|
| Prompt processing (pp2048) | **424.97 ± 1.89** |
| Text generation (tg512) | **20.30 ± 0.36** |

## Comparison with Q6_K

| Quant | Size | pp2048 (t/s) | tg512 (t/s) |
|-------|-----:|-------------:|------------:|
| Q6_K | 27.19 GiB | 373.02 ± 1.76 | 18.39 ± 0.34 |
| Q5_K_M | 23.60 GiB | 424.97 ± 1.89 | 20.30 ± 0.36 |
| **Δ** | **-3.59 GiB** | **+51.95 (+13.9%)** | **+1.91 (+10.4%)** |


Q5_K_M is ~14% faster than Q6_K on prompt processing and ~10% faster on generation, saving 3.59 GiB vs Q6_K.
