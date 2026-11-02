# Beyond Code Generation: What Building an Amharic LLM with Multi-Agent GitHub Copilot Taught Me About Engineering Trust

*An engineering case study of agent coordination, independent review, failure recovery, and reproducible model training.*

**Evidence cutoff: November 1, 2026.**

## 1. Introduction: generating code is not the same as producing evidence

The most revealing result in this project was not a lower model loss. It was a dataset optimization that looked successful until another agent asked the right question.

GitHub Copilot proposed a way to balance training, validation, and test tokens. The resulting proportions were almost perfect. But independent review exposed what the headline concealed: every multi-passage article would have gone into training, leaving both evaluation splits composed entirely of single-passage articles.

The optimization improved the number we were watching while damaging a property we needed.

That incident captures the central question of this project: **what happens when multiple AI coding agents build a real language-model training system, and their work must withstand independent review?**

The setting was Amharic language adaptation. My initial interest was literature, but the eventual pilot used predominantly news and web material, with smaller book, textbook, and Wikipedia contributions. Unicode, extraction quality, source families, limited evaluation coverage, and uncertain linguistic judgments made this a useful setting for studying engineering trust.

## 2. Research questions and scope

I investigated how agents coordinated preparation and review, which objections changed the implementation, how failures were reproduced, and what survived the transition from local development to paid GPU execution.

This is **one observational engineering case study**, not a controlled comparison of multi-agent systems against individual developers. We did not measure developer productivity, isolate the effect of model diversity, or establish that more reviewers necessarily improve outcomes.

There are also two distinct outcomes: whether the engineering system produced auditable experiments, and whether adaptation improved the model. Successful execution cannot answer the second question.

## 3. The agent architecture: separate responsibilities, not shared authority

The main Copilot assistant coordinated implementation, handoffs, evidence collection, and authorized execution. I set goals, made material-acceptance decisions, approved bounded spending, and supplied human evaluation judgments.

The historical record includes two explicitly requested reviews: Claude Sonnet 5.5 inspected bounded Wikipedia material and structural findings; Gemini reviewed ingestion, splitting, provenance, and test gaps. Later, four custom profiles defined dataset preparation, material review, experiment review, and training coordination.

Those are different kinds of evidence. A profile file proves a role was configured, not invoked. The private project retains the journal, review reports, and readiness records; this snapshot publishes a [curated review account](review-evidence.md) and selected metadata. We do not claim that every configured profile ran at every stage.

**Figure 1. Intended handoff and acceptance boundaries**

```text
Human: goals, scope, material decisions, spending authority
                            |
                 Main Copilot coordinator
                            |
       +--------------------+--------------------+
       |                    |                    |
 Dataset preparer     Material reviewer   Experiment reviewer
 Local candidates     Source fidelity     Design and claims
       +--------------------+--------------------+
                            |
              Tests, hashes, source-bound evidence
                            |
              Training-coordinator readiness audit
                            |
             Human authorization -> bounded GPU job
                            |
             Artifact verification -> model evaluation
```

The figure describes the workflow, not an always-running autonomous swarm. Profiles constrained tools and responsibilities: a material reviewer could inspect supplied evidence but not edit or train; the experiment reviewer challenged methodology; the coordinator audited readiness without spending authority.

These were tool permissions and written boundaries, not proof of process isolation or an adversarial security sandbox. Likewise, "independent review" means a separate reviewer context and remit, not an independent institution or a guarantee against correlated errors.

## 4. Building the pipeline: preserve identities before optimizing throughput

The pipeline was deliberately simpler than its module inventory:

```text
Pinned sources -> inspected text -> provenance and duplicate groups
 -> protected splits -> tokenizer accounting -> frozen experiment bundle
 -> original/adapted execution -> verified artifacts -> evaluation
```

Source inspection came before trusting filenames or language labels. Ethiopic script alone does not establish Amharic fluency, and a clean extraction does not establish useful training material. Source suitability, fidelity, and publication permissions remained separate questions.

The data path retained source identities, transformations, offsets where applicable, rejection reasons, and checksums. Exact duplicate-linked articles stayed together; related editions and historical evaluation reservations required their own grouping checks. These mechanisms reduced specific leakage risks without establishing semantic independence or absence from upstream pretraining.

Token accounting was part of the experimental contract. Whole-document and passage token counts can differ because tokenization depends on boundaries. During adaptation, document-bounded overlapping windows reused prefixes as context while masking already scored targets. One original document ending marker was retained; padding was not allowed to erase genuine EOS targets when their token IDs coincided.

Manifests bound data, model revisions, tokenizer behavior, configuration, and executed sources. The larger frozen candidate pool contains 288,116 documents and 1,021,623,880 body tokens, but only the separately accepted five-million-token pilot was trained. Prepared volume was never presented as completed training.

