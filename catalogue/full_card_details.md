# Full Card Detail — Catalogue (41 cards)

Full description, example, fix, and sourcing for each catalogued card, in the order given in `catalogue_table.csv`. Extracted from the project's "Smell Catalogue" working tab.

---

## Card [R-01]
**Smell Name:** Data Leakage via Preprocessing Before Resampling (R)

**Language:** R

**Zhang et al. Mapping:** R11 (Data Leakage) — *Variant*

**Pipeline Stage:** Data Preparation

**Description:** Scaling, PCA, or oversampling is performed on the full dataset before resampling, or random (rather than grouped/spatial/time-aware) splits are used on data with underlying space, time, or group structure — allowing information from the held-out set to leak into training, inflating apparent performance.

**What Differs from the Python Version:** Zhang et al.'s R11 is framed around scikit-learn's `fit`/`transform` split and naive random train/test splits. The R manifestation is broader: beyond the basic preprocessing-before-split error, R's tidymodels/caret ecosystem surfaces a second, equally common failure mode — random splits applied to data with space, time, or group dependency (e.g., repeated subjects, spatial autocorrelation, time series) — which has no single clean Python analog in the original catalogue.

**Example:** Fitting a `recipe()` with `step_normalize()` or `step_pca()` directly on the full dataset, then splitting into train/test — rather than using `step_*()` inside a `workflow()`/`recipe()` that is `prep()`-ped only on the training fold. Similarly, using `initial_split()` (simple random split) on grouped, spatial, or longitudinal data instead of `group_initial_split()`, spatial blocking, or time-based splitting.

**Fix:** Place preprocessing steps inside a `recipe()`/`workflow()` so they are fit only on training folds (`prep()`/`bake()` or `fit_resamples()`), and use grouped, spatial, or time-aware split functions matching the data's dependency structure.

**Sources:**

