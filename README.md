# Capstone Módulo 02 — Prompt Engineering System for Real Business Applications

**Intelligent Incident Triage & Response System** — documento principal (relatório final de avaliação).

> **Nota sobre este repositório.** Este repositório público contém apenas o documento principal. O projeto completo (8 prompts V1/V2, schemas, suíte de 12 casos, scripts de métricas, saídas brutas das 72 execuções do pipeline e das 60 execuções do validador, exemplos e demais documentos) é referenciado abaixo pelos caminhos relativos (`docs/…`, `prompts/…`, `tests/…`, `results/…`) e é entregue separadamente (arquivo .zip). Todos os números deste documento vêm de `results/metrics_v1.json` e `results/metrics_v2.json` do projeto completo.

---

# Final Evaluation Report

## Executive Summary

A four-prompt pipeline for **Intelligent Incident Triage & Response** was designed, tested and improved. V1 (baseline) and V2 (improved) were each run on the same 12 test cases for 3 rounds (**72 full-pipeline runs**), plus a single-defect test of the validator (**60 Prompt-04-only runs**; 78 further runs of an earlier, contaminated multi-defect design are kept as legacy) and an independent unsupported-claim judge pass. Results are real, produced by LLM sub-agents and scored by `tests/evaluate.py`; temperature and other sampling parameters were *not configurable in the current environment*.

- Task success **86.1% → 94.4%** (31/36 → 34/36, +8.3 pp); Overall Quality Score **90.9% → 97.6%** (+6.7 pp).
- The largest differences are in consistency and specification (consistency 80.6% → 91.7%, status coherence 58.3% → 100%, incident-ID stability 0/12 → 12/12, strict-enum validity of validator outputs 87.2% → 100%), fallback accuracy 91.7% → 100%, and validator targeted detection on single-defect fixtures 76.2% → 100% (16/21 → 21/21).
- **Not improved / worse:** category accuracy 100% → 97.2% (−2.8 pp; TC-007 regression), category consistency 97.2% → 94.4%; schema compliance and ambiguity handling were already 100%; one unsupported claim still reached the V2 output.
- **Confidence in the result is limited**: the task-success difference is 3 runs and the 95% intervals overlap (71.3–93.9% vs 81.9–98.5%). The baseline was already strong (ceiling effect), and the suite is small.

## Problem

