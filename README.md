# LLM Evals Masterclass

A practical, source-oriented repository for learning, designing, and shipping evaluation systems for modern LLM products.

This repo is organized around the idea that evaluation has moved beyond a simple `LLM -> RAG -> Agent` ladder. Modern evals cover:

- foundation model benchmarks
- application and workflow evaluation
- RAG, tool calling, memory, and multi-agent systems
- safety, operational, and production monitoring
- judge-based assessment and human-in-the-loop review

The goal is to make this repository both:

- a reference guide you can read top to bottom
- a working scaffold you can extend into your own evaluation framework

## Repository Map

| Folder | Purpose |
| --- | --- |
| `00_eval_mindset/` | Why evals matter, failure modes, and core evaluation philosophy |
| `01_fundamentals/` | Core definitions, evaluation taxonomy, and measurement basics |
| `02_landscape/` | Ecosystem map for frameworks, observability, benchmarks, and judge models |
| `03_benchmarks/` | Foundation model benchmark families and leaderboard hygiene |
| `04_application_evals/` | Evaluating real product experiences instead of just models |
| `05_dataset_engineering/` | Golden sets, synthetic data, regression suites, and versioning |
| `06_eval_pipelines/` | End-to-end pipeline design, CI/CD, tracking, and reporting |
| `07_llm_judge/` | LLM-as-a-judge design, calibration, and bias management |
| `08_rag_evals/` | Retrieval, generation, and end-to-end RAG assessment |
| `09_workflow_evals/` | Multi-step workflows, LangGraph-style flows, and state transitions |
| `10_agent_evals/` | Tool use, planning, reflection, and trajectory scoring |
| `11_multi_agent_evals/` | Coordination, delegation, and communication in multi-agent systems |
| `12_tool_calling_evals/` | Function calling and API invocation reliability |
| `13_memory_evals/` | Short-term, long-term, and vector memory evaluation |
| `14_safety_evals/` | Jailbreaks, prompt injection, privacy, toxicity, and bias |
| `15_operational_evals/` | Latency, cost, throughput, reliability, and observability |
| `16_production_evals/` | Shadow testing, canaries, A/Bs, feedback loops, and drift detection |
| `17_multimodal_evals/` | Vision, audio, video, and document understanding evals |
| `18_coding_agent_evals/` | Code generation, patch success, and repo comprehension |
| `19_research_papers/` | Curated papers by topic and reading path |
| `20_build_your_own_eval_framework/` | Starter architecture for a reusable eval platform |

Supporting material lives in:

- `datasets/`
- `templates/`
- `cheatsheets/`
- `projects/`
- `docs/`
- `reports/`
- `notebooks/`
- `assets/`

## How To Use This Repo

1. Start with `00_eval_mindset/` and `01_fundamentals/`.
2. Use `02_landscape/` to decide which tools or patterns fit your stack.
3. Move into the domain-specific folders that match your product surface.
4. Reuse the templates and sample datasets when designing a new eval suite.
5. Use `20_build_your_own_eval_framework/` as the blueprint for a real internal eval platform.

## What Makes This Repo Different

- It treats application behavior, not model output alone, as the unit of evaluation.
- It includes production realities: monitoring, cost, reliability, governance, and versioning.
- It covers the spaces most repositories miss: workflows, multi-agent systems, memory, and tool use.
- It is structured to be extended into a practical framework rather than remaining a static notes dump.

## Contributing

Contributions should add one of the following:

- a new topic page
- a sharper definition or taxonomy
- a reusable template or checklist
- a paper note with a concise summary
- a small, useful framework primitive

See `CONTRIBUTING.md` for the working conventions.

