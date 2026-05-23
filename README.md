# When Does the Best Result Happen? Mapping Peak Trial Occurrence Across Countermovement Jump Repetitions

Source code and notebook workflow used in the research paper to prepare Hawkin Dynamics CMJ data, select best sessions, and generate figures/statistics.

## Repository layout
- `Pipeline.ipynb` – main pipeline notebook with reusable functions
- `data/00_src/` – place raw Hawkin Dynamics CSV exports here
- `data/01_raw/` – combined intermediate CSVs (auto-created)
- `data/plots/`, `data/metric_stats/`, `data/demographics/` – generated figures, stats, and paper-ready demographics tables
- `data/athletes.csv` – required athlete metadata (`athlete_name,sex,height_cm`)
- `requirements.txt` – Python dependencies (`matplotlib`, `numpy`, `polars`, `scipy`)

## Setup
- Python 3.x with `pip install -r requirements.txt`
- Open the notebook in Jupyter (`jupyter notebook` or `jupyter lab`) and run cells top to bottom.

## Inputs expected
- Hawkin Dynamics CMJ exports as CSVs in `data/00_src/` with at least: `athlete_name`, `timestamp`, `last_sync_time`, `testType_name`, `tag_names`, `system_weight_n`, and the metric columns: `jump_height_m`, `mrsi`, `stiffness_n_m`, `avg_relative_braking_force`, `avg_relative_braking_power_w_kg`, `avg_relative_propulsive_force`, `avg_relative_propulsive_power_w_kg`.
- Athlete metadata in `data/athletes.csv`; missing `sex` or `height_cm` rows will raise an error. This file is collected separately from the Hawkin exports.

## Pipeline outline
1) **Configure paths/thresholds** in the first notebook cell (drop list, sigma threshold, min reps, rest threshold, metrics list).
2) **Combine raw exports**: `load_and_combine` concatenates `data/00_src/*.csv`, drops unwanted columns/athletes, converts timestamps, computes `system_weight_kg`, reorders columns, sorts by time, and writes `data/01_raw/01_combined.csv`.
3) **Isolate CMJs**: `filter_cmj` keeps rows where `testType_name` is “Countermovement Jump”, drops `tag_names`, and writes `data/01_raw/02_combined_cmj.csv`.
4) **Add session/rep info**: `add_session_rep_columns` ranks sessions per athlete, numbers reps within sessions, and computes `rest_before_rep_seconds`.
5) **Filter quality**: removes familiarization sessions, requires ≥3 reps per session before filtering and ≥6 valid reps, keeps only first six reps, and enforces ≥30 s rest before the 4th rep. Progress is logged to `data/pipeline_stats.log`.
6) **Session stats & demographics**: `add_mrsi_session_stats` adds max/avg mRSI per session; `add_sex_height_from_csv` joins metadata. A sex-stratified, athlete-level demographics table is printed and saved to `data/demographics/demographics_by_sex.csv` and `data/demographics/demographics_by_sex.md`; the per-athlete source table is saved to `data/demographics/demographics_athlete_level.csv`. Per-athlete z-scores (abs values) are computed for all metrics and saved to `data/filtered_data.csv`.
7) **Select session paths**: the statistics run for three session-selection paths: best mRSI day (`best_mrsi_day`), typical mRSI day (`median_mrsi_day`, closest to each athlete’s median session-average mRSI), and reproducible random day (`random_day`, controlled by `RANDOM_DAY_SEED`). Session selections are written to `data/<selection_path>_session_per_athlete.csv`; the legacy `data/best_session_per_athlete.csv` is still written for the best-mRSI path.
8) **Outlier handling and plots**: for each metric in `METRICS_CONFIG`, rows beyond ±4 SD in the z-score are dropped, best reps per athlete are chosen within each selected session path, and `plot_raincloud` produces PNG + CSV + stats per metric. The best-mRSI path keeps the legacy plot location (`data/plots/<metric>/...`); added paths write under `data/plots/median_mrsi_day/<metric>/...` and `data/plots/random_day/<metric>/...`. Summary CSVs land in `data/all_summary.csv`, path-specific `data/<selection_path>_all_summary.csv`, and `data/outliers_summary.csv`.
9) **Chi-square + G-Test (LRT) effect size**: for each metric’s best-rep distribution across trials and each selection path, a chi-square and G-test (LRT) goodness-of-fit test (df=5 for six reps) plus Bonferroni-adjusted p-value and Cramer's V are computed. Aggregated results are saved to `data/chi2_g_summary.csv`, with path-specific files in `data/<selection_path>_chi2_g_summary.csv`.

## Customization tips
- Adjust drop lists, thresholds, metric list, and `RECALCULATE_Z_SCORE` in the config/plot cells to match your protocol.
- Fonts default to Times New Roman/Nimbus Roman if installed, otherwise `serif`.
- Directories are created automatically; reruns overwrite log and summary CSVs.

## Typical run
1) Place raw exports in `data/00_src/` and confirm `data/athletes.csv` is complete.
2) Install deps (`pip install -r requirements.txt`).
3) Run `Pipeline.ipynb` sequentially to regenerate combined CSVs, filtered data, plots, and summaries.
