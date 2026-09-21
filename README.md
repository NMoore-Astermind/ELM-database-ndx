# ELM-database-ndx

A labelled training database for **file-archetype classification** — given a source file in a
repository, predict its architectural role (`service`, `utility`, `config`, `model`,
`component`, … 17 classes). Built so an **Extreme Learning Machine** can take over part of a job
that [n-dx](https://github.com/AsterMindAI/n-dx)'s `sourcevision` analyzer currently pays an LLM to
do.

> ## ⚠️ Current state — read before using anything here
>
> **This repository currently holds the methodology, not the data.** It archives how the database is
> built and evaluated, the bars each experiment was held to *before* its numbers existed, and every
> result — including the failures. **The labelled corpus lands here once it has been built and
> certified**, and not before.
>
> **What has been measured so far** (fresh ecosystems = repos the model never trained on; the bar,
> called K1′, is ≥ 30% coverage):
>
> | model | fresh-ecosystem coverage | K1′ |
> |---|---:|---|
> | path-only, corpus v2 (7 ecosystems) | **28.0%** | ❌ FAIL |
> | path + structural features (import graph, inventory) | **27.6%** | ❌ FAIL — no better than path-only |
> | class-targeted harvest | *in progress* | — |
>
> **No model trained on this data is known to generalise yet.** If you train on the corpus once it
> lands, read `methodology/FEATURES.md` first — it lists the failure modes, and none of them are
> visible from the rows alone.

## Six things that will mislead you if you skip them

1. **The labels come from an LLM, not from people.** Against human judgement the teacher is
   **72.3%** correct. Agreement with these labels is not accuracy.
2. **The teacher was not shown only the file path.** It also saw the rules' own guess and the full
   class catalogue, so its labels are not an independent judgement.
3. **The LLM-labelled set is the rules' leftovers, not the repository.** Files the rule pass
   classifies confidently never reach the teacher — which is why some classes are thin or absent.
4. **Coverage is not accuracy.** It counts how often the model answers something other than
   `service`/`utility`, right or wrong. Clearing the bar means the model's predictions widened,
   not that they are correct.
5. **`ELM.train()` in `@astermind/astermind-community` does not train on what you pass it.** It
   trains on the category *names*. Use `trainFromData(X, y)` with one-hot `y`.
   (`methodology/ELM-CORPUS.md` § 10a has the demonstration.)
6. **Do not train on the evaluation repos, `hono` and `trpc`** — including files from them that
   were never sampled. They are the only fresh ecosystems anyone has measured against.

## A naming note

"**v3**" is used for two different things in this history, and the difference matters:

- **`elm-v3-*`, `elm-coverage-v3-*`** — the *structural-features* experiment. It **failed**.
- **corpus v3 / class-targeted harvest** — the experiment in progress now, which adds training rows
  for the classes the model could never predict.

## Layout

| folder | contents |
|---|---|
| `methodology/` | the design decisions (ADRs), implementation plans (IMPLs), `FEATURES.md` (every feature, why it transfers, what is withheld), and `ELM-CORPUS.md` (the corpus's documentation and warnings) |
| `preregistration/` | the pass/fail bars, **each committed before its experiment ran** |
| `results/` | every evaluation artifact, with the exact command and input hashes that produced it |
| `scripts/` | the builder, feature derivation, evaluation harnesses and self-tests |

## Reproducing

The scripts are archived **as they run inside an n-dx checkout**, not as a standalone package: they
import n-dx's `sourcevision` and `llm-client` builds and `@astermind/astermind-community@^3.0.0`.
Run them from the root of [`AsterMindAI/n-dx`](https://github.com/AsterMindAI/n-dx) on the
`Nolan-Work` branch.

Coverage checks re-fit nine 4,096-unit models and **run out of memory at Node's default heap —
pass `--max-old-space-size=6144`**. An out-of-memory run can exit with code 0 and an empty results
table, so check that the table is populated.

## Provenance

Every file is copied verbatim from `AsterMindAI/n-dx` at commit `6dcfaf4e7d10` on `Nolan-Work`.
Relative links inside the archived documents point into that source tree. `PROVENANCE.md` lists a
SHA-256 for every file so a copy can be checked against its source.

Built by Team Nolan's agents for the AsterMind ELM migration.
