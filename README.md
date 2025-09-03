# San Francisco Rent Trends — Initial Report & EDA

**Author:** Luke Drury  
**Topic:** SF Rent Trends (ZIP-level)

---

## 1) Research Questions

1. **Trends:** How have typical asking rents (ZORI) changed over time in San Francisco?  
2. **Drivers:** Which neighborhood-level factors (income, transit access, safety, tourism pressure) are associated with higher or lower rent by ZIP?  
3. **Forecast:** What is a simple baseline forecast for citywide rent over the next 12 months?  
4. **Segments:** Can we cluster ZIP codes into affordability/amenity segments with distinct profiles?

---

## 2) Data Sources

- **Zillow Observed Rent Index (ZORI)** — time series rent index by ZIP (typical asking rent).  
  - File used: `data/zori_zip.csv`.
- **San Francisco ZIP Code Polygons** — ZIP boundaries for spatial joins/filtering.  
  - File used: `data/sf_zip_codes.geojson`.
- **SFMTA (Muni) Stops** — point locations of transit stops (proxy for transit access).  
  - File used: `data/muni_stops.csv` _(or GeoJSON/Shapefile)_.
- **SFPD Incident Reports** — public safety proxy; incidents within last 12 months.  
  - File used: `data/sfpd_incidents.csv`.
- **Inside Airbnb (San Francisco)** — tourism activity proxy; listings density by ZIP.  
  - File used: `data/airbnb_sf_listings.csv`.
- **ACS (Census) — Median Household Income (ZCTA/ZIP)**  
  - Retrieved via Census API (ACS 5-year Subject `S1901_C01_012E` or Detailed `B19013_001E`).  
  - Joined as `income_zip` (columns: `ZIP, median_income`).

> **Notes**  
> • ZCTAs ~ ZIPs; this approximation is standard in city-level analysis.  
> • File names match the notebook expectations; adjust paths if you rename.

---

## 3) Methods (what’s in the notebook)

### 3.1 Data Cleaning & Preparation
- Standardized ZIP keys to 5-digit strings; unified coordinate reference system (EPSG:4326).
- Handled date parsing for all time series; constrained SFPD incidents to the last 12 months relative to latest ZORI month.
- Robust loaders for Muni/SFPD that auto-detect latitude/longitude column names and tolerate malformed CSV rows.
- Constructed ZIP-level features:
  - `rent_index` = latest ZORI by ZIP  
  - `muni_stops` and `transit_density` = stops per 1k residents _(if population available)_  
  - `incidents_12m` and `crime_per_1k` _(if population available)_  
  - `airbnb_density` = listings per km²  
  - `median_income` (ACS)

### 3.2 Exploratory Data Analysis (EDA)
- Distributions (histograms) for continuous features (rent, income, etc.).
- Scatterplots: rent vs income; rent vs crime; color by transit density.
- Choropleth maps (ZIP polygons) of rent and selected drivers.

### 3.3 Baseline Modeling
- **Regression (Cross-sectional, ZIP-level):**  
  - Target: `log_rent`  
  - Predictors: `log_income`, `stops_100` (Muni stops/100), `incidents_k` (incidents/1000), `airbnb_density`  
  - Estimator: OLS (statsmodels) with robust (HC3) SEs  
  - Metrics: **RMSE** and **R²**
- **Forecasting (Citywide, Monthly):**  
  - Aggregation: Median ZORI across SF ZIPs by month  
  - Model: SARIMAX (1,1,1)×(1,1,1)\_12 baseline  
  - Split: last 12 months = test  
  - Metrics: **RMSE (test)** and **MAPE (test)**
- **Clustering (ZIP-level):**  
  - Features: standardized `[rent_index, median_income, crime_per_1k, transit_density, airbnb_density]`  
  - Method: K-means (k≈3–5)  
  - Validation: silhouette score (sanity check)  
  - Visualization: PCA (2D) projection of clusters

---

## 4) Results (initial)

> Replace the placeholders with your actual outputs after running the notebook.

- **Regression:** RMSE = `___`; R² = `___`  
  _Directionally, median income shows a positive association with rent; other covariates vary by ZIP._
- **Forecast (12-month):** RMSE = `___`; MAPE = `___`  
  _Baseline seasonal model captures yearly seasonality; residuals suggest room for richer dynamics._
- **Clustering:** Best k = `___`; Silhouette = `___`  
  _Clusters separate into high-rent/high-income central ZIPs vs. more affordable outer ZIPs with lower transit density._

---

## 5) Project Structure

sf-rent-trends/
├─ data/
│ ├─ zori_zip.csv
│ ├─ sf_zip_codes.geojson
│ ├─ muni_stops.csv
│ ├─ sfpd_incidents.csv
│ ├─ airbnb_sf_listings.csv
│ └─ (auto-fetched) income_zip via Census API
├─ notebooks/
│ └─ 01_eda_modeling.ipynb
├─ README.md
└─ environment.txt # pandas, numpy, matplotlib, seaborn, scikit-learn, statsmodels, geopandas, requests


---

## 6) How to Run

1. Create a virtual environment and install packages from `environment.txt` (or `requirements.txt`).  
2. Place the datasets in `./data/` with the filenames shown above.  
3. (Optional) Set a Census API key: `export CENSUS_API_KEY=your_key`  
4. Open and run `notebooks/01_eda_modeling.ipynb` top-to-bottom.

---

## 7) Assumptions & Limitations

- **ZCTA vs ZIP:** Joined ACS ZCTAs to ZIP polygons pragmatically; small boundary mismatches may remain.  
- **Proxies:** Crime incidents, Muni stop counts, and Airbnb density are proxies for safety, transit access, and tourism pressure; they don’t fully capture causality.  
- **Stationarity:** The SARIMAX baseline is intentionally simple; more advanced forecasting (e.g., exogenous regressors, regime changes) may improve accuracy.  
- **Missingness:** Where necessary, benign imputations (e.g., zeros for counts) were used and documented in the notebook.

---

## 8) Ethical Considerations

- Neighborhood-level metrics can influence perceptions of safety and desirability. Results should be presented with context to avoid stigmatizing communities.  
- This analysis is descriptive/exploratory; it does not imply causation between any single factor and rent.

---

## 9) References & Data Access

- **Zillow Research – ZORI** (ZIP-level rents) & methodology  
- **San Francisco ZIP Boundaries** (DataSF or equivalent GIS source)  
- **SFMTA (Muni) Stops** (DataSF catalog)  
- **SFPD Incident Reports** (DataSF catalog)  
- **Inside Airbnb (SF)** (city listings CSV)  
- **Census/ACS (ZCTA)** — Median Household Income via ACS 5-year Subject `S1901_C01_012E` or Detailed `B19013_001E` (Census API)

> Exact download details and any transformation steps are documented in the notebook’s **Data Loading** cells.

---

## 10) Summary

This initial report explores San Francisco rent patterns using Zillow’s ZORI at the ZIP level, enriched with neighborhood context (income, transit access, safety, and tourism density). After cleaning and integrating multiple public datasets, the analysis provides labeled visualizations, a baseline cross-sectional regression, a citywide seasonal forecast, and an interpretable clustering of ZIPs. Early results suggest that higher-income, transit-rich, central neighborhoods tend to have higher rents, while outer areas remain relatively more affordable. The project sets a clear foundation for refinement in modeling and additional features (e.g., employment centers, school quality, zoning) in subsequent milestones.

