# llama-bench Results — Q4_K_M (Coder)

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
| Model | Qwopus3.6-27B-Coder-MTP-GGUF |
| Quantization | Q4_K_M |
| Size | 15.65 GiB |
| Params | 27.32 B |

## Configuration

| Flag | Value |
|------|-------|
| GPU layers | 25 |
| Threads | 12 |
| Cache type K | q4_0 |
| Cache type V | q4_0 |
| Flash attention | on |
| mmap | off |

## Benchmarks

| Test | Tokens/sec |
|------|-----------:|
| Prompt processing (pp2048) | **322.51 ± 6.50** |
| Text generation (tg128) | **2.13 ± 0.02** |
