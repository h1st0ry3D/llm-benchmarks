# llama-bench Results — Gemma-4 26B A4B IT (UD-IQ4_XS)

**Date:** 2026-06-12
**Build:** 715b86a36 (9559)

## System

| Component | Details |
|-----------|---------|
| GPU | NVIDIA GeForce RTX 4070 (11868 MiB) |
| CPU | AMD Ryzen 5 5600X 6-Core |
| RAM | 31989 MiB |

## Model

| Property | Value |
|----------|-------|
| Model | gemma-4-26B-A4B-it-GGUF |
| Quant | UD-IQ4_XS (4.25 bpw) |
| Size | 12.65 GiB |
| Params | 25.23 B (3.8B active) |
| Architecture | MoE (8/128 + 1 shared) |

## Configuration

| Flag | Value |
|------|-------|
| GPU layers | 99 |
| CPU MoE threads | 12 |
| Threads | 12 |
| Cache type K | q8_0 |
| Cache type V | q8_0 |
| Flash attention | on |
| mmap | off |

## Benchmarks

| Test | Tokens/sec |
|------|-----------:|
| Prompt processing (pp2048) | **1269.75 ± 14.25** |
| Text generation (tg512) | **37.09 ± 0.20** |
