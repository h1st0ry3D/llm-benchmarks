# throughput

**guidellm**: 0.6.0  **profile**: throughput  **model**: http://192.168.0.99:8080  **data**: ['kind=synthetic_text,prompt_tokens=128,output_tokens=128']  **duration**: 30.0s  **rate**: [10.0]

| strategy                | duration | requests             | rps  | tok/s  | TPOT (ms/tok) | prompt_tok | output_tok |
| ----------------------- | -------- | -------------------- | ---- | ------ | ------------- | ---------- | ---------- |
| throughput (workers=10) | 30.0s    | 23/32 (err=0, inc=9) | 0.73 | 100.26 | 84.66         | 143        | 128        |

### Strategy: throughput (workers=10)

**Requests/s**: mean=0.73, p50=0.75, p95=0.81, p99=1.10, min=0.00, max=1.10, std=0.16

**Prompt Tokens**: mean=143.30, p50=144.00, p95=145.00, p99=145.00, min=139.00, max=145.00, std=1.43

**Output Tokens**: mean=128.00, p50=128.00, p95=128.00, p99=128.00, min=128.00, max=128.00, std=0.00

**TPOT (ms/tok)**: mean=84.66, p50=102.38, p95=108.00, p99=108.91, min=10.75, max=108.91, std=31.41

**Output Tokens/s**: mean=100.26, p50=96.15, p95=103.37, p99=227.99, min=83.62, max=227.99, std=23.13

**Prompt Tokens/s**: mean=112.25, p50=108.45, p95=117.10, p99=256.49, min=92.77, max=256.49, std=26.29

**Tokens/s**: mean=212.51, p50=204.48, p95=220.47, p99=484.48, min=176.39, max=484.48, std=49.41

**Scheduler**: queued_avg=12.04s, request_avg=10.84s, resolve_avg=9.32s

---
