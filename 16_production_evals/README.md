# 16. Production Evals

> **Purpose** — Turn evaluation from a pre-deployment activity into a **continuous monitoring loop** in production. This section covers shadow testing, canary releases, A/B testing, online evaluation, user feedback loops, drift detection, regression detection, and alerting — everything you need to maintain quality after deployment.

---

## Table of Contents

- [Why Production Evals Are Different](#why-production-evals-are-different)
- [The Evaluation Lifecycle](#the-evaluation-lifecycle)
- [Production Eval Strategies](#production-eval-strategies)
- [Shadow Testing](#shadow-testing)
- [Canary Releases](#canary-releases)
- [A/B Testing](#ab-testing)
- [Online Evaluation](#online-evaluation)
- [User Feedback Loops](#user-feedback-loops)
- [Drift Detection](#drift-detection)
- [Regression Detection](#regression-detection)
- [Alerting & Incident Response](#alerting--incident-response)
- [The Production Eval Flywheel](#the-production-eval-flywheel)
- [Common Mistakes](#common-mistakes)
- [Key Takeaways](#key-takeaways)

---

## Why Production Evals Are Different

| Dimension | Offline Evals | Production Evals |
|---|---|---|
| **Data** | Curated test sets | Real user inputs (messy, diverse, evolving) |
| **Stakes** | Low (dev/staging) | High (real users, real consequences) |
| **Feedback** | Immediate (run test, see result) | Delayed (users don't report every issue) |
| **Scale** | Hundreds of test cases | Thousands of requests per day |
| **Distribution** | Fixed, representative (hopefully) | Shifting, unpredictable |
| **Speed** | Minutes to hours | Must not add noticeable latency |

> Offline evals catch regressions you can predict. Production evals catch failures you didn't anticipate.

---

## The Evaluation Lifecycle

```
┌──────────────────────────────────────────────────────────────────┐
│                      EVALUATION LIFECYCLE                         │
│                                                                  │
│  Offline Evals ──> Shadow Test ──> Canary ──> Full Deploy        │
│       │                                           │              │
│       │          ┌─────────────────────────────────┘              │
│       │          v                                                │
│       │     Online Eval ──> Drift Detection ──> Alert ──> Fix    │
│       │          │                                    │           │
│       │          v                                    v           │
│       └──── User Feedback ──> Update Eval Dataset ──> Re-eval    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Production Eval Strategies

| Strategy | When to Use | Latency Impact | Risk Level |
|---|---|---|---|
| **Shadow testing** | Before any production traffic | None (parallel) | Zero risk |
| **Canary release** | Initial rollout (1–5% traffic) | None | Very low |
| **A/B testing** | Comparing two versions | None | Low (controlled) |
| **Online scoring** | Continuous quality monitoring | Async (no user impact) | None |
| **User feedback** | Ongoing quality signal | None | None |
| **Drift detection** | Monitoring for distribution changes | Async | None |

---

## Shadow Testing

Run the new system in parallel with the existing one, without serving the new system's output to users.

```
User Request ─┬──> Current System ──> Response (served to user)
              │
              └──> New System ──> Response (logged, not served)
                                        │
                                        v
                                   Compare & Score
```

### Shadow Test Metrics

| Metric | What It Tells You |
|---|---|
| **Quality delta** | Is the new system better or worse on real traffic? |
| **Latency comparison** | Is the new system faster or slower? |
| **Cost comparison** | Is the new system cheaper or more expensive? |
| **Error rate comparison** | Does the new system have more failures? |
| **Agreement rate** | How often do the two systems give the same answer? |

### Shadow Test Checklist

```
☐ New system runs on 100% of traffic (in shadow)
☐ Responses are logged but NOT served to users
☐ Quality scores computed for both systems
☐ Latency and cost tracked for both systems
☐ Run for at least 3-7 days to cover traffic patterns
☐ Analyze performance across slices (topic, user type, etc.)
```

---

## Canary Releases

Gradually route a small percentage of traffic to the new system.

```
Traffic Distribution:
  Day 1:  1% canary, 99% stable
  Day 3:  5% canary, 95% stable
  Day 5:  25% canary, 75% stable
  Day 7:  50% canary, 50% stable
  Day 10: 100% canary → new stable

Rollback trigger: Any metric drops below threshold
```

### Canary Metrics to Monitor

| Metric | Stable Baseline | Canary | Alert If |
|---|---|---|---|
| Quality score | 4.2/5.0 | Monitor | < 3.8 or > 5% regression |
| Error rate | 0.05% | Monitor | > 0.2% |
| Latency P95 | 2.1s | Monitor | > 3.0s |
| User complaint rate | 0.01% | Monitor | > 0.05% |
| Safety incidents | 0 | Monitor | > 0 → immediate rollback |

---

## A/B Testing

Split traffic between two system versions and measure which performs better.

### A/B Test Design

| Element | Details |
|---|---|
| **Hypothesis** | "New prompt v2 will increase task success by 5%" |
| **Control (A)** | Current production system |
| **Treatment (B)** | System with new prompt v2 |
| **Split** | 50/50 random assignment |
| **Primary metric** | Task success rate |
| **Secondary metrics** | Latency, cost, user satisfaction |
| **Duration** | Until statistical significance (typically 1–4 weeks) |
| **Minimum sample** | Depends on effect size (typically 1,000+ per group) |

### Statistical Considerations

| Consideration | Recommendation |
|---|---|
| **Significance level** | α = 0.05 (95% confidence) |
| **Power** | β = 0.80 (80% chance to detect real effect) |
| **Multiple comparisons** | Apply Bonferroni correction if testing multiple metrics |
| **Novelty effects** | Wait at least 1 week to avoid novelty bias |
| **Segment analysis** | Check performance across user segments, not just aggregate |

### A/B Test Results Template

| Metric | Control (A) | Treatment (B) | Delta | Significant? |
|---|---|---|---|---|
| Task success | 78.3% | 82.1% | +3.8% | ✅ (p=0.02) |
| Latency P95 | 2.1s | 2.3s | +0.2s | ❌ (p=0.15) |
| Cost/query | $0.008 | $0.009 | +12.5% | ✅ (p=0.01) |
| User rating | 4.1/5 | 4.3/5 | +0.2 | ✅ (p=0.03) |

**Decision**: Ship Treatment (B) — significant quality improvement with acceptable cost increase.

---

## Online Evaluation

Score a sample of production responses asynchronously, without affecting user experience.

### Online Scoring Architecture

```
User Request ──> System ──> Response ──> Serve to User
                                │
                                v (async)
                          ┌───────────┐
                          │   Sample   │
                          │   (5-10%) │
                          └─────┬─────┘
                                v
                          ┌───────────┐
                          │  LLM Judge │
                          │  + Rules   │
                          └─────┬─────┘
                                v
                          ┌───────────┐
                          │  Store &   │
                          │  Dashboard │
                          └───────────┘
```

### What to Score Online

| Dimension | Scoring Method | Frequency |
|---|---|---|
| **Response quality** | LLM judge (1–5) | 5–10% of requests |
| **Safety** | Safety classifier | 100% of requests |
| **Faithfulness** (RAG) | LLM judge + source verification | 10% of RAG requests |
| **Formatting** | Rule-based validation | 100% of requests |
| **Latency** | System metrics | 100% of requests |
| **Cost** | Token counting | 100% of requests |

---

## User Feedback Loops

### Feedback Collection Methods

| Method | Signal Quality | Volume | Implementation |
|---|---|---|---|
| **Thumbs up/down** | Low (binary) | High | Easy |
| **Star rating (1–5)** | Medium | Medium | Easy |
| **Written feedback** | High (detailed) | Low | Easy |
| **Regenerate button** | Medium (implicit dissatisfaction) | Medium | Easy |
| **Copy/share button** | Medium (implicit satisfaction) | Medium | Easy |
| **Task completion tracking** | High (behavioral) | High | Medium |
| **Follow-up surveys** | High | Low | Hard |

### Turning Feedback into Eval Data

```
User reports bad response
    │
    v
Manual review (triage)
    │
    ├── Systematic issue → Add to eval dataset + fix
    │
    ├── Edge case → Add to edge case eval set
    │
    └── User error / unreasonable expectation → Document, no action
```

### Feedback Metrics

| Metric | Definition | Target |
|---|---|---|
| **Thumbs up rate** | % of responses receiving positive feedback | > 85% |
| **Regeneration rate** | % of responses where user requests regeneration | < 10% |
| **Escalation rate** | % of conversations escalated to human | < 15% |
| **Session completion** | % of users who complete their task | > 70% |

---

## Drift Detection

### Types of Drift

| Drift Type | Description | Detection Method |
|---|---|---|
| **Input drift** | User queries change over time | Embedding distribution monitoring |
| **Output drift** | Model responses change (e.g., after provider update) | Quality score trend analysis |
| **Performance drift** | Quality degrades over time | Rolling window metric comparison |
| **Concept drift** | The meaning of "good" changes | Periodic human re-evaluation |
| **Data drift** | Knowledge base or retrieved data changes | Document freshness monitoring |

### Drift Detection Approaches

| Approach | How It Works | Alert Threshold |
|---|---|---|
| **Rolling window comparison** | Compare last 24h metrics vs last 30d baseline | > 2σ deviation |
| **Embedding clustering** | Monitor query embedding distribution shifts | Cluster composition change > 10% |
| **Quality score trend** | Track quality score moving average | Downward trend over 3+ days |
| **Novelty detection** | Flag queries unlike any in the eval set | Novelty score > threshold |

---

## Regression Detection

### Automated Regression Detection

```
For each metric:
  current = rolling_average(last_24h)
  baseline = rolling_average(last_30d)
  
  if current < baseline - threshold:
    ALERT: Regression detected
    DETAILS: metric, current_value, baseline_value, delta
```

### Regression Response Playbook

| Severity | Condition | Action |
|---|---|---|
| **P3 (Low)** | Quality score drops 2–5% | Investigate within 48h |
| **P2 (Medium)** | Quality score drops 5–10% OR error rate > 0.5% | Investigate within 4h |
| **P1 (High)** | Quality score drops >10% OR safety incident | Immediate investigation |
| **P0 (Critical)** | Multiple users reporting harm OR data breach | Rollback immediately |

---

## Alerting & Incident Response

### Alert Configuration

| Alert | Condition | Channel | Response Time |
|---|---|---|---|
| **Quality regression** | Quality score drops > 5% for 1h | Slack + PagerDuty | 4 hours |
| **Safety incident** | Safety classifier flags critical content | PagerDuty (P0) | 15 minutes |
| **Latency spike** | P95 > 2× baseline for 10 min | Slack | 30 minutes |
| **Error rate spike** | Error rate > 1% for 5 min | PagerDuty | 15 minutes |
| **Cost anomaly** | Daily cost > 2× average | Slack (daily) | Next business day |
| **Drift detected** | Significant distribution shift | Slack (weekly) | Next sprint |

---

## The Production Eval Flywheel

The most powerful production eval systems create a **virtuous cycle**:

```
Deploy ──> Monitor ──> Detect Issues ──> Add to Eval Dataset ──> Fix ──> Re-eval ──> Deploy
              │              │                    │
              v              v                    v
         User Feedback   Drift Alert        Dataset grows
                                            richer over time
```

Each production failure makes your eval suite stronger, which prevents future failures.

### Flywheel Metrics

| Metric | What It Measures | Target |
|---|---|---|
| **Mean time to detection** | How fast do we find issues? | < 1 hour |
| **Mean time to resolution** | How fast do we fix issues? | < 24 hours |
| **Eval set growth rate** | How fast does the eval set improve? | 10+ new cases/month |
| **Recurrence rate** | How often do fixed issues recur? | < 5% |

---

## Common Mistakes

| Mistake | Consequence | Fix |
|---|---|---|
| **No production monitoring** | Issues discovered by users, not by engineering | Deploy online scoring and alerting |
| **Monitoring only aggregate metrics** | Misses slice-specific regressions | Monitor per-slice and per-topic metrics |
| **No feedback loop to eval set** | Eval set becomes stale | Systematically add production failures to eval set |
| **A/B tests without sufficient sample size** | Inconclusive results, wrong decisions | Calculate required sample size before starting |
| **Not testing canary before full rollout** | Bad changes affect all users | Always use staged rollout |
| **Ignoring drift** | Slow degradation goes unnoticed | Implement automated drift detection |

---

## Key Takeaways

| Principle | Details |
|---|---|
| **Production evals are continuous** | Not a one-time activity — a permanent monitoring loop |
| **Use staged rollouts** | Shadow test → canary → A/B → full rollout |
| **Collect and use user feedback** | Users are your most important evaluators |
| **Detect drift proactively** | Don't wait for users to complain |
| **Build the flywheel** | Every production failure should strengthen your eval suite |
| **Alert early, alert precisely** | Right alerts to the right people at the right time |

---

Move to [17. Multimodal Evals →](../17_multimodal_evals/README.md) to learn how to evaluate systems that work with images, audio, video, and documents — not just text.
