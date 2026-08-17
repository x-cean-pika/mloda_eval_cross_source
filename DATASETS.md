# Fixture candidates: healthcare / biomedical data sources

Candidate datasets for the three-source fixture, ranked per layer with backups.

**Read the verification legend first. Not everything here is confirmed.**

| Mark | Meaning |
|---|---|
| ✅ | Verified against a primary source on 2026-08-17 |
| ⚠️ | **Recalled, not verified. Check before relying on it.** |
| ❌ | Confirmed constraint that disqualifies it for this experiment |

Nothing in this file is legal advice. Every licence marked ⚠️ needs a human to read the
actual terms before the data enters a benchmark you intend to publish.

---

## ✅ LICENCE VERIFICATION — completed 2026-08-17

All three selected sources checked against primary sources. **The graph-layer pick changed
as a result.**

### Synthea — CLEAN, use it

Standard Apache-2.0, verified from the LICENSE file. No non-standard clauses. Commercial use
and redistribution both permitted. Obligations: preserve NOTICE attribution in derivative
works; do not use Synthea/MITRE trademarks beyond describing origin. **No further diligence.**

### PMC Open Access — CLEAN, filter structurally

The subset is pre-split into three groupings and the FTP bulk packages follow them:

| Grouping | Licences | Use? |
|---|---|---|
| Commercial Use Allowed | CC0, CC BY, CC BY-SA, CC BY-ND | yes, with a caveat |
| Non-Commercial Only | CC BY-NC, CC BY-NC-SA, CC BY-NC-ND | no |
| Other | no machine-readable CC licence, or custom | no |

Download the commercial package only; filtering needs no per-article work.

**Refinement: restrict further to CC0 and CC BY.** CC BY-ND forbids derivatives, and chunking
articles for retrieval is arguably derivative. CC BY-SA's share-alike could propagate to the
published fixture. Dropping both removes two grey areas at no meaningful cost in volume.

### OptimusKG — BLOCKED for commercial publication

Verbatim from the README:

> "OptimusKG integrates multiple primary data resources, each of which is subject to its own
> license and terms of use. These terms may impose restrictions on redistribution,
> commercial use, or downstream applications."
>
> "Some resources provide data under academic or noncommercial licenses, while others may
> impose attribution or usage requirements."
>
> "Users are responsible for reviewing and complying with the license and terms of use of
> each primary dataset, as specified by the original data providers."

**And the README supplies no per-source licence manifest.** There is no map of which of the
65 sources carry which terms, so establishing commercial rights means identifying and
checking all 65 — with the project stating up front that some are non-commercial.

Access details, recorded for later: Harvard Dataverse DOI `10.7910/DVN/IYNGEV`, PyPI package
`optimuskg`.

**Middle path if the Polars/Parquet fit proves worth it:** use OptimusKG internally and
publish results only, never the fixture. That sacrifices reproducibility, which was the
reason for choosing open data in the first place.

### Hetionet — PROMOTED to graph-layer pick

Not on data quality. OptimusKG is larger, newer, and the better technical fit. Hetionet wins
on the one criterion that decides publishability: **it applies a licence attribute on a
per-node and per-edge basis** for sources with defined licences. The licence metadata ships
*inside the graph*, so it can be filtered programmatically and what you used can be proven.

An afternoon of filtering against a 65-source legal review is not a close call for a
two-person company.


#### Hetionet source-licence detail — the allowlist

The README says only "all original content is CC0" and defers to a table. The table
(`dhimmel/integrate` @ `d482033`, linked from the README) is specific, and the picture is
mixed. CC0 covers Himmelstein's own integration work; the underlying 29 sources vary.

