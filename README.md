# PerformanceMetrics
Performance metrics reference and calculator for distributed systems

How to think about these metrics as a group
---

## They fall into four layers, each answering a different question:
### Latency metrics (p50, p90, p95, p99, p99.9, TTFB, Apdex) answer: "How fast does the system feel?" The critical insight is that averages lie — a single 10-second timeout drags the mean far above what 99% of users experience. Always set SLOs on percentiles, not means. At 10,000 RPS, your p99 fires 100 times per second — those are real users hitting the tail.

### Throughput and concurrency (TPS, RPS, QPS, concurrent connections) answer: "How much work is the system doing?" These are your capacity metrics. Little's Law ties them together: concurrent connections = RPS × average response time. Double your response time and you double your connection count — which is why latency degradation triggers a cascade.

### Reliability metrics (availability, MTTR, MTBF, error rate, SLO/SLA, error budget) answer: "How trustworthy is the system?" Error budget is the most operationally useful — it converts an abstract SLO like "99.9%" into concrete minutes per month you're allowed to fail. When the budget burns fast, you stop deploying features and fix reliability.

### Resource metrics (CPU, memory, IOPS, bandwidth, cache hit ratio, saturation, GC pause, DB replication lag) answer: "What is the system straining against?" The USE method (Utilisation, Saturation, Errors) applied to every resource gives you a systematic checklist to find bottlenecks. Saturation is the most underused — a queue depth of 0 means requests aren't waiting, while a queue depth of 50 means you've hit a ceiling even if CPU looks fine.

## How each is actually measured in practice

### Percentile latencies are computed from histograms, not sorted arrays in production. Tools like Prometheus use HDR histograms (High Dynamic Range) that approximate percentiles in constant memory regardless of request volume. You configure bucket boundaries (0.1 ms, 1 ms, 10 ms, 100 ms, 1 s…) and the histogram counts how many observations fall in each bucket — the percentile is interpolated from those counts.

### Availability is calculated from your incident records: uptime / (uptime + downtime). The "nines" translation is what makes it concrete — 99.9% sounds great until you realize it's 8.76 hours of downtime per year, which is unacceptable for a payment system.

### Apdex is the most user-experience-oriented metric because it converts latency into a satisfaction score that non-engineers can read. You set a threshold T (e.g. 500 ms for your geo-search API), and requests under T count as satisfied, under 4T as tolerating, and over 4T as frustrated. An Apdex of 0.92 means "most users are satisfied, some are tolerating."
