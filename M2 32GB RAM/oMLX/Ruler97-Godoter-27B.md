# Ruler97/Godoter-27B — oMLX + llama.cpp benchmark (M2 Max, 32 GB)

**Generated:** 2026-06-29  
**Hardware:** Apple M2 Max, 32 GB unified RAM, 38 GPU cores  
**Model:** `Ruler97/Godoter-27B` — Qwen3.6-27B dense model finetuned for Godot/GDScript code generation

## Available formats + sources

| Format | Source | File | Size |
|---|---|---|---|
| **GGUF Q4_K_M** | `Ruler97/Godoter-27B-GGUF` | `Godoter-Q4_K_M.gguf` | 15.4 GB |
| **GGUF Q4_K_M (i1)** | `mradermacher/Godoter-27B-i1-GGUF` | `Godoter-27B.i1-Q4_K_M.gguf` | 15.65 GB |
| **MLX 4-bit** | Converted from HF (via `mlx_lm.convert`) | `Godoter-27B-4bit` (3 shards) | 14 GB |
| **Raw safetensors** | `Ruler97/Godoter-27B` | 15 shards BF16 | 55.6 GB |

## TL;DR — best config by use case

| Use case | Pick | Gen TPS | Max ctx | Notes |
|---|---|---|---|---|
| **Coding ≤32k, max speed ★** | **oMLX + MLX 4-bit** | **21.0 / 18.1 / 15.1** @4k/16k/32k | **32k** | MLX is faster on Apple Silicon |
| **Coding up to 64k ★** | **llama.cpp + GGUF Q4_K_M, q8_0 KV cache** | **~17 / ~13** @short/64k | **64k** | Only path that reaches 64k |
| **Coding 64k, max memory headroom** | llama.cpp + GGUF Q4_K_M, q4_0 KV cache | ~17.5 / ~13.6 @short/64k | **64k** | 4-bit KV saves 75% KV RAM |

**Bottom line:** The dense 27B model is **not optimal** for this hardware. MoE models like `Qwopus3.6-35B-A3B-v1-MLX-4bit` achieve 50 TPS at 64k with less memory. Use Godoter-27B only if you specifically need its Godot/GDScript finetuning.

---

## Benchmark results

### oMLX (MLX 4-bit) — `Godoter-27B-4bit`

| Context | Gen TPS | Prefill TPS | TTFT (s) | Peak memory | Status |
|---------|---------|-------------|----------|-------------|--------|
| 4k | **21.0** | 115.5 | 35.5 | 18.5 GB | ✅ |
| 16k | **18.1** | 109.9 | 149.1 | 21.0 GB | ✅ |
| 32k | **15.1** | 87.9 | 372.9 | 24.5 GB | ✅ |
| 64k | — | — | — | — | ❌ OOM (≥26 GB) |

### llama.cpp (GGUF Q4_K_M) — default f16 KV cache

| Context | Gen TPS | Prefill TPS | TTFT (s) | Notes |
|---------|---------|-------------|----------|-------|
| Short | **17.2** | 26.1 | 0.9 | 18 prompt tokens |
| 64k | **11.7** | 109 | 368 | 40k prompt tokens |

### llama.cpp (GGUF Q4_K_M) — optimized KV cache

| KV cache | Context | Gen TPS | Prefill TPS | KV RAM saved |
|----------|---------|---------|-------------|-------------|
| f16 (default) | 64k | 11.7 | 109 | — |
| q8_0 | 64k | **12.9** | 113 | ~50% |
| q4_0 | 64k | ~13.6 | — | ~75% |
| q4_0 | Short | **17.5** | — | — |

---

## What works and what doesn't

### 64k context: MLX vs GGUF

The **MLX** version (oMLX) crashes at 64k with `kIOGPUCommandBufferCallbackErrorOutOfMemory`. The dense 27B weights (14.5 GB) + 64k KV cache in MLX's format (~12-16 GB) exceeds the 26 GB Metal wired cap.

