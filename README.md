# Engineering Trust in an Agent-Assisted Amharic LLM Project

Public documentation and selected experiment evidence for **Beyond Code Generation: What Building an Amharic LLM with Multi-Agent GitHub Copilot Taught Me About Engineering Trust**.

## Read the case study

- **[Main article on Hugging Face](https://huggingface.co/blog/Nardosye/fine-tuning-qwen-for-amharic-with-github-copilot-w)**: agent coordination, a rejected dataset optimization, failure recovery, GPU execution, and model evaluation. [Markdown snapshot](docs/agent-assisted-amharic-llm-case-study.md).
- **[Companion appendix](docs/agent-assisted-amharic-llm-case-study-appendix.md)**: evidence index, artifact identities, and what public readers can verify.
- **[Selected review and failure records](docs/review-evidence.md)**: curated observations and corrections, with explicit provenance limits.
- **[Experiment reports](reports/)**: nine original metadata-only reports and one clearly labeled split-comparison extract.
- **[Snapshot manifest](snapshot-manifest.json)**: file hashes, sizes, and provenance for this publication snapshot.

This is an observational engineering case study, not a tutorial, released model, or controlled demonstration that multi-agent development outperforms other workflows.

## Measured outcomes

A fresh LoRA pilot adapted `McGill-NLP/AfriqueQwen-4B` on 2,500 documents, covering **5,000,865 causal targets, 4,366 windows, and 1,092 optimizer updates**.

| Existing validation domain | Original perplexity | Adapted perplexity | Reduction |
|---|---:|---:|---:|
| Wikipedia | 3.1823 | 3.0533 | 4.05% |
| Grade-10 textbook excerpts | 2.6180 | 2.5603 | 2.20% |
| Children's books | 2.6760 | 2.5945 | 3.05% |

Primary human evaluation favored the adapter in **45 of 83 decisive Amharic comparisons (54.2%)**, with a two-sided 95% Wilson interval of **43.6% to 64.5%**. The prespecified preference/confidence gates were not met. Lower perplexity is not proof of improved fluency.

## Scope of this repository

The original implementation repository and cloud artifacts remain private. This new repository contains selected documentation and metadata only, without private Git history, corpus bodies, PDFs, model weights, raw logs, credentials, generated continuations, individual ratings, or the blind identity mapping.

Public readers can inspect the published evidence and recompute reported aggregate metrics. They cannot rerun the complete training experiment from this snapshot alone. The archived [agent profiles](docs/agent-profiles/) are documentation, not active instructions for this repository.

**Evidence cutoff:** November 1, 2026. Historical receipt fields describe their observation time; later reports supply later evaluation results. Preparing this snapshot does not rerun training or change historical evidence.

Further training on the frozen corpus is planned.
