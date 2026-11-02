# Selected Review and Failure Records

**Provenance:** this is a curated owner-persisted account extracted from the private project's engineering journal and review/readiness reports. It is not a raw reviewer transcript, newly executed review, or independent third-party attestation. Evidence cutoff: November 1, 2026.

Corpus passages, raw review conversations, source PDFs, and the full project journal are not included. The machine-readable reports linked below are exact copies unless explicitly labeled an extract.

## Unicode prefix inspection

The October 27 source-inspection command read a 16,384-byte XML prefix and decoded it strictly as a complete UTF-8 document. A byte cutoff inside a multibyte character caused `UnicodeDecodeError`.

The owner-side correction used an incremental UTF-8-with-BOM decoder with `final=False`, then parsed the complete opening tag. The revised inspection examined 33 source headers. Header-only observations were distinguished from complete-file checksums. The project did not label the complete source corrupt because a prefix ended mid-character.

This was debugging by the implementation owner, not a Claude/Gemini finding. The underlying inventory, source text, and original inspection implementation remain private, so this account is historical rather than publicly re-executable evidence.

## Review provenance and corrections

The October 28 journal records two explicitly requested bounded, read-only reviews:

- Claude Sonnet 5.5 reviewed structural statistics for 256 selected Wikipedia articles and 149 rejected rows, bounded excerpts from 16 selected articles, and code-point contexts for ten Unicode-policy rejections.
- Gemini reviewed ingestion, sampling, duplicate-component splitting, provenance, independent validation, and material test gaps.

The reviewers were not authorized to edit, train, publish, or spend. Neither claimed fluent-speaker approval or factual verification.

Claude clarified that 117 paragraph strings repeated somewhere, 80 occurred in multiple distinct articles, and 23 crossed historical splits. Those are different populations, not interchangeable leakage counts. The cross-split strings were short headings/labels in inspected examples.

Claude also withdrew a markup false positive: an angle-bracketed Amharic quotation was not HTML. The implementation narrowed the detector and added a false-positive regression. This was not permission to strip arbitrary angle-bracketed text or normalize uncertain spelling.

Gemini reported two synthetic hard-wrapped-word/boundary counterexamples where passage selection could emit whitespace-only text. No such passage existed in the historical 620-passage sample. The correction required non-whitespace content at preferred boundaries and explicit independent-validator rejection. Historical data loss was not claimed.

Gemini initially reported 31,967 passage tokens for the dominant historical test article. Repository recomputation measured 31,962. The review's central objection survived, but its exact count required correction.

## Split-policy comparison

The [public metadata extract](../reports/wikipedia_split_comparison_public.json) is derived directly from the saved private report. It is not a regenerated dataset. The original [verification receipt](../reports/wikipedia_review_verification.json) records offline repeated-report byte identity, actual tokenizer recounting, 13 unchanged historical Wikipedia files, and 15 unchanged historical Bible files.

All comparisons retained the same 256 articles, 195,438 passage tokens, passage boundaries, seed, and exact-duplicate components.

| Policy | Training tokens | Validation tokens | Test tokens | Held-out multi-passage articles: validation / test |
|---|---:|---:|---:|---:|
| Historical article targets | 153,258 | 2,598 | 39,582 | 1 / 6 |
| Rejected largest-first absolute deficit | 175,894 | 9,772 | 9,772 | 0 / 0 |
| Seed-ordered midpoint-relative candidate | 175,808 | 9,752 | 9,878 | 6 / 4 |

The rejected proposal nearly matched 90/5/5 token fractions but assigned all 60 multi-passage articles and all 124 passages of at least 500 tokens to training. Both held-out splits contained only single-passage articles.

The alternative retained the existing component-size/seed-hash order and scored destination splits using `(target - actual - component_weight / 2) / target`. It retained some longer held-out material, but 119 of the 124 long passages still entered training. It was a justified candidate comparison, not proof of representativeness, semantic independence, or absence from upstream pretraining.

The comparison did not replace historical assignments. Subsequent curation was a separate project stage.

## Pilot readiness review

The source-bound readiness record documents an executed training-coordinator code-only audit and a narrow follow-up. The reviewer did not inspect source bodies or private keys, edit code, authorize paid actions, or independently execute tests.

The initial finding concerned interrupted pilot-result postprocessing: existing bytes and verified receipts needed preservation, missing files needed recoverable restoration, and the blind identity key needed persistence before page generation. Recovery must not relabel outputs or overwrite human ratings.

The owner corrected collection behavior and added interruption/conflict regressions before launch. The follow-up reported the original finding fixed, no new high-confidence blockers, and readiness for the exact bounded pilot. Owner-run tests were separate from the reviewer's static audit.

The readiness opinion did not establish human fluency, source-use clearance, a final invoice, or a general improvement claim. Configured specialist roles are not evidence that every profile ran at every stage.

## Perplexity collection recovery

The [perplexity execution receipt](../reports/amharic_pilot_perplexity_v1_execution.json) identifies the completed job and separately records the authorized preparer and corrected collector hashes.

The original collector raised `KeyError: 'targets'` during domain reconstruction. Window records used `targets`, while aggregated document records used `scored_tokens`. Remote inference had completed; local acceptance had not.

The owner archived the exact authorized preparer, corrected the domain field, required complete domain coverage, and added regressions covering the real collection schema, missing-domain rejection, archived-source integrity, and changed-source launch rejection. Five focused tests passed for that path.

Collection recovery verified the unchanged runner, input bundle, configuration, original preparer snapshot, adapter identity, and existing remote metrics. No authorization receipt was rewritten and no second paid job was submitted. This fix was owner-side debugging, not an independently reviewed correction.

The [full published metadata](../reports/amharic_pilot_perplexity_v1_results.json) retains the per-window, per-document, and per-domain losses used for reconstruction. The source bodies, archived preparer, implementation, and weights remain private.
