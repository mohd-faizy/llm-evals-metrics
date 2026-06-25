# 15. Operational Evals

> **Purpose** — Evaluate the non-functional characteristics of LLM systems: latency, cost, throughput, reliability, stability, and scalability. A system that produces perfect answers but takes 30 seconds and costs $1 per query is not deployable. Operational evals ensure your system is production-ready beyond just quality.

---

## Table of Contents

- [Why Operational Evals Matter](#why-operational-evals-matter)
- [Quality vs Operational Tradeoffs](#quality-vs-operational-tradeoffs)
- [Operational Metrics](#operational-metrics)
- [Latency Evaluation](#latency-evaluation)
- [Cost Evaluation](#cost-evaluation)
- [Throughput & Scalability](#throughput--scalability)
- [Reliability & Stability](#reliability--stability)
- [Observability](#observability)
- [Setting Operational SLOs](#setting-operational-slos)
- [Operational Eval Pipeline](#operational-eval-pipeline)
- [Common Mistakes](#common-mistakes)
- [Key Takeaways](#key-takeaways)

---

## Why Operational Evals Matter

Quality evals answer: *"Is the output good?"*
Operational evals answer: *"Can we actually serve this in production?"*

| Scenario | Quality Score | Operational Reality | Deploy? |
|---|---|---|---|
| GPT-4 with CoT reasoning | 95% accuracy | P95 latency: 15s, cost: $0.50/query | ❌ Too slow/expensive |
| GPT-4o-mini with simple prompt | 82% accuracy | P95 latency: 1.2s, cost: $0.003/query | ✅ Maybe (depends on use case) |
| Fine-tuned Llama 3.1 8B | 88% accuracy | P95 latency: 0.4s, cost: $0.001/query | ✅ Great balance |

The right model is not always the highest-scoring one — it's the one that meets **all** your requirements.

---

## Quality vs Operational Tradeoffs

| Lever | Quality Impact | Operational Impact |
|---|---|---|
| Larger model | ↑ Better quality | ↓ Higher latency, higher cost |
| Chain-of-thought prompting | ↑ Better reasoning | ↓ 2–5× more tokens (cost + latency) |
| More retrieval chunks (k) | ↑ Better context coverage | ↓ More tokens, higher latency |
| Temperature > 0 | ↑ More creative outputs | ↓ Less consistent, harder to cache |
| Multiple LLM judge calls | ↑ More reliable scoring | ↓ Higher eval cost |
| Streaming | = Same quality | ↑ Better perceived latency |
| Caching | = Same quality (for cache hits) | ↑ Much lower latency + cost |

---

## Operational Metrics

### Core Metrics

| Metric | Definition | How to Measure | Target Example |
|---|---|---|---|
| **Latency (TTFT)** | Time to first token | API response timing | < 500ms |
| **Latency (total)** | Total time to complete response | End-to-end timing | P95 < 3s |
| **Throughput** | Requests handled per second | Load testing | > 100 QPS |
| **Cost per query** | Total $ per user interaction | Token count × price | < $0.05 |
| **Token usage** | Input + output tokens per request | API metering | Monitor distribution |
| **Error rate** | % of requests that fail | Error monitoring | < 0.1% |
| **Retry rate** | % of requests requiring retries | Retry counter | < 5% |
| **Availability** | % of time system is operational | Uptime monitoring | ≥ 99.9% |
| **Cache hit rate** | % of requests served from cache | Cache metrics | > 30% (varies) |

### Percentile-Based Latency

Always measure latency in **percentiles**, not averages:

| Metric | What It Tells You | Why It Matters |
|---|---|---|
| **P50 (median)** | Typical user experience | Baseline performance |
| **P90** | "Slow" user experience (1 in 10) | Regular users hit this |
| **P95** | Most common SLO target | Standard for alerting |
| **P99** | Worst-case for most users | Tail latency matters at scale |
| **P99.9** | Extreme outliers | May indicate infrastructure issues |
| **Average** | ⚠️ Misleading — hides tail latency | Don't use for SLOs |

### Latency Breakdown

```
Total Latency = Network + Retrieval + Model Inference + Post-Processing

Example breakdown:
├── Network:         50ms  (API call overhead)
├── Retrieval:      200ms  (vector search + reranking)
├── Model Inference: 800ms  (LLM generation)
└── Post-Processing:  50ms  (guardrails, formatting)
    ─────────────────────
    Total:          1,100ms
```

---

## Latency Evaluation

### Latency Testing Matrix

| Test Type | Purpose | How to Run |
|---|---|---|
| **Baseline latency** | Measure typical response time | 100 standard queries, measure P50/P95/P99 |
| **Load latency** | Latency under concurrent load | Ramp from 1 to 100+ concurrent users |
| **Streaming latency** | Time to first token + inter-token latency | Measure TTFT and TBT (time between tokens) |
| **Cold start latency** | First request after idle period | Measure after inactivity period |
| **Tail latency** | Worst-case response times | Focus on P99 and P99.9 |
| **Geographic latency** | Impact of user location | Test from multiple regions |

### Latency Optimization Strategies

| Strategy | Latency Reduction | Tradeoff |
|---|---|---|
| **Semantic caching** | 90%+ for cache hits | Stale responses if cache not invalidated |
| **Smaller model** | 50–80% | Lower quality |
| **Streaming** | Perceived latency drops 60–80% | Same total generation time |
| **Parallel tool calls** | Proportional to parallelizable work | More complex orchestration |
| **Edge deployment** | 30–50% (network reduction) | Infrastructure complexity |
| **Speculative decoding** | 20–40% | Requires draft model |
| **Reduced context length** | Proportional to token reduction | May lose important context |

---

## Cost Evaluation

### Cost Breakdown

| Cost Component | How to Calculate | Optimization |
|---|---|---|
| **Input tokens** | Σ input_tokens × price_per_input_token | Compress prompts, reduce context |
| **Output tokens** | Σ output_tokens × price_per_output_token | Set max_tokens, concise prompts |
| **Embedding** | Σ embedding_calls × price_per_call | Cache embeddings, batch requests |
| **Judge/eval** | Σ judge_calls × judge_token_cost | Layer scoring: cheap first |
| **Infrastructure** | Compute, storage, network | Right-size, auto-scale |

### Cost Per Query Calculator

```
Cost = (input_tokens × input_price) + (output_tokens × output_price)
     + (embedding_tokens × embed_price) + (retrieved_chunks × chunk_cost)

Example (GPT-4o):
  1,000 input tokens × $2.50/M  = $0.0025
  500 output tokens × $10.00/M  = $0.005
  1 embedding call               = $0.0001
  ─────────────────────────────────────
  Total per query                = ~$0.008
  Daily (10K queries)            = ~$80
  Monthly (300K queries)         = ~$2,400
```

### Model Cost Comparison

| Model | Input ($/M tokens) | Output ($/M tokens) | Quality | Cost Rating |
|---|---|---|---|---|
| GPT-4o | $2.50 | $10.00 | High | $$$ |
| Claude 3.5 Sonnet | $3.00 | $15.00 | High | $$$ |
| GPT-4o-mini | $0.15 | $0.60 | Medium | $ |
| Claude 3.5 Haiku | $0.80 | $4.00 | Medium | $$ |
| Gemini 1.5 Flash | $0.075 | $0.30 | Medium | $ |
| Llama 3.1 70B (self-hosted) | Infra cost | Infra cost | Medium-High | $–$$ |

---

## Throughput & Scalability

### Throughput Metrics

| Metric | Definition | How to Test |
|---|---|---|
| **QPS (queries per second)** | Maximum queries handled per second | Load test with increasing concurrency |
| **Tokens per second** | Output generation speed | Measure during load test |
| **Concurrent users** | Max simultaneous users without degradation | Ramp up until latency exceeds SLO |
| **Queue depth** | Number of requests waiting to be processed | Monitor during load test |

### Scalability Testing

| Test | What It Reveals | How to Run |
|---|---|---|
| **Horizontal scaling** | Does adding instances help? | Deploy 1x, 2x, 4x instances, measure throughput |
| **Burst handling** | Can the system handle traffic spikes? | Sudden 10× load increase |
| **Sustained load** | Does performance degrade over time? | Run at 80% capacity for hours |
| **Auto-scaling** | How quickly do new instances come up? | Trigger scale event, measure time to healthy |

---

## Reliability & Stability

### Reliability Metrics

| Metric | Definition | Target |
|---|---|---|
| **Uptime** | % of time the system is available | ≥ 99.9% (8.76 hours downtime/year) |
| **Error rate** | % of requests returning errors | < 0.1% |
| **Retry success rate** | % of retried requests that eventually succeed | > 95% |
| **Degradation rate** | % of time the system runs in degraded mode | < 1% |
| **Recovery time** | Time to recover from failures | < 5 minutes |

### Stability Testing

| Test | What It Reveals |
|---|---|
| **Provider failover** | Does the system handle API provider outages? |
| **Rate limit handling** | Does the system gracefully handle rate limits? |
| **Timeout handling** | Does the system handle slow responses correctly? |
| **Consistency across runs** | Same input → similar output quality? |
| **Model update stability** | Does a model version change affect quality/latency? |

---

## Observability

### What to Observe

| Signal | Tool | Purpose |
|---|---|---|
| **Latency per request** | APM (Datadog, New Relic) | Detect slow requests |
| **Token usage per request** | LLM observability (Langfuse, Helicone) | Cost monitoring |
| **Error types** | Error tracking (Sentry) | Diagnose failures |
| **Model response quality** | LLM scoring (Braintrust) | Detect quality drift |
| **Cache performance** | Cache metrics | Optimize cache strategy |
| **Throughput** | Load balancer metrics | Capacity planning |

### Dashboard Essentials

```
╔══════════════════════════════════════════════════╗
║  OPERATIONAL DASHBOARD                           ║
╠══════════════════════════════════════════════════╣
║  Requests: 12,847 today   | Errors: 3 (0.02%)  ║
║  P50 Latency: 1.1s        | P95 Latency: 2.8s  ║
║  Cost Today: $102.40       | Avg $/query: $0.008║
║  Cache Hit Rate: 34%       | Throughput: 45 QPS ║
║  Availability: 99.98%      | Active Models: 2   ║
╚══════════════════════════════════════════════════╝
```

---

## Setting Operational SLOs

### SLO Template

| SLO | Target | Measurement Window | Alert Threshold |
|---|---|---|---|
| Latency P95 | < 3s | Rolling 5 minutes | > 4s |
| Latency P99 | < 8s | Rolling 5 minutes | > 10s |
| Error rate | < 0.1% | Rolling 1 hour | > 0.5% |
| Availability | ≥ 99.9% | Rolling 30 days | < 99.5% |
| Cost per query (avg) | < $0.05 | Daily | > $0.08 |
| Throughput | > 50 QPS | Rolling 5 minutes | < 30 QPS |

### SLO Severity Levels

| Level | Condition | Action |
|---|---|---|
| **INFO** | Approaching SLO boundary (warning) | Log, notify team |
| **WARNING** | SLO breached for < 5 minutes | Alert on-call, investigate |
| **CRITICAL** | SLO breached for > 5 minutes | Page on-call, start incident response |
| **EMERGENCY** | Multiple SLOs breached simultaneously | All-hands incident, consider rollback |

---

## Operational Eval Pipeline

### Integrating Operational Evals

```
Pre-deployment:
  ├── Load test → Verify throughput meets SLO
  ├── Latency test → Verify P95/P99 latency
  └── Cost estimate → Project monthly cost

Post-deployment:
  ├── Monitor latency, errors, cost in real-time
  ├── Alert on SLO violations
  └── Weekly operational review
```

---

## Common Mistakes

| Mistake | Consequence | Fix |
|---|---|---|
| **Using average latency** | Hides tail latency spikes | Always use percentiles (P95, P99) |
| **Not testing under load** | Works at 1 QPS, fails at 50 QPS | Load test before deployment |
| **Ignoring cost until the bill arrives** | Budget surprises | Estimate and monitor cost proactively |
| **No alerting** | Issues discovered by users, not by engineering | Set up SLO-based alerts |
| **Optimizing only for quality** | System is accurate but unusable (too slow/expensive) | Balance quality and operational metrics |
| **Not testing provider failover** | Single point of failure | Test with provider outage simulation |

---

## Key Takeaways

| Principle | Details |
|---|---|
| **Operational evals are deployment gates** | Quality alone doesn't make a system production-ready |
| **Use percentiles, not averages** | P95 and P99 latency reveal the real user experience |
| **Monitor cost continuously** | LLM costs can spike unexpectedly |
| **Set SLOs early** | Define latency, cost, and error rate targets before building |
| **Test under load** | Systems that work at 1 QPS may fail at 100 QPS |
| **Balance quality vs cost vs latency** | The best model is the one that meets all your requirements |

---

Move to [16. Production Evals →](../16_production_evals/README.md) to learn how to turn evaluation into a continuous monitoring loop with A/B testing, shadow scoring, and drift detection.