- [caret issue #335](https://github.com/topepo/caret/issues/335) *(URL constructed from standard repo convention — not individually opened; verify before final use)*
- [themis issue #320](https://github.com/tidymodels/themis/issues/320) *(URL constructed from standard repo convention — not individually opened; verify before final use)*
- [recipes issue #857](https://github.com/tidymodels/recipes/issues/857) *(URL constructed from standard repo convention — not individually opened; verify before final use)*
- [bioLeak (GitHub)](https://github.com/selcukorkmaz/bioLeak) / [bioLeak (CRAN)](https://cran.r-project.org/web/packages/bioLeak/index.html) — confirmed
- BORG — [github.com/gcol33/BORG](https://github.com/gcol33/BORG) *(from earlier grey-literature pass; not re-verified in this search)*
- [leakr (GitHub)](https://github.com/cherylisabella/leakr) / [leakr (CRAN)](https://cran.r-project.org/web/packages/leakr/index.html) — confirmed
- [recipes issue #765](https://github.com/tidymodels/recipes/issues/765) — confirmed, strong: a feature request for `step_percentile()` built around a precise, reproducible side-by-side example — `step_mutate(Mileage = ecdf(Mileage)(Mileage))` (leaky: percentile computed against whatever data is present at `bake()` time) versus `step_mutate(Mileage = ecdf(car_train$Mileage)(Mileage))` (correct: percentile fixed to training data) — producing visibly different values on the same test rows. Julia Silge (tidymodels core) confirms this is a general risk worth documenting, extending the concern to `step_mutate_at()` as well
- [Tidy Modeling with R, Ch. 8 (tmwr.org)](https://www.tmwr.org/recipes) — confirmed, exceptionally strong: the official tidymodels textbook (Max Kuhn and Julia Silge) contains two explicit warnings — Section 8.3 states plainly that "all preprocessing and feature engineering steps use only the training data. Otherwise, information leakage can negatively impact the model's performance," illustrated via `step_normalize()`; Section 8.4.6 names the exact `step_mutate()` mechanism directly: "Consider, for example, the transformation `x = w > mean(w)`. When applied to new data or testing data, this transformation would use the mean of `w` from the new data, not the mean of `w` from the training data."

---

---

## Card [R-02]
**Smell Name:** Incomplete Reproducibility from Seed Setting Under Parallel Execution

**Language:** R

**Zhang et al. Mapping:** R2 (Randomness Uncontrolled) — *Variant*

**Pipeline Stage:** Model Training

**Description:** `set.seed()` called once at the top of a script does not guarantee reproducibility once parallel workers, cross-validation resampling, or an underlying engine's own random steps are involved — each worker process or internal routine can draw from its own, uncontrolled random stream, so results silently change between runs or between sequential and parallel execution of the same code.

**What Differs from the Python Version:** Zhang et al.'s R2 centers on single-process randomness (not setting seeds for NumPy, the ML library, and Python's own `random` module together). The R manifestation is structurally harder: R's parallel backends (`parallel`, `doParallel`, `future`) spawn separate worker processes that do not inherit the main process's RNG state by default, so the failure mode is specifically about process-level seed propagation, not just which libraries get seeded.

**Example:** Calling `set.seed(123)` before `train()` (caret) or `tune_grid()` (tidymodels `tune`) with `trControl`/control set to run in parallel — results differ from a sequential run, and differ again between separate parallel runs, because the global seed was never propagated to the spawned workers.

**Fix:** In caret, pass an explicit list of per-resample seeds via `trainControl(seeds = ...)`. In modern `tune` (≥0.1.2, and substantially reworked in a July 2025 patch), the package itself generates and assigns L'Ecuyer-CMRG seeds matched to each resample for both sequential and parallel execution, removing the need for manual seed lists — but this is a breaking change between `tune` versions, so reproducibility across different `tune` versions on the same code is not guaranteed.

**Sources:**

- [tune issue #11](https://github.com/tidymodels/tune/issues/11) — confirmed: "more explicit seed setting for tune_grid" (2019)
- [tune PR #275](https://github.com/tidymodels/tune/pull/275) — confirmed: "Explicit seed setting in worker processes," explicitly notes it mirrors caret's approach
- [tune Changelog — tune 2.0.0](https://tune.tidymodels.org/news/index.html) — confirmed breaking change: parallel seed construction changed, breaking cross-version reproducibility
- [tune PR #1033](https://github.com/tidymodels/tune/pull/1033) — confirmed, July 2025: further rework generating L'Ecuyer-CMRG seeds matched to resamples — shows the problem was still being actively re-solved years after the original fix
- [caret trainControl() docs](https://topepo.github.io/caret/model-training-and-tuning.html) — documents the `seeds =` argument as the manual mitigation
- [caret issue #1036](https://github.com/topepo/caret/issues/1036) — confirmed: user tried three separate seed-setting approaches before escalating
- mlr issue #2862 — **link not confirmed**; please supply
- Bonus confirmations from the mlr ecosystem (not in original list): [mlr-org/parallelMap #31](https://github.com/mlr-org/parallelMap/issues/31) ("set.seed() doesn't work with parallelMap"), [mlr-org/mlr3 #459](https://github.com/mlr-org/mlr3/issues/459) ("Parallel reproducibility is not given," full reprex)
- [Posit Community: "set.seed() in context of tidymodels"](https://forum.posit.co/t/set-seed-in-context-of-tidymodels/131877) — confirmed, direct match: user confused about where to call `set.seed()` across a workflow; community response confirms the exact pattern — "you would set the seed before `initial_split()`... You might also use it before calling one of the `tune_*()` functions... If you are never going to change your script, you can set the seed at the top. This is a pretty bad assumption, so we often set it multiple times in a script"
- [Posit Community: "When and when not to seed?"](https://forum.posit.co/t/when-and-when-not-to-seed/110472) — confirmed, direct match: same core confusion, with example code showing separate `set.seed()` calls needed before `vfold_cv()` and before `tune_grid()` as two distinct randomness points a single top-of-script seed wouldn't cover
- [tune issue #377](https://github.com/tidymodels/tune/issues/377) ("Unreliable random numbers produced when using doFuture backend") — confirmed, but more nuanced than the title suggests: the original warning is explicitly confirmed a **false positive** by two tidymodels maintainers (Max Kuhn, Julia Silge) — `tune` already generates parallel-safe L'Ecuyer-CMRG seeds internally (PR #349), but `doFuture` hardcodes its own `seed` argument to `FALSE`, so it cannot detect that `tune` is already handling RNG safely and raises an alarm regardless (root cause diagnosed precisely by Davis Vaughan). However, a second user's follow-up reveals a **genuinely real, distinct bug**: applying the natural-seeming fix for this warning (`registerDoRNG()`) in combination with `tune`'s nested `foreach` loops causes an actual downstream error ("nested/conditional foreach loops are not supported yet") when later calling `last_fit()`. The real smell here is subtler than "parallel backends break reproducibility": it's that a false-alarm warning from a backend blind to `tune`'s internal safety mechanism leads users toward a recommended workaround that itself breaks a different part of the pipeline

---

---

## Card [R-03]
**Smell Name:** Many-to-Many Join Ambiguity

**Language:** R

**Zhang et al. Mapping:** R15 (Merge API Parameter Not Explicitly Set) — *Near one-to-one analog*

**Pipeline Stage:** Data Preparation

**Description:** A join (`left_join()`, `inner_join()`, etc.) performed without specifying the expected key relationship can silently produce row duplication when keys match multiple rows on both sides — a many-to-many relationship causing a Cartesian-style explosion in row count, inflating dataset size and corrupting downstream aggregate statistics without any error or obvious symptom.

**What Differs from the Python Version:** Zhang et al.'s R15 centers on pandas' `merge()` lacking an explicit `how`/`on`/`validate` parameter. R's `dplyr` has an almost one-to-one analog: the `relationship` parameter (added dplyr 1.1.0, narrowed in 1.1.1) plays the same role pandas' `validate` parameter does. Notably, dplyr's own maintainers narrowed their warning specifically to many-to-many (not one-to-many) relationships after community discussion, since one-to-many is common and usually intentional while many-to-many is rare and dangerous — a more refined, usage-calibrated version of the same underlying concern.

**Example:** `left_join(orders, customers, by = "customer_id")` where `customer_id` is not actually unique in `customers` — the join silently multiplies matching rows rather than erroring, producing an inflated row count with no indication anything went wrong.

**Fix:** Pass an explicit `relationship` argument (`"one-to-one"`, `"one-to-many"`, `"many-to-one"`, or `"many-to-many"`) to the join call; dplyr ≥ 1.1.0 will error if the actual data violates the declared relationship, catching the issue immediately rather than silently inflating results. In `data.table`, the equivalent guard is `allow.cartesian = FALSE` (the safer non-default).

**Sources:**

- [dplyr issue #6731](https://github.com/tidyverse/dplyr/issues/6731) — confirmed: "On `multiple` matches, and how we can improve," explicitly cross-references data.table #4383 as supporting evidence
- dplyr issue #6717 — confirmed referenced within #6731 (not independently opened)
- [dplyr 1.1.1 release notes](https://github.com/tidyverse/dplyr/releases) — confirmed: documents the `relationship` argument and the narrowing of the many-to-many warning, citing #6731 and #6717 directly
- [data.table issue #4383](https://github.com/Rdatatable/data.table/issues/4383) — confirmed as a real, cross-referenced issue on `allow.cartesian` (not independently opened for full content)
**Corroboration Note:** The strongest signal here is that dplyr's and data.table's maintainers cross-reference each other's issue trackers on this exact problem — independent teams converging on the same diagnosis and a similar fix (`relationship` vs. `allow.cartesian`) is stronger evidence than either ecosystem's issues alone.

---

---

## Card [R-04]
**Smell Name:** In-Place Reference Semantics Misused (data.table)

**Language:** R

**Zhang et al. Mapping:** R16 (In-Place APIs Misused) — *Variant, opposite direction from the Python version*

**Pipeline Stage:** Data Preparation

**Description:** Unlike base R's normal copy-on-modify semantics, `data.table` modifies objects **by reference**. A copy made with ordinary assignment (`<-`), or a saved vector of column names, is not actually independent — subsequently modifying one object (e.g., via `:=`) silently changes the other, including the caller's original data if the data.table was passed into a function.

**What Differs from the Python Version:** This is the mirror image of Zhang et al.'s R16. In pandas, the risk is ambiguous `inplace=` defaults causing uncertainty about whether a copy or in-place modification occurred. In `data.table`, the risk runs the opposite direction: standard R semantics (copy-on-modify) are what users expect, but `data.table` deliberately breaks that convention for performance, so code that *looks* like a normal, safe R assignment is not actually safe. The confusion is caused by `data.table` being "too consistent" with reference semantics while looking syntactically like ordinary R.

**Example:** `DT2 <- DT; DT2[, a := 0]` also modifies `DT`, because `DT2` is not an independent copy — only a second reference to the same underlying object. Similarly, `dt_names <- names(DT)` followed by adding a column to `DT` silently updates `dt_names` as well.

**Fix:** Use `copy()` explicitly wherever an independent object is needed: `DT2 <- copy(DT)`. For column name vectors specifically, `dt_names <- copy(names(DT))`.

**Sources:**

- [Stack Overflow #15913417](https://stackoverflow.com/questions/15913417) — confirmed, cross-referenced directly in data.table issue #512
- [Stack Overflow #18662715](https://stackoverflow.com/questions/18662715) — confirmed, cross-referenced directly in data.table issue #512
- [data.table issue #512](https://github.com/Rdatatable/data.table/issues/512) — confirmed: filed by Matt Dowle (package creator) specifically to track that `copy(names(DT))` needs better documentation, citing both SO threads above
- [dplyr issue #614](https://github.com/tidyverse/dplyr/issues/614) — confirmed, strong, opposite-direction complement: dplyr's own documentation claims "`mutate()` never copies columns, except when you modify an existing column" — but this breaks specifically when the underlying object is a `data.table` rather than a `data.frame`, triggering an unexpected *deep* copy. Precise reproducible memory measurements: 1.2GB → 2GB for a `data.frame` (expected, shallow-copy behavior) vs. 1.2GB → 3.2GB for the identical operation on a `data.table` (unexpected, nearly double). Closed with direct engagement from both dplyr creator Hadley Wickham and data.table co-maintainer Arun Srinivasan. Cross-referenced by a related issue, [dplyr #1431](https://github.com/tidyverse/dplyr/issues/1431) ("Mutate returns nothing from data tables"), confirming this dplyr/data.table copy-semantics friction recurred and was eventually migrated to dtplyr's own issue tracker
- [data.table official vignette: "Reference Semantics"](https://cran.r-project.org/web/packages/data.table/vignettes/datatable-reference-semantics.html) — confirmed: official package documentation devoted entirely to this exact behavior, including the `DT2 <- DT; DT2[, a := 0]` pattern and the `copy()` fix
- [data.table ](https://rdocumentation.org/packages/data.table/topics/copy)[`?copy`](https://rdocumentation.org/packages/data.table/topics/copy)[ documentation](https://rdocumentation.org/packages/data.table/topics/copy) — confirmed: official docs explicitly describe the `names(DT)` reference-sharing trap
- [data.table issue #1086](https://github.com/Rdatatable/data.table/issues/1086) — confirmed relevant: feature request to make a data.table lockable/read-only specifically to prevent accidental reference modification (via `lockBinding()` or a read-only attribute) — direct, on-topic evidence
- [data.table issue #5376](https://github.com/Rdatatable/data.table/issues/5376) — confirmed relevant: "Error on modifying by reference with `data.table::set()` in the context of `future.apply::future_apply()` or `furrr::future_map()`" — extends this smell into parallel/future execution contexts

---

---

## Card [R-05]
**Smell Name:** Fitted Model Objects Retain Training Data and Environments by Default

**Language:** R

**Zhang et al. Mapping:** R10 (Memory Not Freed) — *Variant*

**Pipeline Stage:** Model Training / Deployment

**Description:** Fitted model objects in common R ML packages (caret, base `lm()`/`glm()`, and others) retain the full training data and the environment in which they were created by default, often without the user realizing it. This bloats model objects far beyond what prediction requires, and `rm()` plus `gc()` frequently fail to reclaim this memory in long loops or repeated training runs, since the retained environment can keep references alive.

**What Differs from the Python Version:** Zhang et al.'s R10 centers on explicit resource handles (e.g., TensorFlow sessions, GPU memory) not being released. The R manifestation is structurally different and arguably more insidious: it is not about forgetting to call a cleanup function, but about **default constructor behavior** — base R's lexical scoping means a fitted model silently carries its entire creation environment (and often the full training dataset) unless the user knows to explicitly disable this via function-specific flags.

**Example:** `caret::train()` called without `trControl = trainControl(returnData = FALSE)` retains the full training dataset inside the fitted object. Similarly, base `lm()` carries its creation environment, so `object.size()` on a fitted linear model can be many times larger than the input data alone. In a loop fitting hundreds of models (e.g., cross-validated grid search), this compounds into severe memory pressure that `gc()` does not fully resolve.

**Fix:** Set `trim = TRUE` and `returnData = FALSE` in caret; `store_models = FALSE` in mlr3 benchmarking; use the dedicated `butcher` package (`butcher::butcher()`) to strip unneeded components from any supported fitted model post-hoc; or use the `trimmer` package's `trim()` function. For persistent issues across long loops, restarting the R session or worker process between runs is a common last-resort mitigation.

**Sources:**

- [butcher (CRAN)](https://cran.r-project.org/web/packages/butcher/index.html) / [butcher (GitHub)](https://github.com/tidymodels/butcher) — confirmed: official tidymodels-team package, co-authored by Max Kuhn (also the author of caret itself), supporting 20+ model classes; README explicitly states ML model objects "take up too much memory" due to "lack of selectivity in the construction of the model object itself"
- [caret issue #90](https://github.com/topepo/caret/issues/90) — confirmed: the origin feature request for a 'trim' method on fitted caret models, explicitly citing prior discussion in caret #76 and cross-package workarounds attempted in caretEnsemble#69
- [caret issue #263](https://github.com/topepo/caret/issues/263) — confirmed: real-world report, gbm model on a modest dataset (15k obs, 31 predictors) exhausted 16GB RAM during repeated 10x10-fold CV, and memory was not released after training completed.
- [mlr3batchmark issue #18](https://github.com/mlr-org/mlr3batchmark/issues/18) — confirmed, and notably sharper than the base smell: user explicitly set both `store_models = FALSE` and `store_backends = FALSE` (the documented mitigation flags) and RAM usage was still high — source code showed the task object was stored regardless, with `store_backends` not actually honored in that code path. Shows the documented escape hatch itself can silently fail, not just that retention is the default
- [Stack Overflow #50243219](https://stackoverflow.com/questions/50243219/functionality-of-caret-train-trim-parameter) — confirmed: "Functionality of caret train trim parameter" — real user confusion about what the `trim` parameter actually does, reinforcing that this is a live discoverability issue, not just a resolved one

---

---

## Card [R-06]
**Smell Name:** Column Data Types Inferred from a Sample, Not Set Explicitly

**Language:** R

**Zhang et al. Mapping:** R21 (Columns/DataType Not Explicitly Set) — *Direct analog*

**Pipeline Stage:** Data Preparation (Loading)

**Description:** `readr`'s `read_csv()` and related functions guess each column's data type by examining only a sample of rows (`guess_max`, default 1000) rather than the full file. A value that doesn't fit the guessed type — because it only appears after the sampled rows — is silently coerced to `NA` rather than raising an error, corrupting the loaded dataset before any ML-specific processing begins.

**What Differs from the Python Version:** Zhang et al.'s R21 concerns pandas' `read_csv()` similarly inferring dtypes from a sample. The R manifestation is historically sharper: prior to readr 2.0, the first-edition parser's guessing was documented as substantially worse, and the smell is partly historical — real but declining in severity as the tooling improves, rather than a constant, unaddressed risk.

**Example:** A CSV column containing 1000+ rows of literal `"NA"` followed by real character values past row 1000 gets typed as `logical` (R's guess for an all-NA column) rather than `character` — all subsequent real values in that column silently become `NA` on read.

**Fix:** Pass an explicit `col_types =` specification to `read_csv()`/`read_delim()`, or at minimum increase `guess_max` (up to `guess_max = Inf` to scan the whole file, at a performance cost).

**Sources:**

- [readr issue #839](https://github.com/tidyverse/readr/issues/839) — confirmed: "read_csv column type guess for 1000 rows of 'NA' is logical - should be character"
- [readr column types vignette](https://readr.tidyverse.org/articles/column-types.html) — confirmed: official documentation of the guessing mechanism, including the readr 2.0 improvement and the `col_types =`/`guess_max` fixes

---

---

## Card [R-07]
**Smell Name:** Hyperparameter Silently Ignored via Wrapper Pass-Through

**Language:** R

**Zhang et al. Mapping:** R5 (Hyperparameter Not Explicitly Set) — *Analogous*

**Pipeline Stage:** Model Training

**Description:** Hyperparameters passed through ML wrapper packages (notably `caret::train()`) can be silently ignored when the argument name doesn't match what the underlying engine actually expects — `caret`'s `...` pass-through does not validate or translate argument names, so a typo or a wrong-engine's naming convention fails silently rather than erroring.

**Example:** `train(..., method = "ranger", ntree = 1000)` changes nothing, because `ranger::ranger()` expects `num.trees`, not `ntree` (`ntree` is randomForest's name for the same parameter). The call does not raise an error; the model simply trains with ranger's default tree count instead.

**Fix:** Use the underlying engine's exact argument name, or use `parsnip`, which solves this by design via an explicit, documented argument-translation layer (e.g., `trees` → `num.trees` for ranger, `ntree` for randomForest, `num_trees` for Spark — confirmed via parsnip's own vignette and a community-compiled translation table). Always inspect the fitted model object afterward to confirm the intended hyperparameter value actually took effect.

**Sources:**

- [parsnip issue #774](https://github.com/tidymodels/parsnip/issues/774) — confirmed, distinct mechanism: passing xgboost's native `params = list(tree_method=..., eval_metric=..., objective=...)` via `set_engine("xgboost", params = ...)` causes `tune_bayes()`'s tuned hyperparameters (`tree_depth`, `min_n`, `loss_reduction`, `sample_size`, `mtry`, `learn_rate`) to silently stop varying model behavior — evidenced by "All of the mn_log_loss values were identical" across the grid, which in turn crashes the Gaussian process optimizer (no variance to model). A different pathway to the same smell: tuned hyperparameters not actually taking effect, this time via an engine-native params list overriding parsnip's tuning layer rather than a simple name mismatch
- [ranger documentation (via parsnip engine details)](https://www.rdocumentation.org/packages/parsnip/versions/0.1.6/topics/rand_forest) — confirmed: documents `num.trees` as ranger's argument name
- [parsnip ranger/xgboost/randomForest docs](https://parsnip.tidymodels.org/articles/parsnip_Intro.html) — confirmed: official parsnip documentation and translation table
- [flexsdm issue #474](https://github.com/sjevelazco/flexsdm/issues/474) — confirmed, strong, direct match: "fit_raf/tune_raf/esm_gbm/esm_net silently ignore user hyperparameters" — runtime-verified example showing `fit_raf(..., ntree = 11)` silently used the internal default of 500 instead (`m$model$ntree` returns `500 (fixed: 11)`), with fixes included in the issue
- [tune PR #549](https://github.com/tidymodels/tune/pull/549) — confirmed, strong, maintainer-built safeguard: closes issue #548 ("No error/warning for mismatch between model arguments and `set_engine()`"), adding an explicit error for the case where a parameter marked with `tune()` silently fails to be tuned because the chosen engine doesn't support tuning it — previously completely silent, e.g. `linear_reg(engine = "lm", penalty = tune())` (`lm` doesn't support `penalty`). Also preserves the pre-existing, separate warning for the related but distinct case of calling `tune_grid()` with no parameters marked `tune()` at all: "No tuning parameters have been detected; performance will be evaluated using the resamples with no tuning." The PR author notes this fix should also surface the same silent-non-tuning bug pattern independently confirmed in two other tidymodels extension packages, `tidymodels/rules` (#49) and `tidymodels/bonsai` (#30)

---

---

## Card [R-08]
**Smell Name:** NA/NaN Equivalence Comparison Misused

**Language:** R

**Zhang et al. Mapping:** R18 (NaN Equivalence Comparison Misused) — *Direct analog, structurally richer*

**Pipeline Stage:** Data Preparation

**Description:** `x == NA` always returns `NA` (not `TRUE`/`FALSE`, and not even consistently — every element of the comparison becomes `NA`), under R's three-valued logic for missing data. Consequently, `if (x != NA)` errors outright, and using `x == NA` as a logical mask for subsetting silently produces an all-`NA` selection rather than the intended filter.

**What Differs from the Python Version:** Zhang et al.'s R18 concerns `np.nan == np.nan` returning `False` — a single, consistent misbehavior. R's version is structurally richer and arguably more hazardous: R maintains `NA` (missing/unknown), `NaN` (undefined, e.g. 0/0), and `NULL` (absent) as three distinct concepts, each with different comparison and propagation behavior (`is.na(NaN)` is `TRUE`, but `is.nan(NA)` is `FALSE`), compared to Python's single conflated `NaN`.

**Example:** `x <- c(1, NA, 3); x == NA` returns `c(NA, NA, NA)` for every element, not just the missing one. Using this directly in `if ()` or as a subsetting mask produces either an error or a silently empty/wrong selection.

**Fix:** Use `is.na(x)` to test for missingness; never compare directly to `NA` with `==` or `!=`.

**Sources:**

- [R base documentation (](https://web.mit.edu/r/current/lib/R/library/base/html/NA.html)[`?NA`](https://web.mit.edu/r/current/lib/R/library/base/html/NA.html)[)](https://web.mit.edu/r/current/lib/R/library/base/html/NA.html) — confirmed: official documentation directly describes the three-valued comparison logic and the `is.na()` requirement
- [easyr package, ](https://github.com/oliver-wyman-actuarial/easyr)[`eq()`](https://github.com/oliver-wyman-actuarial/easyr)[ function](https://github.com/oliver-wyman-actuarial/easyr) — confirmed, strong: a CRAN-published package's dedicated NA-safe equality function, documented precisely: "R users will understand that NAs often break equality checks... Returns TRUE if both values are NA, and FALSE when only one is NA. The standard == comparison returns NA in both of these cases and sometimes this is interpreted unexpectedly"
- [common package, ](https://packages.oit.ncsu.edu/cran/web/packages/common/refman/common.html)[`%eq%`](https://packages.oit.ncsu.edu/cran/web/packages/common/refman/common.html)[ operator](https://packages.oit.ncsu.edu/cran/web/packages/common/refman/common.html) — confirmed, independent second CRAN package solving the same problem: "provides a simple, reliable equality check that allows comparing of NULLs, NA values, and atomic data types without error," with `NA %eq% NA` returning `TRUE` and `NULL %eq% NULL` returning `TRUE` — two independent dedicated tools built for the same underlying confusion

---

---

## Card [R-09]
**Smell Name:** NA Keys Silently Match Each Other in Joins

**Language:** R

**Zhang et al. Mapping:** R15 (Merge API Parameter Not Explicitly Set) — *Related to R-03, distinct mechanism*

**Pipeline Stage:** Data Preparation

**Description:** `dplyr`'s join functions (and base R's `merge()`) treat two `NA`/`NaN` values in the join key as **matching each other** by default — directly contradicting the normal rule that `NA == NA` is unknowable, not `TRUE` (see Card R-08). A row with a missing key on one side can silently join to an unrelated row with a missing key on the other side, producing a nonsensical but structurally valid-looking result.

**What Differs from the Python Version:** This is a sharper, more specific case within R15's territory than R-03. Where R-03 is about *duplicate* keys causing row multiplication, this is about *missing* keys being treated as mutually matching — a behavior R inherited into `dplyr` from base R's `merge()`, which many users don't expect given how strictly `NA` is treated as "unknown" everywhere else in R (see R-08's contrasting behavior for `==`).

**Example:** A `songs` table with one row having `movie = NA`, and a `singers` table with one row having `movie = NA`, joined via `inner_join(songs, singers, by = "movie")` — the two unrelated NA-keyed rows are joined together as if they matched, attaching an entirely wrong singer to a song.

**Fix:** Pass `na_matches = "never"` to any `dplyr` join function to treat NA keys as never matching (mirrors `base::merge(incomparables = NA)` and `match(..., incomparables = NA)`). Validate for NA keys explicitly before joining if silent drops/mismatches would be unacceptable.

**Sources:**

- [dplyr issue #2033](https://github.com/tidyverse/dplyr/issues/2033) — confirmed via dplyr's own [NEWS.md/changelog:](http://NEWS.md/changelog:) directly tied to the history of the `na_matches = "na"` default (dplyr 0.7.0+)
- [dplyr issue #3096](https://github.com/tidyverse/dplyr/issues/3096) — confirmed: a second user surprised by the default, requesting it be better documented
- [Posit Community: "Why does NA match NA when joining two dataframes?"](https://forum.posit.co/t/why-does-na-match-na-when-joining-two-dataframes/28785) — confirmed: explains the mechanism, and that base R's `merge()`/`match()` share the same default, with `incomparables =`/`na_matches = "never"` as the fixes
- [horizons issue #139](https://github.com/S-Leuthold/horizons/issues/139) — confirmed, recent (13 days old at time of search): a real production bug where NA-keyed rows silently joined to each other and passed validation checks, because the validation only checked a different column
- [dplyr official join documentation](https://dplyr.tidyverse.org/reference/mutate-joins.html) — confirmed: documents `na_matches` parameter and its default behavior directly

---

---

## Card [R-10]
**Smell Name:** Default Classification Metric Ignores Class Probabilities

**Language:** R

**Zhang et al. Mapping:** R19 (Threshold-Dependent Validation) — *Direct analog*

**Pipeline Stage:** Model Evaluation

**Description:** `caret::train()` tunes and selects classification models using plain Accuracy by default (`defaultSummary`), which does not account for class probabilities, threshold choice, or class imbalance. A more appropriate metric (ROC/AUC via `twoClassSummary`) must be explicitly opted into — and requires an additional, easy-to-forget flag (`classProbs = TRUE`) to even be computable.

**Example:** `train(Class ~ ., data, method = "glm")` silently tunes and selects the "best" model using Accuracy, without regard to probability calibration or class imbalance, unless the user explicitly supplies `trControl = trainControl(summaryFunction = twoClassSummary, classProbs = TRUE)` and sets `metric = "ROC"`.

**Fix:** Set `metric = "ROC"` together with `trainControl(classProbs = TRUE, summaryFunction = twoClassSummary)`. For probability-cutoff selection specifically, caret's `thresholder()` function (added in response to issue #224) analyzes resample results to choose an appropriate probability threshold.

**Evidence Tier:** Strong — upgraded again. CAST #57 shows this smell causing a real, consequential failure (feature selection silently discarding valid predictors at the default threshold), not just a confusing number; combined with caret's own changelog response (#224) and official documentation, this is now one of the better-evidenced cards in the catalogue despite starting from a self-assessed Weak tier.

**Sources:**

- [caret issue #224](https://github.com/topepo/caret/blob/master/RegressionTests/Code/cforest.R) — confirmed via [caret's official changelog](https://cran.r-project.org/web/packages/caret/news.html): directly led to the addition of the `thresholder()` function
- [caret issue #897](https://github.com/topepo/caret/issues/897) — confirmed, adjacent but distinct: user confused why resample-averaged custom-metric results (`model$results`) are more optimistic than a pooled `confusionMatrix()` on the same model's held-out predictions — real confusion in caret's evaluation-metric system, but not a direct confirmation of the "Accuracy is the default" claim specifically; kept as supporting evidence of general metric-interpretation difficulty in caret
- [caret train / summary function documentation](https://rdocumentation.org/packages/caret/versions/6.0-76/topics/postResample) — confirmed: official docs describe `defaultSummary` as the default and `twoClassSummary`'s `classProbs = TRUE` requirement directly
- nestedcv docs — **not independently verified in this pass**; please supply link
- [CAST issue #57](https://github.com/HannaMeyer/CAST/issues/57) — confirmed, strong, directly on-topic: user found caret's fixed 0.5 classification threshold produced Kappa = 0.08, while the actual optimal threshold (0.2) gave Kappa ≈ 0.24 — and this fed directly into `ffs()` (forward feature selection), which discarded real predictor combinations because they scored near-zero Kappa at the wrong threshold. A case where the smell corrupted a downstream feature-selection decision, not just a reported number. The reply thread explicitly cross-references caret #224 as "the only answer," tying this card's two strongest sources together directly
- treesnip issue #58 — dropped per team decision (repository is archived)
- (Bonus, not in original list) [public caret tutorial](https://nader-hotait.github.io/machine-learning/) — confirmed: reproduces the exact warning ("The metric 'Accuracy' was not in the result set. ROC will be used instead") that this default produces in practice
**Corroboration Note:** The core mechanism is confirmed at the highest level available — the package's own changelog and official documentation — which is stronger grounding than the self-assessed Weak tier suggested. The four unconfirmed downstream sources would add breadth but aren't needed to establish the core claim is real.

**Classification:** **Include** — solid catalogue entry once relabeled from Weak to Moderate-Strong.

**Open Items:** Confirm caret #897, nestedcv docs, CAST #57, and treesnip #58.

---

---

## Card [R-11]
**Smell Name:** No Scaling Before Distance-Based/PCA Algorithms

**Language:** R

**Zhang et al. Mapping:** R22 (No Scaling) — *Direct analog*

**Pipeline Stage:** Data Preparation / Model Training

Two related manifestations. (1) `prcomp()` and related PCA/SVD implementations in R default to `scale. = FALSE`, so features are centered but not scaled to unit variance unless explicitly requested, letting high-variance features dominate the resulting components. (2) A more consequential and widely-recurring variant: PCA-adjacent R packages have repeatedly lacked a correct `predict()`-style function to project new data onto an existing fitted PCA space using that model's own training-derived center/scale — instead either re-scaling new data using its own mean/SD (producing wrong, sometimes inverted results) or lacking the capability entirely.

**Example:** `prcomp(data_matrix)` runs with centering only, no scaling, by default — features with naturally larger numeric ranges (e.g., a count in the thousands vs. a proportion between 0 and 1) dominate the first principal components regardless of their actual explanatory importance.

**Fix:** Explicitly pass `scale. = TRUE` to `prcomp()` (or the equivalent parameter in related packages) whenever input features are not already on comparable scales.

**Sources:**

- [R ](https://rdrr.io/cran/irlba/src/R/prcomp.R)[`prcomp()`](https://rdrr.io/cran/irlba/src/R/prcomp.R)[ official documentation](https://rdrr.io/cran/irlba/src/R/prcomp.R) — confirmed: states `scale. = FALSE` is default "for consistency with S, but scaling is often advisable"
- [irlba issue #25](https://github.com/bwlewis/irlba/issues/25) — confirmed: real, maintainer-acknowledged bug in `prcomp_irlba`'s scaling logic, fixed same week
- [irlba issue #56](https://github.com/bwlewis/irlba/issues/56) — confirmed: inconsistent scaling-value storage between `stats::prcomp` and `irlba::prcomp_irlba`
- [irlba issue #21](https://github.com/bwlewis/irlba/issues/21) — confirmed: scale-related arguments producing numerically poor results relative to base `prcomp`
- [PCAtools source](https://github.com/kevinblighe/PCAtools/blob/master/R/pca.R) — confirmed: its own `scale` parameter also defaults to `FALSE`, same convention inherited into a modern, actively-maintained Bioconductor package
- [PCAtools issue #16](https://github.com/kevinblighe/PCAtools/issues/16) — confirmed, strong: user explicitly ties this to ML cross-validation — "In cross-validation, I can't do PCA to the training and testing sets separately... I have to apply PCA to the training set... and use the space to get the components for the test set in the same space to validate the model." Maintainer implemented a fix in response

[nipals issue #1](https://github.com/kwstat/nipals/issues/1) — confirmed: same gap (no `predict()` reusing training center/scale for new data), maintainer shipped a semi-fix
- [dimRed issue #1](https://github.com/gdkrmr/dimRed/issues/1) — confirmed: same gap again, maintainer implemented `predict`/`inverse` functions. Notably, Max Kuhn (caret's own author) is an active participant in this thread, confirming ecosystem-wide awareness of the pattern
- [rSVD issue #9](https://github.com/erichson/rSVD/issues/9) — confirmed, weaker: a feature request showing user awareness of the correct train-then-project pattern, not itself a bug confirmation — supporting context rather than direct evidence
- [r-devel mailing list, Jari Oksanen, May 2012](https://stat.ethz.ch/pipermail/r-devel/2012-May/064165.html) — confirmed, strong, distinct from the other sources: concerns base R's own `prcomp()`, not a third-party package. `prcomp(scale(x))` followed by `predict(sol, newdata = x)` disagrees with `predict(sol)` on the identical data — `all.equal()` reports a 90% mean relative difference. Root cause precisely diagnosed: `scale()`'s `"scaled:scale"` attribute persists through a second `scale()` call even with `scale = FALSE`, so `prcomp()` silently picks up the wrong (stale) scale attribute internally. Oksanen demonstrates `princomp()` (R's other base PCA function) does not have this bug, since it computes scale directly rather than trusting potentially-stale attributes — confirming this is a specific `prcomp()` implementation defect, not a general R quirk. Note: whether this specific bug is still present in current R versions is unverified — not independently reproduced for this catalogue
- [flexsdm issue #334](https://github.com/sjevelazco/flexsdm/issues/334) — confirmed, strongest source on this card: maintainer-confirmed, maintainer-fixed real bug where projecting a species-distribution PCA onto future climate data re-scaled the new data using its own mean/SD instead of the original training data's, causing predictions to be inverted (a warmer future scenario produced higher, not lower, scores). Visually demonstrated with maps; fixed in the same thread
- [practical bioinformatics explainer](https://divingintogeneticsandgenomics.com/post/understanding-prcomp-center-scale-arguments/) — confirmed: explicitly frames unscaled, uncentered PCA as "the worst case," with worked single-cell RNA-seq examples

---

---

## Card [R-12]
**Smell Name:** `as.matrix()` Silently Coerces Mixed-Type Data to Character

**Language:** R

**Zhang et al. Mapping:** R14 (DataFrame Conversion API Misused) — *Direct analog*

**Pipeline Stage:** Data Preparation

**Description:** Calling `as.matrix()` on a data frame containing any text or factor column coerces every column — including purely numeric ones — to character, because R matrices require a single, consistent atomic type. This produces a matrix of character strings, not numbers, with no warning, which silently breaks any downstream numeric computation.

**What Differs from the Python Version:** Zhang et al.'s R14 concerns pandas' `.values`/`.to_numpy()` ambiguity (copy-vs-view uncertainty). R's manifestation is a different, arguably more severe failure mode: not an ambiguity about data sharing, but a silent, whole-matrix type coercion that corrupts every column, not just the non-numeric one(s) responsible for it.

**Example:** `as.matrix(data.frame(value = 1:5, time = as.factor(c("before", "after", ...))))` returns a matrix where the `value` column — originally integers — is now the character strings `"1"`, `"2"`, etc. A related, subtler instance: `apply(x, 2, is.numeric)` on a data frame with any non-numeric column silently coerces the whole thing to character first, so `is.numeric()` incorrectly returns `FALSE` for every column, not just the non-numeric one.

**Fix:** Select only the numeric columns before converting (`as.matrix(df[sapply(df, is.numeric)])`), or explicitly recode factor/character columns (numeric recoding or dummy encoding) before conversion. For column-type checks specifically, use `vapply(x, is.numeric, logical(1))` rather than `apply(x, 2, is.numeric)`.

**Sources:**

- [Posit Community thread](https://forum.posit.co/t/data-frame-converted-to-characters-when-making-it-into-a-matrix-why/137961) — confirmed: clean, direct example with numeric and factor columns, community explains the mechanism and the dummy-encoding fix
- [lintr issue #2886](https://github.com/r-lib/lintr/issues/2886) — confirmed, strong: lintr maintainers actively designing a dedicated linter rule because `apply(x, 2, is.numeric)` is silently broken by this exact mechanism
- [data.table issue #4209](https://github.com/Rdatatable/data.table/issues/4209) — confirmed: real, maintainer-engaged bug, data.table's `as.matrix()` has a secondary formatting quirk on top of the base coercion behavior, traced to base R's own historical implementation

---

---

## Card [R-13]
**Smell Name:** No Single Deterministic-Algorithm Switch Across Engines

**Language:** R

**Zhang et al. Mapping:** R6 (Deterministic Algorithm Option Not Used) — *Direct analog*

**Pipeline Stage:** Model Training

**Description:** Unlike Python's `torch.use_deterministic_algorithms()`, there is no single, package-wide switch in R to force deterministic behavior across model-training engines. Some engines are structurally non-deterministic regardless of `set.seed()`: kernlab's SVM implementation uses a random number generator independent of R's with no exposed control, and GPU-based deep learning (tensorflow, keras, torch in R) is very difficult to make reproducible, since these packages maintain their own C++/CUDA-level RNG that `set.seed()` does not touch.

**Example:** Calling `set.seed()` before training an SVM via `kernlab::ksvm()` has no effect — results vary run to run regardless. Training a GPU-backed Keras or torch model in R similarly varies across runs even with `set.seed()` called, because the underlying RNG lives outside R entirely.

**Fix:** No general fix exists. For kernlab SVMs, none is available. For GPU deep learning, options include disabling GPU execution entirely (forcing CPU, as R's `tensorflow` package does via `use_session_with_seed()`), or using a package that explicitly synchronizes R's and the backend's RNGs (e.g., `survdnn`'s dedicated `.seed` argument, which calls both `set.seed()` and the torch-level seed together).

**Sources:**

- [tune parallel processing docs](https://tune.tidymodels.org/reference/parallelism.html) — confirmed, authoritative: official tidymodels documentation (maintained by Max Kuhn) with a dedicated "Expectations for reproducibility" section naming both exceptions (kernlab SVM RNG independent of R; GPU deep learning difficulty, plus a bonus detail on cross-platform numerical tolerance differences)
- [rstudio/keras issue #42](https://github.com/rstudio/keras/issues/42) — confirmed: R user explicitly frames this as a problem for R's reproducibility-focused culture ("difficult to generate teaching materials, reports, books... if we get different results on every run"), maintainer engaged
- [survdnn package documentation](https://stat.ethz.ch/CRAN/web/packages/survdnn/readme/README.html) — confirmed, independent R package: explicitly states `set.seed()` "only controls R's random number generator" while `{torch}` "relies on its own RNG implemented in C++ (and CUDA when using GPUs)" — directly confirms the card's core claim from a second, unrelated package
- [R tensorflow package documentation](https://rdocumentation.org/packages/tensorflow/versions/2.16.0/topics/use_session_with_seed) — confirmed: shows the severity of the workaround required (disabling GPU execution entirely) to approach reproducibility

---

---

## Card [R-14]
**Smell Name:** Invalid Value (NaN/Inf) Silently Produced Instead of Failing

**Language:** R

**Zhang et al. Mapping:** R7 (Missing the Mask of Invalid Value) — *Direct analog*

**Pipeline Stage:** Data Preparation / Model Training

**Description:** A transformation step — normalization, log transform, geometric mean, or a gradient computation — can silently produce `NaN` or `Inf` on edge-case input (zero variance, zero or negative values, numeric overflow) instead of failing immediately. The error only surfaces much later, often at model fitting or during backpropagation, far from its actual cause, making it far harder to debug.

**Example:** `recipes::step_normalize()` on a zero-variance column divides by a standard deviation of (numerically) zero, silently replacing the entire column with `NaN`. The error doesn't appear until the model is fit: `Error: Missing data in columns: x2`. The same pattern recurs with `log(0) = -Inf` in response/predictor transformations, and with numeric overflow in statistics like geometric mean (`prod(z)^(1/n)` overflowing for large `z` or large `n`).

**Fix:** Catch and explicitly mask invalid values at the point they're created, not downstream. For zero-variance columns, run `step_zv()` before `step_normalize()`. For log transforms, validate inputs are strictly positive first. For numerically unstable formulas (e.g. geometric mean), use a stable reformulation (log-sum-exp: `exp(mean(log(z)))`) and add an explicit guard that errors or warns rather than silently returning `NaN`/`Inf`. Current `recipes` versions raise an error at the preprocessing step itself rather than deferring to model-fit time.

**Sources:**

- [recipes issue #920](https://github.com/tidymodels/recipes/issues/920) — confirmed, exact match: `step_normalize()` on a zero-variance column silently produces `NaN`, error only surfaces at model fitting; maintainer (Julia Silge) proposed and merged a fix (PR #927) adding a proactive error
- [NADEF-RW-2-BIO issue #1](https://github.com/andydawson/NADEF-RW-2-BIO/issues/1) — confirmed: independent dendrochronology analysis repo, `log(0)` silently produces `Inf`, breaking the downstream model until manually replaced
- [EOF_SW2 issue #3](https://github.com/limnoliver/EOF_SW2/issues/3) — confirmed: a second, unrelated research repo hitting the identical `log(0) → -Inf` pattern
- [descriptr issue #134](https://github.com/rsquaredacademy/descriptr/issues/134) — confirmed, strong: geometric mean computation (`prod(z)^(1/n)`) silently overflows to `Inf`/`NaN`; fix specifies the textbook correct pattern — a numerically stable log-sum-exp reformulation plus an explicit guard that warns and returns `NA` rather than propagating silently
- [anvl issue #590](https://github.com/r-xla/anvl/issues/590) — confirmed, deeper manifestation: in an R reverse-mode automatic differentiation library, a forward-pass overflow (`exp(1000) = Inf`) causes `0 × Inf = NaN` to silently corrupt gradient computations during backpropagation — extends this smell into the core of ML training itself, not just preprocessing

---

---

## Card [R-15]
**Smell Name:** Silent Vector Recycling (Broadcasting Inverted)

**Language:** R

**Zhang et al. Mapping:** R1 (Broadcasting Feature Not Used) — *Inverted*

**Pipeline Stage:** Data Preparation / Model Training / Model Evaluation

**Description:** Where Zhang et al.'s R1 concerns Python/TensorFlow users failing to use an available broadcasting feature, R's version runs the opposite direction: R automatically and silently recycles (repeats) the shorter of two vectors to match the longer one in arithmetic operations, by design, as a core language feature. The risk is that this happens even where it shouldn't — typically when two vectors were *expected* to already be the same length (e.g., observations and predictions, or two halves of a bias-corrected statistic), and a mismatch signals a real bug rather than an intentional broadcast. R only emits a warning, never an error, so the silently wrong result propagates.

**Example:** `rmse(1:10, 5:15)` (mismatched-length observation/prediction vectors) returns a plausible-looking number with only a warning, not a failure. In a bias-correction EM algorithm, filtering two related vectors for `NA` independently (rather than jointly) can leave them different lengths, after which all subsequent arithmetic between them is silently misaligned — with measurable downstream impact on computed p-values.

**Fix:** Explicitly check and enforce equal input lengths before arithmetic (e.g., `stopifnot(length(a) == length(b))`), or use a strict-mode variant of a function where available (e.g., `ifelse(..., strict = TRUE)`-style options). When filtering related vectors for missingness, filter jointly on a combined mask, not independently per vector.

**Sources:**

- [dplyr 1.2.0 release notes](https://github.com/tidyverse/dplyr/releases) — confirmed, authoritative but narrow: the dplyr core team soft-deprecated one specific `case_when()` recycling pattern, explicitly because "the recycling behavior... is unsafe, and can result in silent bugs." Note: this only patches one narrow usage inside one function — base R arithmetic recycling (the root mechanism) is untouched and remains fully active everywhere else, including in every other source below
- [trainR issue #17](https://github.com/USGS-R/trainR/issues/17) — confirmed, strong, directly ML-evaluation: `rmse()` silently recycles mismatched observation/prediction vectors, producing a wrong evaluation metric with only a warning
- [NHSRwaitinglist issue #53](https://github.com/nhs-r-community/NHSRwaitinglist/issues/53) — confirmed: a healthcare simulation package building clearer recycling warnings because R's default is too terse to notice
- [OSPSuite.ParameterIdentification issue #335](https://github.com/Open-Systems-Pharmacology/OSPSuite.ParameterIdentification/issues/335) — confirmed, strong, model-fitting context: a length-check function's boolean result was silently discarded (never actually enforced), so recycling still occurred, producing `NA` weights that cascaded into a misleading "Invalid model cost detected" error during pharmacokinetic parameter optimization
- [compstatslib issue #34](https://github.com/compstatslib/compstatslib/issues/34) — confirmed, strong: regression fit statistics (SSR, SSE, SST) computed wrong when `lm()` drops an incomplete row and residual-statistics code doesn't account for the resulting length mismatch; same root cause confirmed recurring in two other functions in the same package
- [ifelse issue #2](https://github.com/jaganmn/ifelse/issues/2) — confirmed: maintainer agrees mismatched-length `yes`/`no` arguments are essentially always user error, considered adding strict-by-default validation
- [r-forge-survey-mirror issue #8](https://github.com/bschneidr/r-forge-survey-mirror/issues/8) — confirmed, strong: the official CRAN `survey` package silently produces a malformed replicate-weights matrix from `NA`contaminated input, yielding a **wrong standard error (0 instead of correct)** with only cryptic recycling warnings, no actionable error
- [ANCOMBC issue #231](https://github.com/FrederickHuangLin/ANCOMBC/issues/231) — confirmed, exceptionally strong: inside an EM bias-correction algorithm, two related vectors filtered for `NA` independently end up different lengths, silently misaligning all subsequent arithmetic; a contributor directly confirmed via patched-vs-unpatched comparison that **p-values and q-values differed** as a result — five+ independent users affected, original maintainer engaged, community fix merged to a bugfix branch

---

---

## Card [R-17]
**Smell Name:** Cross-Language (Reticulate) Seed Coordination

**Language:** R

**Zhang et al. Mapping:** None — *Novel smell, no Python catalogue equivalent*

**Pipeline Stage:** Model Training

**Description:** When an R package wraps a Python ML backend via `reticulate` (e.g., calling Keras/TensorFlow), reproducibility requires separately seeding R's own RNG *and* Python's `random`, NumPy, and TensorFlow generators independently — missing any one of these breaks reproducibility even when the R-side seed looks correct and complete.

**What Makes This ML-Specific (Novel Smell Justification):** This failure mode requires the concept of a cross-language ML framework bridge to exist at all — single-process Python code cannot have this problem, since there is no second language runtime with its own independent RNG state to coordinate. The smell is specific to R packages that wrap Python deep learning backends for training, a pattern with no equivalent anywhere in Zhang et al.'s original Python-only catalogue.

**Example:** `mlr3keras_set_seeds()` must separately call `set.seed()` (R), `reticulate::import("random")$seed()` (Python's `random` module), `reticulate::py_set_seed(..., disable_hash_randomization = TRUE)` (Python/NumPy), and `tensorflow$random$set_seed()` (TensorFlow) — four distinct calls, each necessary, in a specific order (TensorFlow's seed must be set after disabling hash randomization). Omitting any single one silently breaks reproducibility while leaving the others intact.

**Fix:** Use a dedicated multi-seed coordination function rather than a single `set.seed()` call — e.g., `mlr3keras::mlr3keras_set_seeds()`, the official `tensorflow` R package's `set_random_seed()`, or `parsnip::set_tf_seed()`. Be aware that even these coordination functions can break across backend version upgrades and may need periodic maintenance.

**Sources:**

- [mlr3keras_set_seeds.R](https://github.com/mlr-org/mlr3keras/blob/master/R/mlr3keras_set_seeds.R) — confirmed: dedicated function setting R, Python `random`, Python/NumPy, and TensorFlow seeds separately, in a specific required order
- [mlr3keras test suite](https://rdrr.io/github/mlr3keras/src/tests/testthat/test_mlr3keras_set_seeds.R) — confirmed: dedicated tests independently verifying `random.random`, `numpy.random`, and `tf.random` reproducibility as three separate concerns
- [rstudio/tensorflow seed.R](https://github.com/rstudio/tensorflow/blob/master/R/seed.R) — confirmed, highly authoritative: Posit/RStudio's own official R TensorFlow interface, with a dedicated `set_random_seed()` function explicitly documented as setting "the R random seed... the python and Numpy seeds... the TensorFlow seed" together
- [parsnip::set_tf_seed()](https://www.rdocumentation.org/packages/parsnip/versions/1.0.0/topics/set_tf_seed) — confirmed: a third, independent major package (core tidymodels) with its own dedicated dual-seed function, documented: "Some Keras models requires seeds to be set in both R and TensorFlow to achieve reproducible results"
- [rstudio/tensorflow PR #442 / test-seed.R](https://github.com/rstudio/tensorflow/pull/442/files) — confirmed: even the official coordination function has had real breakage across TensorFlow versions, evidenced by a test explicitly skipped with "use_session_with_seed doesn't work with TF >= 2.3"

---

---

## Card [R-21]
**Smell Name:** Default Classification Threshold Danger (tidymodels/probably)

**Language:** R (tidymodels ecosystem: yardstick, probably)

**Zhang et al. Mapping:** R19 (Threshold-Dependent Validation) — *Direct analog, distinct package/mechanism from Card R-10*

**Pipeline Stage:** Model Evaluation

**Description:** Leaving a binary classification threshold at its default 0.5 can silently produce effectively unusable results on imbalanced data, since the default threshold is not calibrated to the actual class distribution. The `probably` package's `make_two_class_pred(threshold = ...)` was built specifically to let users override this default and select an appropriate threshold for their data.

**What Differs from Card R-10:** R-10 concerns `caret::train()` defaulting to plain Accuracy as its model-selection/tuning metric, requiring explicit opt-in to ROC-based evaluation. This card is about a different package family (tidymodels' `yardstick`/`probably`, not `caret`) and a mechanically distinct concern: not which metric is used to select a model, but what probability cutoff is used to convert predicted probabilities into class labels after a model is already fit. Both sit within Zhang et al.'s R19 territory but address different stages of the same broader "threshold/metric choice matters" problem.

**Example:** On a simulated imbalanced churn-prediction dataset, classifying with the default 0.5 threshold produced a sensitivity of just 0.062 — meaning the model correctly identified only about 6% of actual churners, in the tidymodels team's own words: "with the default threshold of 0.5, basically no customers were classified as at risk for churn!" Adjusting the threshold to 0.7 using `make_two_class_pred(.pred_yes, levels(churn), threshold = 0.7)` improved sensitivity to 0.149 — still imperfect, but more than double the true positive rate, illustrating threshold choice as a first-order lever, not a minor tuning detail.

**Fix:** Use `probably::make_two_class_pred()` (or `threshold_perf()`) to explicitly select and apply a probability threshold appropriate to the class distribution and the cost of false negatives/positives, rather than relying on the implicit 0.5 default. `yardstick::roc_curve()`'s `thresholds` argument and `metric_tweak()` can help identify and apply an appropriate cutoff.

**Sources:**

- [Official tidyverse blog, Q2 2021 tidymodels digest](https://www.tidyverse.org/blog/2021/07/tidymodels-2021-q2/) — confirmed exactly, authored by Julia Silge (tidymodels core team): full worked example with exact quoted line ("with the default threshold of 0.5, basically no customers were classified as at risk for churn!") and concrete before/after sensitivity numbers (0.062 → 0.149)
- [tidymodels/probably #149](https://github.com/tidymodels/probably/issues/149) — **not independently verified in this pass**; cited as evidence of inconsistent `threshold_perf()` behavior

---

---

## Card [R-22]
**Smell Name:** `data.table` Chained Assignment Performance Trap (ALTREP)

**Language:** R (data.table)

**Zhang et al. Mapping:** R20 (Chain Indexing) — *Direct analog, different underlying mechanism*

**Pipeline Stage:** Data Preparation

**Description:** Chaining assignments on a `data.table` column after certain type-conversion operations (e.g., `as.character()`) triggers R's ALTREP (ALternative object REPresentation) system to force a full, expensive data expansion on every subsequent chained touch of that column — a hidden cost that doesn't appear in a single, unchained operation on the same data.

**What Differs from the Python Version:** Zhang et al.'s R20 concerns pandas' chained indexing (`df[a][b] = x`) risking a silent no-op due to copy-vs-view ambiguity — a correctness problem. This R mechanism is different: it's a severe, hidden performance cost (not a correctness failure) rooted in R's lazy ALTREP representation, where a type-conversion function returns a deferred placeholder rather than doing the actual (expensive) work immediately, and any subsequent touch forces that deferred work to materialize all at once. The *pattern* — chaining operations silently costing far more than they appear to — is the same as R20's territory, but the specific mechanism (lazy representation forcing, not copy/view ambiguity) is genuinely different.

**Example:** `copy(df)[, x := as.character(x)][]` (single, unchained) runs in ~1.33ms. `copy(df)[, x := as.character(x)][, x := x][]` (with one additional chained touch) runs in ~490ms — roughly a 300-370x slowdown for what looks like an equivalent or barely-different operation. The first `as.character()` call returns only a lazy ALTREP placeholder; the second chained statement is what actually forces the expensive string conversion to happen, inside the timed operation.

**Fix:** Avoid unnecessary chained touches of a column immediately after a type-conversion assignment; if a conversion's cost needs to be measured or amortized predictably, force materialization once (e.g., via an explicit print or a dedicated materializing call) rather than letting it occur implicitly inside a later, seemingly unrelated chained operation.

**Sources:**

- [data.table issue #5408](https://github.com/Rdatatable/data.table/issues/5408) — confirmed exactly: benchmark numbers match precisely (750µs/1.33ms single vs. 416ms/490.03ms chained via `bench::mark()`); community explanation confirms the ALTREP lazy-conversion mechanism, including that simply printing the column or touching it any other way (not just a second assignment) triggers the same forced expansion

---

---

## Card [JL-02]
**Smell Name:** Silent NaN Corruption in BatchNorm from Single-Sample Batches

**Language:** Julia (Flux.jl)

**Zhang et al. Mapping:** R7 (Missing Mask of Invalid Value) — *Related, meaningfully different failure mode*

**Pipeline Stage:** Model Training

**Description:** `BatchNorm` in training mode silently corrupted its running variance with `NaN` whenever a channel saw only a single value (e.g., batch size 1 with no spatial dimensions), with no error raised. The corruption would then propagate into every subsequent use of the layer's running statistics, including at inference time, without any indication of what caused it. Fixed in an unreleased version to throw a clear error instead, matching PyTorch's existing behavior for the same edge case.

**What Differs from the Python Version:** Zhang et al.'s R7 concerns functions like `np.mean()` silently returning `NaN` when any input element is `NaN`, requiring `np.nanmean()` to mask it. This Julia case is a different and arguably more severe mechanism: the corruption isn't caused by `NaN` already present in the input data, but is *manufactured internally* by the layer's own variance-correction formula failing on a valid, non-NaN input (batch size 1) — and it then poisons the layer's persistent internal state (running variance) rather than just one computation's output.

**Example:** `bn = BatchNorm(1); x = zeros(1, 1); bn(x); bn.σ²` returns `NaN`. The root cause: `*track*``stats!` applies an unbiased-variance (Bessel) correction factor `m / (m - 1)` to update the running variance, which evaluates to `1/0` when the batch size `m` equals 1, producing `NaN` with no warning or error at the point of failure.

**Fix:** The maintainer-confirmed fix changes the correction factor to `m / max(1, m - 1)`, avoiding the division by zero, and additionally makes `BatchNorm` throw a clear, explicit error in training mode when a channel sees only a single value, rather than silently producing `NaN` (inference mode via `testmode!` is unaffected, since it doesn't update running statistics).

**Sources:**

- [Flux.jl NEWS.md, "Unreleased" section](https://github.com/FluxML/Flux.jl/blob/master/NEWS.md) — confirmed exactly: "`BatchNorm` in training mode now throws a clear error when a channel sees only a single value... instead of silently corrupting the running variance with `NaN`... this matches PyTorch's behavior," explicitly noting Flux had lagged behind PyTorch's existing handling of this exact case
- [Flux.jl issue #1992, "Batchnorm's variance becomes NaN when minibatch contains just one sample"](https://github.com/FluxML/Flux.jl/issues/1992) — confirmed, strong: precise minimal reproducible example, exact root-cause diagnosis (Bessel correction `m/(m-1)` dividing by zero at `m=1`), proposed one-line fix from the reporter; a Flux.jl core maintainer (CarloLucibello) explicitly re-confirmed the bug "still reproduces on Flux 0.16.11" in a later triage pass, demonstrating the silent-NaN behavior persisted across multiple releases before the eventual fix

---

---

## Card [JL-03]
**Smell Name:** NaN Propagation Without Explicit Skip

**Language:** Julia (DataFrames.jl / Base)

**Zhang et al. Mapping:** R7 (Missing Mask of Invalid Value) — *Direct analog*

**Pipeline Stage:** Data Preparation

**Description:** Unlike pandas, which skips `NaN` in analytic functions (like `mean()`) by default, Julia's analytic functions do not skip `NaN`'s — any `NaN` present in the input silently propagates to make the entire result `NaN`, requiring an explicit opt-out (`skipmissing()` for Julia's `missing` sentinel, or `filter(!isnan, x)`/`Iterators.filter(!isnan, x)` for `NaN` specifically) to get a meaningful aggregate.

**Example:** `mean(df.z)` where `df.z` contains any `NaN` returns `NaN` for the whole computation, mirroring Python's `np.mean()` without `np.nanmean()`. Julia's official pandas-comparison documentation states this directly: `df['z'].mean(skipna = False)` maps to `mean(df.z)` (NaN-propagating), while `df['z'].mean()` (pandas' NaN-skipping default) maps to `mean(skipmissing(df.z))` in Julia — the default behaviors are reversed between the two languages.

**Fix:** Use `mean(skipmissing(x))` to skip Julia's `missing` sentinel, or `mean(Iterators.filter(!isnan, x))` to specifically skip `NaN` values, rather than relying on default NaN-skipping behavior, which Julia does not provide.

**Sources:**

---
- [DataFrames.jl official documentation, "Comparison with Python/R/Stata"](https://dataframes.juliadata.org/stable/man/comparisons/) — confirmed verbatim: "Note that pandas skips `NaN` values in its analytic functions by default. By contrast, Julia functions do not skip `NaN`'s. If necessary, you can filter out the `NaN`'s before processing, for example, `mean(Iterators.filter(!isnan, x))`," with an accompanying explicit side-by-side function-mapping table

---

## Card [JL-04]
**Smell Name:**  Join Missing-Value Matching Enabled via (`matchmissing=:equal`)

**Language:** Julia (DataFrames.jl)

**Zhang et al. Mapping:** R15 (Merge API Parameter Not Explicitly Set) — *Variant, the same hazard behind an explicit opt-in*

**Pipeline Stage:** Data Preparation

**Description:** DataFrames.jl's join functions take a `matchmissing` keyword. The default, `:error`, throws when a join key is missing, because a missing value is unknown and could match anything. Passing `matchmissing=:equal` instead treats `missing` as an ordinary value, so rows with unknown keys match each other. The smell is enabling `:equal` without recognizing that unrelated rows with unknown keys will be joined, the same silent matching described by Card R-09 in R's `dplyr`.

**What Differs from the Python Version, and From Most of This Catalogue:** In pandas and in R, matching missing keys is the default, so the hazard appears with no action from the user. In DataFrames.jl the user has to opt in, so the hazard is a deliberate choice that is easy to make carelessly, for example to silence the default error. The maintainers treat `:equal` as the risky mode: even under `:equal`, DataFrames.jl refuses to join on `NaN` or `-0.0` in a key column, an extra layer of friction the maintainer describes as chosen "for safety reasons."

**Example (illustrative, constructed from the documented semantics, not a reported incident):** `leftjoin(orders, customers, on=:customer_id, matchmissing=:equal)` where several orders and several customers all have a missing `customer_id`. Every order with a `missing` key is matched to every customer with a `missing` key: two such orders and two such customers produce four rows. Unrelated orders are attributed to unrelated customers and the row count is inflated, with no error. With the default `matchmissing=:error`, the same call throws `ArgumentError: missing values in key columns are not allowed when matchmissing == :error`.

**Fix:** Keep the default `:error`, or handle missing keys explicitly before joining, for example by dropping or imputing them. Use `matchmissing=:equal` only when missing keys genuinely denote the same entity.

**Sources:**

- [bkamins.github.io, "DataFrames.jl joins: matchmissing=:notequal"](https://bkamins.github.io/julialang/2021/06/05/notequal.html) — confirmed, maintainer-authored (Bogumił Kamiński, DataFrames.jl lead): explicitly states the `:error` default and its rationale ("missing indicates unknown value so if we knew it it could match to any of the non-missing values")
- [bkamins.github.io, "How is equality checked in DataFrames.jl?"](https://bkamins.github.io/julialang/2023/04/14/isequal.html) — confirmed, maintainer-authored: demonstrates the extra `NaN`/`0.0` safety barrier even under `matchmissing=:equal`, with the maintainer's direct statement that this was "a decision that was made for safety reasons"
- [DataFrames.jl issue #2650, "Add matchmissing = :notequal option"](https://github.com/JuliaData/DataFrames.jl/issues/2650) — confirmed: real design deliberation thread showing the maintainer working through benchmarks and tradeoffs before shipping the feature.

---

---

## Card [JV-01]
**Smell Name:** Inconsistent Seed Handling Across MLlib Algorithms

**Language:** Java / Scala (Apache Spark MLlib)

**Zhang et al. Mapping:** R2 (Randomness Uncontrolled) — *Direct analog, variant*

**Pipeline Stage:** Model Training

**Description:** Different MLlib algorithms handle the `seed` parameter inconsistently across both algorithms and language bindings. `BisectingKMeans` had a hardcoded or inconsistent seed default unlike other clustering algorithms; SparkR's `spark.kmeans` didn't expose a `seed` parameter at all, despite the underlying Scala/Java API supporting one. A deeper issue surfaced during the fix: MLlib's seed type is `Long` (max ~9.2×10¹⁸), but bindings converting seed values through narrower intermediate types risk silent truncation across language boundaries.

**Example:** Prior to SPARK-17311, `BisectingKMeans` did not default to `seed = None`/unset the way other `.mllib` clustering classes did, creating inconsistent reproducibility behavior across otherwise-similar algorithms. Separately, prior to SPARK-19142, calling `spark.kmeans()` from R offered no way to set a seed at all, even though the underlying Spark K-means implementation supported one — R users had no path to reproducible clustering through this interface.

**Fix:** SPARK-17311 standardized `.mllib` classes to consistently accept optional `Long` seeds. SPARK-19142 added `seed`, `initSteps`, and `tol` parameters to SparkR's `spark.kmeans()` interface. Both fixes were merged; however, reviewers noted the existing MLlib K-means test suite (`KMeansSuite.scala`) was "insensitive to seed and tol" even after the fix, meaning seed-correctness was not actually verified by tests at the time — a documented gap between the fix and its validation.

**Sources:**

- [SPARK-17311 / PR #14826](https://github.com/apache/spark/pull/14826) — confirmed: "Standardize Python-Java MLlib API to accept optional long seeds in all cases," explicitly targeting `BisectingKMeans` as the inconsistent outlier, managed by senior Apache Spark committer Sean Owen
- [SPARK-19142 / PR #16523](https://github.com/apache/spark/pull/16523) — confirmed: "`spark.kmeans` should take `seed`, `initSteps`, and `tol` as parameters," explicitly stating SparkR's `spark.kmeans` previously had no interface to set these at all; reviewer discussion additionally surfaces a `Long`to-narrower-type seed truncation risk across the R/JVM boundary, and confirms the existing test suite could not detect seed-insensitivity bugs

---

---

## Card [JV-02]
**Smell Name:** Incomplete Multi-Runtime Seed Coordination (PySpark)

**Language:** Java / Scala (Spark core), surfaced through Python (PySpark)

**Zhang et al. Mapping:** None — *Novel smell, proposed generalizable category: "Incomplete Determinism Control Across Runtime Boundaries," applicable to any ML framework spanning multiple languages or distributed execution*

**Pipeline Stage:** Model Training

**Description:** Full reproducibility in PySpark ML workflows requires setting three separate seeds — Python's `random.seed()`, NumPy's `np.random.seed()`, and `spark.sparkContext.setRandomSeed()` — and setting a seed only on an individual ML algorithm is explicitly insufficient, since it doesn't control Spark's own distributed data-shuffling and partition-level randomness. Seed control alone also does not guarantee reproducibility across different cluster configurations, since Spark's randomness is distributed across executors rather than confined to a single driver process. Additionally, the same seed value can produce different results across Spark versions because Spark's internal RNG algorithm has changed between releases.

**What Makes This a Novel Smell:** This requires the combination of a cross-language binding (Python calling into a JVM-based distributed engine) *and* distributed, partition-based execution — a combination with no single-process Python equivalent. It is conceptually related to Card R-18 (reticulate cross-language seed coordination) but adds a second independent axis of fragility: not just "two runtimes, two RNGs," but "one runtime's RNG behavior is itself inconsistent across the distributed partitions it spans, and across its own version history."

**Example:** Training a `RandomForestRegressor` in PySpark with only the model's own `seed` parameter set reproducibly builds different trees and computes a different AUC on a second run, because Python's `random` and NumPy's RNG (used in feature preprocessing) were never seeded. Separately, a real user (SPARK-27293) found that using the same seed value and code as classmates produced identical results for most seed values but a different result for one specific seed, illustrating that even full seed-setting does not guarantee determinism given cluster/partition variation.

**Fix:** Explicitly call `random.seed()`, `np.random.seed()`, and `spark.sparkContext.setRandomSeed()` together, before data loading and model training, in addition to any per-algorithm `seed` parameter. Be aware this guarantees reproducibility only within a fixed Spark version and fixed cluster/partition configuration — not across version upgrades or differing executor counts.

**Sources:**

- [theneuralbase.com PySpark reproducibility guide](https://theneuralbase.com/pyspark-for-ml/learn/intermediate/reproducibility/) — confirmed, detailed: explicitly states all three seed calls are required, that per-algorithm seeding alone is insufficient, that cluster configuration affects reproducibility independent of seeding, and that Spark's internal RNG algorithm has changed across major versions. Note: this appears to be an AI-compiled tutorial/reference site (every page carries an automated-looking "Verified [date]" annotation) — content confirmed accurate against the sources below, but treated as a secondary rather than primary source on its own
- [SPARK-27293](https://issues.apache.org/jira/browse/SPARK-27293) — confirmed, official Apache JIRA, human-reported: a user got identical RandomForestRegressor results to classmates for most seed values, but a different result for one specific seed, using identical code and data; resolved "Not A Problem" by maintainers but documents real confusion from this exact fragility
- [SPARK-4148 / PR #3010](https://github.com/apache/spark/pull/3010/files) — confirmed, official Apache Spark GitHub, maintainer-fixed historical bug: "the current way of seed distribution makes the random sequences from partition i and i+1 offset by 1" — direct code-level evidence that Spark's partition-based distributed seeding has had real, shipped defects, and that the underlying seeding mechanism has changed over Spark's history

---

---

## Card [JV-03]
**Smell Name:** Memory Not Freed via Off-Heap/Workspace Architecture (DL4J)

**Language:** Java (DeepLearning4J)

**Zhang et al. Mapping:** R10 (Memory Not Freed) — *Direct analog, via a JVM/off-heap-specific mechanism with no Python equivalent*

**Pipeline Stage:** Model Training / Evaluation

**Description:** DL4J uses JavaCPP-managed off-heap memory and an explicit "workspace" system for memory reuse, bypassing normal JVM garbage collection for performance. Several documented bugs arise when operations — notably `doEvaluation` — fail to properly scope output arrays to a workspace, leaving them "garbage-collector scoped" instead. Since the GC does not run mid-loop during tight evaluation cycles, memory builds up unboundedly until an OutOfMemoryError is thrown.

**What Differs from the Python Version:** Zhang et al.'s R10 concerns forgetting to release an explicit resource handle (e.g., a TensorFlow session). DL4J's version is structurally novel: there is no equivalent "workspace" concept in PyTorch or TensorFlow's Python APIs at all — this is a JVM-specific architectural pattern invented specifically to let a garbage-collected language manage off-heap native memory efficiently, and the smell arises precisely from the friction between that workspace system and the JVM's normal GC, a tension with no single-memory-model equivalent in Python.

**Example:** A standard feedforward network's `doEvaluation` creates its output `INDArray` without assigning it to a workspace (`outputWorkspace` set to `null`), so the array falls back to GC-scoped lifetime. Running evaluation over a large test dataset shows memory climbing steadily (confirmed via `nvidia-smi` monitoring) until an OOM exception is thrown, because the GC never gets a chance to run during the tight evaluation loop. A reporter confirmed two fixes: manually forcing `System.gc()` inside the loop eliminated the buildup but caused "a massive performance hit"; properly scoping the output array to a Memory Workspace eliminated the buildup with no performance cost at all.

**Fix:** Explicitly scope evaluation output arrays to a `MemoryWorkspace` rather than leaving them GC-scoped (the fix implemented following #6279, tracked in #6295). More broadly, DL4J's workspace system requires careful, explicit management throughout training and evaluation code; the project underwent a significant "workspace optimization and refactoring" effort (PR #4900) specifically to reduce the frequency of these bugs and add workspace-usage validation to catch them earlier.

**Sources:**

- [deeplearning4j/deeplearning4j #6647](https://github.com/deeplearning4j/deeplearning4j/issues/6647) — confirmed: OOM during prediction, stack trace traces through `outputOfLayerDetached`/`MultiLayerNetwork.output()`, failing to allocate ~4.25GB
- [deeplearning4j/deeplearning4j #3630](https://github.com/deeplearning4j/deeplearning4j/issues/3630) — confirmed: production-scale OOM during `doEvaluation` within early-stopping training ("physicalBytes = 53G > maxPhysicalBytes = 53G")
- [deeplearning4j/deeplearning4j #4335](https://github.com/deeplearning4j/deeplearning4j/issues/4335) — confirmed, notable nuance: OOM occurs only when the workspace feature is enabled ("if don't use workspace, it works fine"); maintainers acknowledge ongoing fix work in PR #4900
- [deeplearning4j/deeplearning4j #6279](https://github.com/deeplearning4j/deeplearning4j/issues/6279) — confirmed, exceptionally detailed: precise root-cause diagnosis (GC-scoped vs. workspace-scoped output arrays in `doEvaluation`), with two compared fixes (`System.gc()` workaround vs. proper workspace scoping) showing the latter eliminates both the leak and any performance cost
- [deeplearning4j/deeplearning4j #4539](https://github.com/deeplearning4j/deeplearning4j/issues/4539) — confirmed: a distinct workspace-related failure mode (memory corruption rather than buildup) in `ComputationGraph`, "very likely workspaces related" per maintainer discussion
- [deeplearning4j/deeplearning4j #5351](https://github.com/deeplearning4j/deeplearning4j/issues/5351) — confirmed: a third distinct failure mode (an outright exception, "Expected no workspace active") during evaluation, resolved by disabling workspace mode entirely
- [Official troubleshooting documentation](https://deeplearning4j.konduit.ai/deeplearning4j/troubleshooting) — confirmed, authoritative: dedicated "ND4J Off-Heap (Native) OOM" and "Workspace Memory Growing Without Bound" troubleshooting entries, directly naming the mechanism and its fix ("Ensure symmetric open/close pairs")
- [Official "Memory and Workspaces" documentation](https://deeplearning4j.konduit.ai/core-concepts/memory-and-workspaces) — confirmed, near-verbatim match to the smell's description: "ND4J's off-heap memory is eventually freed by Java's garbage collector when the INDArray pointer objects are collected. For tight loops without workspaces, GC may not run frequently enough, causing off-heap memory to grow."
- [1.0.0-beta4 release notes](https://deeplearning4j.konduit.ai/en-1.0.0-rewrite/release-notes/1.0.0-beta4) — confirmed exactly: "Removed reliance on periodic garbage collection calls for handling memory management of out-of-workspace (detached) INDArrays... Added `INDArray.close()` method to allow users to manually release off-heap memory immediately" — confirms the precise version and nature of the fix

---

---

## Card [JV-04]
**Smell Name:** Workspace-Related Gradient/Data Corruption (DL4J)

**Language:** Java (DeepLearning4J)

**Zhang et al. Mapping:** None — *Novel smell, tied specifically to DL4J's workspace architecture (no Python equivalent)*

**Pipeline Stage:** Model Training

**Description:** DL4J's workspace memory-reuse system can unexpectedly clear or corrupt gradient arrays when custom external-error (custom loss/gradient) code interacts incorrectly with workspace lifecycle timing — manifesting as apparently corrupted data immediately after backpropagation completes, even though neither the input nor output arrays were corrupted at the point of computation itself.

**What Makes This a Novel Smell:** Related to Card JV-03 (same underlying workspace architecture, same root cause category) but mechanically distinct: JV-03 concerns memory *not being freed* (volume/leaks), while this concerns memory being *silently reused/cleared out from under live data* (correctness/corruption). Neither has a Python equivalent, since PyTorch/TensorFlow have no comparable explicit workspace-reuse abstraction whose lifecycle a custom training loop can get out of sync with.

**Example:** A user implementing custom external errors (custom gradients/loss) with `ComputationGraph` found that neither the output nor the input arrays were corrupted immediately after the relevant computation, but became corrupted shortly afterward, once the computation had "finished." Running with DL4J's `NAN_PANIC` diagnostic mode caused a panic specifically inside the `backpropGradient` call, pointing at gradient-array handling during backpropagation as the trigger. The maintainer noted that array-clearing behavior around this exact code path had changed between the version the user was running (0.9.1) and the current development branch, directly implicating workspace-related array lifecycle management as the likely cause.

**Fix:** No general fix was confirmed in this issue thread (it concludes with a maintainer noting the diagnosis "lowered the probability" of a specific cause rather than confirming a definitive fix) — flagging this honestly rather than asserting a resolution that wasn't clearly reached. Practical mitigation implied by the discussion: be cautious with custom external-error/gradient code interacting with `ComputationGraph`'s workspace-scoped arrays, and test with `NAN_PANIC` mode enabled to catch corruption early during development.

**Sources:**

- [deeplearning4j/deeplearning4j #4539](https://github.com/deeplearning4j/deeplearning4j/issues/4539) — confirmed: detailed real-time diagnostic exchange between reporter and maintainer; `NAN_PANIC` mode isolates the failure to the `backpropGradient` call; maintainer independently confirms suspicion of workspace involvement, noting a related behavior change between versions

---

---

## Card [JV-05]
**Smell Name:** DataType Not Explicitly Set (ND4J)

**Language:** Java (ND4J)

**Zhang et al. Mapping:** R6/R21-adjacent (Columns and DataType Not Explicitly Set) — *Different shape*

**Pipeline Stage:** Data Preparation / Model Training

**Description:** ND4J infers an `INDArray`'s numeric `DataType` from the literal Java type used to construct it — `Nd4j.scalar(1)` produces an `INT`, `Nd4j.scalar(1.0f)` produces a `FLOAT`, `Nd4j.scalar(1.0)` produces a `DOUBLE` — and binary operations require both operands to share the same `DataType`, throwing at runtime (not silently coercing) when they don't. This creates a class of bugs where a seemingly-innocuous choice of numeric literal (an `int` vs a `float` vs a `double`) silently determines downstream type compatibility, and type-inference logic itself has been a source of real, confirmed bugs.

**Example:** `Nd4j.scalar(1)`, `Nd4j.scalar(1L)`, `Nd4j.scalar(1.0)`, and `Nd4j.scalar(1.0f)` produce `INT`, `LONG`, `DOUBLE`, and `FLOAT` type scalars respectively. Operating on two arrays of different inferred types throws `IllegalArgumentException: Op.X and Op.Y must have the same data type, but got FLOAT vs DOUBLE`, requiring an explicit `x.add(y.castTo(x.dataType()))`. Separately, the type-inference logic itself has been buggy: in one confirmed case, `Nd4j.scalar(float)` could recurse infinitely when the library's global default dtype was inconsistently configured (issue #2319); in another, `Nd4j.create()` called with `double[]` data silently returned a `FLOAT`-typed array instead of `DOUBLE`, due to duplicated and inconsistent code paths across overloaded `create()` methods (issue #8057).

**Fix:** Explicitly specify `DataType` when constructing arrays (`Nd4j.create(DataType.FLOAT, ...)`, `Nd4j.zeros(DataType.DOUBLE, ...)`, etc.) rather than relying on literal-type inference, and use `castTo(DataType)` explicitly before any binary operation involving arrays of potentially different types rather than assuming matching types.

**Sources:**

- [1.0.0-beta4 release notes](https://deeplearning4j.konduit.ai/en-1.0.0-beta6/getting-started/release-notes) — confirmed exactly: documents the literal-to-DataType inference behavior and the requirement for matched operand types
- [Official "Operations" documentation](https://deeplearning4j.konduit.ai/nd4j/operations) — confirmed, independent of release notes: "ND4J requires that both operands in a binary operation share the same DataType. Mixing float and double arrays, for example, will produce an error at runtime"
- [Official "Quickstart" documentation](https://deeplearning4j.konduit.ai/en-1.0.0-beta7/nd4j/quickstart) — confirmed: reproducible example showing the exact runtime exception ("`Op.X and Op.Y must have the same data type, but got FLOAT vs DOUBLE`")
- [deeplearning4j/deeplearning4j #2319, "Nd4j.scalar infinite recusrion"](https://github.com/deeplearning4j/deeplearning4j/issues/2319) — confirmed, maintainer-engaged: a genuine bug in the type-dispatch logic underlying `Nd4j.scalar()`, acknowledged directly by maintainer raver119 as stemming from inconsistent dtype configuration being technically possible in prior versions
- [eclipse/deeplearning4j #8057, "Nd4j create method called with double data returns float ndArray"](https://github.com/eclipse/deeplearning4j/issues/8057) — confirmed, independent: duplicated/inconsistent code across overloaded `create()` methods causes `double[]` input to silently produce a `FLOAT`typed array.

---

---

## Card [JV-06]
**Smell Name:** View vs. Copy Depends on the Index (ND4J)

**Language:** Java (ND4J)

**Zhang et al. Mapping:** R3 (Chain Indexing) / R16 (In-Place APIs Misused) — *Loose, dual-mapped*

**Pipeline Stage:** Data Preparation / Model Training

**Description:** Whether an ND4J indexing operation returns a view (sharing underlying memory with the source, so writes propagate back) or a copy depends on which method is used and, in some cases, on the runtime shape of the data being indexed. `getRow()`/`getColumn()` always return views; `getRows()` (plural, multiple rows) always returns a copy, since non-contiguous rows cannot share a single buffer; `get(NDArrayIndex...)` returns a view only when the requested sub-array happens to be contiguous in memory, and silently falls back to a copy otherwise — with no compile-time or obvious runtime signal telling the caller which case they're in.

**Example:** `arr.getRow(i)` returns a view — `arr.getRow(3).addi(1.0)` mutates the original array's third row in place. `arr.getRows(0, 2, 4)` returns a copy, so mutating the result has no effect on `arr`. `arr.get(NDArrayIndex.interval(0, 3), NDArrayIndex.all())` returns a view (contiguous slice), but a differently-shaped `get()` call selecting a discontiguous pattern of indices silently returns a copy instead — confirmed directly by a maintainer-level forum response: *"**`get(...)`** will return a view depending on whether it's possible to... If you ask for any sort of discontiguous array... you'll get those results but it will return a copy only."* A user hit this directly, expecting consistent view semantics across three different `get()`-derived arrays and finding only one of them actually shared memory with the source.

**Fix:** Never assume whether an indexing result is a view or a copy based on the method name alone when `get(NDArrayIndex...)` is involved; check documentation or add an explicit `.dup()` when a copy is required, and avoid relying on in-place mutation through `get()` results unless contiguity is guaranteed by construction (e.g., simple `interval`/`point`/`all` selectors known not to span non-contiguous memory).

**Sources:**

- [Official ND4J "Overview" documentation (multiple versions)](https://deeplearning4j.konduit.ai/en-1.0.0-beta7/nd4j/overview) — confirmed, consistent across versions: "getRow(0) operation returns an INDArray that is a view of the original... getRows(int...)... this will be a copy (not a view)... get(NDArrayIndex...) returns a view of the underlying array, not a copy"
- [Official "Creating NDArrays" documentation](https://deeplearning4j.konduit.ai/nd4j/creating-ndarrays) — confirmed, independently reinforces the same distinction with code examples
- [Official "Indexing and Slicing" documentation](https://deeplearning4j.konduit.ai/nd4j/indexing-and-slicing) — confirmed, most explicit: dedicated "A note on views vs. copies" section explicitly listing which operations return which, including the contiguity caveat for `get(NDArrayIndex...)` implicitly via the `getRows` comparison ("non-contiguous rows cannot share buffer")
- [Konduit Community forum: "Issue with Nd4j get and put"](https://community.konduit.ai/t/issue-with-nd4j-get-and-put/1678) — confirmed, real user confusion matching the exact described mechanism: expected consistent view behavior across multiple `get()` calls, got inconsistent results; maintainer-level response confirms the contiguity-dependent fallback to copy explicitly
- [eclipse/deeplearning4j #7125, "ND4j: Memory leak if calling INDArray.getRow()"](https://github.com/eclipse/deeplearning4j/issues/7125) — confirmed: a genuine bug in `getRow()`'s own view-creation machinery, leaking `LongBuffer`/`AtomicBoolean`/`ArrayList` instances the garbage collector cannot discard
- [eclipse/deeplearning4j #9709](https://github.com/eclipse/deeplearning4j/issues/9709) — confirmed: a shape-handling bug specifically in `getRow()` when the source array has exactly one row, returning the wrong shape and breaking downstream concatenation

---

---

## Card [JV-07]
**Smell Name:** Schema Mismatch Masked by a Wrapper (InputMappedClassifier)

**Language:** Java (Weka)

**Zhang et al. Mapping:** R7 (Missing Mask of Invalid Value) — *Loose*

**Pipeline Stage:** Model Evaluation / Deployment

**Description:** Weka's `InputMappedClassifier` wraps a base classifier specifically to handle incompatible training and test data schemas by constructing a mapping between the attributes the model was trained on and the incoming test instances' actual structure. Rather than erroring on a schema mismatch, it silently fills in missing values: model attributes absent from the incoming data receive missing values, and incoming nominal attribute values the classifier has never seen during training are also mapped to missing — and the diagnostic mapping report documenting exactly what was silently substituted can itself be suppressed via `setSuppressMappingReport(true)`.

**Example:** A model trained with an attribute `region` having nominal values `{North, South, East, West}` is later evaluated on data containing a `region` value of `"Central"` that never appeared during training. Rather than raising an error, `InputMappedClassifier` silently maps this unseen value to missing. If the mapping report is suppressed (the `-M` / `setSuppressMappingReport(true)` option), there is no visible signal that this substitution occurred at all — predictions proceed as if the schema mismatch and the unseen category were simply absent.

**Fix:** Avoid suppressing the mapping report in production or evaluation pipelines; actively review it for unexpected attribute or value substitutions rather than treating silent missing-value imputation as acceptable default behavior for genuine schema drift between training and deployment data.

**Sources:**

- [Official Weka Javadoc, ](https://weka.sourceforge.io/doc.stable-3-8/weka/classifiers/misc/InputMappedClassifier.html)[`InputMappedClassifier`](https://weka.sourceforge.io/doc.stable-3-8/weka/classifiers/misc/InputMappedClassifier.html) — confirmed exactly: "Model attributes that are not found in the incoming instances receive missing values, so do incoming nominal attribute values that the classifier has not seen before," with the `setSuppressMappingReport`/`getSuppressMappingReport` methods confirmed directly
- [The WEKA Workbench, Appendix (Eibe Frank, Mark A. Hall, Ian H. Witten)](https://ml.cms.waikato.ac.nz/weka/Witten_et_al_2016_appendix.pdf) — confirmed, independently worded: "Values for attributes present at training time but not present in the test data receive missing values. Similarly, missing values are used for incoming [nominal values not seen during training]" — authored by Weka's own core development team, the canonical reference text for the toolkit

---

---

## Card [JV-08]
**Smell Name:** Nulls in Feature Assembly (VectorAssembler)

**Language:** Scala / Java (Spark ML)

**Zhang et al. Mapping:** R7 (Missing Mask of Invalid Value) — *Loose*

**Pipeline Stage:** Data Preparation (Feature Engineering)

**Description:** `VectorAssembler`, Spark ML's transformer for merging multiple numeric columns into a single feature vector, originally threw a bare, unhelpful exception on any null value in an input column with no way to handle it gracefully. A `handleInvalid` parameter (`error`/`skip`/`keep`) was added to let users explicitly choose how to handle nulls — but the `keep` option, which substitutes `NaN` for invalid values rather than dropping the row or erroring, performs no further validation: the resulting `NaN` passes through completely unchecked, silently propagating into training and frequently causing confusing, hard-to-trace failures further downstream.

**Example:** Before Spark 2.4, calling `VectorAssembler().transform(df)` on data containing any null in an assembled column failed with `org.apache.spark.SparkException: Values to assemble cannot be null`, offering no built-in remediation. After the fix, `handleInvalid="keep"` avoids that crash but produces a feature vector containing `NaN` (e.g., `[25.0, NaN]`) with no further check — as one independent reference source puts it: *"VectorAssembler doesn't validate for NaN or null values: they propagate silently into the vector and cause NaN in downstream algorithms"* (failures that, per the resolution discussion itself, most Spark ML algorithms are "unlikely to handle... correctly").

**Fix:** Use `handleInvalid="error"` (the default) to fail fast and loudly on invalid data during development, or `"skip"` to filter out invalid rows explicitly and visibly; avoid `"keep"` unless paired with an explicit downstream `Imputer` or equivalent NaN-handling step, since `"keep"` alone defers the problem rather than resolving it.

**Sources:**

- [SPARK-23690](https://issues.apache.org/jira/browse/SPARK-23690) — confirmed exactly: "VectorAssembler should have handleInvalid to handle columns with null values... Affects Version/s: 2.3.0, Fix Version/s: 2.4.0," resolved via PR #20829
- [Official VectorAssembler API docs (Java/Scala/Python, Spark 4.x)](https://spark.apache.org/docs/latest/api/java/org/apache/spark/ml/feature/VectorAssembler.html) — confirmed, consistent across all three language bindings: `handleInvalid` options `'skip'`/`'error'`/`'keep'`, default `'error'`
- ["In use Spark ml, VectorAssembler use null exception" (Programmer Sought)](https://www.programmersought.com/article/86032031583/) — confirmed, real user bug report: exact production exception text, confirms the pre/post-2.4 distinction, and notes `"keep"` is risky since "ML algorithms are unlikely to handle this correctly"
- [SparkCodeHub, "Vector Assembler"](https://www.sparkcodehub.com/pyspark/mllib/vector-assembler) — confirmed: demonstrates `"keep"` substituting `NaN` for null with a concrete example (`[25.0, NaN]`)
- [theneuralbase.com, "VectorAssembler"](https://theneuralbase.com/pyspark-for-ml/learn/intermediate/vectorassembler/) — confirmed, precisely matches the "NaN still passes unchecked" claim: "VectorAssembler doesn't validate for NaN or null values: they propagate silently into the vector and cause NaN in downstream algorithms"

---

---

## Card [JV-09]
**Smell Name:** Train/Test Schema Mismatch from a Refit Preprocessing Step (Weka)

**Language:** Java (Weka)

**Zhang et al. Mapping:** R11 (Data Leakage) / R15 (Merge API Parameter Not Explicitly Set)-adjacent — schema/statistics mismatch between train and test preprocessing

**Pipeline Stage:** Data Preparation

**Description:** A stateful Weka filter (e.g., `StringToWordVector`, `Standardize`) computes its statistics — a word dictionary, a mean/standard deviation — from whatever dataset it is applied to. Applying such a filter separately and independently to the training set and then again to the test set produces two differently-fitted, structurally incompatible outputs (different vocabularies, different standardization parameters), since each run recomputes its statistics fresh from only the data it sees. The correct approach is to fit the filter once on training data and reuse those exact fitted statistics for the test set, via batch mode or `FilteredClassifier`.

**What Differs from Card JV-07:** JV-07 (`InputMappedClassifier`) addresses schema mismatch between an already-trained *model* and new data at deployment time. This card addresses a different, earlier-stage mistake: a *filter* being refit independently on test data during preprocessing, before a model is even evaluated — the test set effectively gets its own separate "training," leaking its own distribution into what should be a held-out evaluation.

**Example:** Running `StringToWordVector` separately on a training ARFF file and a test ARFF file produces two files with different word-attribute vocabularies (since word occurrences differ between the two datasets), triggering Weka's classic, confusing "training and test set are not compatible" error when the resulting test set is evaluated against a model trained on the training set's vocabulary. The same applies to `Standardize`: computing mean/standard deviation separately on train and test produces two incompatibly-scaled datasets.

**Fix:** Use batch mode (the `-b` command-line flag, which pairs `-i`/`-o` for the fitting dataset with `-r`/`-s` for the dataset to be transformed using those same fitted statistics) or wrap the filter and classifier together in `weka.classifiers.meta.FilteredClassifier`, which automatically fits the filter once on training data and reuses it for test-time transformation.

**Sources:**

- [Official Weka Wiki, "Batch filtering"](https://waikato.github.io/weka-wiki/batch_filtering/) — confirmed exactly, using the identical two example filters (`StringToWordVector`, `Standardize`) and the exact fix (`b` batch mode flag with paired `i`/`o` and `r`/`s` options)
- ["Nihil Obstat: Text Mining in WEKA: Chaining Filters and Classifiers"](http://jmgomezhidalgo.blogspot.com/2013/01/text-mining-in-weka-chaining-filters.html) — confirmed: independently recommends `FilteredClassifier` as "the most simple solution," with a full worked command-line example
- [ResearchGate: "How to change text test files to vector in weka?"](https://www.researchgate.net/post/how_to_change_text_test_files_to_vector_in_weka) — confirmed, real user confusion: diagnosis explicitly names both root causes — mismatched headers between train/test, and applying `StringToWordVector` in the Preprocess panel separately from `FilteredClassifier` in the Classify panel
- ["Text Classification in WEKA" (thekandyancode.wordpress.com)](https://thekandyancode.wordpress.com/2013/02/06/text-classification-in-weka/) — confirmed: names the exact error message ("the dreaded 'training and test set are not compatible' error") and walks through the batch-filtering fix with a full CLI command
- [GitHub gist: Java ](https://gist.github.com/alfredfrancis/54ee66341f4f0ef1f9a88de18b3b76ca)[`FilteredClassifier`](https://gist.github.com/alfredfrancis/54ee66341f4f0ef1f9a88de18b3b76ca)[ usage example](https://gist.github.com/alfredfrancis/54ee66341f4f0ef1f9a88de18b3b76ca) — confirmed: working code demonstrating the correct pattern (`classifier.setFilter(filter)` on a `FilteredClassifier` rather than applying the filter independently)

---

---

## Card [JV-10]
**Smell Name:** Nominal Label Order Differs Between Train and Test (Weka)

**Language:** Java (Weka)

**Zhang et al. Mapping:** R6/R21-adjacent (DataType/schema not explicitly set) — index-based categorical encoding mismatch; Python analogue: `LabelEncoder` fitted separately on train and test

**Pipeline Stage:** Data Preparation / Model Evaluation

**Description:** Weka internally stores every value as a double-precision floating-point number; for nominal (and string) attributes, that number is the *index* of the value within the attribute's declared value list, not the value itself. If the order in which nominal values are declared or discovered differs between training and test data — which happens easily with CSV input, where Weka must infer the nominal value list from the order values first appear — the same index silently refers to a different label in each dataset, and predictions become meaningless without ever raising an error.

**Example:** A nominal attribute with values appearing in order `1,2,3` in the training CSV versus `1,3,2` in the test CSV produces two attributes Weka treats as structurally different, even though a human would consider them the same categories. The official Weka Wiki names this directly: *"it is also not guaranteed that all the labels that appeared in the train set also appear in the test set."* A real, independently confirmed user bug report shows the practical consequence precisely: *"Once assigning value to the attribute, Weka uses this map to assign the index of the value to the vector (not the value itself!!)... we were now classifying wrong vectors (for example for the attribute [A,B], we were trying to set value A but due to the values order change our model thought its B)."* The `ClassOrder` filter's existence — which deliberately reorders class values and ships a `originalValue()` method specifically to map predictions back to the original ordering — is itself confirmation that Weka treats nominal order as mutable and index-tracked, not semantically fixed.

**Fix:** Prefer ARFF format over CSV for train/test data where possible, since ARFF declares the nominal value list explicitly in the header rather than inferring it from data order. When CSV must be used, ensure the nominal value list is declared consistently (e.g., via explicit attribute specification) rather than inferred independently for each file. Reuse the exact same `Attribute` definitions (including value order) across train and test data rather than letting each file's loader infer its own.

**Sources:**

- [Weka Wiki FAQ, "Use csv files"](https://waikato.github.io/weka-wiki/faqs/use_csv_files/) — confirmed exactly: "Train and test set may not be compatible... the order of the appearance of these labels create different nominal attributes ('1,2,3' vs '1,3,2')"
- [Weka Wiki FAQ, "Why am i missing certain nominal or string values from sparse instances"](https://waikato.github.io/weka-wiki/faqs/why_am_i_missing_certain_nominal_or_string_values_from_sparse_instances/) — confirmed exactly: "these numbers are interpreted as indexes into the set of values for the attribute in question, with 0 corresponding to the first value, 1 the second and so forth"
- [Official ](https://weka.sourceforge.io/doc.stable-3-8/weka/core/Instance.html)[`weka.core.Instance`](https://weka.sourceforge.io/doc.stable-3-8/weka/core/Instance.html)[ Javadoc](https://weka.sourceforge.io/doc.stable-3-8/weka/core/Instance.html) — confirmed, independent reinforcement: "If an attribute is nominal... the stored value is the index of the corresponding nominal... value in the attribute's definition"
- [Official ](https://weka.sourceforge.io/doc.dev/weka/filters/supervised/attribute/ClassOrder.html)[`weka.filters.supervised.attribute.ClassOrder`](https://weka.sourceforge.io/doc.dev/weka/filters/supervised/attribute/ClassOrder.html)[ Javadoc](https://weka.sourceforge.io/doc.dev/weka/filters/supervised/attribute/ClassOrder.html) — confirmed: a dedicated filter for reordering class values, shipping `originalValue(double value)` specifically to map randomized/reordered class values back to their original meaning — direct evidence the order is mutable and index-tracked by design

---

---

## Card [JV-11]
**Smell Name:** Unstabilized Input Before a Random Split (Spark `randomSplit`)

**Language:** Scala / Java (Apache Spark)

**Zhang et al. Mapping:** R2 (Randomness Uncontrolled)-adjacent — distributed, lazy-evaluation-specific variant

**Pipeline Stage:** Data Preparation (train/test splitting)

**Description:** Calling `randomSplit()` on a DataFrame that has not been cached or stabilized causes each resulting split to independently re-execute Spark's lazy evaluation plan — re-fetching, re-partitioning, and re-sorting the source data for each split. If data distribution across partitions or row ordering shifts between these independent re-executions (which can happen even with the same seed, due to upstream non-determinism in sources, shuffles, or partition scheduling), the resulting "splits" can silently overlap (the same row appearing in both train and test) or lose rows entirely, and the same script can produce different splits on different runs despite an identical seed.

**Example:** `df.randomSplit(Array(0.8, 0.2), seed=42)` on a non-cached DataFrame internally behaves as `sample()` applied twice, each invocation independently re-executing the DataFrame's full lineage. If the underlying computation is not perfectly deterministic end-to-end (a common situation with distributed data sources, certain file formats, or upstream shuffles), the two resulting DataFrames can overlap or have gaps. Azure Synapse's and Microsoft Fabric's built-in Spark Advisor both flag this automatically: *"Inconsistent or inaccurate results may be returned when working with the results of the 'randomSplit' method. Use Apache Spark (RDD) caching before using the 'randomSplit' method."*

**Fix:** Call `.cache()` (or `.persist()`) on the DataFrame immediately before calling `randomSplit()`, forcing the full computation to run once and be held in memory, so all subsequent splits read from the same stabilized, deterministic cached data rather than re-executing the lineage independently. Note this is not a complete guarantee: per Spark maintainer discussion on a related issue (SPARK-28927), if cached blocks are evicted under memory pressure, the underlying computation can still be silently rerun, reintroducing the same risk — repartitioning on a stable column or applying an aggregation like `groupBy` beforehand can provide additional protection.

**Sources:**

- [SPARK-12662 / PR #10626](https://github.com/apache/spark/pull/10626/files) — confirmed: official, maintainer-fixed historical bug, "Fix DataFrame.randomSplit to avoid creating overlapping splits," with a dedicated regression test for reordered-partition scenarios
- [Azure Synapse Analytics: Apache Spark Advisor documentation](https://learn.microsoft.com/azure/synapse-analytics/monitoring/apache-spark-advisor) — confirmed exactly: dedicated built-in advisory, "May return inconsistent results when using 'randomSplit'... Use Apache Spark (RDD) caching before using the 'randomSplit' method"
- [Microsoft Fabric: Apache Spark advisor documentation](https://learn.microsoft.com/en-us/fabric/data-engineering/spark-advisor-introduction) — confirmed, independent product deployment of the identical advisory
- [Microsoft Research: "Getting Deterministic Results from Spark's randomSplit Function: A Deep Dive"](https://www.microsoft.com/en-us/research/group/dynamics-insights-apps-artificial-intelligence-machine-learning/articles/getting-deterministic-results-from-sparks-randomsplit-function/) — confirmed, detailed technical explanation, directly references the SPARK-12662 fix, and confirms caching as the standard workaround
- [Medium/Udemy Engineering: "Spark Under the Hood: RandomSplit() and Sample() Inconsistencies Examined"](https://medium.com/udemy-engineering/pyspark-under-the-hood-randomsplit-and-sample-inconsistencies-examined-7c6ec62644bc) — confirmed, independent technical deep-dive reaching the identical conclusion
- [SPARK-28927 / PR #25789](https://github.com/apache/spark/pull/25789) — confirmed, related and broader: Spark maintainer discussion explicitly generalizing the risk ("this problem exists everywhere in theory in Spark. Any test/train split basically has this problem") and noting that caching alone does not fully eliminate it if cache blocks are evicted

---

---

## Card [JV-12]
**Smell Name:** Workspace Scope Misuse (Leaked/Outdated Pointer) (ND4J)

**Language:** Java (ND4J / DL4J)

**Zhang et al. Mapping:** No direct equivalent — JVM off-heap workspace scoping is a language/runtime-specific mechanism with no Python parallel

**Pipeline Stage:** Model Training

**Description:** An `INDArray` allocated inside an ND4J memory workspace is only valid while that workspace is open and within the same loop iteration it was created in. Using such an array after its workspace has closed (a "leaked pointer") or carrying a reference to it into a subsequent loop iteration after the workspace has reset its backing memory (an "outdated pointer") throws an `ND4JIllegalStateException`/`ND4JWorkspaceException` at the point of use — a validity rule distinct from both Card JV-03 (memory not being freed) and Card JV-04 (silent data corruption): this is an explicit, loud failure rather than a silent one, but it's a correctness trap specific to manual off-heap scope management with no equivalent in garbage-collected, non-workspace-based ML frameworks.

**Example:** `INDArray leaked; try (ws = ...) { leaked = Nd4j.rand(...); } leaked.addi(1.0f);` throws *"Op [set] Y argument uses leaked workspace pointer from workspace [LOOP_EXTERNAL]"* once the array is touched outside the workspace block. A confirmed real-world case shows this occurring during actual training with a UI listener attached: `MultiLayerNetwork.fit()` triggered `"Cannot detach INDArray: Array uses leaked workspace pointer from workspace WS_ALL_LAYERS_ACT"` deep inside `ConvolutionalIterationListener`. A second, maintainer-confirmed case shows the "outdated pointer" variant: a SameDiff loss function executing inside an external DL4J-managed workspace threw `"Op [add] X argument uses outdated workspace pointer from workspace [WS_LAYER_WORKING_MEM]"`, traced by the maintainer to SameDiff not being properly configured to execute correctly within an externally-managed workspace context.

**Fix:** Call `.detach()` to copy an array out of its workspace into independent, workspace-free off-heap memory when it needs to outlive the workspace or survive into a later iteration; use `.leverageTo(name)` to move an array to a specific named (typically parent or ancestor) workspace instead, which falls back to `.detach()` if that named workspace isn't currently active on the calling thread; use `.migrate()` to move an array into whichever workspace is currently active.

**Sources:**

- [Official "Workspaces" documentation](https://deeplearning4j.konduit.ai/nd4j/workspaces) — confirmed, detailed: dedicated sections for both "Leaked Workspace Pointer" and "Outdated Workspace Pointer," with exact exception text and fix code for `.detach()`/`.leverageTo()`
- [Official "Memory and Workspaces" documentation](https://deeplearning4j.konduit.ai/en-1.0.0-beta7/core-concepts/memory-and-workspaces) — confirmed, independent reinforcement: "Scope Panic: Leaked and Outdated Workspace Pointers" section, explicitly noting ND4J validates array pointers at op execution time "to catch these errors early rather than producing silent data corruption" (i.e., deliberately loud rather than silent, by design)
- [GitHub gist: "Leaked workspace pointer in TestConvolutionalListener.testUI()"](https://gist.github.com/printomi/c227d4ebecbcb96f3f103857ffb856ef) — confirmed, real occurrence: full stack trace showing the leaked-pointer exception thrown during actual `MultiLayerNetwork.fit()` execution with a UI listener attached, not a synthetic example
- [deeplearning4j/deeplearning4j #7120](https://github.com/deeplearning4j/deeplearning4j/issues/7120) — confirmed, maintainer-engaged real bug: outdated-pointer exception during SameDiff inference inside a DL4J-managed external workspace, maintainer-diagnosed root cause ("SameDiff wasn't properly set up to work when executing in an external workspace")

---

---

## Card [JV-13]
**Smell Name:** Shared Non-Thread-Safe Inference Handle (DJL `Predictor`)

**Language:** Java (Deep Java Library, DJL)

**Zhang et al. Mapping:** No direct Zhang et al. equivalent — JVM-specific multi-threaded inference handle concern

**Pipeline Stage:** Model Deployment / Inference

**Description:** DJL's `Predictor` object, used to run inference against a loaded `Model`, is explicitly not designed to be thread-safe, because some of DJL's underlying engines are themselves not thread-safe and `Predictor` does not add its own synchronization layer to cover this gap. Sharing a single `Predictor` instance across multiple threads for concurrent inference is unsupported and can produce incorrect results or crashes; the documented pattern is one `Predictor` per thread (via `ThreadLocal` or a pool) built from a single shared, thread-safe `Model`.

**Example:** A multi-threaded inference service that constructs one `Predictor` and reuses it across worker threads violates DJL's documented threading contract. The correct pattern, per DJL's own architecture: `Model` is thread-safe and loaded once per application; `Predictor` is not thread-safe and must be one per thread; `NDManager` is thread-safe and hierarchical.

**Fix:** Create a new `Predictor` per thread (reused across that thread's own multiple predictions, not recreated per call), or use a pool of `Predictor` instances, or use DJL Serving's `WorkLoadManager` which handles this pattern automatically.

**Sources:**

- [Official DJL "Inference Performance Optimization" documentation](https://docs.djl.ai/docs/development/inference_performance_optimization.html) — confirmed exactly: "The DJL Predictor is not designed to be thread-safe... we recommend creating a new Predictor for each thread... Alternatively, you can also use a pool of Predictors"
- [DeepWiki: "Model Loading and Inference"](https://deepwiki.com/deepjavalibrary/djl/3.2-model-loading-and-inference) — confirmed, independent rendering of the same underlying source code: explicit table confirming `Predictor | Not thread-safe | One per thread (ThreadLocal or Pool)`
- [deepjavalibrary/djl #2778](https://github.com/deepjavalibrary/djl/issues/2778) — confirmed real-world occurrence, but an imperfect match: a genuine production multi-threaded GPU inference problem (request queue buildup, infinite latency increase under concurrent load), not a clean demonstration of corrupted/wrong predictions from directly sharing one `Predictor` unsafely

---

---

## Card [CPP-01]
**Smell Name:** Memory Not Freed — Multiple Independent Scenarios (libtorch)

**Language:** C++ (libtorch)

**Zhang et al. Mapping:** R10 (Memory Not Freed) — *Direct analog, substantively amplified*

**Pipeline Stage:** Model Training / Deployment

**Description:** libtorch's C++ API leaks memory in multiple distinct, independently-confirmed scenarios, even in cases where equivalent Python code would not: `torch::load()` repeatedly applied to an optimizer, `.forward()` calls executed in a separate thread, tensor storage inside nested C++ containers, and model evaluation on CPU (even with gradients explicitly disabled). C++'s manual memory model makes this smell both more severe and more frequent than in Python, where garbage collection papers over many of the same underlying reference-counting mistakes.

**What Differs from the Python Version:** Zhang et al.'s R10 concerns forgotten `session.close()`/explicit resource-release calls in Python ML code — a single, well-defined failure mode. The C++ manifestation is structurally broader: because there is no garbage collector to fall back on, reference-counting bugs in libtorch's own internals (not just user code) surface as real, user-visible leaks across at least five independently-reported, mechanically distinct scenarios, several confirmed by PyTorch's own triage process as "high priority."

**Example:** Repeatedly calling `torch::load(optimizer, path)` in a loop causes unbounded memory growth, with a maintainer noting "torch::load should not leak it... maybe it should release previous optimizer states" (#15792). Separately, running `.forward()` in a dedicated thread leaks memory that is never reclaimed at thread exit, while the same call on the main thread correctly reuses memory (#24237).

**Fix:** No single general fix exists across all five scenarios; each was patched independently by PyTorch maintainers as its own distinct bug. General mitigations where available: avoid repeated `torch::load()` calls on long-lived optimizer objects without explicit state clearing; avoid running inference in short-lived threads where possible; flatten nested tensor-containing C++ containers where feasible; and be aware that `torch::NoGradGuard` does not guarantee leak-free CPU evaluation on its own.

**Sources:**

- [pytorch/pytorch #15792](https://github.com/pytorch/pytorch/issues/15792) — confirmed: `torch::load()` on an optimizer leaks memory in a loop (CPU and GPU), maintainer acknowledges prior optimizer state should be released
- [pytorch/pytorch #24237](https://github.com/pytorch/pytorch/issues/24237) — confirmed: `.forward()` in a separate thread leaks memory never reclaimed at thread exit; main-thread calls correctly reuse memory
- [pytorch/pytorch #57457](https://github.com/pytorch/pytorch/issues/57457) — confirmed: Valgrind-detected leak (4,956,000 bytes in 6 blocks "possibly lost") even with `torch::NoGradGuard` explicitly disabling gradient tracking
- [pytorch/pytorch #143564](https://github.com/pytorch/pytorch/issues/143564) — confirmed: tensors stored inside nested C++ containers (e.g. `std::vector<std::vector<torch::Tensor>>`) fail to be released; reporter diagnoses an internal reference-count tracking failure
- [pytorch/pytorch #40973](https://github.com/pytorch/pytorch/issues/40973) — confirmed, tagged "high priority" by PyTorch's own triage: unbounded memory growth evaluating a model on CPU, expected to plateau but does not
- [PyTorch Forums: "Memory leaks in libtorch?"](https://discuss.pytorch.org/t/memory-leaks-in-libtorch/84093) — confirmed: a user systematically Valgrind-testing libtorch finds leaks beyond thread-cleanup noise, asking whether libtorch has undergone "careful memory-leak vetting"
- [PyTorch Forums: "Memory leak in LibTorch, extremely simple code"](https://discuss.pytorch.org/t/memory-leak-in-libtorch-extremely-simple-code/38149) — confirmed: a sixth independent report, explicitly confirming the CPU-specific leak persisted even with `torch::NoGradGuard` applied ("I tried [torch::NoGradGuard no_grad_guard;] but its of no use")

---

---

## Card [CPP-02]
**Smell Name:** Ignored Return Value of Device-Transfer Call

**Language:** C++ (libtorch)

**Zhang et al. Mapping:** No direct Python equivalent in severity — same root mistake as a general pattern, but C++'s stricter memory model amplifies the consequence from silent inefficiency to fatal error

**Pipeline Stage:** Model Training / Deployment

**Description:** `tensor.to(torch::kCUDA)` returns a new tensor rather than mutating in place, identical to Python's `.to()` semantics. In Python, forgetting to reassign the result (`a.to('cuda')` instead of `a = a.to('cuda')`) silently leaves the original tensor on CPU — slower, since CPU/GPU operations can still interoperate loosely at the Python/NumPy level, but the program still runs. In C++, the same mistake causes a hard crash on the next operation, because libtorch enforces strict memory-space separation between CPU and GPU tensors that Python's more permissive runtime does not.

**Example:** A user attempts `tensor2.to(torch::kCUDA);` without reassigning, expecting this to move the tensor to the GPU in place. The original CPU-resident tensor is unaffected, and the next operation expecting a GPU tensor crashes. Confirmed directly by a user's own account: *"I tried creating the tensors CPU-side... and then do a **`tensor.to(torch::kCUDA)`** but it didn't transfer the tensors to CUDA"* — resolved only once they corrected it to `tensor2 = tensor2.to(torch::kCUDA);`.

**Fix:** Always reassign the result of `.to()`: `tensor = tensor.to(device)`, never call it as a bare statement expecting in-place mutation.

**Sources:**

- [pytorch/pytorch #30076](https://github.com/pytorch/pytorch/issues/30076) — confirmed: real user hitting a CUDA-related crash in LibTorch, with the official documentation excerpt quoted directly in the issue confirming `.to()` returns a new tensor
- [PyTorch Forums: "Crash when using torch::from_blob with CUDA"](https://discuss.pytorch.org/t/crash-when-using-torch-from-blob-with-cuda/71298) — confirmed, cleaner direct example: a user explicitly describes making this exact mistake and self-correcting once they realized the return value needed reassignment

---

---

## Card [CPP-04]
**Smell Name:** Train/Eval Mode Not Toggled (libtorch)

**Language:** C++ (libtorch)

**Zhang et al. Mapping:** R4 (Training/Evaluation Mode Improper Toggling) — *Direct analog*

**Pipeline Stage:** Model Training / Evaluation

**Description:** Layers like `Dropout` and `BatchNorm` behave differently during training and evaluation (dropout active vs. disabled; batch statistics vs. stored running statistics), and libtorch requires this to be toggled explicitly by calling `module->eval()` / `module->train()` by hand — there is no automatic mode-switching tied to context (e.g., no automatic eval mode during an inference call).

**Example:** Running inference on a trained model without first calling `model->eval()` leaves `Dropout` active (randomly zeroing activations) and leaves `BatchNorm` normalizing against the current input batch's statistics rather than the learned running statistics — producing, in a real user's words from the broader PyTorch ecosystem, output that is "completely wrong." The official LibTorch FAQ treats this as common enough to warrant a dedicated entry: "Problem: Layers like Dropout and BatchNorm behave differently during training and evaluation, and you need to switch between modes. Fix: Use the `eval()` and `train()` methods."

**Fix:** Call `model->eval()` before any inference/evaluation pass, and `model->train()` before resuming training — explicitly, at the start of each phase, not once at the top of a script, since switching between training and evaluation phases (e.g., per-epoch validation) requires re-toggling each time.

**Sources:**

- [LibTorch C++ FAQ](https://docs.pytorch.org/cppdocs/faq.html) — confirmed exactly: dedicated "How do I set a module to evaluation mode?" entry under the LibTorch section, with the exact problem statement and fix (`model->eval()` / `model->train()`) as described
- [torch::nn Module reference documentation](https://docs.pytorch.org/cppdocs/api/nn/index.html) — confirmed: "Every Module has a boolean associated with it that determines whether the Module is currently in training mode... See the BatchNorm or Dropout modules for examples of Modules that use different code paths depending on this property"
- [Dropout layer documentation](https://docs.pytorch.org/cppdocs/api/nn/dropout.html) — confirmed: "Remember to call `model->train()` during training and `model->eval()` during inference to properly enable/disable dropout behavior"

---

---

## Card [CPP-05]
**Smell Name:** Static Initialization/Destruction Order Fiasco Between CUDA Context and Autograd Engine

**Language:** C++ (libtorch)

**Zhang et al. Mapping:** No Python equivalent — genuinely novel, rooted in a C++-specific language feature (static object destruction ordering across translation units/shared libraries) with no analog in garbage-collected Python

**Pipeline Stage:** Model Training

**Description:** Running CPU-only autograd code (no CUDA usage at all) in a CUDA-enabled libtorch build can crash at program exit with `"CUDA error: driver shutting down"`, because CUDA context initialization happens lazily inside the autograd engine's worker thread, and that thread's destructor can run after the CUDA driver has already begun process-wide teardown — a classic C++ static destruction-order race that only manifests when libtorch is linked into an external program (not from within PyTorch's own internal test suite, where the linking and initialization order happen to avoid the race).

**What Makes This Novel and C++-Specific:** This requires C++'s specific static object lifetime and destruction-order semantics across shared libraries — a category of bug that is definitionally impossible in Python, which has no equivalent concept of ordered static destructors firing during process teardown across separately-compiled translation units. The bug is also specific to libtorch's architecture: a background autograd engine thread that lazily touches CUDA state, racing against the CUDA driver's own shutdown sequence.

**Example:**

```
auto x = torch::randn({2}).requires_grad_();
auto y = MulConstant::apply(x, 5.5);  // simple custom autograd function
y.sum().backward();
```
run in an external C++ program linked against CUDA-enabled libtorch crashes with `terminate called after throwing an instance of 'c10::Error' ... CUDA error: driver shutting down` at `CUDAGuardImpl::setDevice`, inside `torch::autograd::Engine::thread_init`. The same code in a CPU-only libtorch build, or with an explicit early CUDA touch added, runs cleanly. Confirmed independently by multiple users across libtorch 1.3 (original 2020 report), 1.6.0, and 1.9.0+cuda10.2, and separately reproduced via Rust bindings (`tch-rs`) to the same underlying C++ API, ruling out a binding-layer cause.

**Fix:** Force CUDA context initialization early and deterministically, before any autograd computation runs — e.g., call `torch::cuda::is_available()` as the first line of `main()`, or explicitly create and use any CUDA tensor before running CPU-only autograd code. Both workarounds were independently discovered by different users and confirmed to eliminate the crash, by changing the order in which CUDA-related static state gets initialized relative to the autograd engine's thread lifecycle.

**Sources:**

- [pytorch/pytorch #35736](https://github.com/pytorch/pytorch/issues/35736) — confirmed, exceptionally detailed: original report plus independent reproductions in libtorch 1.6.0 and 1.9.0+cuda10.2, a separate independent reproduction via Rust `tch-rs` bindings, maintainer (ezyang) root-cause diagnosis naming the static destruction-order race explicitly, confirmation the bug only reproduces when linked externally (not from PyTorch's own test suite), and two independently-discovered, confirmed workarounds

---

---

## Card [CPP-06]
**Smell Name:** In-Place Operation on a Leaf Tensor Breaks Autograd (LBFGS Optimizer)

**Language:** C++ (libtorch)

**Zhang et al. Mapping:** R16 (In-Place APIs Misused) — *Direct analog*

**Pipeline Stage:** Model Training

**Description:** Autograd forbids in-place modification of a leaf tensor that requires gradients, since doing so would corrupt the value autograd needs for backpropagation; PyTorch raises `"a leaf Variable that requires grad has been used in an in-place operation"` when this happens. In the C++ API, this surfaced inside the `LBFGS` optimizer's own internal implementation — its `add_grad()` helper calls an in-place `add_()` on a tensor that can end up being a leaf requiring grad, something the equivalent Python-side `SGD` optimizer path for the same user code did not trigger.

**Example:** A user's training code ran correctly in Python with the `SGD` optimizer and also compiled and "worked" when ported to C++ with `SGD`, but switching only the optimizer to `LBFGS` in the C++ version crashed: *"My code is working fine in python and with SGD, but I get terminating with uncaught exception of type c10::Error: a leaf Variable that requires grad has been used in an in-place operation"* — with the crash traced precisely to `torch::optim::LBFGS::step()` → `add_grad()` → `VariableType::add_()` → `check_inplace()`.

**Fix:** Avoid using optimizers or custom training code paths that perform in-place operations directly on leaf tensors requiring grad; where a specific optimizer's C++ implementation has this issue, consider cloning/detaching the tensor before the operation that triggers it, or track whether a fix has since landed for the specific optimizer affected.

**Sources:**

- [pytorch/pytorch #15923, "Cannot use LBFGS in C++"](https://github.com/pytorch/pytorch/issues/15923) — confirmed, detailed: precise stack trace through `LBFGS::step()`/`add_grad()`/`VariableType::add_()`/`check_inplace()`, with a clear working-in-Python-and-with-SGD-but-not-C++-LBFGS contrast from the reporter's own account

---

---

## Card [CPP-07]
**Smell Name:** `torch::manual_seed()` Alone Insufficient for Reproducibility (libtorch)

**Language:** C++ (libtorch)

**Zhang et al. Mapping:** R2 (Randomness Uncontrolled) — *Direct analog*

**Pipeline Stage:** Model Training

**Description:** Calling `torch::manual_seed()` alone does not guarantee reproducible training in libtorch, particularly once cuDNN-accelerated operations (e.g., convolutions) are involved. Full determinism additionally requires `at::globalContext().setDeterministicCuDNN(true)` and `at::globalContext().setDeterministicAlgorithms(true, true)` — a less-discoverable, C++-specific API surface distinct from Python's more commonly-documented `torch.backends.cudnn.deterministic`/`torch.use_deterministic_algorithms()` pattern. Compounding this, the C++ API has at times lacked an obvious direct equivalent to Python's `torch.cuda.manual_seed_all()`, leaving C++ users without a clearly-documented, discoverable path to fully seed CUDA RNG across all devices.

**Example:** A user training a DDPG reinforcement learning model called `srand(seed); torch::manual_seed(seed);` at the start of `main()` and still observed different results on every run: *"I still get different result every time."* `torch::manual_seed()` alone left additional, less-discoverable determinism switches — `at::globalContext().setDeterministicCuDNN(...)` and `at::globalContext().setDeterministicAlgorithms(..., true)` — unset, and the user's own report notes that `setDeterministicCuDNN()` did not resolve their case; full non-determinism in this instance remained unexplained in the thread.

**Fix:** `torch::manual_seed()` does not cover cuDNN-accelerated operations, which need `at::globalContext().setDeterministicCuDNN(true)` and `at::globalContext().setDeterministicAlgorithms(true, true)` set explicitly; verify no other source of randomness (e.g., raw C library `rand()`/`srand()` used for exploration or sampling, as in the confirmed case) is seeded separately from libtorch's own RNG. Note that even these additional calls are not guaranteed to resolve every non-determinism case — the user report underlying this card's Example is itself unresolved on that point.

**Sources:**

- [pytorch/pytorch #40361, "C++ API: add torch::cuda::manual_seed like torch.cuda.manual_seed"](https://github.com/pytorch/pytorch/issues/40361) — confirmed, real user question exposing a C++/Python API parity gap in CUDA-wide seeding

---

---

## Card [CPP-09]
**Smell Name:** Silent CUDA Context Overwrite When Embedding libtorch

**Language:** C++ (libtorch)

**Zhang et al. Mapping:** No Python equivalent — novel smell specific to embedding libtorch's device management within a larger, independently CUDA-aware C++ application

**Pipeline Stage:** Model Deployment / Inference

**Description:** When a C++ application has already created and pushed its own valid CUDA context before invoking libtorch (e.g., calling `module.to(at::kCUDA)` to move a loaded model to GPU), libtorch does not detect or reuse the existing context. Instead, it silently creates a new CUDA context and overwrites the current one on the context stack, rather than pushing it as a properly nested context. This corrupts the host application's own CUDA state with no error, warning, or exception raised at any point.

**Example:** An application performs its own CUDA operations and establishes a valid CUDA context before loading a model: `torch::jit::script::Module module = torch::jit::load("test.pt"); module.to(at::kCUDA);`. The reporter confirms the corruption directly: *"The new context is not even push[ed] to the stack of contexts, is overwriting the current context. I know because if after the **`module.to(at::kCUDA)`** I call **`cuCtxPopCurrent`** the current context is null... This causes a lot of problems because I cannot interact with current allocated memory I have in my context."* The issue was triaged and labeled by the PyTorch team (`oncall: jit`) but the underlying behavior described was not resolved in the source thread.

**Fix:** No general fix confirmed in the source thread. Practical mitigation: where possible, let libtorch be the sole owner/initializer of the CUDA context within a process rather than embedding it into an application that independently manages its own CUDA context; if integration with existing CUDA state is required, carefully audit context stack state before and after every libtorch device-transfer call using the CUDA driver API directly (`cuCtxPushCurrent`/`cuCtxPopCurrent`) to detect and correct unexpected overwrites.

**Sources:**

- [pytorch/pytorch #31565, "Force libtorch to use CUDA context"](https://github.com/pytorch/pytorch/issues/31565) — confirmed, officially triaged by the PyTorch team (`oncall: jit`, `triaged` labels): precise technical diagnosis of the context-stack corruption via `cuCtxPopCurrent` verification, directly from the reporter's own debugging

---

---

