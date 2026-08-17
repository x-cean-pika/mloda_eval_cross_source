# mloda-eval: cross-source retrieval

Does an AI agent retrieve data more accurately by **declaring what it needs** through mloda
than by **generating its own queries** against the same sources?

Every mloda monetization path bets on that claim. This repository tests it.

## The experiment

Fifteen questions, each answerable only by combining three source systems — a relational
database, an unstructured document corpus, and a knowledge graph sharing one entity set.
No single query language reaches the answer. Two arms, same model, same questions:

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
| Model (`provider/id@snapshot`, cache off) | |
| API budget ceiling | |

**Author separation is the rule that makes the result trustworthy.** Whoever builds the
fixture and writes the namespace, feature descriptions, and baseline data dictionary must
never see the questions. Whoever writes the questions, answer key, and rubric works from a
plain-English domain description, never from the namespace. One person doing both will
write questions the namespace happens to answer well — not dishonestly, just unavoidably —
and the result will measure nothing.

Known limitation: with two people this separation is partial, since both work against a
fixture one of them designed. Recorded rather than pretended away.

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
