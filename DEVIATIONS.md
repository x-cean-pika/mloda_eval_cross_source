# Deviations

Every change to the evaluation made after the namespace freeze. If this file says "none,"
that is a claim the repository history has to support.

**Status: none recorded. The freeze has not happened yet.**

---

## The rule

Two categories, decided before the freeze so that nobody is deciding them while looking at
partial results. Full reasoning in `README.md` → *Freeze scope*.

| Category | Contents | Policy |
|---|---|---|
| **Frozen hard** | all of `namespace/`; `baseline-arm`'s data dictionary | Any change **voids the primary result**. Re-run from scratch, or log the work as post-freeze and exclude it. |
| **Fixable — behaviour** | MCP server, baseline tools, runner: anything touching tool behaviour, retries, or turn caps | Permitted, logged below, **and all completed runs are discarded — both arms restart from zero.** |
| **Fixable — instrumentation** | Pure measurement that cannot move the primary result: token accounting, timing, transcript formatting | Permitted, logged below, recomputed. **No restart.** |

The restart rule is deliberately rigid. It removes judgment at the moment judgment is least
trustworthy. The budget ceiling in the pre-registration is sized so a discarded run is
affordable.

**It is scoped rather than absolute, because an absolute version punishes the honest act.** If
every fix voids all runs, noticing a bug at run 28 is expensive for whoever would eat the
restart, and a borderline anomaly starts to look like "just behaviour." The dividing line is
whether the defect **could move the primary result** — token cost is reported and explicitly
excluded from the win definition, so an accounting bug is recomputable; a retry-budget bug is
not. Draw the line here, in advance, not at the moment it pays to draw it differently.

**Grading does not begin until every run completes**, and during the run transcripts are
inspected for tool-level faults only. That is what makes this survivable: you cannot be swayed
by which arm a fault favours if you do not yet know which arm is ahead.

**Anything not in either category is frozen hard by default.** If you find yourself arguing
about which column something belongs to, it belongs in the left one.

---

## Entry template

Copy this block per deviation. Fill every field. A blank field is an unlogged deviation.

```markdown
### D<N> — <one-line description>

- **Timestamp (UTC):**
- **Before SHA:**
- **After SHA:**
- **What broke:** <symptom, plus the failing output pasted verbatim>
- **Why this cannot change relative arm performance:** <the argument, in full>
- **Runs discarded:** <which, and the new run count for each arm>
- **Logged by:**
```

The fifth field is the one that matters. Writing the non-contamination argument down, at the
time, in a place a reader can check later, is the whole point of the log. **An argument you
cannot write down is a fix you should not make.**

---

## Entries

_None._
