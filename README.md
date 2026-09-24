# AT2 Weather Intelligence — Experimentation Repository

**Student:** Vignesh Selvam (25740287)
**Subject:** 36120 Advanced Machine Learning, Session 26SP — Assessment Task 2 (Machine Learning as a Service)
**Companion repository:** [36120-at2-api](https://github.com/vigneshcrack/36120-at2-api) (FastAPI deployment, live on Render)

<a target="_blank" href="https://cookiecutter-data-science.drivendata.org/">
    <img src="https://img.shields.io/badge/CCDS-Project%20template-328F97?logo=cookiecutter" />
</a>

This repository contains the data acquisition, feature engineering, experimentation, and model
selection work behind a machine learning as a service solution for Open-Meteo. Two models are
trained on ten years of historical Sydney, NSW weather data:

- **Climate Comfort Index (CCI)** — a 0–100 regression score describing how suitable conditions
  are for outdoor activity, forecast three days ahead.
- **Weather Hazard Category (WHC)** — a four-class risk label (Low / Moderate / High / Extreme
  Risk), forecast exactly seven days ahead.

Both targets are derived from raw weather observations using the fixed formulas set by the
assignment brief, not measured directly. The winning models from this repository (a tuned
Gradient Boosting Regressor for CCI, a class-balanced Random Forest Classifier for WHC) are
served in production by the companion API repository above, currently live at
**https://at2-weather-api-idge.onrender.com**.

## Project organisation

```
36120-at2-experiments/
├── notebooks/
│   ├── 0_data_acquisition.ipynb                                  <- pulls & aggregates Sydney weather via Open-Meteo
│   ├── comfort_climate/
│   │   └── 36120_26SP_AT2_25740287_experiment_comfort_climate.ipynb
│   └── weather_hazard/
│       └── 36120_26SP_AT2_25740287_experiment_weather_hazard.ipynb
├── models/
│   ├── comfort_climate/
│   │   └── comfort_climate_model.joblib      <- final tuned Gradient Boosting Regressor
│   └── weather_hazard/
│       └── weather_hazard_model.joblib       <- final tuned Random Forest Classifier
├── data/                                     <- gitignored; regenerated locally, see below
│   ├── raw/
│   ├── interim/                              <- sydney_daily_weather.csv produced by notebook 0
│   ├── processed/
│   └── external/
├── reports/                                  <- generated analysis output, figures
├── references/                               <- data dictionaries and explanatory material
├── at2_weather/                              <- shared project config/utilities
├── tests/
│   └── test_data.py
├── pyproject.toml                            <- dependencies, incl. the custom mykrml_25740287 package
├── uv.lock
├── Makefile                                  <- make requirements / test / lint / format
└── README.md
```

## Setup

This project uses [uv](https://docs.astral.sh/uv/) for environment and dependency management,
and Python 3.12 (pinned in `.python-version`).

1. Clone this repository and `cd` into it.
2. Install dependencies. The custom `mykrml_25740287` package (shared feature-engineering and
   target-calculation logic, also used by the API repository) is published to TestPyPI rather
   than the main PyPI index, so it needs an extra index:
   ```bash
   uv sync --extra-index-url https://test.pypi.org/simple/ --index-strategy unsafe-best-match
   ```
3. Launch Jupyter Lab:
   ```bash
   uv run jupyter lab
   ```

## Running the notebooks

Run in this order, since each stage depends on the previous one's output:

1. **`notebooks/0_data_acquisition.ipynb`** — pulls ten years of hourly Sydney weather from the
   [Open-Meteo Historical Weather API](https://open-meteo.com/), aggregates it to daily
   resolution, and writes `data/interim/sydney_daily_weather.csv`.
2. **`notebooks/comfort_climate/36120_26SP_AT2_25740287_experiment_comfort_climate.ipynb`** —
   target creation, feature engineering, chronological train/validation/test split, systematic
   model comparison and hyperparameter tuning for the CCI regression task. Saves the selected
   model to `models/comfort_climate/`.
3. **`notebooks/weather_hazard/36120_26SP_AT2_25740287_experiment_weather_hazard.ipynb`** — the
   same process for the WHC classification task, including handling of the severe class
   imbalance in the Extreme Risk category. Saves the selected model to `models/weather_hazard/`.

Both experiment notebooks load `data/interim/sydney_daily_weather.csv` produced by step 1, so
that notebook must be run at least once first on a fresh clone (the `data/` folder is gitignored
and not committed).

## Running the tests

```bash
make test
```
or directly:
```bash
uv run pytest tests/
```

## Convenience commands

```bash
make requirements   # uv sync (see the extra-index-url note above for a first-time install)
make lint            # ruff check
make format          # ruff check --fix && ruff format
```

## Custom package

Shared logic (Open-Meteo data fetching/aggregation, CCI/WHC target calculation) lives in the
`mykrml_25740287` package, developed alongside this repository and published to TestPyPI as
[`mykrml-25740287`](https://test.pypi.org/project/mykrml-25740287/). It is used identically by
both this repository's notebooks and the companion API's prediction endpoints, so the target
definitions and feature engineering stay consistent between training and inference.

## AI usage acknowledgement

Generative AI (Claude) was used to support the design, implementation, and documentation of this
project, in accordance with the subject's AI usage policy. All generated code and analysis were
reviewed, run, and validated by the student, who takes full responsibility for their correctness
and can explain and justify every design decision, including the target formulas, the
chronological split and leakage-prevention approach, the choice of evaluation metrics, and the
final model selections.
