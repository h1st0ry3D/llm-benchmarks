# Benchmark Summary

All benchmarks run with llama.cpp build `715b86a36 (9559)` on CUDA backend.

## Models Compared

| Model | Size | Parameters | Quantization | ngl |
|-------|------|------------|--------------|-----|
| gemma4 12B Q4_0 | 6.48 GiB | 11.91 B | Q4_0 | 99 |

## Performance Comparison

| Model | pp512 (t/s) | tg128 (t/s) |
|-------|-------------|-------------|
| gemma4 12B Q4_0 | **2926.94 ± 73.10** | **54.85 ± 0.07** |

## Configuration

```bash
llama-bench -m gemma-4-12b-it-qat-q4_0.gguf \
  -ngl 99 -ctk q4_0 -ctv q4_0 -fa on \
  -p 512 -n 128 -r 3
```

## Hardware

- **GPU:** NVIDIA GeForce RTX 4070 (12 GB VRAM)
- **VRAM Used:** ~10.8 GB (model) + overhead
- **Backend:** CUDA

## Notes

- **pp512**: Prompt processing (512 tokens) — 2927 t/s
- **tg128**: Text generation (128 tokens) — 54.8 t/s (single-threaded, no speculative decoding)
- With **MTP speculative decoding** (`--spec-draft-n-max 4`), server benchmarks show **~167 t/s** generation (3x speedup)
- Flash Attention (`-fa on`) enabled for faster attention
- KV cache quantized to Q4_0 (`-ctk q4_0 -ctv q4_0`) for memory efficiency

## Comparison: With vs Without Speculative Decoding

| Mode | Generation Speed (t/s) |
|------|------------------------|
| Baseline (llama-bench, no draft) | 54.9 |
| With MTP draft (server, completions) | **~167** |
| With MTP draft (server, chat) | **~109** |

**Speedup: ~3x with MTP speculative decoding**