| Category | Sources | Usable commercially? |
|---|---|---|
| Public domain | Entrez Gene, LabeledIn, MEDLINE, MeSH, Pathway Interaction Database | ✅ |
| CC BY 3.0 | Disease Ontology, Uberon, WikiPathways | ✅ attribution |
| CC BY 4.0 | DISEASES, **DrugCentral**, Gene Ontology, TISSUES | ✅ attribution |
| CC BY-NC-SA | **MEDI**, **PREDICT**, **SIDER 4** | ❌ non-commercial |
| Custom permissive | GWAS Catalog, Reactome, LINCS L1000, BindingDB, DisGeNET (ODbL), **DrugBank 4.2** | ⚠️ read each |
| Restrictive | MSigDB | ❌ |
| No licence | ADEPTUS, Bgee, DOAF, ehrlink, Evolutionary Rate Covariation, hetio-dag, Incomplete Interactome, Human Interactome Database, STARGEO | ❌ no grant to rely on |

**This affects the example question.** `Compound–treats–Disease` edges derive partly from
PREDICT and MEDI (CC BY-NC-SA), and side effects from SIDER 4 (CC BY-NC-SA). Those are among
the most useful edges in the graph and they are not available to a commercial publication.

**DrugCentral rescues it:** CC BY 4.0 and it supplies drug indications, so drug-treats-disease
survives if you take DrugCentral-sourced edges and drop the PREDICT/MEDI ones.

**DrugBank 4.2** carries a custom University of Alberta licence and has historically required
a paid commercial licence. Treat as excluded until someone reads the actual terms.

**Use an allowlist, not a blocklist.** Filter on the per-edge `license` attribute to:

```
Entrez Gene, LabeledIn, MEDLINE, MeSH, Pathway Interaction Database,
Disease Ontology, Uberon, WikiPathways, DISEASES, DrugCentral,
Gene Ontology, TISSUES
```

Surviving content: diseases, genes, anatomy, pathways, disease-gene associations, tissue
expression, drug indications. Sufficient for cross-source questions. Anything outside the
allowlist requires a specific, recorded decision.

⚠️ **The licence table is a 2016-era snapshot** reflecting Hetionet v1.0's build state.
Upstream terms change. Re-check the allowlisted sources before publishing.

Table: https://github.com/dhimmel/integrate/blob/d482033bcaa913a976faf4a6ee08497281c739c3/licenses/README.md
Discussion: https://doi.org/10.15363/thinklab.d107

### Verified stack

| Layer | Use | Status |
|---|---|---|
| Structured | Synthea | Apache-2.0 ✅ clean |
| Graph | Hetionet v1.0 | CC0 + per-edge licence attributes ✅ filterable |
| Documents | PMC OA, CC0 and CC BY only | ✅ structural filter |

Remaining check: re-verify the allowlisted sources' current terms before publishing. The
filter itself is mechanical — see the allowlist above.

---

## The criteria

Ranked in the order that decides the choice.

**1. Information independence — the one that matters most.** Each source must hold facts the
others do not. If the documents merely restate the database, a "cross-source" question is
theatre: the model answers from one source and the experiment measures nothing. This single
criterion eliminates otherwise-attractive options.

**2. Licence clarity for commercial use.** Results will be published as marketing by a
for-profit company. "Free for research" is not the same as "free for us." Several candidates
below are free for academics and restricted for companies.

**3. Redistributability.** A pre-registered benchmark nobody can rerun is weak evidence. If
the fixture cannot ship with the results, reproducibility depends on trusting us.

**4. Entity linkability.** Sources must share resolvable identifiers. In this domain that
means clinical concept codes — SNOMED, RxNorm, MeSH — not patient IDs, since no synthetic
patient appears in published literature.

**5. Scale and difficulty.** Enough volume that aggregate questions return meaningful
numbers, and enough messiness that the easy case is not the only case.

**6. LLM-API compatibility.** The entire experiment pipes data to a frontier model. Any
source forbidding third-party API processing forces self-hosted inference, which changes
what the result means.

---

## Layer 1 — Structured / relational

### 🥇 Top pick: Synthea

Synthetic patient population simulator from MITRE. Models full medical histories:
demographics, conditions, medications, procedures, encounters, observations.

- **Licence: Apache-2.0** ✅ — unambiguous commercial use, redistributable
- Exports HL7 FHIR R4, C-CDA, and CSV ✅
- Repository actively maintained; last updated June 2026 ✅
- **Seeded generation, so the fixture is exactly reproducible** — record the seed as a
  pre-registration field
