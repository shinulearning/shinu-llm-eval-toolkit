# 🧪 Shinu AI Labs — shinu-llm-eval-toolkit

> **Forked from:** [dottxt-ai/outlines](https://github.com/dottxt-ai/outlines) (⭐ 14.7K)  
> **Our Fork:** [github.com/shinulearning/shinu-llm-eval-toolkit](https://github.com/shinulearning/shinu-llm-eval-toolkit)  
> **License:** Apache 2.0 (same as upstream)  
> **Upstream Stars:** 14,724 ⭐

---

## Why This Fork Exists

`outlines` is the gold standard for **structured LLM output generation** — it guarantees valid JSON, Pydantic models, regex patterns, and grammar-constrained outputs *during* generation, not after. This is critical for building reliable AI evaluation pipelines in Quality Engineering.

This fork was created to:

1. **Center LLM evaluation in QE workflows** — structured outputs are the foundation of testable, verifiable AI behavior
2. **Build reusable evaluation primitives** — classification, extraction, scoring, and assertion patterns for LLM-powered QA
3. **Integrate with Shinu AI Labs tools** — QA Copilot, Bug Triage Agent, and LLM evaluation pipelines
4. **Document QE-specific usage patterns** — beyond the upstream's general-purpose examples

---

## What's Added in This Fork

| Artifact | Location | Purpose |
|:---------|:---------|:--------|
| `SHINU_README.md` | Root (this file) | Fork branding, QE rationale |
| `QE_USE_CASES.md` | Root | QA/testing examples for outlines |
| README.md header | Modified | Added shinu- prefix context note |

---

## QE Value Map

| Outlines Feature | QE Application |
|:-----------------|:---------------|
| `Literal["Pass","Fail","Skip"]` | AI-powered test result classification |
| Pydantic models | Structured bug report extraction from free text |
| Regex constraints | Log parsing with guaranteed format compliance |
| JSON Schema | LLM-based test case generation with valid schemas |
| Function calling | Extracting structured assertions from natural language |
| Grammar/CFG | Constrained prompt generation for reproducible testing |

---

## Quick Links

- [Upstream Repo](https://github.com/dottxt-ai/outlines) — All credit to the .txt team
- [Outlines Documentation](https://dottxt-ai.github.io/outlines/)
- [QE Use Cases](./QE_USE_CASES.md) — QA/testing patterns
- [Shinu AI Labs](https://www.shinuailabs.com)
