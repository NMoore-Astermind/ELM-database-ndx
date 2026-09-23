# Certification — corpus v3-classtargeted

**Model:** `data/elm-frozen-model-v3-classtargeted.json` · **Corpus:**
`data/elm-archetype-corpus-v3-classtargeted.json` · **Certified:** 2026-09-22 · **Team Nolan**

The bar below was committed **before the corpus existed**
(`preregistration/elm-harvest-preregistration.json`). The model was frozen **before** any
certification run. Neither was adjusted afterwards.

## The result

| criterion | test | result | |
|---|---|---:|---|
| **PRIMARY** | fresh-ecosystem coverage ≥ 30.0% | **47.2%** | **PASS** |
| **SECONDARY** | distinct labels emitted > 7 | **12** | **PASS** |
| **TERTIARY** | both evaluation repos improve separately | hono 35.8→**80.2**, trpc 24.3→**31.4** | **PASS** |
| **GUARD** | the previous corpus's exact 160 held-out files ≥ 31.8% | **36.9%** | **PASS** |

Trajectory across the project's four models, all on the same 250 fresh files at the same
operating point:

| | v1 | v2 | v3 structural | **v3 class-targeted** |
|---|---:|---:|---:|---:|
| Fresh-ecosystem coverage | 13.2% | 28.0% | 21.6% | **47.2%** |
| Distinct labels emitted | 5 | 7 | 7 | **12** |
| `service`/`utility` share (teacher: 48.4%) | 96.4% | — | 87.2% | **58.8%** |
| Trained-on held-out | 34.9% | 33.8% | 32.5% | **59.1%** |
| GUARD (v2's own 160) | — | 33.8% | — | **36.9%** |

**The GUARD is the load-bearing number.** 36.9% on the *identical* 160 files where the previous
corpus scored 33.8%: the in-distribution case improved rather than being traded away for the
out-of-distribution gain. The instrument was validated before it was used — the same 160-file path
reproduced the previous model's 33.8% exactly, with the same `service`/`utility` share and the same
label count (`results/elm-guard160-parity-v2.log`).

## ⚠️ What this does not license

**Coverage counts predictions outside `service`/`utility` whether or not they are correct.** It is
a label-free metric; it measures whether the class prior widened, not whether any prediction is
right. **No accuracy claim is made by this certification.** Accuracy requires labelling the blind
250-file evaluation set — deliberately unspent — and is capped by the teacher's **72.3%** agreement
with human judgement.

**The two evaluation repos now miss in opposite directions, and the 47.2% average hides both:**

| | coverage | model says S/U | teacher says | reading |
|---|---:|---:|---:|---|
| **hono** | 80.2% | **22.2%** | 45.7% | now **under**-predicts S/U; calls 38 of 81 files `types` |
| **trpc** | 31.4% | **76.3%** | 49.7% | still **over**-predicts S/U; clears by 1.4 pp |

hono's 80.2% is not the model being right about hono; it is the prior having swung past the teacher
in the other direction. **trpc is the honest case and it clears by 1.4 points.** A report of 47.2%
without this table is misleading.

**Per-class ecosystem dominance is untested.** The classes that became cheap came from one repo
each — `config` is 90.8% nest, `middleware` 80.0%, `schema` 78.6%, `model` 67.9% typeorm. The model
may have learned "a config file looks like a NestJS config file", which is the v1 failure
reproduced one class down. Testing it needs a fourth ecosystem per class and a bar declared first.

## 🔴 Do not read the closing block of the published coverage logs

**The logs in this folder end with a verdict that contradicts their own tables.** Until 2026-09-23
`scripts/elm-coverage-check.mjs` printed two **unconditional** lines — *"and collapses where it was
not … It learned this corpus's archetype prior, not a general path→archetype mapping"* — written
when the v1 model genuinely had collapsed, and true of v1. They then printed for **every** model
regardless of its numbers. So `elm-coverage-v3-classtargeted.log` ends by announcing the exact
failure the run above it had just disproved.

**The arithmetic in those logs was always correct. Only the narration was wrong** — and the
narration is the part a reader skims. The logs are published **verbatim rather than edited after
the fact**; this note is the correction.

**The script is fixed** (n-dx `6d844bd3`, and the copy in `scripts/` here is the fixed one). The
closing block is now derived from the numbers: it reports the `service`/`utility` gap against the
teacher on both populations with direction and magnitude, prints the collapse verdict **only** when
the fresh-ecosystem prior over-predicts `service`/`utility` by ≥ 15 pp (the v1 signature),
otherwise states the prior is *not* collapsed and carries the necessary-not-sufficient caveat, and
**names opposite per-repo biases when they occur** — which is the failure mode the aggregate hides.
It was proven red before green: `node scripts/elm-coverage-check.mjs --selftest` runs the real
committed numbers through it with no model work, and the old lines fail its second assertion by
construction. **A rerun would print the right thing; the published logs predate the fix.**

**The refit fingerprint is unverified.** The frozen artifact carries `refitFingerprint` so a
certification run can prove it scored the frozen model; the script never checks it. Every coverage
figure in this repository, including 28.0% and 47.2%, is faithful-by-construction rather than
mechanically verified.

## Method

- Corpus verified before compute was spent: hash matched, all 160 previous held-out files still
  held out, 0 leaked into train, 0 rows from the evaluation repos, 0 gold-set paths.
- Freeze at the pinned operating point (B+su, ELM 4096 / tanh), determinism verified
  (`results/elm-freeze-v3-classtargeted.log`).
- GUARD instrument validated against the previous model **before** use.
- Provenance headers — repo commit, script and model hashes, corpus and row count, full invocation
  including the heap flag — written **before** each run, at the top of each log.
- `scripts/elm-coverage-check.mjs` unmodified at the certified commit. The 160-file GUARD used the
  existing directory mode rather than an edit.
- `--max-old-space-size=6144` throughout, and the artifact-exists test rather than the exit code
  (an out-of-memory run can exit 0 with an empty table).

Roles were separated: the corpus was built by one agent and frozen, measured and certified by
another, against a bar neither could change once the numbers existed. The same machinery returned a
FAIL on the preceding experiment (`results/elm-coverage-v3-primary.log`), which is published here
beside this pass.