## 5. What the agents got wrong

### Case A: a UTF-8 inspection failure mistaken for a source problem

**Initial approach.** An early source-inspection command read a fixed 16,384-byte XML prefix and decoded it as if it were a complete UTF-8 document.

**Observed failure.** It raised `UnicodeDecodeError`: the byte limit landed inside a multibyte character. This was an inspection-method defect, not evidence that the underlying source was corrupt.

**Discovery and correction.** The recovery used an incremental UTF-8-with-BOM decoder with `final=False`. That allowed an incomplete trailing character to wait for more bytes while the inspection parsed the complete opening tag. Strict decoding remained appropriate for complete documents; this was not permission to ignore arbitrary encoding errors.

**Checked outcome.** The revised inspection successfully examined 33 source headers. The inventory distinguished header-only observations from complete-file evidence, rather than inventing whole-file hashes for partially read sources.

This was owner-side debugging, not a finding attributed to Claude or Gemini. Its lesson was foundational: a boundary condition in our reader must not become a data-quality label attached to somebody else's text. The [curated inspection record](review-evidence.md#unicode-prefix-inspection) states what remains private and what the historical inspection established.

### Case B: the split optimizer that made evaluation look better than it was

**Initial proposal.** The first Wikipedia smoke dataset contained 256 articles and 195,438 passage tokens. Article-count allocation produced 153,258 training tokens, 2,598 validation tokens, and 39,582 test tokens. One article contributed 31,962 passage tokens, approximately 80.7% of the test split.

Copilot proposed sorting duplicate-linked components by descending token mass, then assigning each to the largest absolute remaining token deficit. It sounded reasonable: preserve article boundaries, preserve duplicate groups, and improve 90/5/5 token balance.

**Reviewer objection.** Gemini simulated the proposal and found a hidden length-distribution shift. Every one of the 60 multi-passage articles, including all 124 passages of at least 500 tokens, would enter training. Excellent token proportions would conceal evaluation material drawn exclusively from the single-passage category.

The issue was not that those evaluation articles had no value. It was that the proposed improvement implicitly changed what evaluation measured.

**Figure 2. Failure-and-recovery sequence**

```text
Observed: article-count splits have badly imbalanced token totals
    |
Proposed: largest-first allocation nearly achieves 90/5/5 tokens
    |
Reviewed: Gemini finds no multi-passage held-out articles
    |
Reproduced: repository comparison confirms the hidden shift
    |
Rejected: preserve the proposal and its failure, not just its score
    |
Compared: seeded midpoint-relative candidate retains longer held-out units
```

**Correction.** Rather than quietly editing the proposal until it looked successful, we preserved it as a rejected policy. A reproducible offline comparison held the articles, passage boundaries, seed, and exact-duplicate components fixed. Only hypothetical assignments changed.

The alternative retained the existing component-size/seed-hash order and scored destinations by midpoint-adjusted relative deficit: `(target - actual - component_weight / 2) / target`.

| Policy | Train tokens | Validation tokens | Test tokens | Multi-passage articles: validation / test |
|---|---:|---:|---:|---:|
| Historical article targets | 153,258 | 2,598 | 39,582 | 1 / 6 |
| Rejected largest-first policy | 175,894 | 9,772 | 9,772 | 0 / 0 |
| Seed-ordered midpoint candidate | 175,808 | 9,752 | 9,878 | 6 / 4 |

**Checked outcome.** The repository comparison reproduced the objection and the alternative's totals. Repeated offline reports were byte-identical, and checks confirmed the historical Wikipedia and Bible artifacts remained unchanged.

That did not establish a perfect benchmark. Most long passages still entered training, and near-duplicate risks remained. The alternative was a documented candidate; the comparison itself did not replace the historical dataset. Subsequent curation was a separate recorded stage.

