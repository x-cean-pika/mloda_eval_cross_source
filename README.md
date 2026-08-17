# mloda-eval: cross-source retrieval

Does an AI agent retrieve data more accurately by **declaring what it needs** through mloda
than by **generating its own queries** against the same sources?

Every mloda monetization path bets on that claim. This repository tests it.

## The experiment

Fifteen questions, each answerable only by combining three source systems. No single query
language reaches the answer.

| Layer | Source | Licence, verified 2026-08-17 |
|---|---|---|
| Relational | **Synthea**, seeded | Apache-2.0 — clean |
| Graph | **Hetionet v1.0**, licence-filtered | CC0 + per-edge attributes |
| Documents | **PMC Open Access**, CC0 and CC BY only | commercial-use package |

Linkage is on clinical concepts — SNOMED and RxNorm from Synthea, `Disease` and `Compound`
nodes in Hetionet, MeSH in PMC — **not on patients**, since no synthetic patient appears in
published literature. See `DATASETS.md` for the full evaluation, the twelve-source Hetionet
allowlist, and why OptimusKG and MIMIC-IV were rejected.

Two arms, same model, same questions:

| | `mloda-arm` | `baseline-arm` |
|---|---|---|
| Tools | `list_features`, `get_features` | `execute_sql`, `read_documents`, `query_graph` |
| Guidance | Feature descriptions | Data dictionary, matched word budget |
| Sees schema | No | Yes |
| Writes queries | No | Yes |

**Win condition:** gap ≥ 6/15 with two-sided Fisher exact p < 0.05. n=15 is a screen, not a
proof — a positive result that misses significance triggers a 30-question re-run before any
strategic commitment.

## Pre-registration

Fill before the first run. A pre-registration with blanks is not a pre-registration.

| Field | Value |
|---|---|
| Frozen namespace commit SHA | |
| mloda version (PyPI pin) | |
| Model under test (`provider/id@snapshot`, cache off) | |
| Question-authoring model (**different family**) | |
| Dataset versions + Synthea seed | |
| API budget ceiling | |

**Author separation is the rule that makes the result trustworthy.** Whoever builds the
fixture and writes the namespace, feature descriptions, and baseline data dictionary must
never see the questions. Whoever produces the questions, answer key, and rubric never sees
the namespace. One party doing both will produce questions the namespace happens to answer
well — not dishonestly, just unavoidably — and the result will measure nothing.

### Question authorship protocol

Questions are **agent-authored, deterministically verified, human-reviewed.**

**The agent never answers a question. It writes a query that computes the answer.**

| | Circular ❌ | Rigorous ✅ |
|---|---|---|
| Prompt | "what is the answer?" | "write SQL / a traversal / a retrieval that computes this" |
| Truth source | model knowledge | **execution output** |
| Verifiable | no | yes — rerun it |
| Humans review | a medical fact | query logic |

If a model produces the answer key from its own knowledge, the test is graded by the same
class of system under test: when the author model and `mloda-arm` are wrong the same way, a
wrong answer scores correct, and the benchmark measures agreement with a model rather than
correctness.

**Steps.**

1. Agent proposes concept combinations spanning all three sources.
2. Agent writes three deterministic queries — Synthea SQL, Hetionet traversal, PMC retrieval.
3. **Queries are executed. The output is the answer key.** Correct by construction.
4. Agent phrases the natural-language question whose answer is that output.
5. **Clinical reviewer** — is this a question a real person would ask, and is it medically
   coherent? ~1 hour for fifteen. Never sees the namespace or the queries.
6. **Tom** — does the query compute what the question asks, and does answering genuinely
   require all three sources? Needs data literacy, not clinical knowledge. Never sees the
   namespace.
7. Question set, answer key and rubric are frozen together.

**Use a different model family to author than the one under test.** Same model on both sides
yields questions phrased the way that model finds natural, which flatters it. Record both
model pins in the pre-registration table.

Known limitation: with two people the separation is partial, since both work against a
fixture one of them designed. Recorded rather than pretended away.

**When writing questions, work against the licence-filtered graph, not the full one.**
Hetionet's drug-treats-disease edges come partly from non-commercial sources that the
allowlist excludes. See `DATASETS.md`. Questions written against the unfiltered graph will
produce an answer key the fixture cannot reproduce.

## Layout

```
fixture/     three-source corpus, entity IDs, generator seed
namespace/   FeatureGroups and list_features descriptions
questions/   question set, frozen answer key, grading rubric
arms/        MCP server (mloda-arm), tool surface (baseline-arm)
results/     run transcripts, scores, FeatureGroup register
```

All five are committed. They are the evidence.

## Status

Not yet started. Design doc: `mloda_business_plans/2026-08-17-prove-the-runtime-hypothesis.md`
(private).