Triage of unstructured incident reports must be standardised, evidence-backed, and must never fabricate facts or causes; it must hand over to humans when information is insufficient. See `docs/01_business_problem_and_scope.md` (no pre-existing problem for this capstone existed in the module folder, so the brief's fallback scenario was used).

## Architecture

P1 Input Analysis & Normalization → P2 Classification & Context Analysis → P3 Decision & Recommendation → P4 Validation & Final Response, with an orchestrator that supplies trusted metadata, routes blocked input straight to stage 4, and retries stage 3 once on a `CORRIGIR` verdict. Four JSON Schemas define the contracts (frozen between versions). Diagram and contract tables: `docs/02_architecture_and_orchestration.md`. The external source is **simulated** (observations embedded in the test input); the orchestrator was executed by the test-running agent, not by code.

## Prompt System

Each prompt has role/responsibility, input/context, instructions/constraints and output contract sections; V2 adds an instruction hierarchy, a worked example and evidence-driven rules. Techniques (structured prompting, chaining, structured reasoning via auditable fields, ReAct-inspired field protocol, few-shot, constraint prompting, self-checking validator, schema enforcement), where and why: `docs/03_prompt_design_rationale.md`. Files: `prompts/0X_*/v1.md`, `v2.md` (8 prompt files).

## Test Methodology

Fixed rubric written before any output was inspected; 3 independent rounds per version; blind executors (forbidden from reading expected behavior); independent judge for unsupported claims; single-defect fixtures for the validator (7 faults + 3 clean controls, valid under both contracts) to measure the validator because natural runs contained too few upstream errors. Reproducibility record, execution caveats and threats to validity: `docs/05_evaluation_methodology.md`. Two rubric/fixture corrections made after V1 (both applied to both versions) are logged in `tests/evaluation_rubric.md` §6. Key caveats: within a round the cases were produced in one agent context; executor and judge share a model family; catch rate counts any non-`APROVAR` verdict without verifying the reason.

## Test Coverage

12 cases: simple (TC-001, 002), complex (003, 004), ambiguous (005, 006), incomplete (007), contradictory (008), external-dependent (009), external-source failure (010), adversarial/prompt injection (011), mandatory fallback (012; also 005, 007, 010). Expected behavior per case: `tests/expected_behaviors.md`; machine-readable: `tests/test_cases.yaml`.

## Metrics

Ten required metrics plus three diagnostics, defined in `tests/evaluation_rubric.md`. Overall Quality Score = 0.20·Task + 0.15·Classification + 0.10·Schema + 0.10·(1−Hallucination) + 0.10·Fallback + 0.10·Ambiguity + 0.10·Consistency + 0.075·Catch + 0.075·Grounding (weights sum to 1.0).

## V1 Results

| Metric | V1 |
|---|--:|
| Task Success Rate | 86.1% (31/36) |
| Schema Compliance | 100% (144/144 stage outputs) |
| Classification Accuracy | 100% (category 36/36, severity 36/36) |
| Hallucination / Unsupported Claim Rate | 5.6% (2/36) |
| Fallback Accuracy | 91.7% (22/24) — 0 missing, 2 excessive |
| Ambiguity Handling | 100% (9/9) |
| Consistency Across Runs | 80.6% |
| Validation Catch Rate (single-defect fixtures, targeted detection) | 76.2% (16/21); false positives 0% (0/9) |
| Strict-enum validity of validator outputs | 87.2% (34/39) |
| External Evidence Grounding | 83.3% (5/6) |
| Overall Quality Score | 90.9% |
| Diagnostics | status coherence 58.3%; incident-ID consistency 0% |

## Error Patterns

Full table with frequencies, causes and fixes: `results/error_patterns.md`. Main V1 patterns: E09 inconsistent output (ID instability in 12/12 cases; overloaded `escalation_required` semantics → 15/36 incoherent runs; verdict/status variance across rounds), E02 unsupported causal assertions (2/36), E06 excessive escalation on the injection case (2/24), E10 validator blind spots (0/2 natural leaks caught; VF-05 missed 3/3; 5/39 free-text labels). E01, E05, E07, E08 were observed 0 times.

## Improvements Implemented

Ten changes C1–C10, each with problem → evidence → hypothesis → change → result, in `CHANGELOG.md`. Essentials: orchestrator-supplied `incident_id`; `escalation_required ⇔ HUMAN_REVIEW_REQUIRED`, deterministic status table and status mapping; severity rubric and confidence bands; ReAct-style external-evidence fields; conditional mitigation; validator given the raw report with a C1–C8 checklist; router and one-retry loop; one few-shot example per prompt.

## V2 Results

| Metric | V2 |
|---|--:|
| Task Success Rate | 94.4% (34/36) |
| Schema Compliance | 100% (138/138 stage outputs; 6 skipped by router) |
| Classification Accuracy | 98.6% (category 35/36, severity 36/36) |
| Hallucination / Unsupported Claim Rate | 2.8% (1/36) |
| Fallback Accuracy | 100% (24/24) |
| Ambiguity Handling | 100% (9/9) |
| Consistency Across Runs | 91.7% |
| Validation Catch Rate (single-defect fixtures, targeted detection) | 100% (21/21); false positives 0% (0/9) |
| Strict-enum validity of validator outputs | 100% (39/39) |
| External Evidence Grounding | 100% (6/6) |
| Overall Quality Score | 97.6% |
| Diagnostics | status coherence 100%; incident-ID consistency 100%; retry rate 0% |

## V1 vs V2 Comparison

| Metric | V1 | V2 | Δ pp | Δ rel |
|---|--:|--:|--:|--:|
| Task Success | 86.1% | 94.4% | +8.3 | +9.7% |
| Schema Compliance | 100.0% | 100.0% | 0.0 | 0.0% |
| Classification Accuracy | 100.0% | 98.6% | −1.4 | −1.4% |
| Hallucination Rate (↓) | 5.6% | 2.8% | −2.8 | −50.0% |
| Fallback Accuracy | 91.7% | 100.0% | +8.3 | +9.1% |
| Ambiguity Handling | 100.0% | 100.0% | 0.0 | 0.0% |
| Consistency | 80.6% | 91.7% | +11.1 | +13.8% |
| Validation Catch Rate (single-defect, targeted) | 76.2% | 100.0% | +23.8 | +31.3% |
| Strict-enum validity, validator outputs | 87.2% | 100.0% | +12.8 | +14.7% |
| Grounding | 83.3% | 100.0% | +16.7 | +20.0% |
| Overall Quality Score | 90.9% | 97.6% | +6.7 | +7.4% |

Per-case, per-fixture and weighting-sensitivity tables: `tests/comparison.md`.

## Measured Improvement

```
Task success:          V1 = 86.1%  V2 = 94.4%  Improvement = +8.3 percentage points (31/36 → 34/36)
Consistency:           V1 = 80.6%  V2 = 91.7%  Improvement = +11.1 percentage points
Status coherence:      V1 = 58.3%  V2 = 100%   Improvement = +41.7 percentage points (diagnostic)
Validator detection:    V1 = 76.2%  V2 = 100%   Improvement = +23.8 percentage points (16/21 → 21/21; 7 single-defect fixtures × 3 rounds)
Overall score:         V1 = 90.9%  V2 = 97.6%  Improvement = +6.7 percentage points
Category accuracy:     V1 = 100%   V2 = 97.2%  Change = −2.8 percentage points (regression, TC-007 r3)
```

Case-level: TC-011 1/3 → 3/3, TC-004 2/3 → 3/3, TC-009 2/3 → 3/3, TC-007 3/3 → 2/3 (regression), others unchanged. The Overall Score gap stays between +4.3 and +7.7 pp under four different weightings. **Statistical caveat:** the intervals for task success overlap, so treat this as directional evidence. The strongest, least noise-sensitive gains are the deterministic ones (ID stability, state coherence); the ones a reviewer should trust least are the hallucination rate (1 vs 2 events) and the validator detection figures (7 fixtures × 3 rounds, lenient evidence patterns, LLM-simulated validator, cases within a round produced in one context).

Spot checks performed (not a human annotation): the reasons of the V2 validator's blocks on VF-01, VF-02, VF-05, VF-08 (round 1) were read and matched the injected fault; the judge's flagged records (V1: TC-003 r2, TC-009 r2; V2: TC-003 r2) and the TC-007 r3 output were read and are consistent with the labels.

## Post-review corrections

Two independent adversarial reviews (Codex) challenged the results. Every claim was checked against the data (`results/raw/`, `tests/`) before acting.

**Round 1**

| Claim | Verified? | Action taken |
|---|---|---|
| Detection gain came only from VF-05, a rule only V2 defines | **Yes** | Multi-defect bundles demoted to legacy; see round 2 |
| Missing judge records silently count as "no hallucination" | Code defect confirmed; no effect on published numbers | Judge loading made strict (see round 2 for the complete version) |
| Any non-`APROVAR` value counts as detection | Confirmed in code; no invalid verdicts occurred | A block requires a schema-valid output and a recognized blocking verdict |
| `final_category`/`final_severity` accept any string | Confirmed | `schemas/final_output.strict.schema.json`; strict validity reported: pipeline final outputs 36/36 both versions, validator outputs 34/39 (V1) vs 39/39 (V2). Original schema unchanged |
| `[EXTERNAL]` trusts claims inside the untrusted report | Confirmed by design | Documented as a security limitation; not fixed, not tested |
| Final summary is written after the C2 audit and not audited | Confirmed (TC-003 r2, V2) | Documented; V3 fix listed |

**Round 2**

| Claim | Verified? | Action taken |
|---|---|---|
| The "strict" judge check still accepts null / missing external fields | **Yes** — only key presence was checked | Records must have lists of strings for `*_unsupported` and booleans for the TC-009/TC-010 fields; `tests/test_evaluate_guards.py` proves that null, missing, mistyped, duplicate and missing records abort scoring. The published judge files already complied, so no number changed |
| Excluding VF-05 does not remove the contamination of the other bundles | **Yes** — VF-01 and VF-02 have `escalation_required=true` with `SUCCESS`, VF-08 has `true` with `NEEDS_INFORMATION`, and the unmodified VC-01 is blocked by V2 with no injected fault | Replaced by **single-defect fixtures** valid under both contracts (7 faults, 3 clean controls), executed for V1 and V2 × 3 rounds (60 real validator runs), with a requirement that the notes cite the injected fault. Result: V1 16/21 (76.2%), V2 21/21; 0/9 false positives in both. V1 misses: invented external evidence (approved 2/3) and a dropped contradiction (approved 3/3). Overall score recomputed: V1 90.9%, V2 97.6% |
| Conclusions attribute reliability to the pipeline without isolated execution or held-out evaluation | **Yes** (already disclosed as a limitation) | Conclusions rewritten to describe differences in the artifacts of this joint simulation; no per-component causal claims |

Still not done (needs new executions or human work): re-running each stage in a separate context with contractual inputs only, a held-out suite, human blind review of the labels that decide success and hallucination, and forged-source (`[EXTERNAL]`) tests. The evidence-pattern check for "targeted detection" is lenient by design; I read the notes for several fixtures (V2 SF-02/SF-04/SF-06, V1 SF-02/SF-06) and they matched the injected faults.

## Remaining Limitations

1. **Small, easy-ish suite.** 12 cases and 3 rounds; the strong V1 leaves little headroom; borderline cases accept several labels, so accuracy metrics discriminate weakly.
2. **Execution fidelity.** LLM sub-agents simulate stages; cases inside a round share a context; no sampling parameters; executor and judge share a model family; no human gold labels.
3. **Simulated external source and orchestrator.** No live tool loop; schema-validate-then-retry and the retry loop were never triggered in natural runs (retry rate 0%). Invalid format (input), irrelevant information and non-JSON stage output are *design only*.
4. **Diagnostics encode V2 semantics** (coherence), and V2's severity rubric was authored alongside the expected labels.
5. **V2 defects left unfixed to keep the comparison valid:** category on vague reports, confidence threshold boundary, SEV1 wording for finished access windows, summary causal wording, `SKIPPED_BY_ROUTER` label rule (`CHANGELOG.md`, "V2 residual gaps"). Schema enums for `final_*` fields not added.
6. **Validator on natural runs is not proven reliable**: it approved a wrong category and its own summary carried an unsupported causal claim; its measured advantage on single-defect fixtures (V1 76.2% vs V2 100%) is small-sample evidence.
7. **External-source trust.** `[EXTERNAL]` facts come from the untrusted report itself; a sender can fabricate a "consulted" source. Untested (no forgery cases).
8. **Validator evidence is thin**: 7 single-defect fixtures × 3 rounds, lenient evidence patterns, the fixtures come from the same authors and the same model family as the validator, and each round's 10 items were produced in one context.
9. **No cost, latency or token measurements.**

## Production Readiness Considerations

Not production-ready: no orchestrator code, logging, redaction, monitoring or human-review workflow; injection resistance measured on one case and one fixture; sampling parameters unpinned. Required steps and the gap analysis are in `docs/07_production_considerations.md` (shadow-mode pilot, redaction layer, larger labeled suite, promotion gate on these metrics, pinned model and parameters).

## Conclusion

Within this simulation, V2 artifacts are more consistent and more explicitly specified than V1's: stable identifiers, coherent status fields, verdict consistency 100%, no improvised label placeholders, correct fallback on all 24 evaluable runs, and a validator that blocked and correctly cited all 21 single-defect runs (V1: 16/21) with no false positives. It also introduced one small classification regression (TC-007) and still let one unsupported claim through. These are differences between the outputs of a joint LLM simulation, executed once per round in shared contexts, tuned on the same 12 cases and judged by a same-family LLM; they do **not** show that each prompt change caused the differences, nor that the pipeline would behave this way with isolated stages, a held-out suite or human-labelled outcomes. The engineering value of the exercise lies as much in the *method* — frozen baseline, blind runs, fault injection, logged corrections, and two reviews that each overturned part of an earlier conclusion — as in the numbers. The next iteration should fix the documented V2 gaps, add schema enums, run stages in isolation on a held-out and human-labelled suite, add forged-source tests, and build a real orchestrator with a live external-tool loop.

## References

- Wei, J., Wang, X., Schuurmans, D., Bosma, M., Ichter, B., Xia, F., Chi, E., Le, Q., & Zhou, D. (2022). *Chain-of-Thought Prompting Elicits Reasoning in Large Language Models*. arXiv:2201.11903. https://arxiv.org/abs/2201.11903
- Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K., & Cao, Y. (2022). *ReAct: Synergizing Reasoning and Acting in Language Models*. arXiv:2210.03629. https://arxiv.org/abs/2210.03629
- DAIR.AI. *Prompt Engineering Guide*. https://www.promptingguide.ai/ (accessed 2026-09-19).
- OpenAI. *OpenAI Platform Documentation*. https://platform.openai.com/docs (course-provided reference; not consulted for this work).
