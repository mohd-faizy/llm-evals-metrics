# 🚀 Production-Grade LLM Evaluation Engineering & Metrics

[![Python 3.10+](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://www.python.org/)
[![Jupyter Notebook](https://img.shields.io/badge/Jupyter-Notebooks-orange.svg)](https://jupyter.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-Welcome-brightgreen.svg)](CONTRIBUTING.md)

> A practical, source-oriented masterclass and architectural scaffold for designing, testing, benchmarking, and monitoring enterprise LLM applications and foundation models.

---

## 📌 Overview

Transitioning from toy LLM prototypes to production enterprise systems requires moving past **vibe testing** to **rigorous, quantitative, and automated evaluation engineering**. 

This repository provides a complete end-to-end curriculum, architecture blueprints, evaluation taxonomies, and hands-on Jupyter notebooks covering the full LLM evaluation lifecycle:

- **Foundation Model Benchmarks & Harnesses**: MMLU, GSM8K, HumanEval, Chatbot Arena, and HELM.
- **Application & RAG Evals**: Groundedness, Context Relevance, Answer Faithfulness, and Hallucination metrics.
- **Agentic & Workflow Systems**: Tool calling accuracy, multi-step trajectory scoring, memory retrieval, and multi-agent coordination.
- **LLM-as-a-Judge Engineering**: Prompt calibration, pairwise ranking, G-Eval methodology, and bias mitigation.
- **Production Observability**: Offline CI/CD regression suites, real-time online monitoring, drift detection, and cost/latency tracking.

---

## 📓 Interactive Hands-on Notebooks

Dive straight into runnable Jupyter notebooks equipped with structured tutorials, code snippets, visual architecture diagrams, and real-world evaluation pipelines:

| Notebook | Topic / Focus | Link |
| :--- | :--- | :---: |
| **01: LLM Evals Engineering** | Introduction to AI Engineering, prototype vs production transition, & core evaluation taxonomy | [Open Notebook](notebooks/01_llm_evals.ipynb) |
| **02: Model vs Application Evals** | Key differences between model-level capability testing vs end-to-end user-facing app evals | [Open Notebook](notebooks/02_model_vs_app_evals.ipynb) |
| **03: End-to-End Eval Workflow** | Constructing end-to-end evaluation pipelines, dataset preparation, and scoring loops | [Open Notebook](notebooks/03_end_to_end_eval_workflow.ipynb) |
| **04: Multi-Pipeline Eval Architecture** | Designing modular evaluation pipelines for multi-stage LLM chains and agent systems | [Open Notebook](notebooks/04_multi_pipeline_eval_architecture.ipynb) |
| **05: Mechanisms & Paradigms** | Comparing Deterministic Rules, Heuristics, Embeddings/NLP, and LLM-as-a-Judge paradigms | [Open Notebook](notebooks/05_eval_mechanisms_and_paradigms.ipynb) |
| **06: Offline vs Online Evals** | CI/CD unit testing vs production shadow evaluation, user feedback loops, and telemetry | [Open Notebook](notebooks/06_offline_vs_online_evals.ipynb) |
| **07: Model-Level Metrics** | Perplexity, BLEU/ROUGE, Exact Match, Pass@k, and model capability evaluation metrics | [Open Notebook](notebooks/07_model_level_evals.ipynb) |
| **08: Benchmarking Harnesses** | Integrating open-source harnesses (lm-evaluation-harness, Lighteval, DeepEval, Ragas) | [Open Notebook](notebooks/08_benchmarking_and_eval_harnesses.ipynb) |

---

## 🗺️ Curriculum & Repository Structure

The core modules are organized sequentially to build deep competency in LLM evaluation engineering:

| Module Directory | Key Concepts & Focus Areas |
| :--- | :--- |
| **`00_eval_mindset/`** | Evaluation philosophy, failure modes, cost of hallucinations, and baseline setting |
| **`01_fundamentals/`** | Measurement theory, qualitative vs quantitative metrics, and evaluation taxonomy |
| **`02_landscape/`** | Ecosystem mapping: Frameworks (DeepEval, Ragas, TruLens), Observability (LangSmith, Phoenix, Arize) |
| **`03_benchmarks/`** | Standard foundation benchmarks, contamination detection, and leaderboard hygiene |
| **`04_application_evals/`** | Evaluating real product experiences, task-specific success criteria, and user intent alignment |
| **`05_dataset_engineering/`** | Golden dataset creation, synthetic data generation (Evol-Instruct), regression suite versioning |
| **`06_eval_pipelines/`** | Continuous integration for prompts/models, automated test triggers, reporting dashboards |
| **`07_llm_judge/`** | LLM-as-a-Judge system prompt design, position bias, verbosity bias, calibration against humans |
| **`08_rag_evals/`** | The RAG Triad: Context Relevance, Groundedness, Answer Relevance, and Chunking impact |
| **`09_workflow_evals/`** | Multi-step agent workflows, state machine transitions, and graph flow correctness |
| **`10_agent_evals/`** | Agent planning, reflection efficiency, trajectory evaluation, and goal completion rates |
| **`11_multi_agent_evals/`** | Multi-agent collaboration, message passing overhead, delegation efficiency, and deadlock detection |
| **`12_tool_calling_evals/`** | Function call parameter validity, schema matching, tool selection precision/recall |
| **`13_memory_evals/`** | Short-term context window utilization, long-term memory retrieval accuracy, and context rot |
| **`14_safety_evals/`** | Red-teaming, prompt injection resilience, jailbreaks, toxicity, PII leaks, and bias audits |
| **`15_operational_evals/`** | System performance: Time-to-First-Token (TTFT), tokens per second (TPS), latency P99, API costs |
| **`16_production_evals/`** | Live production monitoring, shadow deployment testing, A/B routing, and data drift detection |
| **`17_multimodal_evals/`** | Vision-Language Models (VLM), document layout understanding, image-text alignment evals |
| **`18_coding_agent_evals/`** | Code generation benchmarks, SWE-bench evaluation patterns, unit test execution pass rates |
| **`19_research_papers/`** | Curated research papers, summaries, and reading pathways for LLM evaluation |
| **`20_build_your_own_eval_framework/`** | Complete starter architecture and design patterns for building an in-house evaluation platform |

---

## ⚡ Quick Start Guide

### 1. Clone the Repository
```bash
git clone https://github.com/mohd-faizy/llm-evals-metrics.git
cd llm-evals-metrics
```

### 2. Set Up Virtual Environment
```bash
# Create environment
python -m venv venv

# Activate on Windows (PowerShell)
.\venv\Scripts\Activate.ps1
# Or Linux/macOS
source venv/bin/activate
```

### 3. Launch Jupyter Notebooks
```bash
pip install jupyterlab
jupyter lab notebooks/
```

---

## 📐 Evaluation Architecture Taxonomy

```
                          ┌─────────────────────────────────────────┐
                          │   LLM Evaluation Architecture Matrix    │
                          └────────────────────┬────────────────────┘
                                               │
               ┌───────────────────────────────┴───────────────────────────────┐
               ▼                                                               ▼
 ┌───────────────────────────┐                                   ┌───────────────────────────┐
 │   Offline Evals (Pre-Dev) │                                   │  Online Evals (Production) │
 ├───────────────────────────┤                                   ├───────────────────────────┤
 │ • Golden Test Datasets    │                                   │ • Real-time Telemetry     │
 │ • Deterministic Metrics   │                                   │ • Shadow Deployments      │
 │ • LLM-as-a-Judge Suites   │                                   │ • User Explicit Feedback  │
 │ • CI/CD Regression Tests  │                                   │ • Operational Latency/Cost│
 └───────────────────────────┘                                   └───────────────────────────┘
```

---

## 🤝 Contributing

Contributions are welcome! Please feel free to open an issue or submit a Pull Request.

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingEval`)
3. Commit your Changes (`git commit -m 'Add some AmazingEval feature'`)
4. Push to the Branch (`git push origin feature/AmazingEval`)
5. Open a Pull Request

---

## 📄 License

Distributed under the MIT License. See [`LICENSE`](LICENSE) for more information.
