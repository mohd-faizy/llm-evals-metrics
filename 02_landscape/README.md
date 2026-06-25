# 02. Evaluation Landscape

> **Purpose** — Map the evaluation ecosystem: frameworks, observability platforms, benchmark suites, and judge model options. The goal is not to memorize tools but to understand the *classes* of systems available and when each one is useful.

---

## Table of Contents

- [Ecosystem Overview](#ecosystem-overview)
- [Evaluation Frameworks](#evaluation-frameworks)
- [Observability & Tracing Platforms](#observability--tracing-platforms)
- [Benchmark Suites](#benchmark-suites)
- [Judge Model Options](#judge-model-options)
- [Framework Selection Guide](#framework-selection-guide)
- [Strengths & Limitations Matrix](#strengths--limitations-matrix)
- [Integration Patterns](#integration-patterns)
- [Further Reading](#further-reading)

---

## Ecosystem Overview

The LLM evaluation ecosystem can be divided into four categories:

```
┌──────────────────────────────────────────────────────────────┐
│                    EVALUATION ECOSYSTEM                       │
├──────────────┬──────────────┬───────────────┬────────────────┤
│  Eval        │ Observability│  Benchmark    │  Judge         │
│  Frameworks  │ & Tracing    │  Suites       │  Models        │
├──────────────┼──────────────┼───────────────┼────────────────┤
│ Run & score  │ Monitor &    │ Standardized  │ LLMs used to   │
│ eval         │ trace live   │ test sets for │ score other    │
│ pipelines    │ systems      │ comparison    │ LLM outputs    │
└──────────────┴──────────────┴───────────────┴────────────────┘
```

---

## Evaluation Frameworks

Evaluation frameworks provide the infrastructure to define, run, and score eval suites programmatically.

### Comparison Table

| Framework | Type | Language | LLM Judge | Custom Metrics | CI/CD | Key Differentiator |
|---|---|---|---|---|---|---|
| **[OpenAI Evals](https://github.com/openai/evals)** | Open-source | Python | ✅ | ✅ | ✅ | Large community, simple YAML-based eval definitions |
| **[Braintrust](https://www.braintrust.dev/)** | Commercial | Python/TS | ✅ | ✅ | ✅ | Real-time scoring, experiment tracking, dataset management |
| **[Promptfoo](https://promptfoo.dev/)** | Open-source | Node.js | ✅ | ✅ | ✅ | Model-agnostic, red-teaming, CLI-first |
| **[DeepEval](https://github.com/confident-ai/deepeval)** | Open-source | Python | ✅ | ✅ | ✅ | Pytest integration, 14+ built-in metrics |
| **[Ragas](https://ragas.io/)** | Open-source | Python | ✅ | ✅ | ⚠️ | Purpose-built for RAG evaluation |
| **[Inspect AI](https://inspect.ai-safety-institute.org.uk/)** | Open-source | Python | ✅ | ✅ | ✅ | UK AISI-backed, task-centric design, sandboxed execution |
| **[Eleuther LM Eval Harness](https://github.com/EleutherAI/lm-evaluation-harness)** | Open-source | Python | ❌ | ✅ | ⚠️ | De facto standard for academic benchmarks |
| **[HELM](https://crfm.stanford.edu/helm/)** | Open-source | Python | ❌ | ✅ | ❌ | Stanford's holistic multi-metric evaluation |
| **[MLflow LLM Evaluate](https://mlflow.org/)** | Open-source | Python | ✅ | ✅ | ✅ | Integrates with MLflow experiment tracking |

### Framework Categories

| Category | Frameworks | Best For |
|---|---|---|
| **General-purpose eval** | Braintrust, Promptfoo, DeepEval | Application evals, prompt testing, regression |
| **RAG-specific** | Ragas, TruLens | Retrieval + generation quality |
| **Academic benchmarks** | LM Eval Harness, HELM | Model capability benchmarking |
| **Safety & red-teaming** | Inspect AI, Promptfoo | Adversarial testing, safety evals |
| **Experiment tracking** | Braintrust, MLflow | A/B testing, experiment comparison |

---

## Observability & Tracing Platforms

Observability platforms monitor LLM systems in production, providing traces, logs, and real-time quality signals.

| Platform | Type | Tracing | Scoring | Alerting | Key Feature |
|---|---|---|---|---|---|
| **[Langfuse](https://langfuse.com/)** | Open-source | ✅ | ✅ | ✅ | Open-source tracing + scoring, self-hostable |
| **[Arize Phoenix](https://phoenix.arize.com/)** | Open-source | ✅ | ✅ | ✅ | Embedding analysis, drift detection |
| **[LangSmith](https://smith.langchain.com/)** | Commercial | ✅ | ✅ | ✅ | Deep LangChain integration, dataset management |
| **[Weights & Biases Weave](https://wandb.ai/site/weave)** | Commercial | ✅ | ✅ | ⚠️ | ML experiment tracking heritage, rich visualization |
| **[Braintrust](https://www.braintrust.dev/)** | Commercial | ✅ | ✅ | ✅ | Unified eval + observability platform |
| **[Helicone](https://helicone.ai/)** | Open-source | ✅ | ⚠️ | ✅ | Proxy-based, zero-code integration |
| **[Portkey](https://portkey.ai/)** | Commercial | ✅ | ✅ | ✅ | AI gateway + observability, multi-provider |

### When to Use What

| Need | Tool Category | Examples |
|---|---|---|
| Debug a single LLM call | Tracing | Langfuse, LangSmith |
| Monitor production quality over time | Observability + scoring | Arize Phoenix, Braintrust |
| Track token usage and cost | Observability | Helicone, Portkey |
| Detect distribution drift | Drift detection | Arize Phoenix |
| Correlate eval scores with traces | Integrated platform | Braintrust, LangSmith |

---

## Benchmark Suites

Benchmark suites are standardized collections of eval tasks for comparing model capabilities.

| Suite | Focus | # Tasks | Scoring | Maintained By |
|---|---|---|---|---|
| **[LMSYS Chatbot Arena](https://lmarena.ai/)** | Human preference | Live | ELO rating | UC Berkeley |
| **[Open LLM Leaderboard](https://huggingface.co/spaces/open-llm-leaderboard/open_llm_leaderboard)** | Open-weight models | 6 benchmarks | Accuracy | Hugging Face |
| **[HELM](https://crfm.stanford.edu/helm/)** | Holistic evaluation | 42 scenarios | Multi-metric | Stanford CRFM |
| **[LiveBench](https://livebench.ai/)** | Contamination-resistant | ~18 tasks | Objective scoring | Community |
| **[BIG-Bench](https://github.com/google/BIG-bench)** | Diverse capabilities | 200+ tasks | Varies | Google |
| **[SEAL](https://scale.com/leaderboard)** | Expert human evaluation | Multiple | Human ratings | Scale AI |

> See [03. Foundation Model Benchmarks →](../03_benchmarks/README.md) for deep dives on individual benchmarks (MMLU, GPQA, SWE-Bench, etc.).

---

## Judge Model Options

When using LLM-as-a-Judge, the choice of judge model matters significantly.

### Judge Model Comparison

| Judge Model | Provider | Strengths | Weaknesses | Cost (per 1M tokens) |
|---|---|---|---|---|
| **GPT-4o** | OpenAI | Strong instruction following, well-calibrated | Self-preference bias for OpenAI models | ~$5 input / $15 output |
| **Claude 3.5 Sonnet** | Anthropic | Detailed rubric adherence, low verbosity bias | May be overly cautious on safety-adjacent content | ~$3 input / $15 output |
| **Gemini 1.5 Pro** | Google | Long-context evaluation, multimodal judging | Less community adoption as judge | ~$1.25 input / $5 output |
| **Llama 3.1 70B/405B** | Meta (open) | Free, self-hostable, no data leaves your infra | Weaker calibration than frontier closed models | Self-hosted cost |
| **Prometheus 2** | Open-source | Purpose-built for evaluation, fine-tuned on rubric scoring | Smaller model, may miss nuance on complex tasks | Self-hosted cost |

### Judge Selection Guidelines

| Scenario | Recommended Judge | Rationale |
|---|---|---|
| General quality assessment | GPT-4o or Claude 3.5 Sonnet | Best calibration on broad tasks |
| Evaluating OpenAI model outputs | Claude or Gemini | Avoid self-preference bias |
| Privacy-sensitive data | Llama 3.1 (self-hosted) | Data stays on your infrastructure |
| High-volume, cost-sensitive | Gemini 1.5 Flash or Llama 3.1 8B | Lower cost per judgment |
| Multimodal evaluation | Gemini 1.5 Pro or GPT-4o | Native vision capabilities |
| Standardized rubric scoring | Prometheus 2 | Purpose-built for structured evaluation |

---

## Framework Selection Guide

Use this decision tree to choose the right tool for your situation:

```
What are you evaluating?
│
├── A foundation model's general capabilities
│   └── Use: LM Eval Harness or HELM
│
├── A RAG pipeline
│   └── Use: Ragas or DeepEval (RAG metrics)
│
├── An application or product
│   ├── Need experiment tracking?
│   │   ├── YES → Braintrust
│   │   └── NO  → Promptfoo or DeepEval
│   └── Need CI/CD integration?
│       └── YES → Promptfoo (CLI-native) or DeepEval (pytest)
│
├── Safety and red-teaming
│   └── Use: Inspect AI or Promptfoo
│
└── Production monitoring
    ├── Already using LangChain?
    │   └── YES → LangSmith
    └── Want open-source?
        └── YES → Langfuse or Arize Phoenix
```

---

## Strengths & Limitations Matrix

| Approach | Strengths | Limitations | When It Breaks Down |
|---|---|---|---|
| **Eval frameworks** | Structured, reproducible, CI-friendly | Requires upfront investment in dataset + rubric design | When you don't know what to measure yet |
| **Observability platforms** | Real-time, minimal setup, production visibility | Passive — detects problems but doesn't prevent them | When you need pre-deployment quality gates |
| **Benchmark suites** | Standardized, comparable across models | Static, contamination-prone, may not match your task | When your use case differs from the benchmark distribution |
| **LLM judges** | Scalable, captures nuance, flexible | Biased, expensive at scale, requires calibration | When the judge model doesn't understand the domain |

---

## Integration Patterns

### Pattern 1: Dev → CI → Production

```
┌─────────────┐     ┌──────────────┐     ┌──────────────────┐
│  Dev Loop    │     │  CI Pipeline │     │  Production      │
│              │     │              │     │                  │
│  Promptfoo / │ ──> │  Run eval    │ ──> │  Langfuse /      │
│  Braintrust  │     │  suite on PR │     │  Phoenix monitor │
│              │     │  Gate merge  │     │  Alert on drift  │
└─────────────┘     └──────────────┘     └──────────────────┘
```

### Pattern 2: Eval + Observability Feedback Loop

```
Production traces (Langfuse) → Export failures → Add to eval dataset → Re-run evals → Deploy fix → Monitor
```

This creates a **flywheel** where production failures continuously strengthen your eval suite.

---

## Further Reading

- 📄 [Holistic Evaluation of Language Models (HELM)](https://arxiv.org/abs/2211.09110) — Stanford's multi-metric evaluation framework
- 📄 [Judging LLM-as-a-Judge](https://arxiv.org/abs/2306.05685) — Analysis of LLM judge reliability
- 📝 [Promptfoo Documentation](https://www.promptfoo.dev/docs/intro) — Getting started with model-agnostic eval
- 📝 [Langfuse Documentation](https://langfuse.com/docs) — Open-source LLM observability
- 📝 [Ragas Documentation](https://docs.ragas.io/) — RAG-specific evaluation metrics

---

Move to [03. Foundation Model Benchmarks →](../03_benchmarks/README.md) to learn about MMLU, GPQA, SWE-Bench, and other benchmarks used to compare models.
