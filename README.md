# gridMET → ZIP Code (ZCTA) Humidity & Temperature Statistics
 
An R script that downloads daily [gridMET](https://www.climatologylab.org/gridmet.html) weather data, extracts values at the centroid of every US ZIP Code Tabulation Area (ZCTA), and writes summary statistics for relative humidity and temperature to CSV and Excel.
 
## What it produces
 
One row per ZCTA in the contiguous US, with these columns:
 
| Column | Description |
|---|---|
| `zcta` | 5-digit ZIP Code Tabulation Area |
| `rh_min_daily` | Lowest daily-minimum relative humidity over the period (%) |
| `rh_max_daily` | Highest daily-maximum relative humidity over the period (%) |
| `rh_min_p10` | 10th percentile of daily-minimum relative humidity (%) |
| `rh_max_p90` | 90th percentile of daily-maximum relative humidity (%) |
| `temp_min_daily` | Lowest daily-minimum temperature over the period |
| `temp_max_daily` | Highest daily-maximum temperature over the period |
| `temp_min_p10` | 10th percentile of daily-minimum temperature |
| `temp_max_p90` | 90th percentile of daily-maximum temperature |
| `n_days` | Number of valid days used (about 3,653 for 2015–2024) |
 
Temperature is in °C by default (°F if `to_F = TRUE`).
 
Outputs:
- `zcta_rh_temp_stats.csv`
- `zcta_rh_temp_stats.xlsx`
  
## Data sources
 
| Data | Source | Files used |
|---|---|---|
| Daily min/max relative humidity | gridMET, `http://www.northwestknowledge.net/metdata/data/` | `rmin_2015.nc`, `rmax_2024.nc` |
| Daily min/max temperature | gridMET, `http://www.northwestknowledge.net/metdata/data/` | `tmmn_2015.nc`, `tmmx_2024.nc` |
| ZIP geometry | US Census Bureau cartographic boundary file, 2020 ZCTAs (500k) | `cb_2020_us_zcta520_500k.zip` |

## Requirements
 
- R (4.0 or later)
- Packages: `terra`, `matrixStats`, `writexl`
```r
install.packages(c("terra", "matrixStats", "writexl"))
```
 
- Disk space: roughly 1 GB for 10 years of data (40 NetCDF files, about 20–30 MB each)
- An internet connection for the first run (downloads are skipped if files already exist)

## Usage
 
1. Clone the repo and open the script in R or RStudio.
2. Edit the settings at the top:
```r
years = 2015:2024   # period to summarize
to_F  = FALSE       # TRUE for Fahrenheit
```
 
3. Run the whole script (use **Source** in RStudio rather than line by line).
4. Find the results in `zcta_rh_temp_stats.csv` and `.xlsx`.
 
## How it works
 
1. **Download** the gridMET NetCDF files for each variable and year into `gridmet/`.
2. **Download** the Census ZCTA shapefile into `zcta/` and reproject it to WGS84 (EPSG:4326).
3. **Compute centroids** for each ZCTA (guaranteed to fall inside the polygon) and keep those within the contiguous US bounding box.
4. **Stack** all daily layers per variable with `terra::rast()`.
5. **Extract** gridMET values at each centroid, in chunks of 2,000 ZCTAs to limit memory use.
6. **Summarize** each ZCTA's daily series: minimum, maximum, 10th percentile (of daily minimums), and 90th percentile (of daily maximums).
7. **Clean** any infinite values (from centroids falling on missing cells) to `NA`, then write CSV and Excel.

## Citation and attribution
 
> Abatzoglou, J. T. (2013). Development of gridded surface meteorological data for ecological applications and modelling. *International Journal of Climatology*, 33(1), 121–131.
 
ZCTA boundaries are from the US Census Bureau (TIGER/Line cartographic boundary files).
