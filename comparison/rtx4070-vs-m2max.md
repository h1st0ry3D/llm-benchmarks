# RTX 4070 (12 GB VRAM) vs M2 Max (32 GB Unified) — Model Comparison

> **Hardware:** RTX 4070 + Ryzen 5 5600X (6c/12t) + 32 GB RAM  
> **Hardware:** Apple M2 Max, 38 GPU cores, 32 GB unified RAM  
> **Date:** 2026-06-30

---

## 1. Dense Model: Godoter-27B (27.3B params, Q4)

| Metric | RTX 4070 | M2 Max | Gap |
|--------|:--------:|:------:|:---:|
| Short ctx TPS | 5.4 | **21.0** | **3.9× M2** |
| 32K TPS | 5.1 | **15.1** | **3.0× M2** |
| 64K TPS | 3.8 | **13.6** | **3.6× M2** |
| Max context | **128K** (at 3.1 t/s) | 64K | **RTX** (barely) |
| Prefill TPS | 91 | **116** | 1.3× M2 |
| Memory pressure | 🔴 Constant — model (14.7 GB) > VRAM (12 GB) | 🟢 Spacious — 32 GB unified | **M2** |

**Verdict: M2 Max wins decisively.** The dense 27B model doesn't fit in 12 GB VRAM, forcing partial offload and crippling speed. M2 Max runs it completely on-GPU at 3–4× the throughput.

---

## 2. MoE Model: Qwopus3.6-35B-A3B-Coder-MTP (35.5B / ~3B active)

| Metric | RTX 4070 | M2 Max (MTP variant) | Gap |
|--------|:--------:|:--------------------:|:---:|
| Short ctx TPS | 44 | **79** | **1.8× M2** |
| 32K TPS | 44 | **61** | **1.4× M2** |
| 64K TPS | **44** | ❌ OOM (MTP) / 50 (v1) | **RTX** |
| 128K TPS | **44** | ❌ OOM | **RTX** |
| Max context | **128–262K** ✅ | 32K (MTP) / 64K (v1) | **RTX** |
| Prefill TPS | 75 | **655–737** | **8.8× M2** |
| VRAM / Memory | ~6.2 GB | 22–25 GB | **RTX** (efficient) |

**Verdict: Depends on context length.**

- **≤32K** → **M2 Max** (79 t/s gen, 737 t/s prefill — much snappier)
- **≥64K** → **RTX 4070** (44 t/s flat at any context, M2 MTP OOMs at 64K)
- **Whole-repo 128K+** → **RTX 4070 only option** (M2 MTP OOMs, v1 tops at 64K)

---

## 3. Key Takeaways

### Architecture matters more than the GPU

| Architecture | RTX 4070 experience | M2 Max experience |
|---|---|---|
| **Dense 27B** (Godoter) | 🐢 3–5 t/s, VRAM juggling | 🚀 15–21 t/s, effortless |
| **MoE 35B** (Qwopus) | 🚀 44 t/s, only 6 GB VRAM | 🚀 61–79 t/s, prefill beast |

The MoE model's 3B active params make it efficient **everywhere** — the RTX 4070 can run it at 44 t/s with only half its VRAM used. The dense model punishes the 12 GB card.

### When to use which machine

| Workload | Pick |
|---|---|
| **Dense models (Godoter-27B etc.)** | **M2 Max** — always |
| **MoE models, short context <32K** | **M2 Max** — 1.4–1.8× faster gen, 8.8× faster prefill |
| **MoE models, long context ≥64K** | **RTX 4070** — M2 Max OOMs with MTP variants |
| **Whole-repo agents, 128K+** | **RTX 4070** — only machine that can run full context |
| **Prefill-heavy agent loops** | **M2 Max** — 737 vs 75 t/s is transformative |

### Surprise of the comparison

The **RTX 4070 punches far above its weight on MoE models** — 44 t/s at 128K context using only 6 GB VRAM is impressive. On dense models it's a different story (Godoter at 5 t/s was painful). The M2 Max dominates on short-context throughput and especially prefill, but its 26 GB wired memory cap limits max context on larger quantized models.
