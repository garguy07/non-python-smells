# Replication Package: ML Code Smells Beyond Python

This package accompanies the CAIN 2026 submission "ML Code Smells Beyond Python," a cross-language ML code smell catalogue extending Zhang et al.'s (CAIN 2022) Python-centric catalogue to R, Java, Scala, Julia, and C++.

Its contents reflect the catalogue's state **after the 2026-10-18 inter-rater adjudication**, including the removal of R-20 and CPP-03 (recorded in adjudication as moved to Excluded, but only fully propagated out of the manuscript on 2026-10-26). The manuscript's Table 2 and this package's `catalogue/catalogue_table.csv` are mutually consistent: both report 41 cards (18 R, 3 Julia, 13 Java/Scala, 7 C++).

## Directory structure

```
catalogue/
  catalogue_table.csv       41 catalogued cards: id, name, language, pipeline stage,
                             Zhang et al. mapping type, evidence tier, classification
                             (syntactic / API-runtime / failure-mode / absence-of-counterpart).
                             This is the same data reported in the paper's Table 2.
  full_card_details.md      Full writeup for all 41 catalogued cards: description, what
                             differs from the Python original, a concrete example, the fix
                             (or why no general fix exists), and sourcing for every claim.

discussion_only/
  discussion_only_cards.md  4 cards moved out of the catalogue during adjudication because
                             they fail the ML-specificity test, document the absence of a
                             smell, or read as versioning debt rather than code debt.
                             Referenced in the paper's Discussion section, not Table 2.

excluded/
  excluded_cards.md         9 cards considered during drafting/rating and excluded outright:
                            2 excluded at the 2026-10-18 adjudication (R-20, CPP-03), and
                            7 excluded earlier during drafting for ML-specificity, evidence,
                            or overclaiming reasons (with their rationale recorded inline).

methodology/
  slr_and_adjudication.md   The full systematic literature review (search strings per
                             language, inclusion/exclusion criteria, screening conflicts
                             and their resolution), the grey-literature mining process,
                             and the complete inter-rater adjudication record (Fleiss' κ,
                             every card that changed status and why).

ratings/
  ML_Smell_Catalog_Rated.csv  Raw per-card ratings from all three raters (rating_1 = the
                               catalogue author's own vote, treated as the deciding vote
                               where the majority was ambiguous), plus free-text notes,
                               for all 54 cards considered at any point in the project
                               (41 catalogued + 4 discussion-only + 9 excluded).
```

## How the pieces relate

- `ratings/ML_Smell_Catalog_Rated.csv` is the raw input to adjudication — 54 rows, one per card ever seriously considered, including the ones that didn't survive to the final catalogue.
- `methodology/slr_and_adjudication.md` documents the full process that produced those 54 candidates and the reasoning that sorted them into catalogue / discussion-only / excluded.
- `catalogue/`, `discussion_only/`, and `excluded/` are the three disjoint output sets that process produced (41 + 4 + 9 = 54, matching the ratings file).

## Known open item (not resolved in this package)

The Notes tab this package was built from flags one unresolved inconsistency: three cards currently filed under "excluded during earlier drafting" (JL-05, R-23, R-24) were, at the time of the rating pass, still sitting in the project's Discussion-Only set rather than fully excluded, and all three raters unanimously voted *exclude* rather than *discussion* on them. This package files them as Excluded (matching their final disposition and the raters' unanimous vote), but the underlying question — whether "discussion" in the rating scheme ever meant something different from "belongs in the paper's Discussion section" — was left open in the source material and is reproduced as-is in `methodology/slr_and_adjudication.md`, not resolved here.

## Provenance

All files in this package were generated from the project's working Claude Doc ("ML Code Smells Beyond Python — Paper Draft"), specifically its Smell Catalogue, Notes, and Discussion-Only Smells tabs, plus the rating spreadsheet, as they stood after the manuscript's R-20/CPP-03 correction on 2026-10-26.
