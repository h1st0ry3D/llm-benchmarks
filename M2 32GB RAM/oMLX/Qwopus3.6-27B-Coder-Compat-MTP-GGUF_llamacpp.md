# Qwopus3.6-27B-Coder-Compat-MTP-GGUF — llama.cpp launch summary

**Machine:** Apple M2 Max, 32 GB unified RAM, 38 GPU cores
**Model:** `Jackrong/Qwopus3.6-27B-Coder-Compat-MTP-GGUF` — quant on disk: `Qwopus3.6-27B-Coder-Compat-MTP-Q4_K_S.gguf` (15.8 GB) + `mmproj-F32.gguf` (0.93 GB, vision projector)
**Also covered (non-MTP sibling):** `Jackrong/Qwopus3.6-27B-Coder-GGUF` — `Qwopus3.6-27B-Coder-IQ4_XS.gguf` (15.4 GB) + `mmproj-F32.gguf` (lms key `qwopus3.6-27b-coder`). No "Compat" non-MTP repo exists; this is the original-template coder without MTP heads.
**Engine:** llama.cpp via LM Studio's bundled runtime (selected: `llama.cpp-mac-arm64-apple-metal-advsimd@2.23.0`)
**Server:** OpenAI-compatible API at `http://127.0.0.1:8000` (same port as the oMLX launcher — only one server runs at a time)
**Launcher:** `./launch_qwopus_llamacpp.sh`
**Date:** 2026-06-24

---

## 1. Why a separate launcher (oMLX vs llama.cpp)

oMLX is **MLX-only**. It cannot load GGUF — the model never appears in `/v1/models` and the oMLX admin profile PUT returns `404: Model not found`. The Compat-MTP model is published only as GGUF, so it must be served by llama.cpp. LM Studio ships a bundled, selected llama.cpp engine and exposes it through the `lms` CLI, so no separate llama.cpp install is needed.

| Launcher | Engine | Format | Serves the Compat GGUF? |
|---|---|---|---|
| `launch_qwopus_omlx.sh` | oMLX (MLX) | `.mlx` / safetensors | ❌ no — MLX-only |
| `launch_qwopus_llamacpp.sh` | llama.cpp (LM Studio runtime) | GGUF | ✅ yes |

---

## 2. llama.cpp engine location (the path you asked about)

LM Studio installs llama.cpp as a "runtime extension pack". The selected GGUF engine lives at:

```
~/.lmstudio/extensions/backends/llama.cpp-mac-arm64-apple-metal-advsimd-2.23.0/
```

The standalone OpenAI-compatible server binary inside it:

```
~/.lmstudio/extensions/backends/llama.cpp-mac-arm64-apple-metal-advsimd-2.23.0/llama-server
```

alongside the GGML/Metal shared libraries (`libggml-metal.dylib`, `libllama.dylib`, `libllama-server-impl.dylib`, `libmtmd.dylib` for the vision/mmproj side, …). All engines installed:

```
~/.lmstudio/extensions/backends/llama.cpp-mac-arm64-apple-metal-advsimd-2.{7,9,14,22,23}.0/
```

Check / select engines with the `lms` CLI:

```bash
lms runtime ls        # list installed engines + which is selected for GGUF
lms runtime select --help
```

The launcher itself does **not** call this binary directly — it drives the engine through `lms load` / `lms server start`, which is the supported path. The binary is useful for a manual/direct launch (see §5).

---

## 3. Recommended config — "Codex-sized" coding

