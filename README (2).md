# 🛰️ Annual Coastal Reclamation Monitoring with RGB Visualization (2015–2026)

A Google Earth Engine (GEE) script that combines **Sentinel-1 SAR** and **Sentinel-2 optical** imagery to detect and monitor land reclamation over a Region of Interest (ROI), year by year, complete with true-color visualization and an automated area-trend chart.

## 📌 Overview

Coastal reclamation — converting water bodies into land — is difficult to track using optical imagery alone because clouds frequently obscure coastal areas. This script solves that by using a **dual-sensor approach**:

- **Sentinel-1 (SAR, radar)** — cloud-penetrating, used to reliably classify **water vs. land** and compute reclaimed area statistics.
- **Sentinel-2 (optical, RGB)** — used purely for **visual context**, letting users see actual true-color imagery of the reclaimed area for each year.

The result is a year-by-year map layer stack (RGB + reclamation mask) plus a bar chart showing the trend of newly reclaimed land area (in hectares) from 2015 through 2026.

## ⚙️ How It Works

1. **Baseline (2014):** A Sentinel-1 VV backscatter mean is computed for 2014 and thresholded (`< 1`) to define the original water extent — this is the reference "before reclamation" condition.
2. **Yearly SAR Land Detection:** For each year in the list (2018–2026), a mean VV image is computed and thresholded (`≥ -17 dB`) to classify pixels as land.
3. **Reclamation Mask:** A pixel is flagged as "reclaimed" if it was water in the 2014 baseline **and** is classified as land in the target year.
4. **Yearly RGB Composite:** A cloud-filtered (`< 30%` cloudy pixels) Sentinel-2 median composite (Bands 4-3-2) is generated for the same year, purely for visual reference.
5. **Map Layers:** Both the RGB composite and the reclamation mask (orange overlay) are added to the map for every year, all switched **off by default** to keep the map lightweight.
6. **Area Calculation:** The reclaimed area is calculated in hectares using `ee.Image.pixelArea()` and `reduceRegion()` at 10 m scale.
7. **Trend Chart:** All yearly area values are compiled into a `FeatureCollection` and rendered as a column chart showing the annual reclamation trend.

## 🗺️ Outputs

| Output | Description |
|---|---|
| `RGB Tahun <year>` | True-color Sentinel-2 composite for visual inspection |
| `Area Reklamasi <year>` | Orange mask showing pixels classified as newly reclaimed land |
| Console print | Reclaimed area (Ha) printed per year |
| Column chart | Annual trend of reclamation area (2018–2026) |

## 🧩 Requirements

- A Google Earth Engine account
- A pre-defined `roi` (Region of Interest) geometry variable — must be set before running the script
- Access to the following GEE collections:
  - `COPERNICUS/S1_GRD` (Sentinel-1 GRD)
  - `COPERNICUS/S2_HARMONIZED` (Sentinel-2 Harmonized)

## ▶️ Usage

1. Open [Google Earth Engine Code Editor](https://code.earthengine.google.com/).
2. Define your `roi` geometry (e.g., draw a polygon over your coastal area of interest).
3. Paste the script into a new file.
4. Click **Run**.
5. Open the **Layers** panel, then toggle on the RGB and reclamation layers for the year(s) you want to inspect.
6. Check the **Console** tab for the printed area values and the trend chart.

## 🔧 Parameters You Can Adjust

- `tahunList` — the list of years to analyze.
- SAR land threshold (`sar.gte(-17)`) — adjust based on local backscatter characteristics.
- Baseline water threshold (`getSarMean(2014).lt(1)`) — adjust sensitivity of the initial water mask.
- Cloud filter (`CLOUDY_PIXEL_PERCENTAGE < 30`) — adjust for stricter/looser optical image selection.
- `scale` in `reduceRegion` — spatial resolution of the area calculation (default: 10 m).

## ⚠️ Notes & Limitations

- SAR-based water/land classification can be sensitive to sea state, rainfall, and radar incidence angle; thresholds may need local calibration.
- Sentinel-2 composites are for **visualization only** and are not used in the area calculation.
- Years without sufficient Sentinel-1/2 coverage over the ROI may return incomplete or empty results.
- Reclamation figures represent **net area change relative to the 2014 baseline**, not per-year incremental change.

## 📄 License

Feel free to adapt this script for research, monitoring, or educational purposes. Attribution appreciated.
