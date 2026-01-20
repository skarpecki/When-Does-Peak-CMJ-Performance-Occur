# When Does the Best Result Happen? Mapping Peak Trial Occurrence Across Countermovement Jump Repetitions

Source code and notebook workflow used in the research paper to prepare Hawkin Dynamics CMJ data, select best sessions, and generate figures/statistics.

## Repository layout
- `Pipeline.ipynb` – main pipeline notebook with reusable functions
- `data/00_src/` – place raw Hawkin Dynamics CSV exports here
- `data/01_raw/` – combined intermediate CSVs (auto-created)
- `data/plots/`, `data/metric_stats/` – generated figures and stats
- `data/athletes.csv` – required athlete metadata (`athlete_name,sex,height_cm`)
- `requirements.txt` – Python dependencies (`matplotlib`, `numpy`, `polars`)

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
6) **Session stats & demographics**: `add_mrsi_session_stats` adds max/avg mRSI per session; `add_sex_height_from_csv` joins metadata. Per-athlete z-scores (abs values) are computed for all metrics and saved to `data/filtered_data.csv`.
7) **Select best session**: `best_session_ever` picks each athlete’s session with the highest `max_mrsi_in_session`, writing `data/best_session_per_athlete.csv`.
8) **Outlier handling and plots**: for each metric in `METRICS_CONFIG`, rows beyond ±4 SD in the z-score are dropped, best reps per athlete are chosen, and `plot_raincloud` produces PNG + CSV + stats per metric (e.g., `data/plots/jump_height_m/CMJ_jump_height_m_raincloud.png`). Summary CSVs land in `data/plots/all_summary.csv` and `data/plots/outliers_summary.csv`. Panel images are assembled under `data/plots/raincloud_panels/`.

## Customization tips
- Adjust drop lists, thresholds, metric list, and `RECALCULATE_Z_SCORE` in the config/plot cells to match your protocol.
- Fonts default to Times New Roman/Nimbus Roman if installed, otherwise `serif`.
- Directories are created automatically; reruns overwrite log and summary CSVs.

## Typical run
1) Place raw exports in `data/00_src/` and confirm `data/athletes.csv` is complete.
2) Install deps (`pip install -r requirements.txt`).
3) Run `Pipeline.ipynb` sequentially to regenerate combined CSVs, filtered data, plots, and summaries.
