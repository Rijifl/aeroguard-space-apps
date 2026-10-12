<h1 align="center">AeroGuard API</h1>

<p align="center">
  A single-file <a href="https://fastapi.tiangolo.com/">FastAPI</a> service (<a href="main.py"><code>main.py</code></a>) that serves the NO₂ forecasting model, the city AQI forecast, and the Gemini-backed advice endpoints used by the frontend.
</p>

<p align="center">
  <a href="../README.md">← AeroGuard</a> ·
  <a href="../frontend/README.md">Web app</a> ·
  <a href="../ml/README.md">Machine learning</a> ·
  <a href="../data/README.md">Data</a>
</p>

<h2 align="center">Run</h2>

<p align="center">From the repository root:</p>

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r backend/requirements.txt
uvicorn backend.main:app --reload --port 8000
```

<p align="center">
  Requires Python 3.11 or newer (the pinned NumPy and TensorFlow versions do not support older interpreters). Interactive API docs are served at <code>http://localhost:8000/docs</code>.
</p>

<p align="center">
  The frontend expects this service on <code>http://localhost:8000</code>.
</p>

<h3 align="center">Gemini key</h3>

<p align="center">
  The advice endpoints call <code>gemini-2.5-flash</code>. The key is defined as the <code>GEMINI_API_KEY</code> constant in <code>main.py</code>; set it to your own key from <a href="https://aistudio.google.com/apikey">Google AI Studio</a> before starting the server. The service creates the Gemini client at import time, so it will not start without one.
</p>

<h2 align="center">Endpoints</h2>

<div align="center">

| Method | Path | Purpose |
|:---:|:---:|:---:|
| `GET` | `/` | Health check |
| `POST` | `/predict-no2` | Next-step NO₂ value from the last 10 observations |
| `POST` | `/city-aqi` | Next-day AQI forecast for a South Carolina city |
| `POST` | `/wildfire-advice` | Smoke and evacuation guidance from Gemini |
| `POST` | `/pollution-advice` | Activity-specific air quality guidance from Gemini |

</div>

<h3 align="center"><code>POST /predict-no2</code></h3>

<p align="center">
  Runs the Keras model in <a href="no2_pred_10_window_newer.keras"><code>no2_pred_10_window_newer.keras</code></a>. The input must contain exactly 10 values, reshaped internally to <code>(1, 10, 1)</code>.
</p>

```json
{ "values": [1.2, 1.1, 1.4, 1.3, 1.5, 1.6, 1.4, 1.3, 1.2, 1.1] }
```

```json
{ "prediction": 1.27 }
```

<h3 align="center"><code>POST /city-aqi</code></h3>

```json
{ "city": "Greenville", "days_back": 10 }
```

<p align="center">
  <code>city</code> must be one of: Charleston, Columbia, Greenville, Myrtle Beach, Spartanburg, Florence, Rock Hill, Sumter, Anderson, Clemson. <code>days_back</code> (2 to 30, default 10) controls how many days of history are returned.
</p>

How it works:

1. Fetches 100 days of daily weather for the city from the [NASA POWER API](https://power.larc.nasa.gov/) (temperature, humidity, precipitation, wind speed, surface pressure).
2. Estimates a daily AQI from those weather values with a heuristic formula. This is a weather-derived proxy, not a measured AQI.
3. Trains a `RandomForestRegressor` on a 5-day sliding window of that series, on every request.
4. Predicts the next day and clamps the result to within two standard deviations of the historical mean.

<p align="center">
  The response includes <code>predicted_aqi</code>, <code>aqi_level</code>, <code>description</code>, <code>method</code>, <code>prediction_date</code>, <code>recent_aqi</code>, and a <code>historical_data</code> array of <code>{ date, aqi, ...weather }</code> rows. An unknown city returns <code>400</code>.
</p>

<h3 align="center"><code>POST /wildfire-advice</code></h3>

```json
{
  "location": "Greenville, SC near Paris Mountain",
  "user_context": "Asthmatic senior, light outdoor chores planned"
}
```

<h3 align="center"><code>POST /pollution-advice</code></h3>

```json
{
  "location": "Spartanburg, SC",
  "activity": "Evening 5K run on Rail Trail",
  "user_context": "Mild seasonal allergies"
}
```

<p align="center">
  Both advice endpoints return <code>{ "advice": "..." }</code>, a short plain-text briefing focused on Upstate South Carolina.
</p>

<h2 align="center">Notes</h2>

- CORS is open to all origins, which suits local development only.
- The NO₂ model is loaded once at startup; the service raises on start if the model file is missing.