- Unlimited scale: generate 100 patients or 100,000
- Weakness: synthetic. Coding patterns are cleaner than reality, so it likely *overstates*
  both arms. Acceptable, since it biases neither arm preferentially.

Sources: https://github.com/synthetichealth/synthea · https://synthea.mitre.org/downloads

### 🥈 Backup: MIMIC-IV Clinical Database Demo

100 real patients from MIMIC-IV v2.2, no registration and no credentialing required.

- **Openly available, no DUA** ✅ — also mirrored on the AWS Open Data Registry
- **Excludes free-text clinical notes** ✅ — so it cannot serve Layer 3
- ⚠️ **Licence needs checking.** Believed ODbL-style open, and the credentialed DUA's
  third-party-API prohibition should not attach to the open demo — but that inference has
  not been verified, and it is exactly the kind of thing to confirm rather than assume.
- Weakness: 100 patients caps difficulty. Aggregate questions return single digits.

Choose over Synthea only if realism matters more than volume.

Source: https://physionet.org/content/mimic-iv-demo/2.2/

### 🥉 Backup: Synthea Coherent Data Set

Pre-generated Synthea population on AWS Open Data, bundling clinical notes, DNA, and imaging
alongside structured records for the same patients.

- Entity linkage is free — same synthetic patients across all artifacts
- ⚠️ **Fails criterion 1 if the notes are generated from the structured data**, which would
  make them a restatement rather than an independent source. **Inspect the notes before
  adopting.** This is the single most important check in this file.

Source: https://registry.opendata.aws/synthea-coherent-data/

### ❌ Disqualified: MIMIC-IV (full), MIMIC-IV-Note, eICU

Real clinical data including genuine free-text notes linked by `subject_id`. The best data
available, and wrong for this experiment.

- Requires PhysioNet credentialing + CITI "Data or Specimens Only Research" + a per-dataset
  DUA ✅ — days to weeks
- **Each user must credential individually; team sharing is not permitted** ✅
- **Redistribution prohibited** ✅ — the fixture could never ship with results
- **Sending it through third-party LLM APIs such as OpenAI is explicitly prohibited** ✅ —
  PhysioNet has a dedicated notice. Compliant routes are Azure OpenAI with human-review
  opt-out, Amazon Bedrock, or self-hosted open-weight models.

Publishing *papers* using MIMIC is permitted and extremely common — that is not the problem.
The problem is that this experiment consists entirely of handing data to a frontier model.

Worth noting the irony, and the argument on the other side: that restriction is
*data may not leave your environment for inference* — which is Open Question 7 in the design
doc verbatim. Building on MIMIC would force the inference-residency question in week one
rather than at a sales meeting. A defensible reason to choose it despite the cost.

Sources: https://physionet.org/news/post/llm-responsible-use/ ·
https://physionet.org/about/citi-course/

### ⚠️ Unevaluated: CMS Synthetic Public Use Files (SynPUF)

Synthesized Medicare claims, public. Claims rather than clinical data — different shape,
probably a worse fit for knowledge-base questions. Not investigated.

---

## Layer 2 — Knowledge graph

### 🥇 Top pick: OptimusKG

Zitnik Lab, Harvard Medical School — the same group behind PrimeKG, and its successor in
lineage if not in name. "A modern multimodal knowledge graph with type-specific metadata
across biomedical domains."

- **Code licence: MIT** ✅
- **190,531 nodes across 10 entity types; 21,813,816 edges across 27 relation types** ✅ —
  roughly 5× PrimeKG and 10× Hetionet on edges
- **67,249,863 property instances encoding 110,276,843 values across 150 property keys** ✅
- **65 integrated sources, grounded with 18 ontologies and controlled vocabularies** ✅
- **Pushed 2026-08-16** ✅ — actively maintained, unlike every alternative here
- Distributed as **Apache Parquet via Harvard Dataverse**, with a **Python client on PyPI**;
  loads as **Polars DataFrames** or NetworkX graphs, with local caching ✅
- Standardised on the BioCypher framework and the Biolink Model ✅

**Why it fits this experiment better than a bigger graph normally would:**

