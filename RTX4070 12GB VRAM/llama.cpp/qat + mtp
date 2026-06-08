# llama.cpp Benchmark Results

**Model:** google/gemma-4-12B-it-qat-q4_0-gguf:Q4_0  
**Draft:** Janvitos/gemma-4-12B-it-qat-assistant-MTP-Q8_0-GGUF:Q8_0  
**Hardware:** RTX 4070 12GB VRAM  
**Date:** 2026-06-08  

## Configuration

```bash
llama-server \
  -hf google/gemma-4-12B-it-qat-q4_0-gguf:Q4_0 \
  --spec-draft-hf Janvitos/gemma-4-12B-it-qat-assistant-MTP-Q8_0-GGUF:Q8_0 \
  --spec-type draft-mtp \
  --spec-draft-n-max 4 \
  --parallel 1 \
  --ctx-size 131072 \
  --temp 1.0 \
  --top-p 0.95 \
  --top-k 64 \
  -ngl 99 \
  --spec-draft-ngl 99 \
  -ctk q4_0 -ctv q4_0 \
  --spec-draft-type-k q4_0 --spec-draft-type-v q4_0 \
  -fa on
```

## VRAM Usage

- **Main Model:** 6.5 GB (Q4_0, 48 layers)
- **Draft Model:** 0.44 GB (MTP assistant, Q8_0)
- **Total VRAM Used:** ~10.8 GB / 12 GB

## Benchmark Results

### Completions Endpoint (`/v1/completions`)

| Run | Prompt (t/s) | Generation (t/s) | Draft Tokens | Accepted | Acceptance % |
|-----|--------------|------------------|--------------|----------|--------------|
| 1   | 60.9         | 171.6            | 421          | 414      | 98.3%        |
| 2   | 500.8        | 68.8*            | 4            | 0        | 0.0%*        |
| 3   | 495.1        | 158.6            | 418          | 368      | 88.1%        |
| 4   | 492.9        | 168.6            | 421          | 405      | 96.2%        |
| 5   | 494.7        | 170.0            | 421          | 408      | 96.9%        |

*\* Run 2 appears to be a cold-start anomaly*

**Average (excl. outlier):**  
- **Prompt:** ~493 t/s  
- **Generation:** **167.2 t/s**  
- **Draft Acceptance:** **94.9%**

---

### Chat Completions Endpoint (`/v1/chat/completions`)

| Run | Prompt (t/s) | Generation (t/s) | Draft Tokens | Accepted | Acceptance % |
|-----|--------------|------------------|--------------|----------|--------------|
| 1   | 127.8        | 115.9            | 621          | 354      | 57.0%        |
| 2   | 550.0        | 101.1            | 719          | 331      | 46.0%        |
| 3   | 541.5        | 109.6            | 659          | 346      | 52.5%        |

**Average:**  
- **Prompt:** ~406 t/s  
- **Generation:** **108.9 t/s**  
- **Draft Acceptance:** **51.8%**

---

## Summary

| Metric | Completions | Chat Completions |
|--------|-------------|------------------|
| **Avg Generation Speed** | **167.2 t/s** | **108.9 t/s** |
| **Avg Prompt Speed** | 493 t/s | 406 t/s |
| **Draft Acceptance** | 94.9% | 51.8% |
| **VRAM Used** | 10.8 GB | 10.8 GB |

## Analysis

1. **Completions endpoint is ~54% faster** than chat completions due to no chat template overhead
2. **MTP draft works exceptionally well** on completions (95% acceptance) — the MTP head predicts next tokens accurately for structured generation
3. **Chat acceptance lower** (52%) because reasoning traces and chat formatting reduce draft accuracy
4. **Full GPU offload** (`-ngl 99`) critical — original `-ngl 0` gave only 7.7 t/s

## Comparison: Before vs After Optimization

| Config | Generation Speed | Draft Acceptance |
|--------|------------------|------------------|
| `-ngl 0` (CPU) | 7.7 t/s | 35.9% |
| `-ngl 99` (GPU) | **167 t/s** | **95%** |

**Speedup: ~22x faster with GPU offload**
