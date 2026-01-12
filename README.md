# Spatio-Temporal Relative Humidity Analysis Using ERA5-Land in Agadez region — April 2024 🌦️

---

## Introduction
This repository contains a Google Earth Engine (GEE) script to compute and visualize hourly near-surface relative humidity (RH) across a predefined Area of Interest (AOI) in Agadez region for early April 2024. The analysis converts ERA5-Land hourly temperature and dew point (Kelvin) to RH (%) using the Magnus approximation, then produces a time series of area-mean RH and a classified choropleth map for the latest hour in the range.

---

## Objectives
- Compute hourly relative humidity from ERA5-Land temperature and dew point at 2 m height. 📊
- Produce a spatial map of RH using a 10-class palette with title-based symbols and unique IDs for each class. 🗺️
- Generate an hourly time-series of mean RH over the AOI for the selected period. ⏱️
- Provide an interactive legend with emojis for easier stakeholder interpretation and reproducible exports. 🧾

---

## Methods
1. Data acquisition
   - Dataset: `ECMWF/ERA5_LAND/HOURLY` (GEE ImageCollection).
   - AOI: FeatureCollection asset `projects/gee-trial2/assets/Central_Chanda/Division_Boudry_Chandrapur`.
   - Date window: 2024-04-01 to 2024-04-10.

2. Preprocessing
   - Filter ERA5-Land by AOI and date range.
   - Select `temperature_2m` and `dewpoint_temperature_2m` bands from each hourly image.

3. RH calculation
   - Convert Kelvin to Celsius: T_C = temperature_2m - 273.15; Td_C = dewpoint_temperature_2m - 273.15.
   - Compute RH (%) using Magnus-Tetens approximation:
     RH = 100 * (exp((17.625 * Td) / (243.04 + Td)) / exp((17.625 * T) / (243.04 + T)))

4. Visualization
   - Time series: area-mean RH vs. time (ui.Chart) using ee.Reducer.mean().
   - Map layer: latest RH image clipped to AOI, displayed with a 10-color palette and an on-map legend that includes title, emoji, numeric range, and a unique ID.

---

## AOI / Objects
- AOI object: Earth Engine FeatureCollection:
  `study area`.
- Key objects created:
  - `era5` (ee.ImageCollection filtered by AOI & dates)
  - `rhCollection` (ee.ImageCollection of RH images)
  - `latestRH` (ee.Image, latest RH frame)
  - `chart` (ui.Chart image series)
  - `legend` (ui.Panel for emoji/title-based legend)

---

## Emoji-enhanced Title-based Symbols (Legend)
To make the legend more user-friendly and uniquely identifiable, we use title-based descriptors with emojis and unique IDs. These can be used in the UI legend, in export filenames, and in documentation.

Mapping (Emoji • ID • Title • Range • Color):
- 🔵 RH_01 — Very Low — 0–10% — #0000FF  
- 🟣 RH_02 — Low — 11–20% — #8A2BE2  
- 💗 RH_03 — Low-Moderate — 21–30% — #FF00FF  
- 🟤 RH_04 — Moderate — 31–40% — #800000  
- 🔴 RH_05 — Moderate-High — 41–50% — #FF0000  
- 🟠 RH_06 — High — 51–60% — #FF7F00  
- 🟡 RH_07 — Very High — 61–70% — #FFFF00  
- 💚 RH_08 — Very High+ — 71–80% — #7FFF00  
- 🟩 RH_09 — Near Saturation — 81–90% — #00FF00  
- 🧊 RH_10 — Saturation — 91–100% — #00FFFF

Notes:
- The emoji provides a quick visual cue for stakeholders.
- The unique ID (RH_01 ... RH_10) should be included in export names and metadata for traceability (e.g., `RH_01_VeryLow_2024-04-05.tif`).

---

## How to show emoji-based legend in Earth Engine (snippet)
Paste this into the Earth Engine Code Editor to build an emoji + title + id legend. This snippet extends the previous title-based legend to include emojis.

