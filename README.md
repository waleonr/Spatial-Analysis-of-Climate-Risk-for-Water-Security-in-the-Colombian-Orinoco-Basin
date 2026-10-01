# Spatial Analysis of Climate Risk for Water Security in the Colombian Orinoco Basin

A reproducible, satellite-based fuzzy-logic index for climate risk assessment across the 73 hydrographic subzones of the Colombian Orinoquía, built on the IPCC AR6 risk framework (**Risk = f(Hazard, Exposure, Vulnerability)**) and compared year-over-year (2024 vs. 2025).

> Submitted to the *Water Security and Climate Change Conference 2026* (Giessen, Germany, 6–8 October 2026) — Abstract ID 399.

---

## Overview

Increasing climate variability and hydrometeorological change challenge water security in complex socio-ecological systems. This project develops a **spatially explicit, multi-criteria fuzzy-logic climate-risk index** for the Colombian Orinoquía (345,891 km², 73 IDEAM hydrographic subzones) to identify priority areas for adaptation and territorial planning.

Unlike conventional composite indices that rely on hard thresholds, this approach passes every indicator through a **sigmoid fuzzy-membership function** — calibrated to the actual data distribution — before combining them with a **Fuzzy Gamma operator**, so intermediate conditions are treated as neither fully "at risk" nor fully "safe."

**Highlights**
- 16 indicators across 4 IPCC AR6 components: Hazard (3), Exposure (6), Sensitivity (4), Adaptive Capacity (3)
- 8 open satellite/reanalysis data products, processed entirely in Python and Google Earth Engine
- Year-over-year comparison (2024 vs. 2025) at 250 m resolution, for all 73 subzones
- Dominant-driver attribution: which IPCC AR6 component (Hazard, Exposure, or Vulnerability) explains the change in each subzone
- Full Monte Carlo robustness analysis (expert weights ±20%, fuzzy parameters ±15%)
- End-to-end reproducible pipeline, from Earth Engine export to the final poster/manuscript figures

## Study Area

The Colombian Orinoquía spans the binational Orinoco basin (most of its area lies in Venezuela) and is one of Colombia's fastest-expanding agricultural frontiers, under compounding climate stress. The analysis covers the **73 IDEAM hydrographic subzones** (IDEAM, 2013) within Colombian territory.

## Methodology

```
Satellite/reanalysis data  →  Local materialization  →  1. Fuzzification  →  2. Indicators
 (CHIRPS, ERA5-Land,           (per-variable GeoTIFF      (sigmoid           (16 fuzzified
  Dynamic World, WorldPop,      mosaic, 250 m grid,        membership          indicators:
  MODIS, SRTM, VIIRS, GHSL)     clipped to 73 subzones)    function)           3H · 6E · 4S · 3AC)
                                                                                    │
                                                                                    ▼
                                           4. Sensitivity   ←──────────────  3. Aggregation
                                           (Monte Carlo:                     (Fuzzy Gamma, γ = 0.90,
                                            30 runs weights ±20%,             combines indicators into
                                            30 runs fuzzy params ±15%)        H, E, V → Climate Risk Index)
                                                    │                                  │
                                                    └──────────────┬───────────────────┘
                                                                    ▼
                                      External validation: risk classification,
                                      dominant-driver mapping, 2024 vs. 2025 comparison by subzone
```

1. **Fuzzification.** Each of the 16 indicators is passed through a sigmoid membership function `1 / (1 + exp(-k·(x - x0)))`. Thresholds (`x0`) and slopes (`k`) are auto-calibrated to the real 2024–2025 median and interquartile range of each variable, rather than fixed a priori.
2. **Aggregation.** A Fuzzy Gamma operator (γ = 0.90) combines the fuzzified indicators into Hazard, Exposure and Vulnerability indices (Vulnerability itself is a Fuzzy Gamma combination of Sensitivity and inverted Adaptive Capacity), and finally into the Climate Risk Index.
3. **Classification.** Pixel-level risk is classified into five classes (Very low → Very high) using quintiles of the 2024 baseline.
4. **Dominant-driver attribution.** For every subzone, the component (Hazard, Exposure, or Vulnerability) that contributes most to the 2024→2025 change is identified.
5. **Sensitivity analysis.** Monte Carlo simulation (30 runs perturbing the six expert weight blocks ±20%; 30 runs perturbing the fuzzy membership parameters ±15%) tests the robustness of the spatial risk ranking.

### Data sources

