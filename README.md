# Replication Package: ML Code Smells Beyond Python

This package accompanies the CAIN 2026 submission "ML Code Smells Beyond Python," a cross-language ML code smell catalogue extending Zhang et al.'s (CAIN 2022) Python-centric catalogue to R, Java, Scala, Julia, and C++.

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

## Provenance

All files in this package were generated from the project's working Claude Doc ("ML Code Smells Beyond Python — Paper Draft"), specifically its Smell Catalogue, Notes, and Discussion-Only Smells tabs, plus the rating spreadsheet, as they stood after the manuscript's R-20/CPP-03 correction on 2026-10-26.
