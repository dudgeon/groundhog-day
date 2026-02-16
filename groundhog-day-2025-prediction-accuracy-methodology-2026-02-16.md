# Methodology — Groundhog Day 2025 Accuracy Analysis

## Overview

This document describes the methodology used to evaluate the accuracy of 83 Groundhog Day prognosticators' predictions for 2025. Each animal's "shadow" or "no shadow" prediction was compared against actual observed temperatures at their location to determine whether they correctly forecast the remainder of winter.

**Analysis date:** February 16, 2026
**Weather API:** [Open-Meteo](https://open-meteo.com/) (free, open-source, no API key required)

---

## Step 1: Collecting Predictions

**Source:** [groundhog-day.com](https://groundhog-day.com)

All 83 recorded predictions for Groundhog Day 2025 (February 2, 2025) were collected. Each prediction includes:

- The prognosticator's name and home location (city, state/province, country)
- Their prediction: **shadow** (6 more weeks of winter) or **no shadow** (early spring)

The dataset includes traditional groundhogs as well as honorary prognosticators such as Scramble the Duck (Eastford, CT), Benny the Bass (Buckeye Lake, OH), Lucy the Lobster (Barrington, NS), Featherstone the Flamingo (Leominster, MA), Walnut the Hedgehog (Dayton, OH), Sylvia the Apex Armadillo (Apex, NC), and Athene the Burrowing Owl (Cape Coral, FL).

---

## Step 2: Geocoding Locations

Each prognosticator's home location was converted to latitude/longitude coordinates using the [Open-Meteo Geocoding API](https://geocoding-api.open-meteo.com/v1/search).

### Geocoding Strategy

For each location, the geocoder was queried with up to two search strategies:

1. **Primary query:** `"{city}, {state/province}"` — filtered by country code (`US` or `CA`) to prefer results in the correct country
2. **Fallback query:** `"{city}"` alone — used if the primary query returned no results

When multiple results were returned, the first result matching the expected country code was selected. If no country match was found, the top result was used as a best guess.

### API Call Example

```
GET https://geocoding-api.open-meteo.com/v1/search?name=Punxsutawney%2C%20PA&count=5&language=en
```

### Geocoding Results

- **81 of 83** locations were successfully geocoded
- **2 locations could not be resolved:**
  - **Fred la Marmotte** (Val-d'Espoir, QC, Canada) — a very small community in the Gaspé region not present in the geocoding database
  - **Dover Doug** (Dover Township, PA, USA) — townships are administrative areas rather than named settlements and do not geocode reliably

### Known Geocoding Caveats

Some locations share names with larger cities in other countries or states. The country code filter mitigates most mismatches, but a few edge cases likely resolved incorrectly:

- **Melbourne, ON, Canada** (#34 Middlemiss Mike) — likely resolved to Melbourne, Australia instead of the small Ontario community. The reported avg temp of 70.6°F strongly suggests Australian coordinates (Feb–Mar is summer in the Southern Hemisphere).
- **Stonewall, MB, Canada** (#14 Manitoba Merv) — likely resolved to Stonewall, LA, USA. The reported avg temp of 55.2°F is far too warm for Manitoba in February.
- Other locations with common names (Jackson, Concord, Hope, etc.) may have similar issues, though the country code filter catches most of these.

These geocoding mismatches affect the accuracy of individual results but do not significantly impact the overall trend (2025 was colder than baseline across the vast majority of North American locations).

---

## Step 3: Defining the Verification Window

**Window:** February 3, 2025 through March 16, 2025 (42 days / 6 weeks)

The Groundhog Day tradition states that if the groundhog sees its shadow, there will be "6 more weeks of winter." The verification window spans exactly 6 weeks starting the day after Groundhog Day (February 2), covering the full period the predictions are meant to describe.

---

## Step 4: Retrieving Historical Weather Data

**Source:** [Open-Meteo Historical Weather API](https://archive-api.open-meteo.com/v1/archive)

For each of the 81 verifiable locations, five API calls were made to retrieve daily mean 2-meter air temperature (`temperature_2m_mean`) in degrees Fahrenheit:

### 4a. Observed 2025 Temperatures

- Daily mean temperatures for February 3 – March 16, 2025
- Averaged across the full 6-week window to produce a single mean temperature

### API Call Example

```
GET https://archive-api.open-meteo.com/v1/archive?latitude=40.94&longitude=-78.97&start_date=2025-02-03&end_date=2025-03-16&daily=temperature_2m_mean&temperature_unit=fahrenheit&timezone=auto
```

### 4b. Baseline Temperatures (2022–2024)

- The same Feb 3 – Mar 16 date range was queried for each of the three preceding years (2022, 2023, 2024) — one API call per year
- Each year's daily temperatures were averaged into a single mean
- The three yearly means were then averaged together to produce the **baseline temperature** for the location

### Why a 3-Year Baseline?

The standard approach in climatology is a 30-year normal (e.g., 1991–2020). We used a shorter 3-year window (2022–2024) because:

1. **Simplicity:** Fewer API calls, simpler computation
2. **Recency:** Captures what people recently experienced as "normal" rather than a decades-old average
3. **Climate responsiveness:** Better reflects current climate trends rather than smoothing over 30 years of change

**Trade-off:** A 3-year baseline is noisier. If one baseline year was anomalously warm or cold, it skews the comparison. A 30-year normal would be more robust and less sensitive to individual outlier years.

### Rate Limiting

API calls were spaced with a 150ms delay between requests to avoid overwhelming the Open-Meteo servers. With ~5 calls per location × 83 locations = ~415 total calls, the data collection took approximately 3–5 minutes.

---

## Step 5: Calculating the Temperature Anomaly

For each location, a **temperature anomaly** was calculated:

```
anomaly = observed_2025_mean − baseline_mean
```

- A **negative anomaly** (below baseline) indicates the 6-week period was **colder than normal** → extended winter
- A **positive anomaly** (above baseline) indicates the period was **warmer than normal** → early spring

All anomaly values are reported in degrees Fahrenheit (°F).

---

## Step 6: Classifying Actual Outcomes

Each location was classified based on its temperature anomaly:

| Anomaly | Classification | Emoji |
|---------|---------------|-------|
| Below −2.0°F | Extended winter (well below normal) | 🥶 |
| −2.0°F to 0.0°F | Extended winter (slightly below normal) | ❄️ |
| 0.0°F to +2.0°F | Early spring (slightly above normal) | 🌤️ |
| +2.0°F to +4.0°F | Early spring (above normal) | 🌸 |
| Above +4.0°F | Early spring (well above normal) | ☀️ |

**The classification threshold is 0.0°F** — any location with a negative anomaly (even −0.1°F) counts as "extended winter," and any location with a positive anomaly counts as "early spring." The emoji tiers are purely cosmetic to indicate the magnitude of the deviation.

---

## Step 7: Scoring Predictions

Each prognosticator's prediction was scored against the actual outcome at their location:

| Predicted | Actual | Verdict |
|-----------|--------|---------|
| Shadow (winter) | Extended winter (anomaly < 0) | ✅ Correct |
| Shadow (winter) | Early spring (anomaly ≥ 0) | ❌ Incorrect |
| No shadow (spring) | Early spring (anomaly ≥ 0) | ✅ Correct |
| No shadow (spring) | Extended winter (anomaly < 0) | ❌ Incorrect |

Locations with no available weather data (geocoding failures) were marked as 🤷 and excluded from accuracy tallies.

### Final Results

| Group | Correct | Total | Accuracy |
|-------|---------|-------|----------|
| All verifiable predictions | 33 | 81 | 40.7% |
| "Shadow" predictors (winter) | 27 | 34 | 79.4% |
| "No shadow" predictors (spring) | 6 | 47 | 12.8% |

---

## Limitations and Caveats

### 1. Binary Classification at Zero
The threshold between "early spring" and "extended winter" is exactly 0.0°F deviation from baseline. A location at −0.1°F is classified identically to one at −6.0°F. Several predictions were decided by fractions of a degree (e.g., General Beauregard Lee at +0.1°F, Athene the Burrowing Owl at +0.2°F). The anomaly values in the data table provide this nuance, but the binary ✅/❌ scoring does not.

### 2. Temperature as Sole Metric
We used only mean daily air temperature. The perception of "spring arriving" also depends on:
- Snowfall, snow cover, and snow melt timing
- Frost-free date advancement
- Phenological events (tree budding, flower blooming, bird migration)
- Precipitation type and amount
- Day length and sunshine hours

A more comprehensive analysis could weight multiple factors, but temperature is the most universal, readily available, and comparable metric across 83 diverse locations.

### 3. Geocoding Accuracy
As noted in Step 2, some locations likely geocoded to the wrong place. Locations in small towns, townships, or communities sharing names with larger international cities are most at risk. This affects individual row accuracy but not the overall trend.

### 4. Baseline Period Length
Three years (2022–2024) is short. One anomalous year can meaningfully shift the baseline. A 30-year climatological normal would be more robust. We chose the shorter window for simplicity and because it better represents "recent normal" conditions people have experienced.

### 5. Spatial Resolution of Weather Data
Open-Meteo provides gridded weather model data (reanalysis), not point station observations. The reported temperature represents a grid cell (~1–11 km resolution), not the exact location of the groundhog's burrow. For this analysis, this resolution is more than sufficient.

### 6. This Is Not Science
Groundhog Day predictions are a beloved cultural tradition, not a scientific forecasting method. This analysis is meant to be fun. Please do not fire your local meteorologist and replace them with a groundhog.

---

## Data Sources

| Source | URL | Purpose |
|--------|-----|---------|
| Groundhog Day predictions | https://groundhog-day.com | All 83 prognosticators' predictions for 2025 |
| Open-Meteo Archive API | https://archive-api.open-meteo.com/v1/archive | Daily mean temperatures (2022–2025) |
| Open-Meteo Geocoding API | https://geocoding-api.open-meteo.com/v1/search | City → lat/lon coordinate lookup |

---

## Tools

| Component | Tool |
|-----------|------|
| Data collection | Python 3 script (standard library: `urllib`, `json`, `re`) |
| Weather data | Open-Meteo Historical Weather API (free, no API key) |
| Geocoding | Open-Meteo Geocoding API (free, no API key) |
| Temperature unit | Fahrenheit (°F) |
| Timezone handling | Location-native via `timezone=auto` parameter |
| Table formatting | Manual edit, one row at a time |

---

## Reproducibility

This analysis can be fully reproduced by:

1. Parsing the 83 groundhog sighting locations from [groundhog-day.com](https://groundhog-day.com)
2. Geocoding each location via the Open-Meteo Geocoding API
3. Querying the Open-Meteo Archive API for daily mean temperatures:
   - **2025 window:** `start_date=2025-02-03&end_date=2025-03-16`
   - **Baseline years:** Same date window for 2022, 2023, and 2024
4. Computing the arithmetic mean temperature for each window
5. Averaging the three baseline years into a single baseline mean
6. Subtracting baseline from 2025 to get the anomaly
7. Classifying each location (positive = early spring, negative = extended winter)
8. Comparing the classification to the prediction

Historical weather data is stable and deterministic — the same queries should return identical results on future runs. All data was fetched on February 16, 2026.
