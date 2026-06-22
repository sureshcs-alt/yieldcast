# YieldCast — ML Yield Forecasting API & UI

A **Random Forest** based 1-year-ahead crop yield forecasting system for Indian oilseed panel data (1980–2023).

---

## Features

- **ML Model**: Random Forest Regressor (400 trees) trained on lagged yields, rolling statistics, and time-trend features — no leakage from contemporaneous production or area
- **Features used**: `yield_lag1..5`, `yield_roll3_mean`, `yield_roll5_mean`, `yield_roll3_std`, `area_lag1`, `area_lag2`, `t` (time index), `t²` (quadratic trend) + one-hot encoded geography & crop
- **Holdout evaluation**: Rolling-origin split (test = last 8 years) to avoid data leakage
- **Performance**: RMSE ~268 kg/ha | MAE ~198 kg/ha | MAPE ~16.7%
- **REST API**: FastAPI backend (`/forecast`, `/meta`, `/health`)
- **Web UI**: Responsive dashboard with forecast table, bar chart, dark mode

---

## Setup

```bash
pip install -r requirements.txt
uvicorn app.main:app --reload
```

Then open http://localhost:8000 in your browser.

---

## API Reference

### `POST /forecast`
```json
{
  "geography": "Rajasthan",
  "crop": "Soybean",
  "include_current_area": false
}
```

**Response:**
```json
{
  "evaluation": { "rmse": 268.3, "mae": 197.6, "mape": 16.65, "test_start_year": 2015 },
  "forecasts": [
    { "geography": "Rajasthan", "crop": "Soybean", "year": 2023, "yield_forecast_kg_ha": 841.2 }
  ]
}
```

### `GET /meta`
Returns available geographies, crops, and year range.

### `GET /health`
Service health check.

---

## Deployment on Render

1. Push this repo to GitHub
2. Create a new **Web Service** on [render.com](https://render.com)
3. Connect your GitHub repo — `render.yaml` handles build & start commands automatically
4. Mount a **Render Disk** at `/data` and upload `crop_panel_data-1.csv` there

---

## Model Design Notes

| Design Choice | Rationale |
|---|---|
| Lagged features only | Avoids leakage from contemporaneous production/area |
| Pooled model (all series) | More training data than per-series models |
| Rolling-origin split | Preserves time order; no future data in training |
| Random Forest | Handles nonlinear trends, robust to small sample sizes |
| One-hot encoding | Allows geography & crop effects without separate models |

---

## Limitations

- Dataset has only 3 geographies × 6 crops — pooled model has ~380 training rows
- No weather, soil, irrigation, or price features
- MAPE ~16–17% is reasonable for agricultural data without climate covariates
