# Methodology: SLR, Grey-Literature Mining, and Inter-Rater Adjudication

Transcribed from the project's working "Notes" tab. This is the full methodological record behind the catalogue, including material not reproduced in the paper for space.

## Definition

An ML-specific code smell is a sub-optimal implementation choice in ML pipeline code, not a general programming smell, that risks reproducibility, correctness, efficiency or maintainability.

## Pipeline stages

1. Data preparation and pre-processing
2. Model training
3. Evaluation
4. Deployment

## SLR: inclusion / exclusion criteria (overall)

**Include** if the source:
1. documents a concrete, recurring implementation problem specific to ML pipeline code (not model accuracy, not general software smells)
2. is in R, Java, Scala, or Julia, or is language-agnostic conceptual work relevant to those ecosystems
3. is peer-reviewed academic work OR a grey-literature source meeting a minimum credibility bar (official docs, maintainer-authored changelog, a well-upvoted Stack Overflow answer, not a random blog post)

**Exclude**: general (non-ML specific) code smells studied in an ML context; pure model-performance/accuracy papers; sources with no ML-pipeline-relevant content.

### Formal inclusion criteria

| Code | Criterion |
|---|---|
| IC1 | Subject matter: describes one or more concrete, recurring implementation-level quality issues specific to ML/DL pipeline code — not general software quality, not model accuracy/performance alone. |
| IC2 | Language scope: primary subject is R, Java, Scala, Julia, or C++ ML/DL code, OR is language-agnostic conceptual/taxonomic work relevant to ML-specific quality issues — even if demonstrated on Python, as long as the underlying concept is not tied to a Python-specific API. |
| IC3 | Publication date: 2015 or later (Sculley et al. as the field's origin point). |
| IC4 | Venue type: peer-reviewed (journal/conference/workshop), OR a credible arXiv preprint. |
| IC5 | Published in English. |
| IC6 | Contribution type: primary empirical study, systematic review, or mapping study providing a taxonomy, catalogue, detector, or empirical characterization of ML-specific smells, anti-patterns, or technical debt. |

### Formal exclusion criteria

| Code | Criterion |
|---|---|
| EC1 | Wrong direction: uses ML/AI techniques to DETECT code smells (ML as the method), rather than studying smells IN ML code (ML as the subject). |
| EC2 | Wrong specificity: studies general, non-ML-specific code smells in an ML-adjacent codebase, with no discussion of anything unique to ML pipeline logic. |
| EC3 | Wrong focus entirely: purely about model accuracy, algorithmic performance, or statistical correctness, with no code-level or implementation angle. |
| EC4 | Python-specific with no broader claim. Routed to the "Python-side related work" list instead of discarded. |
| EC5 | Not a primary/secondary research contribution: editorials, posters, tutorials, keynote abstracts, panel summaries, front-matter entries. (Grey literature is out of scope here — separate Phase 2 pipeline.) |
| EC6 | Duplicate or superseded: keep only the most complete version. |
| EC7 | Full text not retrievable after a documented access attempt. Logged as a limitation, not silently excluded. |

## Search terms (per language)

Date range: since 2015 (based on Sculley et al.'s "Hidden Technical Debt in Machine Learning Systems").

**R**
```
("machine learning" OR "deep learning" OR "data science" OR "statistical learning")
AND
("code smell*" OR "anti-pattern*" OR "antipattern*" OR "technical debt" OR "defect pattern*" OR "code quality issue*")
AND
("R language" OR "R programming" OR "tidyverse" OR "tidymodels" OR "mlr3" OR "caret package" OR "recipes package")
```

**Java**
```
("machine learning" OR "deep learning" OR "data science")
AND
("code smell*" OR "anti-pattern*" OR "antipattern*" OR "technical debt" OR "defect pattern*" OR "code quality issue*")
AND
("Java" AND ("machine learning" OR "Weka" OR "Deeplearning4j" OR "DL4J" OR "Spark MLlib" OR "Apache Mahout"))
```

**Scala**
```
("machine learning" OR "deep learning" OR "data science")
AND
("code smell*" OR "anti-pattern*" OR "antipattern*" OR "technical debt" OR "defect pattern*" OR "code quality issue*")
AND
("Scala" OR "Spark MLlib" OR "Smile machine learning")
```

**Julia**
```
("machine learning" OR "deep learning" OR "data science")
AND
("code smell*" OR "anti-pattern*" OR "antipattern*" OR "technical debt" OR "defect pattern*" OR "code quality issue*")
AND
("Julia language" OR "Julia programming" OR "Flux.jl" OR "Knet.jl" OR "MLJ.jl")
```

**C++**
```
("machine learning" OR "deep learning" OR "reinforcement learning" OR "data science")
AND
("code smell*" OR "anti-pattern*" OR "antipattern*" OR "technical debt" OR "defect pattern*" OR "code quality issue*")
AND
("C++" AND ("libtorch" OR "PyTorch C++" OR "TensorFlow C++" OR "ONNX Runtime" OR "TensorRT" OR "MuJoCo" OR "mlpack"))
```
C++ was expected to return the fewest hits of the five; the plan going in was to lean more heavily on grey-literature mining (libtorch/mlpack/MuJoCo repos, C++ style guides, RL-framework changelogs) for this language if the formal SLR came up thin, which is in fact what happened.

## SLR results

Initial: 2,128 records; 1,887 after deduplication. Screening (abstract, then full text) proceeded via IC/EC criteria above, with snowballing. 11 conflicts arose during screening and were resolved by discussion:

| Candidate | Resolution |
|---|---|
| Automated detection of inter-language design smells in multi-language deep learning frameworks | Excluded — Python/C++ bridge, design-level rather than code-level |
| MLSmellHound: A Context-Aware Code Analysis Tool | Excluded — Python only |
| Clones in deep learning code: what, where, and why? | Excluded — code clones are generic |
| An Empirical Study of Self-Admitted Technical Debt in Machine Learning Software | Excluded — studies Python (for ML) vs. Java (for non-ML projects) |
| 23 shades of self-admitted technical debt: an empirical study on machine learning software | Included — contains smells related to ML pipeline |
| TFX: A TensorFlow-Based Production-Scale Machine Learning Platform | Excluded — TensorFlow-based (Python-only) |
| "Project smells": experiences in analysing the software quality of ML projects with mllint | Excluded — a linter, not a smell detector |
| Data Leakage in Notebooks: Static Detection and Better Processes | Included — check for analogs |
| A First Look at the Lifecycle of DL-Specific Self-Admitted Technical Debt | Excluded — not code-level |
| Is Using Deep Learning Frameworks Free? Characterizing Technical Debt in Deep Learning Frameworks | Included |
| An Empirical Study of Refactorings and Technical Debt in Machine Learning Systems | Included |

Final set of papers carried forward (via SLR and specific supplementary searches):
1. 23 shades of self-admitted technical debt: an empirical study on machine learning software
2. Data Leakage in Notebooks: Static Detection and Better Processes
3. Is Using Deep Learning Frameworks Free? Characterizing Technical Debt in Deep Learning Frameworks
4. An Empirical Study of Refactorings and Technical Debt in Machine Learning Systems
5. Characterizing Technical Debt and Antipatterns in AI-Based Systems: A Systematic Mapping Study
6. Code smells for machine learning applications (Zhang et al. — the original paper this catalogue extends)

The last two were not part of the SLR proper; they were included specifically because they offered a way to mine grey literature further. Other citations used for related work and the introduction are tracked separately in the project's References tab.

## Grey-literature mining

Results of grey-literature mining are the smell catalogue itself (see `../catalogue/`). Cards excluded from the catalogue during this phase but worth discussing are the Discussion-Only set (see `../discussion_only/`) and the Excluded set (see `../excluded/`).

## Final smells acquired: 54 (before adjudication)

## Inter-rater adjudication (2026-10-18)

Three raters rated all 54 candidate cards (include / exclude), recorded in `../ratings/ML_Smell_Catalog_Rated.csv` (`rating_1`, `rating_2`, `rating_3`; `rating_1` is the catalogue author's own vote).

Fleiss' κ = 0.712 on the full 54-row rated set; 0.636 restricted to the real catalogue-candidate cards only (excluding the trivially-unanimous pre-excluded tail) — both land in or near the substantial-agreement band (Landis & Koch, 1977). 10 of 54 cards had rater disagreement. Resolved as follows — the catalogue author's own rating (`rating_1`) was treated as the deciding vote wherever the majority was ambiguous or a conceptual objection was raised, consistent with her being the catalogue's final decision-maker:

**Moved catalogue → Discussion-Only:**
- **R-16** (Saved Preprocessing Pipeline Silently Breaks After Package Upgrade) — Novel smell, Strong evidence; no general fix confirmed at the package level (only a practical re-validate/pin-versions mitigation).
- **R-19** (Train/Eval Mode API Evolution, R torch) — noted as "versioning debt, not code debt"; borderline whether this qualifies as a code smell at all.
- **JL-01** (Design-Level Data Leakage Prevention, MLJ.jl) — noted as "not a smell, worth mentioning the severity" rather than a defect.
- **CPP-08** (Silent Numerical Divergence, C++/Python TorchScript) — no general fix confirmed.

**Moved catalogue → Excluded:**
- **R-20** (Tensor-to-R-Object Conversion Bugs, R torch) — 2/3 raters, including the catalogue author's own vote, voted exclude.
- **CPP-03** (Dual-ABI Silent Binary Incompatibility, libtorch) — 2/3 raters, including the catalogue author's own vote, voted exclude.

**Kept in catalogue despite disagreement:**
- **R-22** (data.table Chained Assignment Performance Trap) — Strong evidence, no stated objection, majority voted include.
- **JL-04** (Join Missing-Value Matching Parameter) — carries the identical "not a smell" note as JL-01, but majority favored include; flagged as an inconsistency worth a deliberate call rather than leaving implicit.
- **JV-02** (Incomplete Multi-Runtime Seed Coordination, PySpark) — Novel card; majority voted include, but the single exclude vote plus its Novel/no-precedent status means it should get one more adversarial evidence check before submission.

**Open flag, not yet resolved:** the three original Discussion-Only entries (Type Piracy, `rbind()`-in-a-loop, Random State Changed by Loading a Package — listed as JL-05 / R-23 / R-24 in the rated CSV) came back unanimous *exclude* from all three raters in this rating pass, not *discussion*. This conflicts with their placement in the Discussion-Only tab at rating time (they are treated as fully Excluded in this replication package, under "Group 2" in `../excluded/excluded_cards.md`, which predates and is independent of this specific inconsistency — but the inconsistency itself is flagged here, unresolved, exactly as recorded). Needs a decision: either "discussion" in the rating scheme means something narrower than "belongs in the paper's Discussion section," or these three should be explicitly re-filed as Excluded for that reason too. Left untouched pending that call.

**Net effect:** catalogue shrinks from 47 to 41 cards (4 moved to Discussion-Only: R-16, R-19, JL-01, CPP-08; 2 moved to Excluded: R-20, CPP-03).
