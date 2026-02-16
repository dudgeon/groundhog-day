# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a data journalism repository analyzing the accuracy of Groundhog Day 2025 predictions. It contains Markdown documents with research findings, methodology, and supporting context — no application code.

## Repository Structure

- **groundhog-sightings-2025.md** — Main analysis: table of 83 prognosticators with predictions, actual weather outcomes, accuracy verdicts, and summary statistics
- **groundhog-day-2025-prediction-accuracy-methodology-2026-02-16.md** — Methodology: data collection, geocoding, verification window (Feb 3–Mar 16, 2025), temperature baseline, anomaly thresholds, scoring, and known limitations
- **wiarton-news-summary.md** — Historical local context for Wiarton, Ontario around Groundhog Day 2022–2026

## Data Sources

- **Predictions:** groundhog-day.com (WebFetch access granted in `.claude/settings.local.json`)
- **Weather verification:** Open-Meteo Historical Weather API (no API key required)
- **Geocoding:** Open-Meteo Geocoding API

## Key Context

- The data collection was done with Python (urllib, json, re) and Open-Meteo APIs, but the script is not committed to this repo
- Temperature anomalies are calculated against a 2022–2024 baseline
- Known geocoding issues: Melbourne, ON may resolve to Australia; Stonewall, MB to Louisiana
- Key finding: shadow-seers (predicting extended winter) were 79.4% accurate vs 12.8% for early-spring predictors
