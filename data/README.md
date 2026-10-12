# Data

Sample satellite data and the scripts that produced it.

| File | Description |
|---|---|
| [`tempo_no2_data.csv`](tempo_no2_data.csv) | Tropospheric NO₂ columns extracted from one NASA TEMPO Level 2 granule (`TEMPO_NO2_L2_V03_20240717T232209Z_S015G06`), about 249,000 rows |
| [`extract_data.py`](extract_data.py) | Converts a TEMPO L2 NetCDF granule into the CSV above |
| [`scratch.py`](scratch.py) | Prints the groups and variables in a TEMPO NetCDF file, used to explore the format |

## CSV schema

| Column | Meaning |
|---|---|
| `latitude` | Pixel centre latitude, degrees north |
| `longitude` | Pixel centre longitude, degrees east |
| `NO2_troposphere` | NO₂ tropospheric vertical column, molecules/cm² |

Rows with missing or infinite values are dropped. No quality-flag filtering is applied, so the file includes negative retrievals.

## Regenerating the CSV

The source `.nc` granule is not committed (`*.nc` is git-ignored). Download it from [NASA Earthdata](https://search.earthdata.nasa.gov/) (collection `TEMPO_NO2_L2`, version V03), place it in this directory, then:

```bash
pip install netCDF4 pandas numpy
cd data
python extract_data.py
```

Both scripts read the granule filename from a constant at the top of the file; edit it to process a different granule.
