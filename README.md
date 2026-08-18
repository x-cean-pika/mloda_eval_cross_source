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

**The mloda pin is the field most likely to break the run.** mloda shipped five releases in
three months (v0.6.3 → v0.11.0, ~180 commits in July alone). At that cadence an unpinned run
is a run whose treatment arm changes underneath it, and the frozen namespace SHA means
nothing if the framework beneath it moved. Pin to PyPI, never to a working copy. Freeze it,
and do not bump it mid-experiment even for a fix that looks unrelated.

**Author separation is the rule that makes the result trustworthy.** Whoever builds the
fixture and writes the namespace, feature descriptions, and baseline data dictionary must
never see the questions. Whoever produces the questions, answer key, and rubric never sees
the namespace. One party doing both will produce questions the namespace happens to answer
well — not dishonestly, just unavoidably — and the result will measure nothing.

### The freeze

The frozen namespace SHA is the only part of author separation that can be made into hard
evidence rather than an assurance. Everything below exists to make it worth something.

**What it pins.** Three artifacts, one commit: the FeatureGroups and `list_features`
descriptions in `namespace/`, and `baseline-arm`'s data dictionary in `arms/`. It does not
pin mloda — that is the separate PyPI field above.

**A local SHA proves nothing.** Both git dates are environment variables
(`GIT_AUTHOR_DATE`, `GIT_COMMITTER_DATE`), `--amend --date` rewrites them, and a rebase
rewrites SHAs wholesale. A commit reading "frozen 1 September" could have been made on the
20th, after the questions existed, and the repository would not show it. The hash proves
*this content goes with this hash*; it does not prove *this hash existed by this date*.
Recording it in a design doc you also control is circular — evidence and claim sit in the
same custody.

**So put the hash in someone else's hands before the questions exist.** Cheapest first:

```bash
git tag -a freeze/namespace-v1 -m "namespace + descriptions + data dictionary frozen before question authoring"
git push origin main
git push origin freeze/namespace-v1
```

Then enable **tag protection** in the repo settings so the tag cannot be quietly moved.

Two cheap reinforcements, both worth doing:

- **Let CI stamp it.** A workflow triggered on tag push that does nothing but echo the SHA.
  The Actions run record carries a GitHub-side timestamp bound to the commit and persists,
  where the public events API retains push records only ~90 days.
- **Email the SHA to the external clinical reviewer.** They are already contracted and
  already required never to see the namespace, so a 40-character hash reveals nothing to
  them. It puts a dated copy on infrastructure neither party controls, and it is the most
  legible form to a reader: an outside party held this hash on this date.

**The ordering is the substance.** The push must precede the existence of any question, and
the record must show that. Have the question author's first commit reference the freeze SHA,
so their work visibly starts from a known frozen point rather than both timelines being
asserted afterwards.

### Freeze scope, and what happens when the harness breaks

`arms/` holds two categorically different things, and one commit SHA pins both:

| | What it is | If it changes |
|---|---|---|
| Baseline data dictionary | **Treatment variable**, matched against `list_features` descriptions | The comparison measures something else |
| MCP server, baseline tools, runner, grader | **Plumbing** | The measurement runs, or it does not |

Partway through the runs a harness bug will surface. A `get_features` timeout that reaches
the agent as a tool error rather than rows. `query_graph` returning nondeterministic row
order that the grader mis-scores. Token accounting double-counting tool results on one arm.
Missing rate-limit backoff producing failures that are not evenly distributed across arms.
Each corrupts data while saying nothing about the hypothesis.

**Decide the policy now, because you will be deciding it while looking at partial results.**
By then you will know which arm the bug is hurting, and the fix that helps your arm will
genuinely look more necessary than the one that does not. That is how motivated reasoning
works on everyone. Do not plan to rely on judgment at that moment.

**Frozen hard — any change voids the primary result.** All of `namespace/`, and the
`baseline-arm` data dictionary. These are the treatment variable on their respective arms.

**Fixable, under a deviation log.** MCP server, baseline tool implementations, runner,
grading scripts. Every fix is recorded in `DEVIATIONS.md` with: timestamp, before SHA, after
SHA, what broke with the failing output, **why the fix cannot change relative arm
performance**, and which completed runs are discarded. The fourth field is the one doing the
work — an argument you cannot write down is a fix you should not make.

**Any harness fix discards all completed runs. Both arms restart from zero.** Deliberately
rigid: it removes judgment at the exact moment judgment is least trustworthy. Check the cost
before flinching — 15 questions × 2 arms × 3 runs × ≤15 turns is an API bill and a few hours,
not a month. This is why the sixth pre-registration field is a currency ceiling rather than a
turn count: the budget is sized so a discarded run is affordable. A rigid protocol you can
afford to obey beats a flexible one you will rationalise around.

**Smoke-test before freezing, so you rarely need any of the above.** Before the freeze, the
*fixture author* writes two or three throwaway questions and runs the complete pipeline on
them: both arms, every tool, grading, token accounting. Those questions are contaminated by
construction — the same person wrote the namespace — which is exactly what makes them safe
here. They exist to shake out timeouts, ordering bugs, accounting errors and rate limits
while fixing them is still free. Then delete them, freeze, and hand off. Half a day, and it
catches most of what would otherwise become a deviation.

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
fixture/         three-source corpus, entity IDs, generator seed
namespace/       FeatureGroups and list_features descriptions   [frozen hard]
questions/       question set, frozen answer key, grading rubric
arms/            MCP server (mloda-arm), tool surface (baseline-arm)
                 data dictionary is [frozen hard]; harness code is fixable under DEVIATIONS.md
results/         run transcripts, scores, FeatureGroup register
DEVIATIONS.md    every post-freeze harness change, or a statement that there were none
```

All of it is committed. It is the evidence.

## Status

**Not yet started.** Nothing in `fixture/`, `namespace/`, `questions/`, `arms/` or `results/`.
The blocking item is the pre-registration table above: six fields, all blank.

Design doc: `mloda_business_plans/2026-08-17-prove-the-runtime-hypothesis.md` (private),
**Revision 9** (2026-08-18). Revision 9 changed nothing in this experiment — no deliverable,
estimate, success criterion, or decision-table row moved. It corrected the lineage/audit
findings in the design doc after an upstream sync, and sharpened the mloda pin warning above.
