# Discussion-Only Smells (4 cards)

Real, well-evidenced findings moved out of the main catalogue during inter-rater adjudication (2026-10-18) because they fail the ML-specificity litmus test, document the absence of a smell rather than a defect, or read as versioning debt rather than code debt. Referenced in the paper's Discussion section, not Table 2.

---

## Card [R-16]
**Smell Name:** Saved Preprocessing Pipeline Silently Breaks After Package Upgrade

**Language:** R

**Zhang et al. Mapping:** None — *Novel smell, no Python catalogue equivalent*

**Pipeline Stage:** Deployment / Inference

**Why Discussion-Only, Not Catalogued:** Rater split (discussion/exclude/discussion). A well-evidenced Novel smell, but no general fix exists at the package level -- only a practitioner-side mitigation (re-validate output shape after upgrades, pin versions) -- which was decisive in routing it here rather than into the main catalogue.

**Description:** A fitted preprocessing pipeline (a `recipes` "recipe" including a PCA step) is saved to disk and reused for scoring/inference over an extended period. When the package that created it is later upgraded, internal changes to the object's expected structure can cause a saved pipeline step to silently become a no-op — no error, no warning — because the new code checks for an internal field that didn't exist when the object was originally serialized. The pipeline still runs and produces output; it just silently stops doing what it was trained to do.

**What Makes This ML-Specific (Novel Smell Justification):** Unlike generic serialization-compatibility problems (which are a general software engineering concern, not ML-exclusive), this smell requires the concept of a *fitted, saved preprocessing pipeline reused for consistent inference* — a concept with no meaning outside ML/data-science workflows. The harm is specific to the ML deployment pattern of "train once, serialize, apply repeatedly to new data," where structural package changes between training time and inference time can silently desynchronize the saved artifact from the code now reading it.

**Example:** A recipe with `step_pca(all_predictors(), threshold = 0.65)` is trained and saved via `saveRDS()` under `recipes` 0.1.17. Used in production scoring for months. After upgrading to `recipes` 0.2.0 (which added an internal `columns` field to `step_pca()` to support an unrelated feature), loading and applying the old saved recipe silently skips the PCA step entirely — `bake()` returns data with the original (pre-PCA) column count, not the reduced PCA components, with no indication anything went wrong.

**Fix:** No general fix exists at the package level for consumers; the specific case was manually repairable by patching the loaded object's internal structure to add the missing field (requires deep knowledge of the package's internals, not practical for most users). More robust general practice: re-validate a saved pipeline's output shape/structure against an expectation after every package upgrade, rather than trusting silent successful execution; pin package versions for production scoring pipelines; or retrain and re-save pipelines after any upgrade of a preprocessing package.

**Sources:**