```javascript
// Title-based symbol descriptors with emoji + id
var classDescriptors = [
  { id: 'RH_01', emoji: '🔵', title: 'Very Low', range: '0–10%',   color: '#0000FF' },
  { id: 'RH_02', emoji: '🟣', title: 'Low',      range: '11–20%',  color: '#8A2BE2' },
  { id: 'RH_03', emoji: '💗', title: 'Low-Moderate', range: '21–30%', color: '#FF00FF' },
  { id: 'RH_04', emoji: '🟤', title: 'Moderate', range: '31–40%',  color: '#800000' },
  { id: 'RH_05', emoji: '🔴', title: 'Moderate-High', range: '41–50%', color: '#FF0000' },
  { id: 'RH_06', emoji: '🟠', title: 'High',     range: '51–60%',  color: '#FF7F00' },
  { id: 'RH_07', emoji: '🟡', title: 'Very High', range: '61–70%', color: '#FFFF00' },
  { id: 'RH_08', emoji: '💚', title: 'Very High+', range: '71–80%', color: '#7FFF00' },
  { id: 'RH_09', emoji: '🟩', title: 'Near Saturation', range: '81–90%', color: '#00FF00' },
  { id: 'RH_10', emoji: '🧊', title: 'Saturation', range: '91–100%', color: '#00FFFF' }
];

var rhPalette = classDescriptors.map(function(d) { return d.color; });

// Add map layer with descriptive name + timestamp
var latestRH = rhCollection.sort('system:time_start', false).first();
var latestTime = ee.Date(latestRH.get('system:time_start')).format('YYYY-MM-dd HH:mm').getInfo();
var layerName = 'Latest RH (%) — ' + latestTime;
Map.addLayer(latestRH.clip(aoi), {min:0, max:100, palette: rhPalette}, layerName);

// Build emoji legend
var legend = ui.Panel({ style: { position: 'bottom-left', padding: '8px 15px' }});
legend.add(ui.Label('🌦️ RH (%) Classes — Emoji • Title • Range • ID', { fontWeight: 'bold', fontSize: '14px' }));

classDescriptors.forEach(function(d) {
  var colorBox = ui.Label({ style: { backgroundColor: d.color, padding: '8px', margin: '0 6px 0 0' }});
  var description = ui.Label(d.emoji + '  ' + d.title + ' (' + d.range + ')  —  ' + d.id, { margin: '0 0 4px 0' });
  var row = ui.Panel({ widgets: [colorBox, description], layout: ui.Panel.Layout.Flow('horizontal') });
  legend.add(row);
});
Map.add(legend);
```

---

## Analysis
- Temporal: The script produces an hourly time-series (area-mean RH) for the AOI — use it to detect diurnal cycles and short-term trends. 📈
- Spatial: The emoji-enhanced legend with unique IDs helps quickly interpret classes across stakeholders and link exported assets to class metadata. 🗂️
- Interpretation caveats: ERA5-Land resolution (~9 km) and Magnus formula limitations still apply.

---

## Outputs
- Interactive Earth Engine outputs:
  - Chart: "Relative Humidity Time Series (April 2024)" — mean RH (%) vs date/time for the AOI.
  - Map: "Latest RH (%) — <timestamp>" with color classification and emoji legend.
- Export recommendations:
  - Use unique IDs in file names and metadata, e.g., `RH_05_ModerateHigh_2024-04-05.tif`.
  - Provide an export mapping JSON or CSV containing: id, emoji, title, color, numeric range for downstream users.

---

## How to run (Google Earth Engine Code Editor)
1. Open https://code.earthengine.google.com/
2. Paste the full script (including the emoji legend snippet).
3. Ensure you have read access to the AOI asset or replace it with your own geometry.
4. Run the script — chart and emoji legend will appear in Console/Map.
5. (Optional) Export images/tables and include the class id in filenames and metadata.

---

## Caveats and Notes
- ERA5-Land variables are in Kelvin — the script converts to Celsius prior to RH computation.
- Spatial resolution (~9 km) limits fine-scale microclimate detection.
- Emojis improve readability in many UIs but may not render in some export contexts; always include the textual title and ID in metadata.

---

## 🏆 Author

Souleymane Maman Nouri Souley  
* GIS Expert  
* ssouley@uta.cv  
* +22799717923  

---

## License
Specify your preferred license (e.g., MIT). If unspecified, add one before reuse.
