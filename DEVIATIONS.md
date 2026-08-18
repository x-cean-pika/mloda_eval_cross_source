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
| **Fixable** | MCP server, baseline tool implementations, runner, grading scripts | Permitted, logged below, **and all completed runs are discarded — both arms restart from zero.** |

The restart rule is deliberately rigid. It removes judgment at the moment judgment is least
trustworthy. The budget ceiling in the pre-registration is sized so a discarded run is
affordable.

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
