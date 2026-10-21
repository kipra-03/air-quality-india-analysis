# Air Quality in India: Data Analysis and AQI Forecasting Baseline
**Author:** Kipra
## Problem Statement
Air pollution is a serious public-health issue in India. This project analyses daily air-quality data from 26 Indian cities to understand how pollution varies across cities, seasons and years, and tests whether simple machine-learning models can forecast the next day's AQI better than a naive baseline.

## Objective
- Clean and understand a real-world, messy dataset
- Answer specific questions through EDA
- Engineer features for forecasting
- Evaluate models honestly against a baseline

## Dataset
- **Name:** Air Quality Data in India (2015-2020), file `city_day.csv`
- **Source:** Kaggle (originally from India's Central Pollution Control Board, CPCB)
- **Size:** 29,531 rows, 16 columns, 26 cities, 2015-01-01 to 2020-07-01
- **Contents:** daily pollutant readings (PM2.5, PM10, NO, NO2, NOx, NH3, CO, SO2, O3, Benzene, Toluene, Xylene), AQI and AQI_Bucket

## Technologies
Python, pandas, NumPy, Matplotlib, Seaborn, scikit-learn, Jupyter (VS Code)

## Project Structure
```
air-quality-india/
├── data/        # raw and processed CSV files
├── notebook/    # numbered notebooks, run in order
├── README.md
├── requirements.txt
└── .gitignore
```

| Notebook | Purpose |
|---|---|
| `01_data_understanding.ipynb` | Load data, understand columns, clean |
| `02_eda.ipynb` | Exploratory analysis and findings |
| `03_feature_engineering.ipynb` | Create features for forecasting |
| `04_ml_baseline.ipynb` | Baseline and model comparison |

## Methodology

### Data cleaning
| Issue | Finding | Action |
|---|---|---|
| Date stored as text | `Date` was a string | Converted to datetime |
| Time features | Needed for trend/season analysis | Added Year, Month, Season |
| Duplicates | 0 duplicate rows, 0 duplicate City+Date pairs | None needed |
| Xylene | 61% missing, 100% missing in about 12 cities | Dropped |
| Short gaps | Gaps in pollutant columns | Filled from the same city's neighbouring days (max 3 days) |
| Long gaps | Some cities never measured some pollutants | Left as missing |
| Negative values | 0 found | None needed |
| Ahmedabad CO | CO identical to NO in 1,346 rows, likely a data-entry error | Set CO, AQI and AQI_Bucket to missing for those rows |
| AQI above 500 | 131 rows remain, mostly Delhi, Patna, Gurugram, Lucknow | Kept, flagged in `AQI_Above_500` |

### Feature engineering
DayOfWeek, IsWeekend, DayOfYear, AQI_Yesterday, PM25_7day_avg (previous 7 days only, to avoid leakage) and AQI_Trend.

## Key Findings

1. **2020 lockdown:** The first all-city comparison was misleading because the number of cities with PM2.5 data grew from 7 (2015) to 26 (2020). Using only cities present in all six years, average March-June PM2.5 was 58.8 in 2019 and 37.1 in 2020. This is consistent with a lockdown effect, but weather was not controlled.
2. **Seasons:** Winter had the highest average AQI (204.0) and Monsoon the lowest (100.3). Winter and Post-Monsoon also had the widest spread of values.
3. **Pollutants:** AQI correlates most strongly with PM10 (0.90) and PM2.5 (0.89), and least with Benzene (0.01). PM2.5 and PM10 are strongly correlated with each other (0.84).
4. **Delhi:** PM2.5 peaks in winter and is lowest in the monsoon every year. January-June average PM2.5 was highest in 2016 (131.9) and lowest in 2020 (80.5).
5. **Cities:** Among cities with 500+ AQI readings, Delhi, Patna, Gurugram and Lucknow had the highest average AQI, and Thiruvananthapuram the lowest (75.9). Cities were measured over different years, so this is a rough ranking.

## ML Roadmap and Results

**Task:** predict a city's AQI for a day using only information from before that day (regression).
**Metric:** MAE (mean absolute error, in AQI points).
**Split:** by time, because a random split would leak the future into training.
**Not used as inputs:** today's pollutant readings and AQI_Bucket (data leakage).

| Test year | Model | MAE |
|---|---|---|
| 2020 (train 2015-2019) | Baseline (today = yesterday) | **16.9** |
| 2020 | Linear Regression | 17.9 |
| 2020 | Linear Regression (predicting change) | 17.9 |
| 2020 | Random Forest | 19.0 |
| 2019 (train 2015-2018) | Baseline | **24.0** |
| 2019 | Linear Regression | 24.3 |

**Conclusion:** no model beat the baseline. A likely reason is that yesterday's AQI already explains most of today's AQI (correlation 0.91), so the other features add little. This explanation was not tested directly.

## Limitations
- Cities have different monitoring periods, so city and year comparisons are not perfectly fair.
- Ahmedabad's CO issue is likely a data error, not confirmed.
- No weather data, so seasonal explanations (temperature inversion, low wind) are not proven by this dataset.
- 2020 includes the lockdown, which makes it an unusual test year.
- MAE values from different test years cannot be compared with each other.
- The data ends in July 2020.

## Future Improvements
- Add weather data (temperature, wind, rainfall)
- Predict the AQI category (classification)
- Forecast several days ahead
- Try gradient boosting, cross-validation and hyperparameter tuning
- Deploy a Streamlit app

## How to Run
1. Download the dataset from Kaggle ("Air Quality Data in India 2015-2020") and place `city_day.csv` in `data/`.
2. Install the requirements:
```
   pip install -r requirements.txt
```
3. Run the notebooks in `notebook/` in order (01 to 04).

## Acknowledgements
Data originally from the Central Pollution Control Board (CPCB), India, via Kaggle.
