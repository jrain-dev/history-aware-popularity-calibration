# Curated results

This folder holds the small, reviewable summaries of completed experiment runs: CSV tables, figures, and the provenance files that identify how each run was produced. Full run exports (Parquet recommendation lists, evaluator inputs, Spark `_SUCCESS`/`.crc` markers) stay in the local, git-ignored `outputs/` folder; regenerate them by rerunning the notebooks with the run settings recorded here. No external archive of the full exports exists yet. If one is created, link it from this file.

Only split seed 42 is complete. Seeds 43 and 44 have not been run, so nothing here supports final three-seed claims.

## Seed 42

| Stage | Source run in `outputs/` | Files kept here |
| --- | --- | --- |
| ALS baseline | `als-baseline/2026-11-16T01-44-41.706431Z` | `als-baseline/split-config.txt` |
| ALS tuning | `als-tuning/2026-11-16T01-34-52.248706Z` | `als-tuning/grid.csv` (see the caveat below) |
| Popularity profiles | `popularity-profiles/2026-11-16T01-50-34.019005Z` | `popularity-profiles/split-config.txt` |
| Baseline UPD | `baseline-upd/2026-11-16T02-06-03.108645Z` | `baseline-upd/provenance.txt` |
| Calibration | `calibration/2026-11-16T01-57-11.331215Z` | `calibration/manifest.txt`, `results.csv`, `selected_on_validation.csv`, `metrics-popularity-diversity.csv`, `upd-crosscheck.csv`, and `analysis/` |

`calibration/analysis/` was regenerated from `calibration/results.csv` with the current `analysis/history_group_analysis.py`:

```bash
python analysis/history_group_analysis.py results/seed-42/calibration/results.csv --out results/seed-42/calibration/analysis
```

## Reading these results

- **Validation and test are different files.** `calibration/results.csv`, `metrics-popularity-diversity.csv`, and everything in `calibration/analysis/` (including `ndcg_vs_upd.png` and the matched-UPD tables and plot) are computed on the validation split carved from training data. Held-out test results exist only for the validation-selected settings, in `calibration/selected_on_validation.csv`. Validation curves are not independent test confirmation of the history-aware conjecture.
- **Matched-UPD gains are interpolated estimates.** Each CP and HA family is sampled at a few lambda settings. The gain at a given UPD is the difference between linear interpolations of the two NDCG@10-vs-UPD curves, restricted to the UPD range both curves cover (the `overlap` column). It is not a measurement at identical UPD. A group with no overlap is reported as `none` with a missing gain, never as zero gain. The `basis` column states this on every row, and `seedsWithOverlap` reports how many seeds contributed.
- **The ALS tuning grid is out of date.** `als-tuning/grid.csv` was produced before the tuning notebook was corrected to rank every unrated catalog movie. Its `precisionAt10` and `ndcgAt10` rank only each user's validation-rated movies, so they are far higher than the pipeline's top-N metrics and must not be used to judge tuning. Its RMSE column is unaffected. Replace the file after rerunning `notebooks/als-tuning.ipynb`.
- **Selection was rerun on paper after a fix.** The calibration run predates the change that compares each group's HA metrics with the same group's CP curve during matched-UPD selection. Replaying the corrected rule on `calibration/results.csv` selects the same setting, `ha_n100_l0.9` (mean gain +0.00247 instead of +0.00244), so the selected settings and their test results are unchanged.
- **Diagnostics are not included yet.** `accuracy-popularity-diagnostics.csv` has not been generated for this run.
