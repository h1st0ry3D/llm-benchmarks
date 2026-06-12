# concurrency

**guidellm**: 0.6.0  **profile**: sweep  **model**: http://192.168.0.99:8080  **data**: ['kind=synthetic_text,prompt_tokens=128,output_tokens=128']  **duration**: 30.0s  **rate**: [5.0]

| strategy                | duration | requests                | rps  | tok/s  | TPOT (ms/tok) | prompt_tok | output_tok |
| ----------------------- | -------- | ----------------------- | ---- | ------ | ------------- | ---------- | ---------- |
| synchronous (workers=1) | 30.0s    | 22/22 (err=0, inc=0)    | 0.70 | 98.93  | 10.73         | 143        | 128        |
| throughput (workers=10) | 30.0s    | 22/533 (err=0, inc=511) | 0.70 | 95.11  | 129.30        | 144        | 128        |
| constant (workers=10)   | 30.0s    | 15/21 (err=0, inc=6)    | 0.50 | 100.01 | 77.12         | 144        | 128        |
| constant (workers=10)   | 30.0s    | 20/21 (err=0, inc=1)    | 0.67 | 93.22  | 11.31         | 143        | 128        |
| constant (workers=10)   | 30.0s    | 20/21 (err=0, inc=1)    | 0.67 | 95.01  | 11.06         | 144        | 128        |

### Strategy: synchronous (workers=1)

**Requests/s**: mean=0.70, p50=0.73, p95=0.83, p99=0.84, min=0.00, max=0.84, std=0.17

**Prompt Tokens**: mean=143.32, p50=143.00, p95=145.00, p99=146.00, min=139.00, max=146.00, std=1.52

**Output Tokens**: mean=128.00, p50=128.00, p95=128.00, p99=128.00, min=128.00, max=128.00, std=0.00

**TPOT (ms/tok)**: mean=10.73, p50=10.72, p95=13.33, p99=13.68, min=8.62, max=13.68, std=1.22

**Output Tokens/s**: mean=98.93, p50=93.53, p95=107.47, p99=187.45, min=76.71, max=187.45, std=20.94

**Prompt Tokens/s**: mean=110.76, p50=105.22, p95=120.76, p99=213.81, min=85.10, max=213.81, std=23.75

**Tokens/s**: mean=209.69, p50=198.74, p95=227.37, p99=401.26, min=161.81, max=401.26, std=44.68

**Scheduler**: queued_avg=5.07s, request_avg=1.37s, resolve_avg=1.31s

### Strategy: throughput (workers=10)

**Requests/s**: mean=0.70, p50=0.71, p95=0.79, p99=1.14, min=0.00, max=1.14, std=0.15

**Prompt Tokens**: mean=143.59, p50=143.00, p95=146.00, p99=146.00, min=141.00, max=146.00, std=1.47

**Output Tokens**: mean=128.00, p50=128.00, p95=128.00, p99=128.00, min=128.00, max=128.00, std=0.00

**TPOT (ms/tok)**: mean=129.30, p50=125.28, p95=231.21, p99=243.20, min=11.93, max=243.20, std=69.29

**Output Tokens/s**: mean=95.11, p50=91.47, p95=158.23, p99=166.88, min=82.45, max=166.88, std=16.63

**Prompt Tokens/s**: mean=106.69, p50=102.89, p95=176.77, p99=190.35, min=91.46, max=190.35, std=19.14

**Tokens/s**: mean=201.80, p50=193.66, p95=335.00, p99=357.23, min=173.91, max=357.23, std=35.75

**Scheduler**: queued_avg=1.16s, request_avg=16.55s, resolve_avg=29.75s

### Strategy: constant (workers=10)

**Requests/s**: mean=0.50, p50=0.67, p95=0.85, p99=1.06, min=0.00, max=1.06, std=0.33

**Prompt Tokens**: mean=143.73, p50=144.00, p95=146.00, p99=146.00, min=141.00, max=146.00, std=1.29

**Output Tokens**: mean=128.00, p50=128.00, p95=128.00, p99=128.00, min=128.00, max=128.00, std=0.00

**TPOT (ms/tok)**: mean=77.12, p50=77.48, p95=81.65, p99=81.65, min=74.27, max=81.65, std=1.77

**Output Tokens/s**: mean=100.01, p50=93.03, p95=166.42, p99=228.39, min=83.90, max=228.39, std=27.48

**Prompt Tokens/s**: mean=112.30, p50=104.65, p95=189.82, p99=253.37, min=94.67, max=253.37, std=30.64

**Tokens/s**: mean=212.31, p50=197.68, p95=356.24, p99=481.77, min=178.95, max=481.77, std=58.11

**Scheduler**: queued_avg=17.42s, request_avg=9.87s, resolve_avg=8.10s

### Strategy: constant (workers=10)

**Requests/s**: mean=0.67, p50=0.69, p95=0.75, p99=0.76, min=0.00, max=0.76, std=0.14

**Prompt Tokens**: mean=143.15, p50=143.00, p95=145.00, p99=146.00, min=140.00, max=146.00, std=1.53

**Output Tokens**: mean=128.00, p50=128.00, p95=128.00, p99=128.00, min=128.00, max=128.00, std=0.00

**TPOT (ms/tok)**: mean=11.31, p50=11.46, p95=12.82, p99=13.10, min=9.76, max=13.10, std=0.99

**Output Tokens/s**: mean=93.22, p50=88.29, p95=164.73, p99=174.73, min=80.61, max=174.73, std=18.95

**Prompt Tokens/s**: mean=104.25, p50=99.83, p95=180.18, p99=196.58, min=91.95, max=196.58, std=20.85

**Tokens/s**: mean=197.47, p50=187.95, p95=344.91, p99=371.31, min=172.56, max=371.31, std=39.79

**Scheduler**: queued_avg=17.42s, request_avg=1.45s, resolve_avg=1.38s

### Strategy: constant (workers=10)

**Requests/s**: mean=0.67, p50=0.69, p95=0.77, p99=0.83, min=0.00, max=0.83, std=0.16

**Prompt Tokens**: mean=143.60, p50=144.00, p95=145.00, p99=145.00, min=140.00, max=145.00, std=1.24

**Output Tokens**: mean=128.00, p50=128.00, p95=128.00, p99=128.00, min=128.00, max=128.00, std=0.00

**TPOT (ms/tok)**: mean=11.06, p50=10.80, p95=13.21, p99=13.59, min=9.27, max=13.59, std=1.19

**Output Tokens/s**: mean=95.01, p50=91.31, p95=150.38, p99=172.91, min=82.68, max=172.91, std=17.59

**Prompt Tokens/s**: mean=106.58, p50=102.01, p95=170.35, p99=191.82, min=90.43, max=191.82, std=19.68

**Tokens/s**: mean=201.59, p50=193.32, p95=320.73, p99=364.72, min=173.11, max=364.72, std=37.27

**Scheduler**: queued_avg=17.42s, request_avg=1.42s, resolve_avg=1.35s

---
