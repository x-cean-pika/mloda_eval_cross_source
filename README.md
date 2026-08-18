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

**Win condition: two-sided McNemar exact p < 0.05** on discordant pairs. Both arms answer the
same questions, so the data are paired and Fisher exact — which assumes independent samples —
is the wrong test. Let `b` = questions `mloda-arm` got right and `baseline-arm` did not, and
`c` = the reverse. At n=15 the criterion reduces exactly to **b ≥ 6 + 2c**:

| Losses `c` | Wins `b` needed | Gap | p |
|---|---|---|---|
| 0 | 6 | 6 | 0.031 |
| 1 | 8 | 7 | 0.039 |
| 2 | 10 | 8 | 0.039 |
| 3 | 12 | 9 | 0.035 |
| 4 | — | — | impossible within 15 questions |

**`mloda-arm` can afford at most three losses, and each one costs two additional wins.** A raw
gap threshold is not used because this test implies it (`gap = b − c ≥ 6 + c`). A gap of 6 is
significant at 6 wins / 0 losses and *not* significant at 7/1 or 8/2, which is why the marginal
totals are not sufficient: **`results/` must record each arm's per-question outcome**, or the
primary analysis cannot be computed at all.

n=15 is a screen, not a proof — a positive result that misses significance triggers a
30-question re-run before any strategic commitment.

## Pre-registration

Fill before the first run. A pre-registration with blanks is not a pre-registration.

**And a pre-registration nobody outside can read is not a registration.** Every rule below is
enforced by two people who both want the answer to be yes. Pre-registration binds because it is
filed publicly *before* results exist, so that abandoning it is visible.

> **Registered 2026-08-18.** This repository was made public at commit `087787c`, before the
> fixture was built, before the namespace existed, and before a single question was written.
> https://github.com/x-cean-pika/mloda_eval_cross_source
>
> **The proof is the empty directory, not the timestamp.** `namespace/`, `questions/` and
> `results/` contain nothing in the public history. Anyone can verify that the thresholds, the
> win condition, the decision rules and the deviation policy were all published before any
> result existed to shape them. That is the whole claim, and it does not depend on trusting a
> commit date.

Publishing at this moment was also the cheapest it will ever be: with the namespace not yet
built, nothing anyone is undecided about was disclosed, and the question of whether to publish
the namespace itself stays open until it is actually due.

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

**Know what each anchor actually buys.** Tag protection defends against accident and sloppiness
— a tag moved without thinking, a rebase that rewrites history. It does **not** resist the
repository owner, who can disable protection, move the tag, re-enable it, and delete Actions
logs. Only a copy held outside your custody answers a determined skeptic.

Two reinforcements, the second of which is the one that counts:

- **Let CI stamp it.** A workflow triggered on tag push that does nothing but echo the SHA.
  The Actions run record carries a GitHub-side timestamp bound to the commit and persists,
  where the public events API retains push records only ~90 days.
- **Email the SHA to the external clinical reviewer. This is the one that resists the owner.**
  They are already contracted and already required never to see the namespace, so a
  40-character hash reveals nothing to them. It puts a dated copy on infrastructure neither
  party controls, and it is the most legible form to a reader: an outside party held this hash
  on this date. Do all three; rely on this one.

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

**A harness fix that could move the primary result discards all completed runs. Both arms
restart from zero.** Deliberately rigid: it removes judgment at the exact moment judgment is
least trustworthy. Check the cost before flinching — 15 questions × 2 arms × 3 runs × ≤15 turns
is an API bill and a few hours, not a month. This is why the sixth pre-registration field is a
currency ceiling rather than a turn count: the budget is sized so a discarded run is affordable.
A rigid protocol you can afford to obey beats a flexible one you will rationalise around.

**Scoped, because an unscoped version punishes the honest act.** If *every* fix voids all runs,
then noticing a bug at run 28 becomes expensive for the person who would eat the restart, and a
borderline anomaly acquires a pull toward "that is just behaviour." So: faults touching **tool
behaviour, retries, or turn caps** trigger the restart. Pure **instrumentation** that feeds only
reported-never-headline metrics — token accounting is the live example, since token cost is
explicitly excluded from the win definition — may be fixed and recomputed under a logged
deviation with no restart. The test is whether the defect could move the primary result, and it
is drawn here in advance rather than at the moment it pays to draw it differently.

**Smoke-test immediately after freezing, so you rarely need any of the above.** The *fixture
author* writes two or three throwaway questions and runs the pipeline on them: both arms, every
tool, token accounting. **Plumbing only. The pass criterion is "no tool faults, no timeouts,
accounting reconciles" — never "did the agent answer correctly."** They exist to shake out
timeouts, ordering bugs, accounting errors and rate limits while fixing them is still free
(nothing has run, so a fix discards nothing). Then delete them and hand off. Half a day, and it
catches most of what would otherwise become a deviation.

**This runs after the freeze, not before, and the ordering is not incidental.** Placed before,
it is a loophole: the namespace is still editable, and the author has just watched their own
arm succeed or fail on question-shaped inputs. That is a compliant channel for tuning the
namespace against questions, which is the one thing author separation exists to prevent.

**No grading until every run completes.** During the run, transcripts are inspected for
tool-level faults only — never for answer correctness. This is what makes the restart rule
survivable: you cannot be swayed by which arm a bug is hurting if you do not yet know which arm
is ahead.

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
6. **Tom Kaltofen, mloda's maintainer** — does the query compute what the question asks, and
   does answering genuinely require all three sources? Needs data literacy, not clinical
   knowledge. Never sees the namespace. **Disclosed plainly: this reviewer is an interested
   party.** He is checking query logic, not correctness of the answer key, which is produced by
   execution rather than judgement — but a reader should know the relationship rather than
   infer it.
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

Five of the six can be filled today. The sixth — the frozen namespace SHA — is an output, not a
choice: it exists only once D1 and the namespace are done, and filling it is the signal that
they are.

Design doc: `mloda_business_plans/2026-08-17-prove-the-runtime-hypothesis.md` (private),
**Revision 9** (2026-08-18). Revision 9 changed nothing in this experiment — no deliverable,
estimate, success criterion, or decision-table row moved. It corrected the lineage/audit
findings in the design doc after an upstream sync, and sharpened the mloda pin warning above.
