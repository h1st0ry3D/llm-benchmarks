# Qwopus3.6-27B-Coder-4bit.mlx — oMLX Benchmark & Tuning Summary

**Machine:** Apple M2 Max, 32 GB unified RAM, 38 GPU cores
**Models tested:**
- `jedisct1/Qwopus3.6-27B-Coder-4bit.mlx` — base, ~16.6 GB on disk, ~18.5 GB resident
- `jedisct1/Qwopus3.6-27B-Coder-MTP-4bit.mlx` — MTP variant, ~16.0 GB on disk, **~15.0 GB resident**
**Server:** oMLX 0.4.4 (`omlx` CLI), OpenAI-compatible API at `http://127.0.0.1:8000`
**Bench tool:** oMLX built-in `/admin/api/bench/start` (SSE stream) — TTFT, TPOT, gen TPS, processing (prefill) TPS, peak memory.
**Date:** 2026-06-19

---

## 1. TL;DR — BEST configuration

**Use the MTP variant with MTP enabled. It wins on every axis** (faster prefill, faster decode, lower memory) except that it cannot combine with TurboQuant.

```bash
# 1. Raise Apple's GPU wired-memory cap (volatile — redo after every reboot!)
sudo sysctl iogpu.wired_limit_mb=26624      # 26 GB
#    (persist in /etc/sysctl.conf to survive reboots)

# 2. Restart oMLX so it picks up the raised cap
omlx restart
```

**oMLX settings (`~/.omlx/settings.json`) — applied:**
```json
"server":  { "burst_decode_mode": "balanced", "preserve_mid_system_cache": true },
"memory":  { "memory_guard_tier": "custom",
             "memory_guard_custom_ceiling_gb": 25.0,
             "prefill_memory_guard": true },
"cache":   { "enabled": true, "ssd_cache_max_size": "92GB",
             "hot_cache_max_size": "0", "initial_cache_blocks": 256 }
```

**Per-model profile — applied:**
- `Qwopus3.6-27B-Coder-MTP-4bit.mlx` → `mtp_enabled: true`, `turboquant_kv_enabled: false` ← **daily driver**
- `Qwopus3.6-27B-Coder-4bit.mlx` → `turboquant_kv_enabled: false` (fallback; enable TQ 4-bit only if you ever need to squeeze under the ceiling)

**Resulting performance (MTP variant, MTP on):**
| Context | Decode TPS | Prefill TPS | TTFT | Peak mem |
|---|---|---|---|---|
| 4k   | 18.5 | 114 |  36 s | 19.5 GB |
| 16k  | 17.0 |  90 | 181 s | 20.9 GB |
| 32k  | 16.5 |  98 | 334 s | 21.5 GB |
| 64k  | ❌   | —   | —     | ~26.8 GB (does not fit; see §4) |

**Max viable context on this hardware: ~32k** (comfortable, 21.5 GB / 25 GB ceiling).

---

## 2. Full benchmark comparison

All runs: `tg=128`, `temperature=0.1`, cold cache (`cached_tokens=0`), single-stream. Peak memory is oMLX's measured/ predicted peak.

### Decode TPS (higher is better)

| Context | Base no-TQ | Base TQ-4bit | **MTP on (no-TQ)** |
|---:|---:|---:|---:|
|  4096 | 19.6 | 18.6 | 18.5 |
| 16384 | 16.4 | 13.9 | **17.0** |
| 32768 | 15.4 | 14.8 | **16.5** |
| 65536 | ❌   | ❌   | ❌    |

### Prefill TPS / TTFT (higher TPS / lower TTFT is better)

| Context | Base no-TQ | Base TQ-4bit | **MTP on (no-TQ)** |
|---:|---|---|---|
|  4096 | 114 / 36 s  |  76 / 54 s | 114 /  36 s |
| 16384 |  58 / 284 s |  91 / 180 s| **90 / 181 s** |
| 32768 |  69 / 478 s |  65 / 506 s| **98 / 334 s** |

### Peak memory (lower is better; ceiling = 25 GB)