| Product | Variables | Reference |
|---|---|---|
| CHIRPS | Daily precipitation | Funk et al., 2015 |
| ERA5-Land | Hourly temperature and climate variables | Muñoz-Sabater et al., 2021 |
| Dynamic World V1 | Near-real-time land cover | Brown et al., 2022 |
| WorldPop | Gridded population | WorldPop, 2020 |
| MODIS NDVI (MOD13Q1) | Vegetation | — |
| MODIS burned area (MCD64A1) | Fire disturbance | — |
| SRTM | Elevation, slope | — |
| VIIRS DNB | Nighttime lights | — |
| GHSL | Built-up surface | — |

All layers are retrieved, mosaicked and pre-processed through **Google Earth Engine**, reprojected to a common 250 m grid, and clipped to the 73 IDEAM subzone polygons for consistent year-to-year comparison.

## Key Results

<img width="1453" height="477" alt="image" src="https://github.com/user-attachments/assets/8aaf61a7-21e5-4c3c-9f45-496be77fd154" />

- Mean climate risk across subzones **fell slightly on average, from 0.458 (2024) to 0.428 (2025)**, but rose sharply in specific subzones: 57 of 73 improved, 16 worsened.
  - Largest increase: *Directos Río Arauca (md)* (+0.132)
  - Largest decrease: *Caño Guanápalo y otros directos al Meta* (−0.121)
- **Vulnerability, not Hazard, drives most of the 2024–2025 change** — dominant in 48 of 73 subzones (66%) vs. 25 (34%) for Hazard; Exposure never dominates a single subzone's change.
- In 2025, **20.98%** of the study area remained in the **High** risk class and **18.89%** in **Very High**.
- The spatial risk pattern is **highly robust to the expert weights** (Spearman ρ = 0.992–1.000 across 30 perturbation runs); the **fuzzy membership parameters are the larger source of uncertainty** (ρ = 0.90 on average, min 0.80).
- Exposure (land-use: cropland, wetlands, water, population) shifted little year-to-year — the signal comes from Vulnerability and Hazard, not from what is on the ground.

## Getting Started

### Requirements

- Python 3.11
- A Google Earth Engine account (for the data-export steps)
- Core packages: `geemap`, `rasterio`, `geopandas`, `numpy`, `pandas`, `matplotlib`

```bash
pip install geemap rasterio geopandas numpy pandas matplotlib
```

### Running the pipeline

1. Authenticate Earth Engine (`earthengine authenticate` or `ee.Authenticate()` inside the notebook).
2. Open `orinoco_climate_risk_fuzzy_v48_EN.ipynb` and run the cells in order:
   - **Data retrieval & local materialization** — exports and mosaics each variable to a shared 250 m grid.
   - **Fuzzification (Section 9.0b)** — auto-calibrates sigmoid thresholds to the 2024–2025 data and builds the 16 fuzzified indicators.
   - **Aggregation** — Fuzzy Gamma combination into Hazard, Exposure, Vulnerability and the final Climate Risk Index, for both years.
   - **Classification & dominant-driver mapping** — produces the risk-class rasters and per-subzone driver attribution.
   - **Sensitivity analysis** — runs the two Monte Carlo experiments (weights, fuzzy parameters).
   - **Figures export** — writes the final figures to `outputs/08_poster_figures/`.

All intermediate rasters and the final risk surfaces are kept, so every figure in the poster/manuscript is fully reproducible from the open-source code and open data alone.

## Citation

If you use this code, data pipeline, or results, please cite:

> León-Rueda, W. A., Ospina Noreña, J. E., Rincon Romero, V. O., & Barrientos Fuentes, J. C. (2026). *Spatial Analysis of Climate Risk for Water Security in the Colombian Orinoco Basin*. Water Security and Climate Change Conference 2026, Giessen, Germany.

## Authors

- **William Alfonso León-Rueda** — Faculty of Agricultural Sciences, Universidad Nacional de Colombia – Bogotá
- Jesús Efrén Ospina Noreña
- Víctor Orlando Rincón Romero
- Juan Carlos Barrientos Fuentes

## Acknowledgments

The authors thank Universidad Nacional de Colombia – Facultad de Ciencias Agrarias for institutional support, the **SDGnexus Network**, and the open-data providers whose datasets made this study possible: CHIRPS, ERA5-Land, Google Earth Engine, Dynamic World, WorldPop and IDEAM.


## Contact

William Alfonso León-Rueda — waleonr@unal.edu.co