- [recipes issue #921](https://github.com/tidymodels/recipes/issues/921) — confirmed, exceptionally strong: production pipeline silently broke after a package upgrade, explicitly confirmed "no error or warning" by the reporter; independently reproduced by a second user; root cause precisely diagnosed by maintainer Emil Hvitfeldt (a new internal field absent in pre-upgrade serialized objects caused a silent no-op check); core developer Julia Silge confirmed this broke an explicit backward-compatibility guarantee the team had been actively maintaining
- [parsnip issue #1413](https://github.com/tidymodels/parsnip/issues/1413) — confirmed, strong, directly on-topic: an update to an upstream dependency (LiblineaR, not parsnip/recipes themselves) caused a model's underlying fit to become "materially different, not just numerically noisier" — caught only by CRAN's own automated reverse-dependency testing, not by any end user
- [bundle package documentation](https://rdocumentation.org/packages/bundle) — confirmed, exceptionally strong corroboration: an entire dedicated R package built specifically because saved model/pipeline objects (parsnip, recipes, keras, caret, xgboost, h2o, luz, tidymodels workflows) don't reliably survive being saved and reloaded in production; its stated purpose is to "prepare model objects so that they can be effectively saved and re-loaded for use in new R sessions" — the same "ecosystem built dedicated tooling to catch this" signal that strengthened several of the catalogue's strongest cards

---

---

## Card [R-19]
**Smell Name:** Train/Eval Mode API Evolution (R torch)

**Language:** R (torch package)

**Zhang et al. Mapping:** R4 (Improper Train/Eval Mode Toggling) — *Direct analog*

**Pipeline Stage:** Model Training

**Why Discussion-Only, Not Catalogued:** Rater split (exclude/discussion/discussion). Treated as versioning debt -- an API return-type change between package releases -- rather than a structural code-level defect in user code; borderline whether it qualifies as a code smell at all.

**Description:** R's `torch` package mirrors PyTorch's `nn_module$eval()`/`nn_module$train()` mode-toggling methods, but the R binding has had its own independent history of API instability around these methods — including a documented breaking change to their return type.

**Example:** In `torch` v0.2.1, a breaking change was introduced: `nn_module$eval()` and `nn_module$train()` changed from returning a plain `nn_Module` to returning a callable `nn_module`, a type change affecting any code chaining off the return value of these calls.

**Fix:** Pin `torch` package versions for production training code where exact mode-toggling return behavior matters; check the package changelog when upgrading across versions that span this change.

**Sources:**

- [torch NEWS.md / GitHub release v0.2.1](https://github.com/mlverse/torch/releases/tag/v0.2.1) — confirmed exactly: "`nn_module$eval()` and `nn_module$train()` now return a callable `nn_module` instead of a `nn_Module`. (#425)"

---

---

## Card [JL-01]
**Smell Name:** Design-Level Data Leakage Prevention (confirms R11 transfers to Julia)

**Language:** Julia (MLJ.jl)

**Zhang et al. Mapping:** R11 (Data Leakage) — *Direct analog*

**Pipeline Stage:** Data Preparation

**Why Discussion-Only, Not Catalogued:** Rater split (discussion/include/discussion). Documents the absence of a smell -- a proactive design choice that prevents a Zhang et al. R11-type risk -- rather than a defect a developer could introduce; kept here for its severity-confirming value rather than as a catalogued smell.

**Description:** MLJ.jl's own architecture explicitly separates a "prefit" stage (hyperparameters wrapped around the full dataset, but not told which rows are training data) from a final "fit" stage that declares which rows to actually use — a design decision made specifically to prevent data leakage during cross-validation with dynamic, look-through training.

**What This Confirms:** Unlike the R cards in this catalogue, this is not a bug report or community confusion — it's evidence that leakage risk is significant enough that a non-Python ML framework's core maintainers deliberately architected around it from the design stage, rather than patching it after the fact. This is a different, complementary kind of evidence from reactive bug fixes: proactive design prevention, confirming R11 is recognized as a serious enough risk to shape a framework's fundamental training API before any bugs were ever reported.

**Example:** `scale = prefit(scale_, X)` wraps a standardizing transformer around the full dataset `X` without yet specifying which rows belong to the training fold. Only when `fit!(scale, fold1)` is called is the transformer actually trained, and only on `fold1`'s rows — with the maintainer's own code comment stating directly: "construct a transformer to standardize the inputs, using the training fold to prevent data leakage."

**Fix:** Not applicable in the traditional sense — this card documents a design pattern that *is* the fix (built into MLJ.jl's architecture), rather than a smell requiring a fix. Included in the catalogue as confirmation that the underlying risk (R11) was taken seriously enough to shape fundamental API design in a second, independent, non-Python ML framework.

**Sources:**

- [MLJ.jl issue #14, "Learning networks and dynamic data"](https://github.com/JuliaAI/MLJ.jl/issues/14) — confirmed exactly: maintainer's own code comment states "construct a transformer to standardize the inputs, using the training fold to prevent data leakage," with detailed explanation of the prefit/fit two-phase design specifically intended to prevent leakage during cross-validation (note: repository has since moved from the `alan-turing-institute` org to `JuliaAI`; same issue, same number, under the current canonical URL)

---

---

## Card [CPP-08]
**Smell Name:** Silent Numerical Divergence Between C++ and Python TorchScript Runtimes

**Language:** C++ (libtorch)

**Zhang et al. Mapping:** R6 (No Single Deterministic-Algorithm Switch) — *Variant, novel manifestation*

**Pipeline Stage:** Model Deployment / Inference

**Why Discussion-Only, Not Catalogued:** Rater split (discussion/discussion/exclude). No general fix confirmed at the framework level, and evidence rests on a single (if detailed) PyTorch Forums report rather than independently corroborated sources -- both counted against full catalogue inclusion.

**Description:** Loading and running the identical scripted TorchScript model (`torch::jit::load` in C++ vs. `torch.jit.load` in Python) on the identical input can produce silently different numerical outputs between the two runtimes, with no error or warning raised during saving, loading, or inference. The divergence is not confined to model-specific computation — it was confirmed to appear even in basic tensor creation (`torch::ones` vs. `torch.ones`), suggesting the root cause sits at the level of the C++ and Python runtime environments themselves (library versions, build configurations, or low-level numerical kernel selection) rather than anything specific to the model being deployed.

**Example:** A Faster R-CNN object detection model trained and scripted in Python (`torch.jit.script(model)`) produces high-confidence, correctly-labeled predictions when reloaded and run in Python via `torch.jit.load`. The same `.pt` file, loaded in C++ via `torch::jit::load` and run on the same image, produces a *different* result — with the reporter noting plainly: "I am not sure what is going on as there was no error in saving and loading the torchscripted models." Follow-up testing confirmed `torch.ones`/`torch::ones` themselves returned different raw values between the two runtimes for the same shape and dtype.

**Fix:** No general fix confirmed in the source thread; practical mitigation is to pin exact matching library versions and build configurations between the Python environment used for training/scripting and the C++ environment used for deployment, and to add explicit numerical consistency checks (comparing a known reference input/output pair) as part of any C++ deployment pipeline rather than assuming identical TorchScript execution across languages.

**Sources:**

- [PyTorch Forums: "Torch::jit::load in c++ and Torch.jit.load in python for object detection models give dissimilar results"](https://discuss.pytorch.org/t/torch-load-in-c-and-torch-jit-load-in-python-for-object-detection-models-give-dissimilar-results/126568) — confirmed, detailed: real user report with a specific model (scripted Faster R-CNN), explicit confirmation no error occurred anywhere in the pipeline, and a critical follow-up finding that even basic tensor creation diverges between the two runtimes for the same code

---

---