The [public comparison account](review-evidence.md#split-policy-comparison) also preserves a reviewer correction: Gemini initially reported 31,967 passage tokens for the dominant article; repository recomputation measured 31,962. The objection survived, but the reviewer's number did not.

This was the strongest example of verification changing a decision. A reviewer supplied a counterexample to our success criterion, and the pipeline made that objection reproducible.

### Case C: the GPU evaluation succeeded, but collecting its results failed

**Initial approach.** A separate GPU job evaluated original and adapted perplexity, after which a local collector independently reconstructed window, document, and domain totals before accepting the metrics.

**Observed failure.** The GPU job reached `COMPLETED`, but local collection raised `KeyError: 'targets'`. Window records used `targets`; aggregated document records used `scored_tokens`. The collector confused those schemas during domain reconstruction.

**Discovery.** The monitor's nonzero exit did not tell us whether inference failed. The terminal job receipt, immutable remote metrics, and local traceback separated successful GPU execution from failed postprocessing. No result was labeled verified at the failed collection step.

**Correction.** We archived the exact authorized preparer before modifying collection, corrected the field, and required complete domain coverage. Regression tests exercised collection across the real window/document/domain schema and checked that a missing domain was rejected.

Recovery still verified the original preparer snapshot, unchanged runtime, bundle, and configuration. The execution receipt recorded the authorized source hash and the corrected collector hash separately. The changed source could collect existing evidence, not silently authorize another launch.

**Checked outcome.** Existing results were retrieved and reconstructed successfully, with no second GPU job. Five focused tests passed for the evaluation/recovery path. This was local debugging, not an independently reviewed fix; the article does not upgrade it into one.

The earlier training coordinator had already identified a different interrupted-collection problem before the pilot launched. Fixing that issue did not prevent this later schema mistake. Review reduced a specific risk; it did not certify the entire pipeline.

## 6. Independent reviews: what actually changed?

Useful review produced consequences beyond another paragraph of agreement.

| Finding and evidence source | Consequence |
|---|---|
| Gemini: balanced token shares hide held-out length shift | Reject the first split proposal; compare length profiles and contributor concentration |
| Gemini: two synthetic boundary counterexamples | Fix passage selection and add independent whitespace-only passage rejection |
| Claude: an angle-bracketed quotation was falsely flagged as HTML | Narrow the detector and add a false-positive regression |
| Training coordinator: interrupted pilot collection could damage recovery or relabel pairs | Preserve verified bytes, persist the blind key, and test recovery before launch |

The latent passage defect illustrates why scope matters. A hard-wrapped long word followed by a boundary could produce a whitespace-only passage. Gemini supplied counterexamples, and tests now cover them. None occurred in the saved 620-passage sample; we did not report existing data loss that had not happened.

Claude's material review also needed clarification. Its initial paragraph-repeat counts referred to different populations: strings repeated anywhere, strings repeated across distinct articles, and strings crossing splits. Conflating those counts would have overstated leakage. Its markup heuristic falsely flagged an Amharic quotation, and the reviewer withdrew that finding.

These episodes made the reviewers' uncertainty operational. Review flags became inspection queues, not automatic deletion rules. Structural heuristics did not authorize spelling repair, establish fluency, or turn a local source into publishable material.

Later readiness evidence was equally explicit about limits. The pilot coordinator performed a code-only audit without source bodies, keys, paid actions, or independent test execution. Owner-run regressions supplied separate evidence. Calling both steps "review passed" would erase who actually checked what.

## 7. From local experiments to NVIDIA GPU training

Early experiments used Qwen3-0.6B LoRA on an Apple M2. Cloud work subsequently used `McGill-NLP/AfriqueQwen-4B`. This was a change in base model, scale, and hardware, not a controlled acceleration comparison.

The first A100 throughput calibration failed before producing measurements. A compound hardware assertion conflated provider accelerator kind with requested flavor and did not expose its individual conditions. The original log could not identify every failed subcondition.

A separately authorized follow-up reported actual CUDA availability, device count, GPU name and memory, and BF16 support. It observed `ACCELERATOR="gpu"` on an A100-SXM4-80GB and completed nine context/batch configurations. It also explicitly pinned CUDA 12.6. We did not infer that CUDA caused the first failure.

**Figure 3. Selected cloud execution history, not a complete project invoice**

| Run and job reference | Terminal state | Provider total | What was verified |
|---|---|---:|---|
| [Calibration v1](../reports/amharic_full_corpus_v1_calibration_execution.json) | ERROR | 57 s | Preflight failure; no throughput measurements |
| [Diagnostic calibration v2](../reports/amharic_full_corpus_v1_calibration_v2_execution.json) | COMPLETED | 186 s | Nine configurations; no retained adapter |
| [Five-million-target pilot](../reports/amharic_five_million_pilot_v1_execution.json) | COMPLETED | 3,180 s | Complete pass, adapter integrity, matched generations |
| [Matched perplexity](../reports/amharic_pilot_perplexity_v1_execution.json) | COMPLETED | 122 s | Both scoring schedules and reconstructed metrics |

These public receipts identify the jobs and their observation-time status. The underlying cloud artifacts remain private.

The pilot trained a fresh rank-8/alpha-16 query/value LoRA adapter at learning rate 0.00002, using a BF16 base, float32 adapter parameters, SDPA, gradient checkpointing, and true batch four. One pass covered **4,366 windows, 1,092 updates, and 5,000,865 causal targets**.

Verification checked unchanged frozen-base parameters and exact saved/fresh-base-reloaded adapter tensors. The job produced 100 original and 100 adapted continuations under matched decoding. Private partial adapters were saved every 256 updates; these were recovery artifacts, not optimizer-resumable checkpoints or completed results.

Submission intent, remote reservations, one-attempt limits, and provider/runtime deadlines bounded execution independently of an awake laptop. A failed approval was not reused for a retry.

At the checked A100 rate of $0.041667/minute, rounding the pilot's timestamp interval upward to 54 minutes gives a **$2.250018 reference**, below its $5 ceiling. The perplexity reference was $0.208335, using three rounded total minutes plus a two-minute margin. These are documented estimates, not verified invoices or a claim about total project spending.

## 8. What the model evaluations revealed

The completed engineering pipeline did not earn a fluent-model claim.

A separately authorized inference-only job scored the original and saved adapter on the same 56 validation units/windows and **12,886 causal targets**, using BF16/SDPA, context 2,048, stride 1,024, one EOS per unit, and once-only target coverage. Loaded adapter tensors matched, and disabling the adapter reproduced the original likelihood.

**Figure 4. Prediction-loss gains beside the separate human outcome**

| Existing validation domain | Targets per model | Original perplexity | Adapted perplexity | Reduction |
|---|---:|---:|---:|---:|
| Wikipedia: 51 articles | 10,081 | 3.1823 | 3.0533 | 4.05% |
| Grade-10 textbooks: 3 excerpts | 1,335 | 2.6180 | 2.5603 | 2.20% |
| Children's books: 2 readers | 1,470 | 2.6760 | 2.5945 | 3.05% |

| Human evaluation, primary Amharic analysis | Result |
|---|---|
| Included comparisons | 89 |
| Adapter / original / neither / tie | 45 / 38 / 5 / 1 |
| Adapter preference among decisive comparisons | 54.2%, or 45/83 |
| Two-sided 95% Wilson interval | 43.6% to 64.5% |
| Prespecified preference/confidence gates | Not met |

Perplexity was computed from token-weighted negative log likelihood, not average document perplexities. Lower scores show improved prediction on these texts; a 4% reduction is not 4% better fluency. These small reused validation cohorts do not cover the dominant news/web training domains, establish statistical significance, or establish upstream novelty. Historical short-context evaluations are not identical baselines.

Human provenance was kept separate: the first submitted assessment was AI-scored; the second was user-confirmed human judgment with AI formatting only. We did not pool them. One exposed pair was conservatively excluded from the primary analysis. Amharic prompt naturalness was user-confirmed, but the prompts were assistant-authored exploratory continuations, not a validated general-language benchmark.

The required 60% preference and confidence lower bound above 50% both failed. Mean coherence rose only from approximately 2.52 to 2.60 out of five. Ten English controls were too few to establish general absence of regression. Both models hit the 256-token generation cap on every prompt without EOS.

The appropriate conclusion was modest prediction-loss improvement and inconclusive perceived-quality improvement, not automatic escalation to full-corpus training.

## 9. Lessons, limitations, and reproducibility

The practices I would retain are bounded work orders, separate review and spending authority, explicit acceptance criteria, preserved failed proposals, source-pinned artifacts, and recovery that does not require rerunning paid compute.

The most valuable checks challenged the definition of success. Token balance needed length-distribution analysis. A completed job needed artifact validation. Lower perplexity needed separate human judgment.

I would also separate three questions whenever an agent reports completion. Can we identify exactly what ran? Can we reconstruct its outputs from the retained evidence? Does that outcome support the intended claim? Hashes, collectors, and statistical evaluation answer different parts of that sequence. None substitutes for the others.

Tests written by the implementation agent can faithfully encode the wrong assumption. The split-policy case mattered because another reviewer challenged the acceptance criterion itself, not just its implementation. Conversely, another model's objection is not automatically correct: the token-count discrepancy and markup false positive both required correction. The useful unit of review was a finding connected to a reproducible check or an explicitly bounded observation.

Limitations remain substantial: one project, no non-agent control, potentially correlated model errors, limited language evaluation, unresolved source-use questions, and no broad independent final test. Checksums prove identity, not quality; separate agent contexts do not prove independent reasoning.

The [companion appendix](agent-assisted-amharic-llm-case-study-appendix.md) maps claims to public reports, archived profiles, and recovery records in this [documentation repository](https://github.com/nardosye/amharic-llm-agent-case-study). Readers can inspect selected evidence and recompute published aggregate metrics. The implementation, corpus, weights, and cloud artifacts remain private, so this snapshot does not permit complete experiment re-execution.

Further training on the frozen corpus is planned.

The central lesson is not that agents can automate everything. It is that their proposals become useful engineering evidence only when objections can be reproduced, failures remain visible, and the final claim survives its acceptance criteria.
