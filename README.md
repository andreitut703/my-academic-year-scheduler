# README

## Overview
This repository provides a full pipeline for forecasting seat occupancy at KTH Library, integrating data from multiple sources:
- **Seating occupancy**
- **Academic calendars**
- **Weather forecasts**
- **Library opening hours**

It uses several notebooks to demonstrate different modeling approaches (XGBoost and Prophet), with **OBT_3_inference_3.ipynb** functioning as the central notebook that covers both training and inference tasks in one place.

---

## 1. OBT_3_inference_3.ipynb

**Purpose**: Merges raw seating data with external features, trains Prophet models for each library location, performs forecasts on new time ranges, and saves or uploads resulting CSVs.  
Below are the key steps in this single file:

### Data Download and Merging
- Pulls occupancy percentages from `davnas/occupancy_perc` and converts them into a uniform 10-minute series, filling gaps via interpolation.
- Retrieves academic events from `andreitut/kth-academic-scraper`, matching them to the same timestamps.
- Incorporates weather forecasts (temperature, precipitation, wind) from Open-Meteo or a Hugging Face weather dataset, aligning them to the occupancy timeline.
- Adds library open/close hours from `davnas/date_kth` and determines if the library is open at each timestamp (`is_open`).

### Model Loading or Training
- Downloads Prophet models (one per library location) from `davnas/library_model`.
- By default, it uses these existing models, but you can adapt the notebook to retrain them on your newly merged data if needed.

### Inference
- Concatenates historical seat-occupancy data with future timestamps initially filled with `NaN`.
- Merges future weather forecasts and open/close hours for the next day(s).
- Uses each Prophet model to generate predictions, enforcing realistic constraints (e.g., occupancy forced to zero if `is_open == 0`).
- Produces final DataFrames (e.g., `occupancy_today.csv` and `forecast_tomorrow.csv`), optionally resampled to 30-minute intervals.

### Visualization & Upload
- Plots seat occupancy over time, highlighting actual vs. predicted periods.
- Demonstrates how to save results locally or upload them to Hugging Face.

By covering these tasks in one file, **OBT_3_inference_3.ipynb** lets you progress from raw data to final seat predictions without juggling multiple notebooks.

---

## 2. XGBOOST_NO_ROLLING.ipynb

A **separate** notebook that demonstrates how to train and validate an **XGBoost** model with a classical time-series split:
- Uses features like occupancy counts, weather variables, academic events, and `is_open`.
- Evaluates performance with MAE/RMSE.
- Serves as an alternative approach for those preferring XGBoost over Prophet or wishing to compare both models’ results.

---

## 3. Prophet_Training.ipynb

A **standalone** workflow showcasing how to train a Prophet model for seat occupancy:
- Treats seat usage as a logistic growth problem (e.g., `cap = 100`).
- Integrates regressors such as `is_open`, weather features, and academic event codes.
- Evaluates predictions against actual seat occupancy to measure accuracy.
  
If you prefer a simpler Prophet pipeline, **Prophet_Training.ipynb** offers a direct example without combining inference steps or multiple data sources in one place.

---

## 4. Prophet_NextDay.ipynb

An additional **Prophet-based** notebook focusing on **short-horizon** (next-day) forecasts:
- Reads a "future" dataset with upcoming timestamps.
- Applies a trained Prophet model, again enforcing zero occupancy when the library is closed.
- Provides a compact script for day-ahead predictions, useful if you want a minimal example before adopting the more expansive approach in **OBT_3_inference_3.ipynb**.

---

## 5. Scraping.ipynb

A **Selenium-based** crawler that updates academic schedules directly from the KTH Intranet:
- Parses exam periods, re-exam slots, self-study days, holidays, and normal schedules from the website’s HTML.
- Converts these data into daily event flags (Normal, Exam, Re-exam, etc.).
- Optionally uploads the resulting dataset to a Hugging Face repository so that the entire pipeline remains accurate as KTH’s schedule changes.

---

_This README provides an overview of the workflow and notebooks contained in the repository. For any questions, refer to the individual notebooks or contact the contributors._ 
