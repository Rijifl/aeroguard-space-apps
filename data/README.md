<h1 align="center">Data</h1>

<p align="center">Sample satellite data and the scripts that produced it.</p>

<p align="center">
  <a href="../README.md">← AeroGuard</a> ·
  <a href="../frontend/README.md">Web app</a> ·
  <a href="../backend/README.md">API</a> ·
  <a href="../ml/README.md">Machine learning</a>
</p>

<div align="center">

| File | Description |
|:---:|:---|
| [`tempo_no2_data.csv`](tempo_no2_data.csv) | Tropospheric NO₂ columns extracted from one NASA TEMPO Level 2 granule (`TEMPO_NO2_L2_V03_20240717T232209Z_S015G06`), about 249,000 rows |
| [`extract_data.py`](extract_data.py) | Converts a TEMPO L2 NetCDF granule into the CSV above |
| [`scratch.py`](scratch.py) | Prints the groups and variables in a TEMPO NetCDF file, used to explore the format |

</div>

<h2 align="center">CSV schema</h2>

<div align="center">

| Column | Meaning |
|:---:|:---:|
| `latitude` | Pixel centre latitude, degrees north |
| `longitude` | Pixel centre longitude, degrees east |
| `NO2_troposphere` | NO₂ tropospheric vertical column, molecules/cm² |

</div>

<p align="center">
  Rows with missing or infinite values are dropped. No quality-flag filtering is applied, so the file includes negative retrievals.
</p>

<h2 align="center">Regenerating the CSV</h2>

<p align="center">
  The source <code>.nc</code> granule is not committed (<code>*.nc</code> is git-ignored). Download it from <a href="https://search.earthdata.nasa.gov/">NASA Earthdata</a> (collection <code>TEMPO_NO2_L2</code>, version V03), place it in this directory, then:
</p>

```bash
pip install netCDF4 pandas numpy
cd data
python extract_data.py
```

<p align="center">
  Both scripts read the granule filename from a constant at the top of the file; edit it to process a different granule.
</p>
