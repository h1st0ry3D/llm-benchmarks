# Godoter-27B Q4_K_S Benchmark Summary

## Hardware

| Component | Spec |
|-----------|------|
| GPU | NVIDIA GeForce RTX 4070 (12 GB VRAM) |
| CPU | AMD Ryzen 5 5600X 6-Core (12 threads) |
| RAM | 32 GB |
| Model | Godoter-27B Q4_K_S (14.73 GB, 27.3B dense params) |

## Key Architecture Details

- **Base model**: Qwen3.5 27B (dense, not MoE)
- **64 layers** (hybrid attention: 16 full-attention + 48 linear-attention)
- **Model size**: 14.73 GB (Q4_K_S quantization)
- **Full model in VRAM**: impossible (14.73 GB > 12 GB) — requires partial GPU offload

## Benchmark Methodology

- **llama-bench** (batch 2048, ubatch 512) for raw TG t/s
- **llama-server** with OpenAI-compatible API + chat template for real-world TPS
- Default params: q4_0 KV cache, flash attention on, 200 generated tokens, prompt ~36 tokens

### llama-bench Raw Results (fast test, no chat template)

| Config | TG t/s | PP t/s | Notes |
|--------|--------|--------|-------|
| ngl=25, t=6 | 3.86 | 43.2 | Baseline |
| ngl=25, t=8 | 3.71 | 42.1 | |
| ngl=25, t=10 | 3.56 | - | |
| ngl=25, t=12 | 3.09 | - | Too many threads (cache thrash) |
| ngl=15, t=8 | 3.06 | - | |
| ngl=20, t=8 | 3.35 | - | |
| ngl=25, t=8 | 3.71 | 42.1 | |
| ngl=30, t=8 | 4.15 | - | |
| ngl=35, t=8 | 4.66 | - | |
| ngl=40, t=6 | 5.54 | 62.7 | |
| ngl=45, t=6 | 6.49 | - | |
| ngl=46, t=6 | 6.77 | - | |
| ngl=47, t=6 | 7.01 | - | |
| ngl=48, t=6 | 7.30 | - | |
| **ngl=49**, t=6 | **7.58** | **90.7** | **Max ngl before OOM** |
| ngl=50, t=6 | OOM | - | CUDA error at ngl=50 |

**Best thread count**: 6 (saturates 6 physical cores without hyperthread overhead)

### llama-server Real-World Results (with chat template)

| ngl | TPS | VRAM | VRAM % | Profile |
|-----|-----|------|--------|---------|
| **45** | **5.41** | 11,728 MiB | 95.5% | 🔥 Max speed, tight fit |
| 40 | 3.99 | 10,616 MiB | 86.4% | Anomaly: slower than ngl=38 |
| **38** | **5.08** | 10,160 MiB | 82.7% | ⭐ Best balance speed/headroom |
| 35 | 4.68 | 9,504 MiB | 77.4% | Good for multi-turn convos |
| 30 | 4.16 | 8,374 MiB | 68.2% | Safe, room for 64K context |
| 28 | 3.96 | 7,938 MiB | 64.6% | |
| 25 | 3.73 | 7,264 MiB | 59.1% | Baseline |

**Note**: ngl=40 showed an anomalous speed regression (3.99 < 5.08 at ngl=38). This is likely due to memory pressure causing CUDA graph capture failures or fallback to suboptimal execution paths. The 38-layer boundary is a sweet spot on this GPU.

### Context Size Scaling

| Config | Context | TG t/s | VRAM Est.* | Use Case |
|--------|---------|--------|------------|----------|
| ngl=38, t=6 | 32K | 5.08 | 10,160 MiB | General purpose |
| ngl=35, t=6 | 32K | 4.68 | 9,504 MiB | Multi-turn |
| ngl=30, t=6 | 32K | 4.16 | 8,374 MiB | Large prompts |
| ngl=25, t=6 | 32K-64K | 3.73-3.84 | 7,264 MiB | Long context |
| ngl=20, t=6 | 64K-128K | 3.35 | ~7,000 MiB | Very long context |
| ngl=15, t=6 | 128K+ | 3.07 | ~6,000 MiB | Extreme context |

*Actual VRAM usage depends on KV cache size which grows with context.

## Recommended Configurations

### 🚀 Max Speed (32K context)
```
ngl=45, t=6, kv=q4_0, ctx=32768, fa=on
```
**5.41 t/s** — 95% VRAM used. Best for short, fast generation. Limited headroom for KV growth.

### ⭐ Best Balance (32K context)
```
ngl=38, t=6, kv=q4_0, ctx=32768, fa=on
```
**5.08 t/s** — 83% VRAM. Sweet spot: fast generation with room for multi-turn conversations.

### 🎯 General Purpose (32K context)
```
ngl=35, t=6, kv=q4_0, ctx=32768, fa=on
```
**4.68 t/s** — 77% VRAM. Comfortable headroom, good for most use cases.

### 📚 Long Context (64K-128K)
```
ngl=25, t=6, kv=q4_0, ctx=65536, fa=on
```
**3.84 t/s** — 59% VRAM at load, leaves room for KV cache growth.

### 📖 Extreme Context (128K)
```
ngl=20, t=6, kv=q4_0, ctx=131072, fa=on
```
**3.35 t/s** — Enough headroom for full 128K context. ngl=15 also works at 3.07 t/s.

## KV Cache Notes

- **q4_0 KV cache** vs **f16 KV cache**: Identical TPS (3.71 both) — always use q4_0 for more VRAM savings
- Flash attention (`-fa on`): enabled in all tests
- For 128K context with q4_0 KV, expect ~2-3 GB additional VRAM for KV cache
- ngl=45 at full 128K context would likely OOM; use ngl=25-30 for 64K, ngl=20 for 128K

## Comparison with Other Models

| Model | Type | Params | Quant | TPS@32K | VRAM |
|-------|------|--------|-------|---------|------|
| Godoter-27B | Dense | 27.3B | Q4_K_S | 5.41 | 11.7 GB |
| Qwopus35B Q6_K | MoE | 35.5B (3B active) | Q6_K | 10.7 | 7.0 GB |
| Qwopus9B Q6_K | Dense | 9B | Q6_K | 57.3 | 12.3 GB |

Godoter-27B is a **dense 27B model** — all 27B params are computed per token. This makes it ~2x slower than the MoE Qwopus35B which only activates ~3B params per token. However, for Godot/GDScript tasks it may provide superior quality due to domain-specific fine-tuning.

## Performance Profile Summary

```
Speed:        3-5 t/s (server) / up to 7.6 t/s (raw TG)
Prompt Proc:  55-91 t/s (depending on ngl)
Memory:       7-12 GB VRAM (depending on ngl)
Context:      Up to 128K (with reduced ngl)
Best ngl:     38 (balance) / 45 (max speed)
Best threads: 6 (not more!)
```