1. **Parquet and Polars are mloda-native.** mloda's core dependency is pyarrow and it ships a
   `PolarsDataFrame` compute framework. The graph layer drops in with almost no glue.
2. **150 property keys is where the hypothesis should be most visible.** A declarative
   interface's advantage over query generation should widen as the attribute surface grows.
   The baseline arm must discover and navigate 150 keys by hand; `mloda-arm` names a feature.
   If *declaring beats generating* holds anywhere, it holds here.
3. **Ontology grounding makes concept linkage to SNOMED, RxNorm and MeSH cleaner** than
   hand-mapping against an ungrounded graph.

⚠️ **Licence diligence is larger here, not smaller.** MIT covers the codebase. The graph
integrates **65 upstream sources with varying licences**, and the project states explicitly
that it "does not alter or override these source-specific licensing conditions." That is a
bigger surface than Hetionet's 29 sources. The per-source metadata should make the check
tractable, but it is real work and it must happen before publishing.

⚠️ Also unverified: the PyPI client's package name, and whether 21.8M edges needs subsetting
for a 15-question evaluation. Parquet plus Polars makes subsetting cheap either way.

Source: https://github.com/mims-harvard/optimuskg

### 🥈 Backup: PrimeKG — *pending licence check*

Precision Medicine Knowledge Graph, Harvard MIMS.

- 100,000+ nodes, 4,000,000+ relationships, 29 edge types ✅
- Integrates 20 public resources, covering 17,000+ diseases ✅
- **Carries clinical guideline text descriptions** ✅ — meaning the graph layer holds prose
  the database does not, which strengthens information independence
- Published 2023 — materially newer than Hetionet
- ⚠️ **Licence not verified.** Could not confirm from search. **Check the repository before
  committing.** If the licence is restrictive for commercial use, fall back to Hetionet.

Source: https://github.com/mims-harvard/PrimeKG

### 🥉 Backup: Hetionet v1.0

Integrative biomedical hetnet built for drug repurposing.

- **Licence: CC0** ✅ — the strongest licence position of any candidate here
- 47,031 nodes across 11 types; 2,250,197 edges across 24 types ✅
- Ships in JSON and **Neo4j** formats, both carrying node and edge properties ✅ (TSV and
  matrix formats drop those properties — use JSON or Neo4j)
- Per-node and per-edge licence attributes, since it integrates 29 upstream sources ✅ —
  meaning even under CC0 for original content, **check the attributes for sources you rely
  on**
- Weakness: published 2017. Nine years old.

The safety pick. Choose it if OptimusKG's 65-source licence check or PrimeKG's licence
proves awkward, or if unambiguous CC0 is worth more to you than graph richness or recency.
It is a defensible choice, not a bad one — just a nine-year-old one.

Source: https://github.com/hetio/hetionet

### ⚠️ Unverified alternatives

- **Open Targets** — drug-target-disease associations, open, actively maintained. Not
  verified this session. Worth a look if PrimeKG's licence fails.
- **SPOKE** — large biomedical KG; access terms unknown.
- **UMLS** — authoritative medical terminology. Free but requires licence registration, and
  it is a terminology rather than a rich graph.
- **SNOMED CT** — licensing varies by country; free in member countries. ⚠️ Germany's
  membership status should be confirmed if you want to rely on it, and you are Berlin-based
  so this may be free for you.
- **RxNorm, MeSH, ICD-10** — free, and useful as the *linkage vocabulary* rather than as the
  graph layer itself. RxNorm and MeSH are how Layers 1 and 3 connect to Layer 2.

---

## Layer 3 — Unstructured documents

This layer is the hardest to source well, and the one where my first recommendation was
weakest.

### 🥇 Top pick: PubMed Central Open Access Subset

Full-text biomedical literature, bulk downloadable.

- **Passes criterion 1 decisively** — it is the only candidate holding knowledge that is
  genuinely absent from both the patient record and the graph: what studies actually report.
  That independence is what makes a three-source question real rather than decorative.
