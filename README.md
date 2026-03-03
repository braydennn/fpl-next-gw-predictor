
# FPL Next-GW Points Predictor: End-to-End Modeling Project

Welcome to a project that predicts **per-player Fantasy Premier League points for any chosen Gameweek**! It builds on the FPL-Elo-Insights dataset by combining official FPL stats with team Elo and opponent defensive strength to generate a ranked leaderboard of expected points.

It brings together three powerful ingredients:

1. **Official FPL Player Data** – per-GW player performance (points, minutes, etc.).
2. **Team Context & Opponent Strength** – ClubElo-style team ratings, home/away splits, fixture difficulty.
3. **Rolling Form Features** – last 3/5 GW form and minutes trends.

---

## Data Updates

This project uses CSVs under `data/<season>/...` that you keep up to date.
If your repo syncs data automatically, your predictions will reflect the most recent Gameweek. Otherwise, drop in the latest CSVs and re-run the notebook.

* Expected layout:

  ```
  data/
  ├─ 2024-2025/
  │  ├─ playerstats/playerstats.csv
  │  ├─ players/players.csv
  │  └─ teams/teams.csv
  └─ 2025-2026/
     ├─ playerstats/playerstats.csv
     ├─ players/players.csv
     └─ teams/teams.csv
  ```

> Tip: You can add a cron/CI to refresh data and re-run the notebook on a schedule.

---

## Using the Project

Data is pulled from [olbauday/FPL-Elo-Insights](https://github.com/olbauday/FPL-Elo-Insights). 

Visit this repository to download the most up-to-date gameweek 
data to input into this project.


Feel free to use this project and adapt it—blog posts, analysis threads, or your own tools.
If it helps you, a link back to this repo is appreciated. If you build something cool, email me and I can showcase it here!


</details>

## What’s New in This Project?

<details>
<summary>Click to expand</summary>

* **Per-GW Predictions on Demand**: Enter a Gameweek (e.g., GW 9) and get a leaderboard of predicted points.
* **Season-Aware Training**: Trains on all of **2024–25** plus **current 2025–26** up to GW-1, then predicts **GW**.
* **Robust Features**:

  * Rolling form: mean `event_points` over last 3 & 5 GWs
  * Minutes trend: mean minutes over last 3 & 5 GWs
  * **Elo differential**: team Elo − opponent Elo
  * Opponent defensive strength: home/away aware
  * Position one-hot features
* **Models**: Ridge (scaled, numerically stable) and Random Forest (default).

</details>

## Project Structure

```
projects/player-points/
├─ notebooks/
│  └─ 01_predict_next_gw_points.ipynb   # end-to-end workflow
├─ outputs/                              # metrics.json + gw<k>_predictions_2025-2026.csv
├─ README.md                             # (this file)
└─ requirements.txt                      # minimal deps for the notebook
```

---


## How It Works

* **Target (`next_points`)**: For each player+season, we shift `event_points` by +1 GW so features at GW=k predict points at GW=k+1.
* **Training**: All rows from **2024–25** + current season rows **before** target GW.
* **Inference**: Rows at **GW = target_gw − 1** (they carry the features to predict the next GW).
* **Validation**: The function prevents invalid inputs (e.g., can’t predict GW 9 if GW 8 rows don’t exist).

---

## Running Predictions for Any Gameweek

Inside the notebook:

1. Run **Cells 1→7** to load data, engineer features, and train baseline models.

2. Run **Cell 8-prep** to cache arrays & masks.

3. Use **Cell 8A** (`train_and_predict_for_gw_fast`) to predict a chosen GW.

4. In **Cell 9**, set:

   ```python
   user_target_gw = 9  # choose any valid GW
   ```

   and run to get a top-20 leaderboard with `last_name` and `pred_points`.

5. **Cell 10** saves:

   * `projects/player-points/outputs/metrics.json`
   * `projects/player-points/outputs/gw<k>_predictions_2025-2026.csv`

---

## Example Features

* `form_last3`, `form_last5` — rolling mean of `event_points`
* `minutes_last3`, `minutes_last5` — rolling mean minutes (if available)
* `elo_diff` — team Elo minus opponent Elo
* `opp_def_str` — opponent defensive strength (home/away aware)
* `pos_*` — position one-hot
* (Optional) `now_cost`, `selected_by_percent` if present in your CSVs

---