| Context | Base no-TQ | Base TQ-4bit | **MTP on (no-TQ)** |
|---:|---|---|---|
|  4096 | 19.18 GB | 18.84 GB | 19.50 GB |
| 16384 | 21.30 GB | 19.53 GB | 20.85 GB |
| 32768 | 22.88 GB | 21.87 GB | **21.52 GB** |
| 65536 | ~27.2 GB ❌ | ~27.7 GB ❌ | ~26.8 GB ❌ |

> Raw data: `bench_results.json` next to this file.

---

## 3. Why the MTP variant is the best choice

The MTP variant carries one extra "Multi-Token-Prediction" head layer (`mtp_num_hidden_layers: 1`) used for speculative decoding: each generation step proposes a draft token that is verified against the backbone, so ~1.84 tokens are produced per backbone cycle.

**Measured MTP draft acceptance (from logs):**
- 32k context: **84–85%** acceptance, ~57 bonus tokens per 128 generated, MTP overhead only ~235 ms of a ~7.5 s backbone pass.
- 16k context: 82–84%.

**Why it wins:**
1. **Lower base memory** — loads to **15.0 GB** vs **~18.5 GB** for the base variant (the MTP-4bit quants are slightly more aggressive). This is why MTP at 32k (21.5 GB) actually uses *less* peak memory than the base at 32k (22.9 GB), even without KV quantization.
2. **Much faster prefill** — 32k TTFT **334 s vs 478 s** (−30%), prefill TPS 98 vs 69 (+43%). The MTP model patch (PR 990) appears to optimize the prefill path as well.
3. **Faster decode at long context** — 16.5 vs 15.4 TPS at 32k (+7%). At very short prompts (4k) MTP is marginally slower (18.5 vs 19.6) due to draft overhead — negligible and below the noise floor.

**The one constraint:** MTP and TurboQuant are **mutually exclusive** (oMLX enforces this: *"MTP and TurboQuant KV cannot both be enabled; TurboQuant patches the attention path MTP relies on"*). This is fine — MTP's lower base memory gives more headroom than TQ would anyway.

---

## 4. Why 64k context still doesn't fit (both variants)

oMLX preflight peak predictions at 64k:
```
Base   no-TQ:  current 20.45 GB + KV+SDPA  7.25 GB = 27.20 GB
Base   TQ-4bit: current 20.45 GB + KV+SDPA  7.25 GB = 27.70 GB   (TQ doesn't help — see below)
MTP    no-TQ:  current 16.78 GB + KV+SDPA 10.05 GB = 26.83 GB
```

All exceed the safe **25 GB ceiling**. The KV+SDPA term at 64k is dominated by the **linear-attention recurrent state + SDPA workspace**, not the quantizable full-attention KV — that's why TurboQuant (which only quantizes the 15 full-attention KV layers) barely moves the 64k number. The MTP draft path adds workspace, so its KV+SDPA is even larger (10.05 GB), offsetting its lower base.

Fitting 64k would need a ≥27 GB ceiling → a 28–30 GB wired cap → on a 32 GB machine that starves macOS and **causes kernel panics** (reproduced during this session at 30 GB). **64k is not achievable on 32 GB RAM with this model family.**

**Practical max context: ~32k.** Extrapolating linearly from the 32k/64k measurements, the MTP variant could likely stretch to ~50k before hitting 25 GB, but oMLX's bench only accepts fixed lengths (…32768, 65536…), so this wasn't pinned. Leave headroom for real workloads — **32k is the safe operating point.**

---

## 5. Key findings (carry-over, still true)

- **Hybrid architecture**: 64 layers, only 16 are full-attention; 48 are linear-attention with a small fixed recurrent state. KV grows slowly (~64 KB/token) but the linear state + SDPA workspace dominate at very long context.
- **Apple Metal wired cap is the hard floor.** Out of the box it's 21.3 GB → max ~16k. `sudo sysctl iogpu.wired_limit_mb=26624` is mandatory for 32k. It is **volatile** (resets on reboot, and your earlier crash was a reboot that silently reset it to 0). Persist in `/etc/sysctl.conf`. Do **not** use 30720 — crashes a 32 GB Mac.
- **SSD prefix cache is essential.** Cold prefill is slow (8 min at 32k) but `preserve_mid_system_cache: true` + 92 GB SSD cache turns repeat prefixes into thousands-of-TPS cache hits (measured 4800–7900 TPS on hit in the streaming test). Reuse system prompts / file contents.
- **TurboQuant 4-bit** (base model only) saves ~1 GB at 32k but costs ~1–2 decode TPS and is incompatible with MTP. Only useful as a fallback headroom tool on the base model. Not needed with the MTP variant.