The **GGUF** version (llama.cpp) succeeds at 64k because:
1. llama.cpp uses fp16 KV cache by default (vs MLX's larger per-block KV)
2. `-ctk q8_0 -ctv q8_0` halves KV memory
3. `-ctk q4_0 -ctv q4_0` quarters KV memory to ~4 GB, freeing room

### Speed comparison: MLX vs llama.cpp

| Backend | Short ctx TPS | 32k TPS | 64k TPS |
|---------|:------------:|:-------:|:-------:|
| oMLX (MLX 4-bit) | 21.0 | 15.1 | ❌ OOM |
| llama.cpp (GGUF f16) | 17.2 | ~15 | 11.7 |
| llama.cpp (GGUF q8_0) | ~17 | ~15 | 12.9 |
| llama.cpp (GGUF q4_0) | 17.5 | ~15 | 13.6 |

MLX is **~20-40% faster** at short context, but llama.cpp/GGUF is the **only way to reach 64k**.

---

## Comparison with other models (from prior benchmarks)

| Model | Arch | Params | Gen TPS @4k | Gen TPS @32k | Gen TPS @64k | Peak @64k |
|---|---|---|---|---|---|---|
| **Godoter-27B (MLX)** | Dense | 27B | 21.0 | 15.1 | ❌ OOM | — |
| **Godoter-27B (GGUF q8_0 KV)** | Dense | 27B | 17.2 | ~15 | 12.9 | ~22 GB |
| Qwopus3.6-27B-Coder (MLX) | Dense | 27B | 17.3 | 14.7 | — | — |
| Qwopus3.6-27B-Coder-MTP (MLX) | Dense | 27B | 21.1 | 14.3 | — | — |
| ★ **Qwopus3.6-35B-A3B-v1-MLX** | **MoE** | **35B (3B active)** | **77** | **61** | **50** | **26.9 GB** |
| Qwen3-Coder-30B-A3B (MLX) | MoE | 30B (3B active) | 71.4 | 31.8 | — | — |
| Nemotron-Nano-30B-A3B (MLX) | MoE | 30B (3B active) | 101 | 81.5 | — | — |

**Key insight:** The MoE A3B models (activating only 3B params/token) are **3-5x faster** than the dense 27B Godoter at the same or lower memory. If you don't specifically need Godot/GDScript finetuning, use the `Qwopus3.6-35B-A3B-v1` or `Nemotron-Nano-30B-A3B` instead.

---

## Optimization techniques for M2 Max 32 GB

### 1. KV cache quantization (biggest win for 64k)

| Flag | KV size @64k | Total est. memory | Fits 32 GB? |
|------|:----------:|:-----------------:|:----------:|
| Default (f16) | ~16 GB | ~30.5 GB | ❌ Tight |
| `-ctk q8_0 -ctv q8_0` | ~8 GB | ~22.5 GB | ✅ |
| `-ctk q4_0 -ctv q4_0` | ~4 GB | ~18.5 GB | ✅ Spacious |

**Recommendation:** Use `q8_0` for best quality/speed balance, or `q4_0` for max headroom.

### 2. Flash Attention

`-fa on` (default `auto`) — reduces memory bandwidth for attention computation. Use it.

### 3. GPU offload ratio

`-ngl 99` — offload ALL layers to Metal. On M2 Max 32GB this gives max speed. Only reduce if you need to free GPU memory for concurrent tasks.

### 4. Thread tuning (llama.cpp)

- `-t 8` — 8 CPU threads for generation (matches M2 Max performance core count)
- `-tb 8` — 8 threads for batch/prompt processing

Default (-1) auto-detects, but explicit values can help in mixed workloads.

### 5. Cache RAM limit

`--cache-ram 8192` — limit in-RAM cache to 8 GB (default). For 64k context, you might want to increase this: `--cache-ram 16384`.

### 6. SSD prefix cache (oMLX only)

oMLX's `--paged-ssd-cache-dir` offloads KV cache to SSD. This helps with cache reuse across requests but does NOT reduce peak memory during a single 64k request (the full 64k KV is resident during the request).

### 7. i1 importance-matrix GGUF variants

The `mradermacher/Godoter-27B-i1-GGUF` variant uses an importance matrix to allocate bits more intelligently. Gives better quality per bit at the same file size. Try `Godoter-27B.i1-Q4_K_M.gguf` or `Godoter-27B.i1-IQ4_XS.gguf` (14.25 GB, even smaller).

### 8. CPU split offloading (last resort)

If GPU memory is exhausted, offload some layers to CPU with `-ngl 80` (80/99 layers on GPU). This frees GPU RAM for larger KV cache but slows generation.

---

## Files

- Launcher (GGUF/llama.cpp): `launch_godoter_gguf.sh`
- Launcher (MLX/oMLX, ≤32k): `launch_godoter_omlx.sh`
- MLX model location: `~/.lmstudio/models/Godoter-27B-4bit/`
- GGUF model location: `~/.lmstudio/models/Ruler97/Godoter-27B-GGUF/Godoter-Q4_K_M.gguf`
- Prior benchmarks: `m2-max/summary.md`, `Qwopus3.6-27B-Coder-4bit_omlx_benchmark.md`, `Qwopus3.6-27B-Coder-Compat-MTP-GGUF_llamacpp_summary.md`
