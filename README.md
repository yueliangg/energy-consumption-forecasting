# Energy Consumption Forecasting (Time Series)

Forecasting a building's monthly gas, electricity and water consumption for the next 60 months using ARIMA, SARIMA and exponential smoothing models, trained on 33 years of data.

**Final result:** a seasonal SARIMA model forecasts electricity with a 3.3% test MAPE. Each utility gets its own model, chosen from its trend and seasonal behaviour.

---

## Problem Overview

| | |
|---|---|
| **Task** | Univariate time series forecasting |
| **Dataset** | `CA2-Energy-Consumption-Data.csv`: 397 monthly records (Jan 1990 to Jan 2023) |
| **Targets** | Gas (tons), Electricity (MWh), Water (tons) |
| **Horizon** | 60 months (Feb 2023 to Jan 2028), with 95% confidence intervals |

---

## Tech Stack

Python, pandas, NumPy, statsmodels, scikit-learn, SciPy, Matplotlib, Seaborn, Jupyter Notebook

---

## Workflow

1. **Data Preparation:** parsed day-first dates and validated a continuous monthly frequency.
2. **Exploratory Data Analysis:** Z-score outlier detection, additive seasonal decomposition and correlation analysis.
3. **Chronological Split (80/10/10):** no shuffling, to prevent data leakage.
   - Train: Jan 1990 to May 2016
   - Validation: Jun 2016 to Aug 2019
   - Test: Sep 2019 to Jan 2023
4. **Stationarity Testing:** Augmented Dickey-Fuller (ADF) test on the training data to set the differencing order.
5. **Baseline Modelling:** candidate models for each utility, compared against Seasonal Naive and Mean benchmarks.
6. **Hyperparameter Tuning:** grid search over ARIMA and SARIMA orders on the validation set.
7. **Test Evaluation:** final models retrained on train and validation data, then scored once on the unseen test set.
8. **Forecasting:** models retrained on the full dataset to produce a 60-month forecast.

---

## Exploratory Findings

| Utility | Trend | Seasonality | Residual Noise | Forecastability |
|---|---|---|---|---|
| Gas | No long-term growth, slowly drifting level | Weak (±1 ton) | Low | Moderate |
| Electricity | Rising until about 2013, then levelling off | Strong (-95 to +135 MWh) | Low | Very good |
| Water | Wandering, no clear direction | Weak (±40 tons) | Very high | Poor |

- The utilities are only weakly correlated (all |r| < 0.4), so each one is modelled independently.
- A single anomaly in **February 2014** shows gas use doubling while water use nearly stopped. This points to one operational event rather than two separate errors.

---

## Results

### Final Models: Test Set Performance

| Utility | Model | Validation MAPE | Test MAPE | Test RMSE |
|---|---|---|---|---|
| Electricity | SARIMA(0,1,1)(1,1,1)12 | 2.22% | **3.30%** | 44.46 |
| Gas | ARIMA(1,0,1) | 15.39% | **15.03%** | 5.06 |
| Water | ARIMA(3,0,0) | 19.57% | **17.79%** | 106.57 |

### Impact of Seasonality (Validation MAPE)

| Utility | Without Seasonality | With Seasonality |
|---|---|---|
| Electricity | 9.42% | **2.22%** (76% lower error) |
| Gas | **15.39%** | 28.62% |
| Water | **19.57%** | 21.45% |

---

## Key Insights

- **Modelling seasonality was the most important decision.** It cut electricity forecast error by 76%, while hyperparameter tuning improved results by less than 1%.
- **Seasonality only helps when it is present.** For gas and water it made the forecasts worse, because the models fitted noise in their weak seasonal patterns.
- **Water is largely unpredictable from its own history.** The best model only slightly beats a simple mean baseline.
- **Gas is non-stationary** (ADF p = 0.32). Its mean-reverting forecast under-predicted the higher consumption in 2021 and 2022.

---

## Repository Structure

```
├── energy_forecasting.ipynb             # Full analysis notebook
├── CA2_PartB_Energy_Forecasting.pptx    # Presentation slides
├── data/
│   └── CA2-Energy-Consumption-Data.csv
└── README.md
```

---

## How to Run

```bash
git clone <this-repo-url>
cd <repo-folder>
pip install pandas numpy statsmodels scikit-learn scipy matplotlib seaborn
jupyter notebook energy_forecasting.ipynb
```

Set the dataset path in the loading cell:

```python
df = pd.read_csv("data/CA2-Energy-Consumption-Data.csv")
```

Then select **Kernel > Restart & Run All**. The grid search takes a few minutes.

---

## Limitations and Future Work

- **Gas:** testing ARIMA with first-order differencing (d = 1), as the ADF result suggests, should help it follow level shifts.
- **Electricity:** the forecast extends recent growth even though the long-term trend has flattened. A damped-trend model would give a more conservative projection.
- **Water:** external drivers such as occupancy, weather or operations data are needed for meaningful forecasts.
- **Validation:** rolling-origin (walk-forward) validation would give a more robust estimate of accuracy than a single validation window.

---

## Author

**Thawdar Lin Wai**
Diploma in Applied AI and Analytics, Singapore Polytechnic