The Compat model is a **heavy thinker**: even trivial prompts burn 300–950+ reasoning tokens before emitting content. A large output budget is mandatory, otherwise the response comes back empty with `finish_reason=length` (this is **not** a loop — it's an under-sized budget).

The launcher's ★ default config:

| Setting | Value | Why |
|---|---|---|
| Context window (`-c`) | **65536** | 24.66 GiB @ 100% GPU — fits the 26 GB wired cap |
| Output budget (`max_tokens`) | **32768** | "Codex size": 32K for prompt + 32K for output |
| GPU offload (`--gpu`) | `max` (100% Metal) | All layers on-device; CPU idle |
| `temperature` | 0.6 | between old 0.5 preset and Qwen no-thinking 0.7 |
| `top_p` | 0.8 | Qwen3-Coder non-thinking default |
| `top_k` | 20 | Qwen3 family default |
| `min_p` | 0 | off |
| `repetition_penalty` | 1.05 | safety net; empirically shortens reasoning vs 1.0 with no quality loss |

Memory estimates (`lms load --estimate-only`, 100% GPU, 26 GB wired cap):

| Context | Est. GPU |
|---|---|
| 16k | ~19 GB |
| 32k | 20.96 GiB |
| 48k | 22.81 GiB |
| **64k** | **24.66 GiB** ← ★ default |

Full config table in the launcher:

| Config | Model | Context | Output budget |
|---|---|---|---|
| ★ Qwopus-Coder-Compat-MTP GGUF (64k coder, 32k out) | compat-mtp (Q4_K_S) | 65536 | 32768 |
| Qwopus-Coder-Compat-MTP GGUF (32k safe, 16k out) | compat-mtp (Q4_K_S) | 32768 | 16384 |
| Qwopus-Coder GGUF non-MTP (64k coder, 32k out) | coder non-MTP (IQ4_XS) | 65536 | 32768 |
| Qwopus-Coder GGUF non-MTP (32k safe, 16k out) | coder non-MTP (IQ4_XS) | 32768 | 16384 |
| Qwopus-Coder-Compat-MTP GGUF (short, 16k) | compat-mtp (Q4_K_S) | 16384 | 8192 |

The launcher writes these sampler values (incl. `max_tokens`) to the LM Studio config preset `Qwopus Compat-MTP Coder` (`@local:qwopus-compat-mtp-coder`) at `~/.lmstudio/config-presets/` on every run, and runs a warm-up verification request that distinguishes "length-capped while thinking" from a genuine repeated-line loop.

---

## 4. Speed benchmark (GGUF Q4_K_S, llama.cpp via LM Studio, 100% GPU, ctx 65536)

Measured 2026-06-24 against the running server, SSE streaming, `repetition_penalty=1.05`, padded system prompt. `decode_tps` includes reasoning tokens (they are produced by the same decode loop). TTFT = time to first streamed chunk (includes prefill).

| Prefix | Prompt tok | Completion tok (reasoning) | TTFT | Prefill TPS | **Decode TPS** | Wall |
|---|---|---|---|---|---|---|
| ~4k  | 3394 | 327 (303 reasoning) | 10.9 s | 311 | **9.8** | 44.4 s |
| ~16k | 13834 | 415 (391 reasoning) | 101.3 s | 137 | **10.1** | 142.4 s |
| ~32k | 28234 | 436 (412 reasoning) | 168.6 s | 167 | **10.0** | 212.1 s |

**Decode throughput is ~10 TPS for the Compat-MTP (Q4_K_S), flat across context lengths.** This is the dense 27B running on llama.cpp's Metal backend.

### Non-MTP sibling — `Qwopus3.6-27B-Coder-GGUF` (IQ4_XS, 64k ctx, 100% GPU)

Same benchmark method, measured 2026-06-24. Uses the **original** chat template (no Compat tool-call hardening) and ships as IQ4_XS (15.23 GiB resident vs 16.76 GiB for the Compat-MTP Q4_K_S) — so the small speed difference vs Compat-MTP is partly the quant, not just MTP-head overhead.

| Prefix | Prompt tok | Completion tok (reasoning) | TTFT | Prefill TPS | **Decode TPS** | Wall |
|---|---|---|---|---|---|---|
| ~4k  | 3395  | 516 (493 reasoning) | 23.3 s  | 146 | **13.7** | 60.8 s |
| ~16k | 13835 | 1320 (1297 reasoning) | 82.9 s | 167 | **11.5** | 197.8 s |
| ~32k | 28235 | 148 (125 reasoning)   | 174.8 s | 161 | **8.9**  | 191.5 s |

**Non-MTP vs Compat-MTP (both llama.cpp Metal, 64k ctx):** ~13.7/11.5/8.9 TPS vs ~9.8/10.1/10.0 TPS. The non-MTP/IQ4_XS build is a bit faster at short context and roughly even at 32k, within run-to-run variance and the quant difference. Pick by template need, not speed: use Compat-MTP when you want the tool-call loop-hardened template; use the non-MTP coder when you don't need MTP/Compat and want the smaller IQ4_XS footprint (~0.5 GiB less resident).

### Comparison vs the same model family on oMLX (MLX)

| Backend | Decode TPS @4k | Decode TPS @32k | TTFT @32k | Notes |
|---|---|---|---|---|
| **llama.cpp Metal (GGUF Q4_K_S, this doc)** | ~10 | ~10 | 169 s | LM Studio runtime, 100% GPU |
| oMLX MLX (`Qwopus3.6-27B-Coder-MTP-4bit.mlx`, MTP on) | ~18.5 | ~16.5 | 334 s | MLX is better tuned for Apple Silicon dense models |
| oMLX MLX (`Qwopus3.6-27B-Coder-4bit.mlx`, base) | ~19.6 | ~15.4 | 478 s | |

**Honest take:** llama.cpp's Metal backend is ~40% slower at decode than MLX for this dense 27B on Apple Silicon. The Compat GGUF is the only way to get the *Compat chat-template* fixes (tool-call loop hardening, JSON-string/mapping arg interop) — if you don't need the Compat template fixes, the `.mlx` MTP variant on oMLX is ~1.7× faster decode. Use the GGUF/llama.cpp path when you specifically want the Compat release or the bundled MTP-via-llama.cpp speculative path.

---

## 5. Hint: running the GGUF directly with the llama.cpp binary

The `lms` CLI wraps the engine and is what the launcher uses. For debugging or to access flags the wrapper doesn't expose (notably **no-thinking via `--chat-template-kwargs`**, which LM Studio's server does **not** forward from the request body), you can launch the bundled `llama-server` binary directly:

