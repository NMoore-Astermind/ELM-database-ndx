# ELM-database-ndx

A labelled training database for **file-archetype classification** — given a source file in a
repository, predict its architectural role (`service`, `utility`, `config`, `model`,
`component`, … 17 classes). Built so an **Extreme Learning Machine** can take over part of a job
that [n-dx](https://github.com/AsterMindAI/n-dx)'s `sourcevision` analyzer currently pays an LLM to
do.

> ## ✅ Current state — certified 2026-09-22
>
> **A model trained on this data generalises to repositories it has never seen, and the bar it
> cleared was committed before the data existed.** All four pre-registered criteria pass. The data
> is in `data/`, the evaluation artifacts in `results/`, and the full result with its caveats is in
> [`results/CERTIFICATION.md`](results/CERTIFICATION.md).
>
> | model | fresh-ecosystem coverage | K1′ (≥30%) |
> |---|---:|---|
> | path-only, corpus v1 (2 ecosystems) | 13.2% | ❌ FAIL |
> | path-only, corpus v2 (7 ecosystems) | 28.0% | ❌ FAIL |
> | path + structural features (import graph, inventory) | 21.6% | ❌ FAIL — worse than path-only |
> | **corpus v3, class-targeted harvest** | **47.2%** | ✅ **PASS** |
>
> **Both failures are published here beside the pass.** The structural-features experiment was a
> real, expensive negative result: adding the import graph and file inventory did not help, and the
> diagnosis that followed — the model could not emit six classes because it had almost no training
> rows for them — is what this corpus was built to fix.
>
> ### ⚠️ Coverage is not accuracy
>
> Coverage counts how often the model answers something other than `service`/`utility`, **right or
> wrong**. Clearing the bar proves the model's predictions widened; it does **not** prove they are
> correct. No accuracy number exists for any model in this repository, and none is claimed. Read
> [`results/CERTIFICATION.md`](results/CERTIFICATION.md) § "What this does not license" before
> quoting 47.2% anywhere — the two evaluation repos now miss in *opposite* directions and the
> average hides both.

## Seven things that will mislead you if you skip them

1. **The labels come from an LLM, not from people.** Against human judgement the teacher is
   **72.3%** correct. Agreement with these labels is not accuracy.
2. **The teacher was not shown only the file path.** It also saw the rules' own guess and the full
   class catalogue, so its labels are not an independent judgement.
3. **The LLM-labelled set is the rules' leftovers, not the repository.** Files the rule pass
   classifies confidently never reach the teacher — which is why some classes are thin or absent,
   and why a second, rule-labelled dataset ships beside it.
4. **Coverage is not accuracy** — see the box above.
5. **`ELM.train()` in `@astermind/astermind-community` does not train on what you pass it.** It
   trains on the category *names*. Use `trainFromData(X, y)` with one-hot `y`.
   (`methodology/ELM-CORPUS.md` § 10a has the demonstration.)
6. **Do not train on the evaluation repos, `hono` and `trpc`** — including files from them that
   were never sampled. They are the only fresh ecosystems anyone has measured against, and the
   250-file blind set is still unlabelled and reusable.
7. **`text` is not a unique key.** `lib/request.js` exists in both express and fastify. De-duplicate
   on `repo` + `text`; de-duplicating on `text` alone can move a held-out file's twin into training
   and quietly contaminate the split.

## The data

| file | rows | labels come from | covers |
|---|---:|---|---|
| `data/elm-archetype-corpus-v3-classtargeted.json` | **2,195** | the LLM teacher | 16 classes, 10 repos — **the certified corpus** |
| `data/elm-archetype-corpus-resolved-v3-classtargeted.json` | **1,886** | `sourcevision`'s rules | **all 17 classes**, including `page`, `component` and `hook` |
| `data/elm-frozen-model-v3-classtargeted.json` | — | — | the certified model, as a recipe (see below) |

**These are two different populations with two different warranties. Do not pool them.** The first
is what the rules could *not* classify, labelled by an LLM. The second is what the rules *could*
classify — free, algorithmic, and the only artifact covering the classes the LLM residue
structurally cannot supply.

Each row is a repo-relative path string, a label, and optional measured columns joined from
`sourcevision`'s analysis (`role`, `language`, `loc`, `inDegree`, `outDegree`, `zone`). **Read
`provenance.rowColumns` for what a given artifact actually carries rather than assuming**, and read
`methodology/FEATURES.md` before deriving features — it documents every column, why it transfers
across repositories, and what is deliberately withheld.

**The split is seeded, stratified and carried forward.** Seed 42, 25% held out. Every row from the
previous corpus keeps its previous train/held-out assignment (`provenance.carriedSplit`), which is
what made the GUARD comparison possible. **Do not re-split** — your numbers stop being comparable
to anything published here.

**The frozen model is a recipe, not weights.** Scoring it re-fits nine 4,096-unit models, which
needs `--max-old-space-size=6144`. An out-of-memory run **exits 0 with an empty results table**, so
check that the table is populated.

## Using the corpus in a different feature space

**Every row pins its repository's commit and remote**, so the rows can be re-derived into any
feature space without re-labelling a single file — which is the property this dataset was built
for. All 2,195 rows were verified retrievable at their pinned commits, and **all ten commits still
resolve on their public remotes** (`results/elm-content-availability.log`), so this works from a
fresh clone rather than only on the machine that built it.

`scripts/elm-corpus-featurise.mjs` is the bridge into `sourcevision`'s content-based feature space
(651 dimensions: extension, path scalars, hashed path and content tokens, structural counts):

```
2,195 / 2,195 rows featurised   0 content missing   17 truncated at the 64 KB cap
15 models x 128 hidden trained in 4.6s   (results/elm-corpus-featurise.log)
```

Two things it does that anything reading this dataset should also do:

- **Bytes come from git at the pinned commit, not from a working tree**, so the output does not
  depend on what happens to be checked out.
- **The output pins `featureVersion`, `featureVectorSize`, `maxContentBytes`, the extractor's
  sha256 and the corpus sha256.** A feature-version number alone is not enough — a changed content
  cap, or a rebuilt extractor at the same version, invalidates the matrices with no error.

The derived matrices are **not** published here: they regenerate in about 30 seconds and are tied
to one extractor build, so a stale copy would be worse than none.

⚠️ **The labels are feature-space-independent. The corpus's *shape* is not.** Which classes were
harvested was decided from the starvation diagnosis of a path-only model, so a different feature
space may be weak at different classes. Re-featurising inherits that choice silently.

## Layout

| folder | contents |
|---|---|
| `data/` | the two labelled corpora and the certified frozen model |
| `results/` | [`CERTIFICATION.md`](results/CERTIFICATION.md) and every evaluation artifact — passes and failures — each with the exact invocation and input hashes that produced it |
| `preregistration/` | the pass/fail bars, **each committed before its experiment ran** |
| `methodology/` | the design decisions (ADRs), implementation plans (IMPLs), `FEATURES.md` (every feature, why it transfers, what is withheld) and `ELM-CORPUS.md` (the corpus's full documentation and warnings) |
| `scripts/` | the builder, feature derivation, evaluation harnesses and self-tests |

## How it was kept honest

Every bar in `preregistration/` was committed before the experiment that it judges. The model was
frozen before it was measured. The corpus was built by one agent and frozen, measured and certified
by a second, against a bar neither could move once the numbers existed. The measuring instrument
was itself validated first — it had to reproduce the previous model's number exactly before it was
trusted with a new one. The same machinery returned two FAILs before it returned this PASS, and all
three are published.

**One defect, found and fixed:** the closing narration in the published coverage logs is hardcoded
— written for the original failure and never made conditional, so it contradicts the passing result
printed directly above it. The logs are published **verbatim rather than edited after the fact**;
the script in `scripts/` is the fixed version, which derives that verdict from the numbers and is
covered by a self-test (`--selftest`). [`results/CERTIFICATION.md`](results/CERTIFICATION.md)
carries the full correction. **A remaining gap is recorded there too:** the frozen artifact carries
a refit fingerprint so a run can prove it scored the frozen model, and the script does not yet check
it — so every coverage figure here is faithful-by-construction rather than mechanically verified.

## Reproducing

The scripts are archived **as they run inside an n-dx checkout**, not as a standalone package: they
import n-dx's `sourcevision` and `llm-client` builds and `@astermind/astermind-community@^3.0.0`.
Run them from the root of [`AsterMindAI/n-dx`](https://github.com/AsterMindAI/n-dx) on the
`Nolan-Work` branch.

## A naming note

"**v3**" names two different things in this history, and the difference matters:

- **`elm-v3-*`, `elm-coverage-v3-primary/secondary`** — the *structural-features* experiment. It
  **failed**.
- **`*-v3-classtargeted`** — the class-targeted harvest. It is the one that passed, and it is the
  data in `data/`.

## Provenance

Every file is copied verbatim from `AsterMindAI/n-dx` on `Nolan-Work`. `PROVENANCE.md` lists a
SHA-256 for every file so a copy can be checked against its source. Artifacts are **not** edited
after the fact — including the absolute paths inside them, which record where each repository was
analyzed from and are not paths you can use.

Built by Team Nolan's agents for the AsterMind ELM migration.
