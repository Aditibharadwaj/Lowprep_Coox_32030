# COOX — Outskirts Booking Geo-Blocking Analysis

## Overview
This project analyzes 500+ non-serviceable / outskirts booking records across 13 Indian cities to identify high-frequency problem zones, cluster them geospatially, and produce actionable pincode-level geo-blocking recommendations.

---

## Repository Structure
```
coox_geoblocking_analysis/
│
├── coox_geoblocking_analysis.ipynb   # Main analysis notebook
│
├── outputs/
│   ├── coox_analysis_summary.json     # Full machine-readable results (clusters, pincodes, financials)
│   ├── coox_cluster_blocking_list.csv # 63-row blocking list with severity & loss estimates
│   ├── coox_isolated_pincode_watchlist.csv  # Borderline pincodes for monitoring
│   ├── coox_financial_sensitivity.csv # 3-scenario revenue loss analysis
│   ├── coox_epsilon_crossvalidation.csv     # DBSCAN hyperparameter tuning results
│   │
│   ├── coox_cluster_map.html          # Master interactive cluster map (all cities)
│   ├── coox_analysis_dashboard.html   # Analytics dashboard
│   ├── coox_kdistance_plot.html       # K-distance elbow plot (epsilon selection)
│   │
│   ├── city_map_ahmedabad.html
│   ├── city_map_bengaluru.html
│   ├── city_map_chennai.html
│   ├── city_map_hyderabad.html
│   ├── city_map_indore.html
│   ├── city_map_jaipur.html
│   ├── city_map_kolkata.html
│   ├── city_map_lucknow.html
│   ├── city_map_mumbai.html
│   ├── city_map_navi_mumbai.html
│   ├── city_map_pune.html
│   └── city_map_surat.html
```
---

## Methodology

### 1. Data Loading & Preprocessing
- Loaded 546 outskirts/non-serviceable booking records
- Extracted/validated pincodes from address strings using regex
- Reverse-geocoded coordinates to fill missing pincodes

### 2. Epsilon Selection (DBSCAN Hyperparameter Tuning)
- Generated k-distance plot (k = MIN_SAMPLES = 3) to identify elbow
- Cross-validated over ε ∈ {3, 5, 8} km using silhouette score

| ε (km) | Clusters | Noise | Coverage | Silhouette |
|--------|----------|-------|----------|------------|
| 3      | 63       | 235   | 57.0%    | **0.755**  |
| 5      | 43       | 139   | 74.5%    | 0.515      |
| 8      | 32       | 86    | 84.2%    | 0.484      |

**Selected: ε = 3 km (highest silhouette score → tightest clusters)**

---

### 3. DBSCAN Clustering
- Algorithm: DBSCAN with Haversine distance metric  
- Parameters: ε = 3 km, min_samples = 3  
- Result: **63 clusters**, 235 noise points  

---

### 4. Cluster Severity Tiering

| Severity | Booking Count | Action |
|----------|--------------|--------|
| High     | ≥ 10         | Block Week 1–2 |
| Medium   | 5–9          | Block Month 1 |
| Low      | 3–4          | Block Month 2+ |

---

### 5. Pincode Extraction
- Primary: parsed from address strings  
- Fallback: reverse geocoding via Nominatim  
- For clusters without pincodes: convex hull polygons provided  

---

### 6. Financial Impact Estimation
- Average booking value: ₹15,000 + ₹200 processing fee  
- Total estimated loss: **₹83,00,000**  
- Cluster-attributed loss: **₹47,27,200 (57%)**

---

## Key Findings

- **Top affected cities:** Bengaluru, Pune, Chennai, Hyderabad, Ahmedabad  
- **Highest-priority cluster:** Ravet (Pune) — 18 bookings, ₹2,73,600 loss  
- **High-severity clusters across pincodes:**  
  382722, 410506, 411033, 412101, 560049, 560115, 562125, 603112, 654321  

- **7 pincodes on watchlist** (low frequency)  
- **235 noise bookings (43%)** → isolated cases (no geo-blocking)
---

## Requirements

pandas  
numpy  
scikit-learn  
folium  
geopy  
plotly  

Install dependencies:

pip install pandas numpy scikit-learn folium geopy plotly

---

## Dataset

https://docs.google.com/spreadsheets/d/19V2nf3Hdi2tqhv_jrrzJeCXYMaJnYUdllm8eZX2exU/edit?gid=86078783#gid=86078783

---

## Summary

This solution combines:
- Geospatial clustering (DBSCAN with tuned ε)
- Pincode-level decisioning
- Financial impact estimation
- Phased rollout strategy

to deliver a scalable, data-driven geo-blocking framework for COOX.
