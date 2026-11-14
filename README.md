# Energy Consumption Forecasting

Forecasting a building's monthly gas, electricity and water consumption for the next 60 months with ARIMA and SARIMA models, trained on 33 years of monthly data.

**Result:** electricity is forecast with a **2.61% test MAPE** using SARIMA(1,0,2)(0,1,1)12. Each utility has its own model, chosen by validation error across four rolling validation windows. The test set was used only once, for the final evaluation.

---

## Problem Overview

| | |
|---|---|
| **Task** | Univariate time series forecasting |
| **Dataset** | `CA2-Energy-Consumption-Data.csv`: 397 monthly records, January 1990 to January 2023 |
| **Targets** | Gas (tons), Electricity (MWh), Water (tons) |
| **Horizon** | 60 months (February 2023 to January 2028), with 95% confidence intervals |
| **Purpose** | Support utility budgeting and capacity planning |

---

## Results

### Final Models

| Utility | Model | Validation MAPE (4-window average) | Test MAPE | Test RMSE | Bias (actual minus predicted) |
|---|---|---|---|---|---|
| Electricity | SARIMA(1,0,2)(0,1,1)12 | 2.83% | **2.61%** | 36.76 MWh | -7.6 MWh |
| Gas | ARIMA(1,0,1) | 17.55% | **15.03%** | 5.06 tons | +3.4 tons |
| Water | ARIMA(1,0,1) | 30.42% | **17.77%** | 106.38 tons | +19.7 tons |

All final models converged. Residual diagnostics (Ljung-Box test) show white-noise residuals for gas and water, and a weak remaining pattern at 24 lags for electricity.

### 60-Month Forecast (average per month)

| Year | Electricity (MWh) | Gas (tons) | Water (tons) |
|---|---|---|---|
| 2024 | 1,055 | 24.4 | 475 |
| 2025 | 1,061 | 23.9 | 484 |
| 2026 | 1,066 | 23.8 | 485 |
| 2027 | 1,072 | 23.8 | 485 |

Electricity keeps its strong yearly cycle with slow growth of about 0.5% per year, consistent with the flat trend since 2013. Gas and water return to their long-run averages, with wide confidence intervals.

---

## Approach

1. **Data preparation:** parsed day-first dates and confirmed a continuous monthly frequency with no missing values.
2. **Exploratory data analysis:** Z-score outlier detection, additive seasonal decomposition and correlation analysis.
3. **Chronological split (80/10/10):** no shuffling, to prevent data leakage.
   - Train: January 1990 to May 2016
   - Validation: June 2016 to August 2019
   - Test: September 2019 to January 2023
4. **Stationarity analysis:** ADF test, rolling statistics and visual inspection on the training data, used together to decide whether each series needs differencing.
5. **Baseline modelling:** candidate models for each utility, compared against Seasonal Naive and Mean benchmarks.
6. **Hyperparameter tuning:** grid search over ARIMA and SARIMA orders, including the differencing order, scored by average RMSE over four rolling validation windows. When configurations scored within 1% of each other, the simpler model was chosen.
7. **Test evaluation:** final models retrained on training and validation data, then scored once on the unseen test set, followed by residual analysis.
8. **Forecasting:** models retrained on the full dataset to produce the 60-month forecast.
9. **Model improvement analysis:** effect of seasonality, of tuning, of the February 2014 outlier, and of forced seasonal alternatives.

---

## Key Findings

### Exploratory analysis

| Utility | Trend | Seasonality | Residual noise | Forecastability |
|---|---|---|---|---|
| Electricity | Rising until about 2013, then levelling off | Strong (-95 to +135 MWh) | Low | Very good |
| Gas | No long-term growth, slowly drifting level | Weak (about ±1 ton) | Low | Moderate |
| Water | Wandering, no clear direction | Weak (-38 to +45 tons) | Very high | Poor |

- The utilities are only weakly correlated (all |r| < 0.4), so each one is modelled independently.
- One anomaly, in February 2014, shows gas use doubling while water use nearly stopped. Smoothing it changed validation error by less than 2%, so it was kept.

### What worked

- **Seasonality for electricity:** cut validation MAPE from 9.42% to 2.22%, a 76% reduction.
- **Differencing only where needed:** only electricity was differenced. For gas and water, d = 1 made forecasts worse.
- **A seasonal difference alone for electricity:** d = 0 with D = 1 beat d = 1 with D = 1, a 10.1% improvement over the baseline and a more realistic long-term forecast.
- **Rolling-origin validation:** on the single original validation window, a doubly differenced electricity model looked best (RMSE 29.47), but averaged over four windows it was 11% worse than the chosen model.

### What did not work

- **Seasonal terms for gas and water:** gas error nearly doubled (15.39% to 28.62% MAPE) and water showed no gain.
- **Forced long seasons:** Holt-Winters with 36- and 60-month seasons, and other seasonal alternatives, scored 16% to 45% worse in validation.
- **Fine-tuning p and q:** the top configurations for gas and water were within about 2% of each other. Choosing the right model type mattered far more.

---

## Limitations and Future Work

- **Gas:** the model cannot anticipate shifts in level, such as the higher use in 2021 and 2022. Regular refitting and external operational data are the most promising improvements.
- **Electricity:** the seasonal swing grows with consumption, which leaves a weak residual pattern at 24 lags. A log transform could stabilise it.
- **Water:** only modestly better than the historical mean. External drivers such as occupancy, weather or maintenance data are needed for meaningful forecasts.
- **Test horizon:** the 41-month test set is shorter than the 60-month forecast, so accuracy in the final year of the forecast is not directly measured.

---

## Tech Stack

Python, pandas, NumPy, statsmodels, scikit-learn, SciPy, Matplotlib, Seaborn, Jupyter Notebook

---

## Repository Structure

```
├── Energy_Forecasting.ipynb             # Full analysis notebook
├── CA2_PartB_Energy_Forecasting.pptx    # Presentation slides
├── CA2-Energy-Consumption-Data.csv      # Dataset
└── README.md
```

---

## How to Run

```bash
git clone https://github.com/yueliangg/energy-consumption-forecasting.git
cd energy-consumption-forecasting
pip install pandas numpy statsmodels scikit-learn scipy matplotlib seaborn jupyter
jupyter notebook Energy_Forecasting.ipynb
```

The notebook reads the dataset from the same folder. Select **Kernel > Restart & Run All**. The grid searches take about one to two minutes.

---

## Author

**Yue Liang (Thawdar Lin Wai)**
Diploma in Applied AI and Analytics, Singapore Polytechnic

[Portfolio](https://yueliangg.github.io/yueliang-portfolio) | [LinkedIn](https://www.linkedin.com/in/thawdarlinwai) | [GitHub](https://github.com/yueliangg)
