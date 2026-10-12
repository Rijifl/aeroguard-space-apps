# AeroGuard API

A single-file [FastAPI](https://fastapi.tiangolo.com/) service ([`main.py`](main.py)) that serves the NO₂ forecasting model, the city AQI forecast, and the Gemini-backed advice endpoints used by the frontend.

## Run

From the repository root:

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r backend/requirements.txt
uvicorn backend.main:app --reload --port 8000
```

Requires Python 3.11 or newer (the pinned NumPy and TensorFlow versions do not support older interpreters). Interactive API docs are served at `http://localhost:8000/docs`.

The frontend expects this service on `http://localhost:8000`.

### Gemini key

The advice endpoints call `gemini-2.5-flash`. The key is defined as the `GEMINI_API_KEY` constant in `main.py`; set it to your own key from [Google AI Studio](https://aistudio.google.com/apikey) before starting the server. The service creates the Gemini client at import time, so it will not start without one.

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/` | Health check |
| `POST` | `/predict-no2` | Next-step NO₂ value from the last 10 observations |
| `POST` | `/city-aqi` | Next-day AQI forecast for a South Carolina city |
| `POST` | `/wildfire-advice` | Smoke and evacuation guidance from Gemini |
| `POST` | `/pollution-advice` | Activity-specific air quality guidance from Gemini |

### `POST /predict-no2`

Runs the Keras model in [`no2_pred_10_window_newer.keras`](no2_pred_10_window_newer.keras). The input must contain exactly 10 values, reshaped internally to `(1, 10, 1)`.

```json
{ "values": [1.2, 1.1, 1.4, 1.3, 1.5, 1.6, 1.4, 1.3, 1.2, 1.1] }
```

```json
{ "prediction": 1.27 }
```

### `POST /city-aqi`

```json
{ "city": "Greenville", "days_back": 10 }
```

`city` must be one of: Charleston, Columbia, Greenville, Myrtle Beach, Spartanburg, Florence, Rock Hill, Sumter, Anderson, Clemson. `days_back` (2 to 30, default 10) controls how many days of history are returned.

How it works:

1. Fetches 100 days of daily weather for the city from the [NASA POWER API](https://power.larc.nasa.gov/) (temperature, humidity, precipitation, wind speed, surface pressure).
2. Estimates a daily AQI from those weather values with a heuristic formula. This is a weather-derived proxy, not a measured AQI.
3. Trains a `RandomForestRegressor` on a 5-day sliding window of that series, on every request.
4. Predicts the next day and clamps the result to within two standard deviations of the historical mean.

The response includes `predicted_aqi`, `aqi_level`, `description`, `method`, `prediction_date`, `recent_aqi`, and a `historical_data` array of `{ date, aqi, ...weather }` rows. An unknown city returns `400`.

### `POST /wildfire-advice`

```json
{
  "location": "Greenville, SC near Paris Mountain",
  "user_context": "Asthmatic senior, light outdoor chores planned"
}
```

### `POST /pollution-advice`

```json
{
  "location": "Spartanburg, SC",
  "activity": "Evening 5K run on Rail Trail",
  "user_context": "Mild seasonal allergies"
}
```

Both advice endpoints return `{ "advice": "..." }`, a short plain-text briefing focused on Upstate South Carolina.

## Notes

- CORS is open to all origins, which suits local development only.
- The NO₂ model is loaded once at startup; the service raises on start if the model file is missing.
