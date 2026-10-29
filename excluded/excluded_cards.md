# Excluded Cards (9 cards)

Cards considered during drafting and rating but excluded from the catalogue entirely (not Discussion-Only). Two groups:

1. **Excluded after the 2026-10-18 inter-rater adjudication** (R-20, CPP-03) — both were full catalogue entries that 2-of-3 raters, including the catalogue author's own vote, judged should be dropped. See `methodology/slr_and_adjudication.md` for the exact rationale recorded for each.
2. **Excluded during earlier drafting** (JL-05, R-23, R-24, R-25, R-26, JV-14, CPP-10) — identified before the rating pass as failing a specific catalogue-inclusion test (ML-specificity, evidence tier, overclaiming); rationale is given inline in each card below, exactly as recorded in the working tab.

Note: the rating CSV in `ratings/ML_Smell_Catalog_Rated.csv` also records that all three raters unanimously voted *exclude* on JL-05, R-23, and R-24, even though at rating time they were still sitting in the project's Discussion-Only tab rather than fully excluded. This is flagged as an open, unresolved inconsistency in the Notes tab and reproduced verbatim in the adjudication file in `methodology/`; it was not resolved as part of this package.

---

## Group 1: excluded after inter-rater adjudication (2026-10-18)

## Card [R-20]
**Smell Name:** Tensor-to-R-Object Conversion Bugs (R torch)

**Language:** R (torch package)

**Zhang et al. Mapping:** R14 (DataFrame Conversion API Misused) — *Direct analog, distinct mechanism from Card R-12*

**Pipeline Stage:** Data Preparation / Model Evaluation

**Why Excluded:** 2/3 raters, including the catalogue author's own deciding vote, voted exclude (exclude/exclude/include). No further substantive rationale beyond the vote itself is recorded in the source material -- this is flagged here honestly as a gap rather than papered over with an invented justification.