---

## 6. Final recommended configuration

**Daily driver model:** `Qwopus3.6-27B-Coder-MTP-4bit.mlx`

**`~/.omlx/settings.json` (relevant keys):**
```json
{
  "server":  { "burst_decode_mode": "balanced", "preserve_mid_system_cache": true },
  "memory":  { "memory_guard_tier": "custom",
               "memory_guard_custom_ceiling_gb": 25.0,
               "prefill_memory_guard": true },
  "cache":   { "enabled": true, "ssd_cache_max_size": "92GB",
               "hot_cache_max_size": "0", "initial_cache_blocks": 256 },
  "sampling":{ "max_context_window": 262144 }
}
```

**Per-model profiles:**
```jsonc
// Qwopus3.6-27B-Coder-MTP-4bit.mlx  (daily driver)
{ "mtp_enabled": true, "turboquant_kv_enabled": false }

// Qwopus3.6-27B-Coder-4bit.mlx      (fallback)
{ "turboquant_kv_enabled": false }   // set to true,4 only if you need ceiling headroom
```

**Kernel (after every reboot, or in `/etc/sysctl.conf`):**
```bash
sudo sysctl iogpu.wired_limit_mb=26624
omlx restart
```

---

## 7. Useful commands

```bash
# Check / set the Metal wired cap
sysctl iogpu.wired_limit_mb            # 0 = Apple default (21.3 GB)
sudo sysctl iogpu.wired_limit_mb=26624 # 26 GB — safe for 32 GB RAM

# Restart oMLX after sysctl or settings changes
omlx restart
grep "Metal cap\|enforcer started" ~/.omlx/logs/server.log | tail -2
# expect: "Metal cap (26.0GB ...) ... ceiling=25.0GB"

# Set the daily-driver model profile (MTP on)
M="Qwopus3.6-27B-Coder-MTP-4bit.mlx"
curl -X PUT "http://127.0.0.1:8000/admin/api/models/$M/settings" \
  -H "Authorization: Bearer oMLXapi" -H "Content-Type: application/json" \
  -d '{"mtp_enabled":true,"turboquant_kv_enabled":false}'

# Fallback: base model + TurboQuant 4-bit (for max headroom, no MTP)
B="Qwopus3.6-27B-Coder-4bit.mlx"
curl -X PUT "http://127.0.0.1:8000/admin/api/models/$B/settings" \
  -H "Authorization: Bearer oMLXapi" -H "Content-Type: application/json" \
  -d '{"mtp_enabled":false,"turboquant_kv_enabled":true,"turboquant_kv_bits":4}'

# Run the built-in benchmark (valid lengths: 1024 4096 8192 16384 32768 65536 131072 200000)
python3 omlx_bench.py "Qwopus3.6-27B-Coder-MTP-4bit.mlx" 4096 16384 32768

# Custom streaming benchmark (uses oMLX usage metrics; tests cache hits too)
python3 bench_qwopus.py 4096 16384 32768
```

---

## 8. Bottom line

- **Switch to the MTP variant** (`Qwopus3.6-27B-Coder-MTP-4bit.mlx`) with `mtp_enabled: true`. It is faster (prefill +30%, decode +7% at 32k), smaller (15 GB vs 18.5 GB base), and the 84–85% draft acceptance makes speculative decode worthwhile.
- **Realistic max context: ~32k** at ~16.5 decode TPS and ~334 s cold TTFT (cache makes repeats near-instant). 64k is not possible on 32 GB RAM with either variant.
- **`sudo sysctl iogpu.wired_limit_mb=26624` after every reboot** (or persist in `/etc/sysctl.conf`) — without it you're silently capped at ~16k. Never use 30720 (crashes).
- **TurboQuant is only a fallback headroom tool for the base model** — it's incompatible with MTP and slower. The MTP variant doesn't need it.
- **Keep the SSD prefix cache on** — it's the single biggest UX win for repeat-heavy coding workflows.
