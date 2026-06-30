# Qwopus3.6-35B-A3B-Coder-oQ4-mtp — oMLX benchmark

**Model:** `fritskarl/Qwopus3.6-35B-A3B-Coder-oQ4-mtp` (Jackrong coder finetune + MTP heads)
**Base:** `Jackrong/Qwopus3.6-35B-A3B-v1` → `unsloth/Qwen3.6-35B-A3B`
**Hardware:** Apple M2 Max, 32 GB unified RAM, 38 GPU cores
**Engine:** oMLX (MLX 4-bit mixed-precision quant, MTP enabled, DFlash + SSD cache)
**Date:** 2026-06-30

## TL;DR — best coding model on this hardware

| Config | 4k | 16k | 32k | 64k | 128k | Weights |
|--------|:--:|:---:|:---:|:---:|:----:|:-------:|
| **★ Coder MTP + MTP + DFlash/SSD (NEW)** | **79 TPS** | **70 TPS** | **61 TPS** | ❌ OOM | ❌ OOM | 20 GB |
| v1 + DFlash/SSD (prev daily) | 77 TPS | 69 TPS | 61 TPS | **50 TPS** | ❌ OOM | 18.3 GB |

**Key finding:** The Coder MTP model is the **fastest coder config** at ≤32k (79 TPS @4k!) but cannot reach 64k due to the larger 20 GB weight footprint from MTP heads. Use the v1 model for 64k tasks.

## Recommended sampling for agentic coding (from HF model card)

| Parameter | Value | Source |
|-----------|-------|--------|
| `temperature` | **0.6** | Qwen3-Coder non-thinking default |
| `top_p` | **0.8** | Qwen3-Coder non-thinking default |
| `top_k` | **20** | Qwen3 family default |
| `min_p` | **0** | off |
| `repetition_penalty` | **1.05** | safety net, shortens reasoning |
| `thinking` | **off** | "thinking-off" agent mode (fast) |

The Coder MTP model is designed for **"thinking-off" agentic coding** — it avoids excessive reasoning tokens, making it ~40% more token-efficient in agent loops compared to the v1 model with thinking enabled.

## Full benchmark results

### Gen TPS (tok/s, higher = better)

| Config | MTP | DFlash | 4096 | 16384 | 32768 | 65536 |
|--------|:---:|:------:|:----:|:-----:|:-----:|:-----:|
| **★ Coder oQ4-mtp** | **on** | **ssd** | **79.2** | **70.2** | **61.4** | ❌ OOM |
| v1-MLX-4bit | off | ssd | 76.5 | 65.9 | 56.8 | 50.1 |
| v1-MLX-4bit | off | off | 76.9 | 68.6 | 61.0 | ❌ OOM |

### Prefill / Processing TPS

| Config | 4096 | 16384 | 32768 |
|--------|:----:|:-----:|:-----:|
| **Coder oQ4-mtp** | **737.0** | **722.4** | **655.0** |
| v1-MLX-4bit (DFlash) | 609.8 | 544.6 | 447.7 |
| v1-MLX-4bit (base) | 708.6 | 698.0 | 605.9 |

### TTFT (s, lower = better)

| Config | 4096 | 16384 | 32768 |
|--------|:----:|:-----:|:-----:|
| **Coder oQ4-mtp** | **5.6** | **22.7** | **50.0** |
| v1-MLX-4bit (DFlash) | 6.7 | 30.1 | 73.2 |
| v1-MLX-4bit (base) | 5.8 | 23.5 | 54.1 |

### Peak memory (GB)

| Config | 4096 | 16384 | 32768 | 65536 |
|--------|:----:|:-----:|:-----:|:-----:|
| **Coder oQ4-mtp** | **22.23** | **23.27** | **25.12** | ❌ OOM |
| v1-MLX-4bit (DFlash) | 21.52 | 22.56 | 24.40 | 26.88 |

## Maximum context

The Coder MTP model has **20 GB weights** (2 GB more than v1 due to MTP heads). On M2 Max 32 GB:

- **32k** ✅ — 25.12 GB peak, safe
- **48k** → Theoretical ~26.5 GB — borderline, may crash
- **64k** ❌ — exceeds 26 GB wired cap

**For 64k context**, use the `Qwopus3.6-35B-A3B-v1-MLX-4bit` (18.3 GB weights) instead.

## Memory calculation

```
Weights:      ~20.0 GB  (MTP heads add ~2 GB vs v1)
KV cache/64k: ~6.9 GB   (40 layers, 2 KV heads, 256 head_dim)
Overhead:     ~1-2 GB
Total @64k:   ~28 GB → exceeds 26 GB wired cap ❌
Total @32k:   ~25.1 GB → fits ✅
Total @16k:   ~23.3 GB → fits ✅
```

## Files

- Model location: `~/.lmstudio/models/fritskarl/Qwopus3.6-35B-A3B-Coder-oQ4-mtp/`
- Launcher: `launch_qwopus_omlx.sh` (select ★ Qwopus3.6-35B-A3B-Coder-oQ4-mtp)
- Prior v1 benchmark: `m2-max/summary.md`
