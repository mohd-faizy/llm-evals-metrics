# 03. Foundation Model Benchmarks

> **Purpose** — Understand the major public benchmarks used to evaluate foundation models, how to read their scores, and why they must be complemented by application-level evals before shipping.

---

## Table of Contents

- [Why Benchmarks Matter](#why-benchmarks-matter)
- [Benchmark Categories](#benchmark-categories)
- [Key Benchmarks at a Glance](#key-benchmarks-at-a-glance)
- [Benchmark Deep Dives](#benchmark-deep-dives)
- [Interpreting Benchmark Scores](#interpreting-benchmark-scores)
- [Common Pitfalls](#common-pitfalls)
- [Benchmarks vs Application Evals](#benchmarks-vs-application-evals)
- [Further Reading](#further-reading)

---

## Why Benchmarks Matter

Benchmarks provide a **standardized, reproducible signal** for comparing models across capabilities. They serve three primary purposes:

1. **Model selection** — Narrow the field before running your own evals.
2. **Regression detection** — Track whether a model update improves or degrades core capabilities.
3. **Community alignment** — Give researchers a shared vocabulary to discuss progress.

> ⚠️ **Benchmarks are proxies, not ground truth.** A model that tops a leaderboard can still fail catastrophically on your specific task, distribution, or risk profile.

---

## Benchmark Categories

| Category | What It Tests | Example Benchmarks |
|---|---|---|
| **Reasoning** | Logical deduction, multi-step problem solving | GPQA, ARC, BBH |
| **Knowledge** | Factual recall across domains | MMLU, MMLU-Pro |
| **Math** | Numerical computation, proof-style reasoning | GSM8K, MATH, MathVista |
| **Coding** | Code generation, bug fixing, repository-level tasks | HumanEval, MBPP, SWE-Bench |
| **Truthfulness** | Avoiding common misconceptions & hallucinations | TruthfulQA |
| **Language Understanding** | Sentence completion, NLI, commonsense | HellaSwag, WinoGrande, ARC |
| **Long Context** | Retrieval & reasoning over large input windows | RULER, Needle-in-a-Haystack, LongBench |
| **Multimodal** | Vision-language understanding, visual QA | MMMU, MathVista, VQAv2 |
| **Instruction Following** | Adherence to complex, multi-constraint instructions | IFEval, MT-Bench |
| **Safety & Alignment** | Refusal of harmful prompts, bias measurement | ToxiGen, BBQ, HarmBench |
| **Agentic / Tool Use** | Multi-step tool calling, environment interaction | SWE-Bench, WebArena, ToolBench |

---

## Key Benchmarks at a Glance

| Benchmark | Category | Format | Metric | Approx. Scale | Notes |
|---|---|---|---|---|---|
| **MMLU** | Knowledge | 57-subject MCQ | Accuracy (%) | 14 k questions | De facto standard; watch for data contamination |
| **MMLU-Pro** | Knowledge | Harder MCQ (10 options) | Accuracy (%) | 12 k questions | Reduced guessing via more answer choices |
| **GPQA** | Reasoning | Graduate-level science MCQ | Accuracy (%) | 448 questions | "Diamond" subset is expert-validated |
| **ARC** (Challenge) | Reasoning | Grade-school science MCQ | Accuracy (%) | 2.6 k questions | Tests commonsense + science reasoning |
| **HellaSwag** | Language | Sentence completion MCQ | Accuracy (%) | 10 k questions | Near-saturated; useful as a baseline |
| **TruthfulQA** | Truthfulness | Open-ended + MCQ | % truthful | 817 questions | Tests resistance to popular misconceptions |
| **HumanEval** | Coding | Function completion (Python) | pass@k | 164 problems | Simple functions; use HumanEval+ for stricter tests |
| **MBPP** | Coding | Short Python programs | pass@k | 974 problems | Broader than HumanEval |
| **SWE-Bench** | Coding (Agentic) | Real GitHub issues → patches | % resolved | 2.3 k instances | Gold standard for agentic coding; use "Verified" subset |
| **GSM8K** | Math | Grade-school word problems | Accuracy (%) | 8.5 k problems | Near-saturated for frontier models |
| **MATH** | Math | Competition-level problems | Accuracy (%) | 12.5 k problems | Levels 1-5 difficulty |
| **LiveBench** | Mixed | Regularly refreshed questions | Composite score | Varies monthly | Designed to resist contamination |
| **MT-Bench** | Instruction | Multi-turn conversation | LLM-judge score (1-10) | 80 questions | Uses GPT-4 as judge |
| **IFEval** | Instruction | Verifiable constraint following | Accuracy (%) | 541 prompts | Objective, rule-based scoring |
| **MMMU** | Multimodal | College-level vision MCQ | Accuracy (%) | 11.5 k questions | Spans 30 subjects |
| **Needle-in-a-Haystack** | Long Context | Retrieval from long docs | Recall (%) | Configurable length | Tests effective context window |

---

## Benchmark Deep Dives

### Reasoning — GPQA

- **Full name**: Graduate-Level Google-Proof QA
- **What it tests**: Expert-level reasoning in biology, chemistry, and physics
- **Why it matters**: Questions are hard enough that domain experts without the specific sub-specialty score ~65%, making it a strong ceiling test
- **Key variant**: The **GPQA Diamond** subset (198 questions) is double-validated by experts and is the most cited split

### Knowledge — MMLU / MMLU-Pro

- **MMLU** (Massive Multitask Language Understanding) covers 57 subjects from abstract algebra to world religions
- **Known issues**: Some answer keys contain errors; 4-option MCQ allows ~25% random-guess baseline
- **MMLU-Pro** mitigates this by using 10 options per question and integrating more reasoning-intensive questions, making it more discriminating for frontier models

### Coding — HumanEval & SWE-Bench

| Aspect | HumanEval | SWE-Bench (Verified) |
|---|---|---|
| Scope | Single function | Full repository patch |
| Language | Python only | Multi-language |
| Interaction | One-shot generation | Agentic (multi-step) |
| Realism | Low (toy problems) | High (real GitHub issues) |
| Metric | pass@1 / pass@10 | % Resolved |

- **HumanEval** is easy to run but does not reflect production coding tasks
- **SWE-Bench Verified** (500 instances, human-validated) is the most reliable subset for comparing agentic coding systems

### Math — GSM8K & MATH

- **GSM8K**: Grade-school word problems. Frontier models now score >95%, making it primarily a sanity check
- **MATH**: Competition-level problems across 7 subjects (algebra, geometry, number theory, etc.). Scored by difficulty level (1-5). Much more discriminating than GSM8K

### Truthfulness — TruthfulQA

- Tests whether a model reproduces common human misconceptions (e.g., "We only use 10% of our brains")
- Scored in two dimensions: **truthfulness** and **informativeness** — a model that always says "I don't know" would be truthful but not informative
- Important limitation: the 817-question set is static and likely contaminated in modern training data

### Long Context — Needle-in-a-Haystack & RULER

- **Needle-in-a-Haystack**: Inserts a target fact at various positions within a long document and tests recall. Simple but effective as a smoke test
- **RULER**: More comprehensive — tests multi-hop reasoning, variable tracking, and aggregation across long contexts, not just retrieval
- **Key insight**: A model may pass retrieval tests at 128K tokens but fail reasoning tasks at the same length

### Contamination-Resistant — LiveBench

- Refreshes questions monthly using recent data sources
- Avoids LLM judges entirely — uses objective, verifiable answers
- Categories: math, coding, reasoning, language, instruction following, data analysis
- **Why it matters**: Addresses the biggest weakness of static benchmarks — training data contamination

---

## Interpreting Benchmark Scores

### Dos ✅

| Practice | Why |
|---|---|
| Compare models on the **same benchmark version and split** | Scores across versions are not comparable |
| Check the **evaluation harness** used (lm-eval, simple-evals, etc.) | Prompt format and sampling settings can shift scores by 5-10% |
| Look at **confidence intervals** or variance across runs | Single-run numbers can be noisy, especially on small benchmarks |
| Prefer **contamination-resistant** benchmarks for frontier comparisons | Static benchmarks get memorized over time |
| Weight benchmarks **closest to your use case** | A coding benchmark matters more if you're building a coding tool |

### Don'ts ❌

| Anti-Pattern | Why It's Dangerous |
|---|---|
| Ranking models by a **single benchmark** | Models have different strength profiles |
| Trusting **self-reported scores** without reproduction | Prompt engineering and cherry-picked configs inflate results |
| Ignoring **saturation** | When top models cluster at 95%+, the benchmark loses discriminative power |
| Conflating **benchmark performance with production readiness** | Benchmarks test capability, not reliability under your constraints |
| Using **exact-match accuracy on open-ended tasks** | Penalizes valid alternative phrasings |

---

## Common Pitfalls

### 1. Data Contamination
Models trained on web-scale corpora may have memorized benchmark questions. Scores on older benchmarks (MMLU, HellaSwag, GSM8K) are increasingly unreliable as true measures of capability.

**Mitigation**: Prefer benchmarks with anti-contamination mechanisms (LiveBench, private held-out splits) or design your own eval sets.

### 2. Prompt Sensitivity
A model's score can change dramatically based on:
- System prompt phrasing
- Few-shot example selection and ordering
- Answer extraction method (regex, constrained decoding, etc.)

**Mitigation**: Always document the exact prompt template and parsing strategy used.

### 3. Metric Gaming
- **pass@k with high k** inflates coding scores (generating 100 samples and checking if any pass is very different from getting it right on the first try)
- **Chain-of-thought prompting** can boost MCQ accuracy but may not transfer to real tasks

**Mitigation**: Report pass@1 as the primary metric. Be explicit about prompting strategy.

### 4. Benchmark Saturation
When the top 10 models all score within 1-2% of each other, the benchmark is no longer useful for discriminating between them.

| Benchmark | Saturation Status (Mid-2025) |
|---|---|
| HellaSwag | ⚠️ Saturated (>95%) |
| GSM8K | ⚠️ Near-saturated for frontier models |
| MMLU | ⚠️ Approaching saturation |
| MATH | ✅ Still discriminating |
| GPQA Diamond | ✅ Still discriminating |
| SWE-Bench Verified | ✅ Still discriminating |
| LiveBench | ✅ Refreshes regularly |

---

## Benchmarks vs Application Evals

This is the central message of this module:

| Dimension | Benchmarks | Application Evals |
|---|---|---|
| **Purpose** | Compare general model capabilities | Validate fitness for your specific product |
| **Data** | Public, standardized | Private, task-specific |
| **Distribution** | Academic / synthetic | Your actual user inputs |
| **Metrics** | Accuracy, pass@k | Task success, user satisfaction, cost, latency |
| **Contamination Risk** | High (public data) | Low (private data) |
| **Who runs them** | Model providers, researchers | Your team |
| **When to use** | Model selection & tracking | Before every deployment |

> **Bottom line**: Use benchmarks to **shortlist models**. Use application evals to **make shipping decisions**. Never ship based on benchmark scores alone.

---

## Leaderboard Resources

| Resource | URL | Notes |
|---|---|---|
| **Chatbot Arena (LMSYS)** | [lmarena.ai](https://lmarena.ai/) | ELO-based human preference ranking |
| **Open LLM Leaderboard** | [Hugging Face](https://huggingface.co/spaces/open-llm-leaderboard/open_llm_leaderboard) | Open-weight model comparisons |
| **LiveBench** | [livebench.ai](https://livebench.ai/) | Contamination-resistant, refreshed monthly |
| **SWE-Bench Leaderboard** | [swebench.com](https://www.swebench.com/) | Agentic coding leaderboard |
| **Artificial Analysis** | [artificialanalysis.ai](https://artificialanalysis.ai/) | Speed, cost, and quality comparisons |
| **SEAL Leaderboards** | [scale.com/leaderboard](https://scale.com/leaderboard) | Expert-human evaluated rankings |

---

## Further Reading

- 📄 [Measuring Massive Multitask Language Understanding](https://arxiv.org/abs/2009.03300) — Original MMLU paper
- 📄 [GPQA: A Graduate-Level Google-Proof QA Benchmark](https://arxiv.org/abs/2311.12022) — GPQA paper
- 📄 [SWE-bench: Can Language Models Resolve Real-World GitHub Issues?](https://arxiv.org/abs/2310.06770) — SWE-Bench paper
- 📄 [Holistic Evaluation of Language Models (HELM)](https://arxiv.org/abs/2211.09110) — Stanford's comprehensive evaluation framework
- 📄 [Chatbot Arena: An Open Platform for Evaluating LLMs by Human Preference](https://arxiv.org/abs/2403.04132) — LMSYS Arena paper
- 📄 [LiveBench: A Challenging, Contamination-Free LLM Benchmark](https://arxiv.org/abs/2406.19314) — LiveBench paper
- 📝 [How to Read LLM Benchmarks](https://www.latent.space/p/benchmarks) — Practical guide from Latent Space

---

## Guidance

Benchmarks are useful signals, but they do not replace application evals. A model that wins a benchmark can still fail your product if the task, distribution, or risk profile is different.

**The evaluation workflow should be:**

```
Benchmarks → Shortlist → Application Evals → Ship/No-Ship Decision
```

Move to [04. Application Evals →](../04_application_evals/README.md) to learn how to build the eval sets that actually drive shipping decisions.