```bash
LLAMA_DIR="$HOME/.lmstudio/extensions/backends/llama.cpp-mac-arm64-apple-metal-advsimd-2.23.0"
GGUF_DIR="$HOME/.lmstudio/models/Jackrong/Qwopus3.6-27B-Coder-Compat-MTP-GGUF"

# Raise the Metal wired cap first (same as the launcher)
sudo sysctl iogpu.wired_limit_mb=26624

"$LLAMA_DIR/llama-server" \
  -m "$GGUF_DIR/Qwopus3.6-27B-Coder-Compat-MTP-Q4_K_S.gguf" \
  --mmproj "$GGUF_DIR/mmproj-F32.gguf" \
  -c 65536 -n 32768 \
  -ngl 99 \
  -fa on \
  --jinja \
  --host 127.0.0.1 --port 8000 \
  --api-key oMLXapi
```

Key flags:

- `-ngl 99` — offload all layers to Metal (100% GPU).
- `-c 65536` — 64K context window (24.66 GiB; fits the 26 GB wired cap).
- `-n 32768` — default completion budget (Codex-sized output).
- `-fa on` — flash attention (default `auto`).
- `--jinja` — use the chat template embedded in the GGUF (the Compat template; required for correct tool-call rendering).
- `--mmproj …` — the vision projector. Add `--no-mmproj` instead if you do text-only and want to save the ~0.9 GB.

### No-thinking mode (only reachable with the direct binary)

The HF model card's headline SWE-bench run is **no-thinking**. LM Studio's `lms server` does **not** forward `chat_template_kwargs.enable_thinking` from the request body (verified: 4 request-body variants all still produced reasoning tokens). The standalone binary **does** support it via `--chat-template-kwargs`:

```bash
"$LLAMA_DIR/llama-server" \
  -m "$GGUF_DIR/Qwopus3.6-27B-Coder-Compat-MTP-Q4_K_S.gguf" \
  -c 65536 -ngl 99 -fa on --jinja \
  --chat-template-kwargs '{"enable_thinking": false}' \
  --reasoning-format deepthink \
  --host 127.0.0.1 --port 8000
```

With `enable_thinking=false`, the Compat chat template injects an empty `ILD…ILD_end` block and the model skips reasoning — this is the fast agentic-coding mode from the card (~100 tok/s on an RTX 5090 per the card; expect ~10–20 tok/s on M2 Max Metal). `--reasoning-format` controls how any residual thought tags are surfaced in the OpenAI response (`deepseek` → `message.reasoning_content`, `deepthink` → strips tags, `none` → raw).

---

## 6. About the "loop" symptom

The Compat release is specifically built to **fix** tool-call format loops (HF card: "0 loops / 10 cases", vs 3/10 with the official template). Sampling is rarely the cause. What can look like a loop:

1. **Under-sized `max_tokens`** — the thinker spends the whole budget reasoning → empty content + `finish_reason=length`. **Fix:** use the ★ 64k/32k-out config (≥32K output budget). Not a loop.
2. **Tool-call history serialization** — if a client serializes `tool_call.function.arguments` in a shape the template doesn't accept, the model can repeat a malformed call. The Compat template widens this (accepts JSON-string, mapping, and list args). Make sure your client sends well-formed tool calls.
3. **Genuine repetition** — rare; `repetition_penalty=1.05` is the safety net. Bump to `1.1` only if repeated-line loops persist (verified: 1.0, 1.05, 1.1 all answer correctly with adequate budget; higher values shorten reasoning).

The launcher's warm-up step reports `finish_reason`, `reasoning_tokens`, and a repeated-line heuristic so you can tell these apart.

---

## 7. Operational notes

- **Port:** `8000` (same as the oMLX launcher). Only one server runs at a time; the launcher stops anything on the port first.
- **API key:** `oMLXapi` (matches the oMLX launcher; LM Studio's server ignores auth unless you enable it in LM Studio's server settings).
- **Wired cap:** `sudo sysctl iogpu.wired_limit_mb=26624` after every reboot (volatile), or persist in `/etc/sysctl.conf`. Mandatory for 64k context.
- **MTP:** llama.cpp has no MTP on/off toggle. The model's bundled MTP/NextN heads (`nextnPredictLayers: 1`, `supportsMtp: true` in the GGUF metadata) are used automatically by the engine for speculative decoding when supported — nothing to enable from the CLI.
- **Engine selection:** `lms runtime ls` shows installed engines; the selected one for GGUF is `llama.cpp-mac-arm64-apple-metal-advsimd@2.23.0`.

---

## 8. Files

- Launcher: `launch_qwopus_llamacpp.sh` (GGUF / llama.cpp via LM Studio)
- Sibling launcher: `launch_qwopus_omlx.sh` (MLX / oMLX — cannot serve this GGUF)
- oMLX benchmark + tuning (MLX variants): `Qwopus3.6-27B-Coder-4bit_omlx_benchmark.md`
- M2 Max cross-model best-fit: `m2-max/summary.md` (updated with the GGUF/llama.cpp row)
