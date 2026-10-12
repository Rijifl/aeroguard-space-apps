# Machine learning

Notebooks, data access scripts and model artifacts from the research side of AeroGuard. Nothing in this directory is imported by the running application: the API serves [`backend/no2_pred_10_window_newer.keras`](../backend/no2_pred_10_window_newer.keras) and trains its city AQI forecaster on the fly.

## Notebooks

| Notebook | What it does |
|---|---|
| [`sc_city_aqi_forecast.ipynb`](notebooks/sc_city_aqi_forecast.ipynb) | Pulls daily weather from the NASA POWER API for South Carolina cities, derives an AQI estimate, and fits a random forest to forecast the next day. This is the prototype of the API's `/city-aqi` endpoint. |
| [`tempo_data_collection_and_aqi_model.ipynb`](notebooks/tempo_data_collection_and_aqi_model.ipynb) | Queries NASA's CMR catalogue for TEMPO Level 3 and SEDAC air quality collections, then builds a feature-engineering and training pipeline (random forest, gradient boosting) on a synthetic dataset shaped like those sources. |
| [`omi_no2_granule_download.ipynb`](notebooks/omi_no2_granule_download.ipynb) | Searches CMR for OMI/Aura daily NO₂ (`OMNO2d`) granules over a bounding box and downloads them with Earthdata credentials. |

## Scripts

| Script | What it does |
|---|---|
| [`data_extract.py`](data_extract.py) | Uses `earthaccess` and `xarray` to search for TEMPO NO₂, HCHO, O₃ and radiance products and stream a NO₂ granule from the cloud without downloading it. Requires a free [NASA Earthdata](https://urs.earthdata.nasa.gov/) login. |

## Model artifacts

| File | Format |
|---|---|
| [`models/aqi_forecaster_model.h5`](models/aqi_forecaster_model.h5) | Keras HDF5 |
| [`models/aqi_forecaster_model.pkl`](models/aqi_forecaster_model.pkl) | Python pickle |

These are experimental AQI forecasters saved during the hackathon. They are kept for reference and are not loaded by the API.

## Outputs

[`outputs/global_air_quality_map.html`](outputs/global_air_quality_map.html) is a standalone Leaflet map of global air quality generated during analysis. Open it directly in a browser.

## Environment

The notebooks were written against Python 3.13. Beyond `backend/requirements.txt` they use `matplotlib`, `seaborn`, `earthaccess`, `xarray` and `netCDF4`.