- Links to the other layers through MeSH terms
- ⚠️ **Licence needs care, and this is the trap.** The PMC OA Subset is split into
  commercial-use and non-commercial-only portions. A for-profit company publishing a
  benchmark must restrict itself to the commercial-use subset. **Filter on licence at
  download time, not afterwards.**
- Weakness: genre mismatch. It is published literature, not the internal documents a
  knowledge-base product would actually index. Accept knowingly — the independence is worth
  more than the genre fidelity for a first experiment.

### 🥈 Backup: MTSamples

Several thousand transcribed medical reports, freely available. Closer in genre to real
clinical prose than literature is.

- ⚠️ **Licence genuinely unclear.** Widely used, rarely with an explicit grant. Verify before
  publishing anything built on it.
- Weakness: no entity link to your patients or your graph. Would need concept extraction to
  connect, which is real work.

### 🥉 Conditional: Synthea Coherent clinical notes

See Layer 1. Free entity linkage, but **only usable if the notes contain information not
already in the structured records.** Inspect first.

### ❌ Disqualified: MIMIC-IV-Note, n2c2 / i2b2

Real clinical notes, and the best fit on genre by a wide margin. Both require DUAs; MIMIC-IV-
Note inherits the third-party-LLM-API prohibition above.

---

## Recommended stack

| Layer | Pick | Holds what nothing else does | Licence |
|---|---|---|---|
| Structured | Synthea | who has which condition, who takes which drug | Apache-2.0 ✅ |
| Graph | **Hetionet v1.0** (OptimusKG blocked on licence) | how drugs, diseases and genes relate | CC0 + per-edge attributes ✅ |
| Documents | PMC OA, **CC0 and CC BY only** | what the literature reports | ✅ structural filter |

Linkage vocabulary: **SNOMED and RxNorm** (from Synthea) → **Hetionet concept nodes** → **MeSH** (in PMC). Concepts are the shared entity set. Patients are not.

Example of a question this stack makes genuinely three-source:

> Among patients prescribed drug D who also have condition C, what do published studies
> report about contraindications, and which alternative compounds does the graph associate
> with C?

- patients on D with C → **Synthea**, relational query
- contraindications in the literature → **PMC**, retrieval over text
- alternatives for C → **Hetionet**, graph traversal

No single query language reaches the answer. That asymmetry is the experiment.

---

## Verification checklist — do these before the fixture build

- [ ] **OptimusKG's 65 upstream source licences** — MIT covers the code only, and the project
      explicitly does not override source terms. Largest diligence surface of any candidate,
      and the one most likely to change the recommendation.
- [ ] **PrimeKG licence** — read the repository licence. Commercial use permitted? Only
      matters if OptimusKG's source check fails.
- [ ] **PMC OA commercial subset** — confirm how to filter, and filter at download.
- [ ] **Synthea Coherent notes** — read a sample. Do they contain facts absent from the
      structured records, or restate them? This decides whether the option exists at all.
- [ ] **MIMIC-IV Demo licence** — confirm the open demo does not inherit the credentialed
      DUA's third-party-API prohibition. Only matters if you use it.
- [ ] **Hetionet per-source licence attributes** — CC0 covers original content; upstream
      sources carry their own terms on node and edge attributes.
- [ ] **MTSamples licence** — only if you adopt it.
- [ ] **SNOMED CT** — confirm Germany's member-country status if you rely on SNOMED beyond
      what Synthea already emits.

Record every answer in this file. The next person to ask will be you, in four months, and
"I think it was fine" will not be a good enough answer to publish on.

---

## What this changes in the design doc

**A fifth pre-registration field.** Which data was used is as load-bearing as which model:
Synthea version **and generation seed**, OptimusKG (or PrimeKG/Hetionet) version and
Dataverse DOI, PMC snapshot date.
Without the seed the fixture is not reproducible even by you.

**Fixture effort drops from 10–15 days to roughly 5–8.** You still load three stores, verify
concept linkage, and define a noise model — but you are no longer inventing a corpus, which
was the genuinely hard part.

**Reviewer Concern 1 improves.** The limitation was that both authors work against a fixture
one of them designed. With third-party data, neither built it. Not fully closed — you still
choose which slice — but a real gain at zero cost.
