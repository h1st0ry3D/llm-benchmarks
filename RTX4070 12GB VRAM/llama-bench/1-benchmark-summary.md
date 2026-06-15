# Benchmark Summary

All benchmarks run with llama.cpp build `d05fe1d7d (9010)` on CUDA backend.

## Models Compared

| Model | Size | Parameters | Quantization | ngl |
|-------|------|------------|--------------|-----|
| gemma4 E4B | 4.95 GiB | 7.52 B | Q4_K_M | 99 |
| qwen35 9B | 5.23 GiB | 8.95 B | Q4_K_M | 99 |
| qwen35moe 35B.A3B | 16.10 GiB | 34.66 B | Q4_K_M | 20 |

## Performance Comparison

| Model | pp512 (t/s) | tg128 (t/s) |
|-------|-------------|-------------|
| gemma4 E4B | **5875.71 ± 309.86** | **114.34 ± 0.50** |
| qwen35 9B | 3756.47 ± 63.53 | 77.79 ± 0.61 |
| qwen35moe 35B.A3B | 703.91 ± 84.76 | N/A |

## Analysis

### Prompt Processing (pp512)
- **Gemma4 E4B** is **~1.6x faster** than Qwen3.5 9B and **~8.3x faster** than Qwen3.6 MoE
- Gemma4 shows highest throughput for prompt processing
- Qwen3.6 MoE is significantly slower due to larger model size (34.66B params vs ~8B)

### Text Generation (tg128)
- **Gemma4 E4B** is **~1.5x faster** than Qwen3.5 9B
- Qwen3.6 MoE has no tg128 benchmark data
- Gemma4 shows more consistent performance (lower std dev: ±0.50 vs ±0.61)

### Model Characteristics
- **Gemma4 E4B**: Best overall performance, smallest size, most efficient
- **Qwen3.5 9B**: Middle ground, larger than Gemma but smaller than MoE
- **Qwen3.6 35B MoE**: Largest model with Mixture of Experts architecture, much slower due to size and ngl=20 (fewer layers offloaded to GPU)

### Notes
- Qwen3.6 MoE uses `type_k=q8_0`, `type_v=q8_0`, `fa=1`, `mmap=0` (different configuration)
- Lower ngl (20 vs 99) for MoE model suggests GPU memory constraints due to larger size
- All tests run on same CUDA backend with same build version