**Description:** R's `torch` package needed a dedicated `as.matrix()` method added for tensors (it did not exist from the start), and separately had a confirmed bug where `as.array()` did not correctly move a GPU-resident tensor to CPU before copying its data into a native R object. This is a direct parallel to pandas' `.values()`/`.to_numpy()` ambiguity (Zhang et al.'s R14), but manifesting as a cross-boundary (GPU tensor → R native object) correctness bug rather than a copy-vs-view ambiguity.

**What Differs from Card R-12:** R-12 concerns base R's `as.matrix()` silently coercing a data.frame's numeric columns to character when any column is non-numeric — a pure data-type coercion bug with no device/memory-location dimension. This card's mechanism is specific to GPU-accelerated tensor libraries: the conversion function needs to know to move data off the GPU device before the R-side copy can succeed correctly, a concern that doesn't exist for base R's `as.matrix()` at all. Distinct package, distinct root cause, same Zhang et al. mapping.

**Example:** Prior to `torch` v0.12.0, calling `as.array()` on a tensor resident on a CUDA device did not first move it to CPU, risking incorrect or failed conversion. The fix explicitly added this device-transfer step: "`as.array` now moves tensors to the cpu before copying data into R." Separately, `as.matrix()` support for tensors did not exist at all until v0.1.1, meaning earlier versions of the package offered no direct path from a tensor to a native R matrix.

**Fix:** Both issues were fixed at the package level (v0.1.1 added `as.matrix()` support; v0.12.0 fixed the GPU-to-CPU transfer in `as.array()`). For users on older package versions, explicitly call `$cpu()` on a tensor before conversion rather than relying on the conversion function to handle device transfer implicitly.

**Sources:**

- [torch NEWS.md / changelog, v0.1.1](https://torch.mlverse.org/docs/news/) — confirmed: "Added an `as.matrix` method for torch Tensors. (#282)"
- [torch NEWS.md / changelog, v0.12.0](https://torch.mlverse.org/docs/news/) — confirmed: "`as.array` now moves tensors to the cpu before copying data into R. (#1080)"

---

---

## Card [CPP-03]
**Smell Name:** Dual-ABI Silent Binary Incompatibility (libtorch)

**Language:** C++ (libtorch)

**Zhang et al. Mapping:** No Python equivalent — Python's packaging/ABI model does not expose end users to this kind of binary-layout mismatch

**Pipeline Stage:** Deployment

**Why Excluded:** 2/3 raters, including the catalogue author's own deciding vote, voted exclude (exclude/exclude/include). No further substantive rationale beyond the vote itself is recorded in the source material -- this is flagged here honestly as a gap rather than papered over with an invented justification.

**Description:** libtorch ships in two mutually incompatible binary builds — a legacy "pre-CXX11 ABI" (for compatibility with older systems like CentOS 7) and the modern "CXX11 ABI." Linking the wrong one against a project's own compiler settings causes memory corruption, cryptic crashes, or (in some cases) silently wrong numerical results, because fundamental types like `std::string` have a genuinely different memory layout between the two ABI variants.

**Example:** A user following the official AOTInductor example with the CXX11-ABI build of libtorch hits unexplained "Missing Symbols" errors (#122313), officially tagged by PyTorch maintainers under both `module: abi` and `module: crash` ("Problem manifests as a hard crash, as opposed to a RuntimeError"). Separately, a user compiling PyTorch themselves to interoperate with a Python-embedded application found that neither ABI variant linked cleanly ("tons of linking error"), and that using the officially-distributed build at all produced incompatibilities with their own compiled `_C.cpython...so`, forcing a full self-compile just to get consistent behavior (#102142: `free(): invalid pointer Aborted`, resolved only by switching ABI versions).

**Fix:** Ensure the libtorch binary ABI variant matches the ABI your own project's compiler toolchain was configured to use (verify via `torch._C._GLIBCXX_USE_CXX11_ABI` from the Python side, or by checking your own compiler's `*GLIBCXX*``USE_CXX11_ABI` macro setting). When embedding libtorch within a larger application that also links against a Python-built PyTorch, explicitly verify both use the same ABI, since a mismatch here is not caught by the compiler or linker in all cases and can surface only at runtime.

**Sources:**

- [pytorch/pytorch #122313](https://github.com/pytorch/pytorch/issues/122313) — confirmed: officially tagged `module: abi` ("libtorch C++ ABI related problems") and `module: crash`, confirming this is a maintainer-recognized, dedicated problem category, not an isolated incident
- [pytorch/pytorch #102142](https://github.com/pytorch/pytorch/issues/102142) — confirmed, independent: a second user hitting `free(): invalid pointer Aborted` from an ABI mismatch between their own third-party C++17 libraries and PyTorch's default pre-CXX11 ABI build, resolved only by switching to the CXX11 ABI libtorch build
- [PyTorch Forums: "Wrong results on CUDA"](https://discuss.pytorch.org/t/wrong-results-on-cuda/199097) — confirmed, reveals a more insidious variant: a user reports that both ABI variants caused linking errors in their cross-compiled setup, and that the officially-distributed PyTorch build's bundled libtorch is incompatible with its own Python extension module unless self-compiled — with the user's genuine confusion recorded directly: "I don't understand why, in the year 2024, the official PyTorch release does not use the c++11 abi"
- `p**ytorch/pytorch #73829**`
- `pytorch/ao #2908`

---

---

## Group 2: excluded during earlier drafting

## Card [JL-05]
**Type Piracy (Julia)**

Julia allows any package to silently override functions it doesn't own (e.g. `Base.sum`), which can corrupt unrelated code anywhere, ML included. General language-design risk, not ML-specific. Evidence Tier: Strong (Aqua.jl has a dedicated check for it; multiple Flux.jl incidents). Excluded because it fails the ML-specificity litmus test: *would this be possible and meaningful in non-ML code, with the ML occurrence incidental?* — yes.

---

---

## Card [R-23]
**Unnecessary Iteration / ****`rbind()`**** in a Loop (R)**

Growing a data frame with repeated `rbind()` calls is O(n²) and one of the most famous general R performance pitfalls (documented in Patrick Burns' *The R Inferno*, 2012). Evidence trail is dominated by general scientific/spatial/statistics R code (sf, WREG, smoothr), not ML pipelines specifically. Excluded for two reasons: fails the ML-specificity test, and it's a performance issue, not a correctness issue (a second way it differs from the main catalogue's cards).

---

---

## Card [R-24]
**Random State Changed by Loading a Package (R)**

`set.seed(1); library(forecast); runif(1)` gives a different result than without the `library()` call, purely from the package's load-time side effects on the RNG. Confirmed by a tidymodels maintainer-opened issue (`tune` #389), including a second case where `tune_grid()`'s own deliberate RNG-kind change sometimes fails to reset afterward. Evidence Tier: Strong. Zhang mapping: R2 (direct analog). Excluded despite being real and well-evidenced: **no actionable fix exists on the user's side** — reseeding before use doesn't close the gap, since a package loaded afterward can still perturb state before the next draw.

---

---

## Card [R-25]
**Broadcasting Documentation Confusion (R torch)**

R torch's broadcasting rules (how tensors of different shapes combine in elementwise ops) are inherited from libtorch's C++ semantics but documented thinly in the R-facing docs, leading to confusion about which dimension aligns with which. Zhang mapping: R1 (loose/weak analog — the Python-side smell is about broadcasting being *silently used* in ways that mask bugs; here the issue is closer to a documentation gap than an actual runtime misbehavior). Evidence Tier: Weak — thin, scattered forum evidence, no maintainer acknowledgment or dedicated issue found. Excluded for insufficient evidence to support a Strong/Moderate claim.

---

## Card [R-26]
**1-Based Indexing Dimension Drop (R torch single-element indexing)**

Indexing a tensor with a single integer in R torch (1-based) drops a dimension the same way NumPy/PyTorch's 0-based single-element indexing does — this was initially drafted as a "novel" R-specific smell, but on closer check it's simply R torch faithfully mirroring the documented NumPy/PyTorch convention, with the only "difference" being R's 1-based offset, not new risk. Excluded as overclaimed: it needed reframing as "same smell, cosmetic indexing-base difference" rather than a new entry, and once reframed it didn't add anything the Python catalogue didn't already cover.

---

---

## Card [JV-14]
**Memory Order Left Implicit — ****`dup()`**** Returns C Order by Default (ND4J)**

ND4J's `dup()` method (used to force a copy of an array) defaults to C (row-major) order regardless of the source array's original memory layout, which could silently change strides/performance characteristics for code assuming order preservation. Zhang mapping: R20-adjacent (data.table/memory-layout family). Evidence Tier: Low — no concrete evidence surfaced of this actually causing misuse or bugs in practice beyond the documented default behavior itself; it reads as expected, documented API behavior rather than a smell. Excluded for lacking a real misuse case.

---

---

## Card [CPP-10]
**Forward Hooks Added to Match Python Making R8 Directly Applicable (C++ / libtorch)**

The hypothesis was that libtorch had added forward-hook support matching PyTorch's Python API closely enough that Zhang et al.'s R8 (hook-related smell) would transfer directly. Thread/changelog evidence showed this feature was never fully shipped in the form assumed — so the "direct applicability" claim was overclaimed relative to what's actually in libtorch today. Excluded for resting on a feature that doesn't exist as described.

---

