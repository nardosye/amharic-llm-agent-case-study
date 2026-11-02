# Companion Appendix: Evidence and Reproducibility

Companion to [the engineering case study](agent-assisted-amharic-llm-case-study.md). Evidence cutoff: November 1, 2026.

This is a public documentation snapshot of a private implementation project. It contains selected original metadata reports, a clearly labeled metadata extract, curated review records, and archived profile definitions. It is not a full implementation release, training dataset, model release, or paid-job launcher.

## 1. Public evidence index

| Claim | Public evidence | Limitation |
|---|---|---|
| UTF-8 prefix inspection failed and recovered | [Curated inspection record](review-evidence.md#unicode-prefix-inspection) | Owner-recorded history; original inventory and inspection code are not published here |
| Claude and Gemini reviews actually occurred | [Curated review record](review-evidence.md#review-provenance-and-corrections) | Summarized from the project journal, not raw reviewer transcripts |
| Token balance concealed a held-out length shift | [Split-comparison extract](../reports/wikipedia_split_comparison_public.json), [original verification receipt](../reports/wikipedia_review_verification.json), [interpretation](review-evidence.md#split-policy-comparison) | Full source report, corpus, and implementation are not included |
| Four specialist profiles were configured | [Archived profile definitions](agent-profiles/) | Configuration does not prove invocation; these are not active repository agents |
| A code-only coordinator audit changed pilot recovery | [Curated audit record](review-evidence.md#pilot-readiness-review) | No independent test execution by that reviewer |
| First calibration failed; revised calibration measured nine cases | [v1 execution](../reports/amharic_full_corpus_v1_calibration_execution.json), [v2 execution](../reports/amharic_full_corpus_v1_calibration_v2_execution.json), [v2 metrics](../reports/amharic_full_corpus_v1_calibration_v2_metrics.json) | Short accepted-book timing cases, not full-corpus throughput |
| The exact five-million-target pilot completed | [Training execution](../reports/amharic_five_million_pilot_v1_execution.json), [training results](../reports/amharic_five_million_pilot_v1_results.json) | No training weights, generation text, or private input bundle |
| Human preference was inconclusive | [Human analysis](../reports/amharic_five_million_pilot_v1_human_review.json) | User-confirmed human provenance; individual ratings, key, and AI assessment are not published |
| Matched per-domain perplexity improved | [Verified execution](../reports/amharic_pilot_perplexity_v1_execution.json), [full metadata results](../reports/amharic_pilot_perplexity_v1_results.json) | Existing small validation sets; no fresh final test or news/web cohort |
| Local collection recovered without another paid run | [Recovery record](review-evidence.md#perplexity-collection-recovery), [source hashes in execution receipt](../reports/amharic_pilot_perplexity_v1_execution.json) | Owner-side debugging, not an independently reviewed recovery fix |

The [snapshot manifest](../snapshot-manifest.json) hashes every included documentation/evidence file except the manifest itself. Its provenance entries distinguish exact source copies, adapted publication documents, and the derived split-comparison extract. Hashes prove identity against a supplied reference, not correctness or independent attestation.

## 2. Split-policy comparison

The public extract omits corpus bodies, rejection contexts, review queues, per-article assignments, and paragraph examples. It retains the policy sizes, concentration and length summaries, token-fraction errors, exact-passage overlap counts, and original source-report hash.

The source hash matches `review_report_sha256` in the original [verification receipt](../reports/wikipedia_review_verification.json). That receipt records a byte-identical offline rerun, actual token recounting, and preservation of historical artifacts. The full original report is not released, so the hash binds the extraction to retained private evidence; it does not let a public reader regenerate that evidence.

The candidate split was a comparison outcome, not an automatic adoption or a contamination-free benchmark.

## 3. Agent configurations versus executions

The four archived definitions are:

- [Dataset preparer](agent-profiles/amharic-dataset-preparer.agent.md)
- [Material reviewer](agent-profiles/amharic-material-reviewer.agent.md)
- [Experiment reviewer](agent-profiles/experiment-reviewer.agent.md)
- [Training coordinator](agent-profiles/training-coordinator.agent.md)

They are copied unchanged from the private project's `.github/agents/` directory, but stored under `docs/` here so they are not active agent configuration. Their instructions reference private-project paths and are not a standalone runnable workflow.

Historical Claude Sonnet 5.5 and Gemini attribution refers to documented review executions, not a fixed model binding in these profiles. Separate contexts and tool boundaries do not establish independent institutions, uncorrelated reasoning, or fluent-human judgment.

## 4. Experiment and artifact identities

| Binding | Recorded value |
|---|---|
| Pilot base model | `McGill-NLP/AfriqueQwen-4B` |
| Model revision | `f4a44d5b4997b52350e07304f4985777e5563d1c` |
| Pilot private result revision | `eed7202a113d3bea50d674db6ea10aebcdb5c399` |
| Saved adapter SHA-256 | `75b6bd0b250131ca8871f1b1cb86f671a505e0d2361bba3f187c2c6f8122fb5f` |
| Perplexity private result revision | `92b5ba14f9ca7c3b242981c0f72d99dea90583ad` |
| Perplexity result SHA-256 | `14be2f372c08d44742708bfaef305b7aad8ce2e692e152dcfcd5a8e44c6f9c4e` |

Original report copies retain historical observation-time flags. For example, the training execution receipt reports pending human review; the later human-analysis report supplies the completed assessment. A historical `publication_performed: false` describes its experiment, not the current visibility of this documentation repository.

Any Hugging Face job URLs inside receipts identify historical jobs; access to private cloud artifacts is not provided by this repository.

## 5. What can be reproduced publicly?

Public readers can recompute per-domain perplexity and reductions from the published negative log-likelihood sums and token counts. From the repository root, using Python 3:

```python
import json
import math
from pathlib import Path

path = Path("reports/amharic_pilot_perplexity_v1_results.json")
result = json.loads(path.read_text())
for domain, comparison in result["comparison"].items():
    values = {}
    for role in ("original", "adapted"):
        metrics = comparison[role]
        nll = metrics["negative_log_likelihood_sum"]
        count = metrics["scored_tokens"]
        values[role] = math.exp(nll / count)
        assert math.isclose(values[role], metrics["perplexity"])
    reduction = 100 * (1 - values["adapted"] / values["original"])
    assert math.isclose(reduction, comparison["perplexity_reduction_percent"])
    print(domain, values, reduction)
```

The full perplexity metadata also supports reconstruction from windows to documents to domains without source text. The human report exposes aggregate counts and Wilson bounds, not a public independent re-rating exercise.

Local training, ingestion, and recovery commands from the private implementation are deliberately not presented as runnable commands in this documentation-only snapshot. Full re-execution requires the implementation, appropriate source access, pinned inputs/environments, and a separate bounded execution decision.

## 6. Publication boundaries

The main article is a public documentation draft, not an accepted official Hugging Face Blog submission. Its Markdown tables and text diagrams do not require a custom renderer.

Model and dataset cards remain separate future release resources. Raw logs, weights, data bodies, PDF pages, credentials, private review keys, and private Git history are not included. Artifact identities and source access do not grant redistribution rights.

When citing this snapshot from Hugging Face, use GitHub's commit-pinned file URLs for a stable article/appendix/report version.
