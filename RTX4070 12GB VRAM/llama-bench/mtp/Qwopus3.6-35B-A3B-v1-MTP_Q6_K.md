# llama-bench Results

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
| Quantization | Q6_K |
| Size | 27.19 GiB |
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
| Prompt processing (pp2048) | **373.02 ± 1.76** |
| Text generation (tg512) | **18.39 ± 0.34** |
