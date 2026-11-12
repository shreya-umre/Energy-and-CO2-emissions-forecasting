# Forecasting Germany's Energy Consumption and CO₂ Emissions with SARIMAX

Master's thesis project (M.Sc. Data Analytics and Decision Science, RWTH Aachen University) that forecasts Germany's daily total energy consumption and CO₂ emissions for **2025–2026**, using time-series models enriched with weather, economic, energy-mix and event variables (COVID-19, Russia-Ukraine war).

📄 Full thesis: [Master Thesis Report (PDF)](Master_Thesis_Report.pdf)

---

## Table of Contents

- [Motivation](#motivation)
- [Objectives](#objectives)
- [Data](#data)
- [Methodology](#methodology)
- [Key Results](#key-results)
- [Policy Implications](#policy-implications)
- [Repository Structure](#repository-structure)
- [How to Run](#how-to-run)
- [Tech Stack](#tech-stack)
- [Limitations and Future Work](#limitations-and-future-work)
- [Author](#author)

---

## Motivation

Germany aims to be climate neutral by 2045, and its energy system (the *Energiewende*) is changing fast: coal and nuclear are being phased out while wind and solar grow. At the same time, external shocks such as the COVID-19 pandemic and the Russia-Ukraine war have disrupted demand, prices and supply. Reliable forecasts of energy demand and emissions help policymakers, utilities and businesses plan resources, grid stability and decarbonisation strategies.

## Objectives

- Understand historical patterns in German energy consumption, generation mix and CO₂ emissions (2019–2024).
- Quantify how weather, economic indicators and major events relate to energy demand and emissions.
- Build and compare SARIMA / SARIMAX models, with and without exogenous variables.
- Forecast daily energy consumption and CO₂ emissions for 2025–2026.

## Data

Daily, national-level data for Germany from **1 January 2019 to 31 December 2024** (2,192 daily records).

| Category | Variables | Source |
|---|---|---|
| Energy consumption and generation | Total consumption; generation by source (coal, natural gas, nuclear, wind, solar, hydropower, biomass, pumped storage, others); electricity prices | [SMARD](https://www.smard.de) |
| Weather | Mean temperature, HDD, CDD, precipitation, wind speed, solar radiation | [Meteostat](https://meteostat.net) (via API) |
| Economic and demographic | GDP (quarterly, interpolated to daily), population (annual, interpolated to daily) | GDP: [FRED](https://fred.stlouisfed.org); population: [Statista](https://www.statista.com) |
| CO₂ emissions | Total and sectoral (aviation, transport, residential, industry, electricity) | [Carbon Monitor](https://carbonmonitor.org) |
| Event indicators | COVID-19 waves (Feb–May 2020, Oct 2020–Feb 2021); Russia-Ukraine war (from 24 Feb 2022) | Derived binary flags |

The full feature list with units is in Table 4.1 of the thesis.

**Preprocessing highlights**
- Translated German generation categories (e.g. *Braunkohle*, *Steinkohle*) to English and consolidated overlapping ones (e.g. onshore/offshore wind).
- Standardised dates, enforced datatypes, and interpolated missing weather values.
- Outliers: values beyond ±3σ of the seasonal-decomposition residuals were smoothed with a rolling median, and IQR-based capping was applied. Event-driven anomalies (COVID-19, war) were deliberately **kept**.

## Methodology

1. **Exploratory data analysis**: trends, seasonal decomposition, ADF stationarity tests, ACF/PACF, energy mix evolution, weather-vs-generation relationships, and event impact analysis.
2. **Feature selection**: correlation analysis plus iterative Variance Inflation Factor (VIF) filtering (VIF > 10 removed).
3. **Train/test split**: train on 2019–2023, test on 2024.
4. **Baseline models** (no exogenous variables)
   - Energy consumption: SARIMA(2,1,1)(1,0,2,12), chosen via ACF/PACF and empirical tuning against AIC/BIC/MAE/RMSE.
   - CO₂ emissions: ARIMA(4,0,0) with no seasonal terms, chosen after Auto-ARIMA suggested a purely seasonal model.
5. **SARIMAX models** adding exogenous variables
   - Energy: nuclear, natural gas, solar and wind generation, energy prices, mean temperature, COVID-19 flag, war flag.
   - CO₂: natural gas, coal, mean temperature, wind, solar and population.
6. **Future exogenous inputs**: because SARIMAX needs exogenous values over the forecast horizon, each variable was projected for 2025–2026 using seasonal decomposition of the last three years, linear trend extrapolation, and forward/backward filling.
7. **Final forecast**: models retrained on the full 2019–2024 dataset and forecast over 730 days (2025–2026) with confidence intervals.

## Key Results

**Energy consumption (test year 2024)**

| Model | MAE (MWh) | RMSE (MWh) |
|---|---|---|
| Baseline SARIMA(2,1,1)(1,0,2,12) | 117,758 | 150,958 |
| SARIMAX with exogenous variables | 87,918 | 112,569 |

Adding exogenous variables reduced MAE and RMSE by roughly 25%.

**CO₂ emissions (test year 2024)**

| Model | MAE | RMSE |
|---|---|---|
| Baseline ARIMA(4,0,0) | 0.8262 | 0.9293 |
| SARIMAX with exogenous variables | 0.1819 | 0.2684 |

The baseline drifted downward and underestimated emissions; exogenous variables reduced MAE by about 78% and RMSE by about 71%. AIC/BIC were higher for the SARIMAX model, a complexity trade-off for much better forecast accuracy.

**2025–2026 energy consumption forecast**
- Mean daily consumption: about **1.37 million MWh** (std. dev. about 119,000 MWh)
- Range: about 1.07 to 1.61 million MWh, with clear winter peaks and summer troughs
- Confidence intervals widen toward 2026, as expected for long-horizon forecasts

**2025–2026 CO₂ emissions forecast**
- Mean forecasted emissions: about **1.19 MtCO₂** (std. dev. 0.34), 25th–75th percentile 0.92 to 1.41 MtCO₂
- Maximum forecasted value: about 2.03 MtCO₂, reflecting winter peaks
- Confidence intervals widen toward 2026

**Model insights reported in the thesis**
- Higher energy prices were associated with lower total consumption.
- For CO₂, coal and natural gas had the largest positive effects, while wind and solar had negative effects.
- Mean temperature had a slight negative relationship with emissions (less heating demand in warmer weather).

**Selected EDA findings**
- Energy consumption and CO₂ emissions are strongly seasonal; CO₂ dropped visibly during 2020 lockdowns.
- Nuclear generation falls to zero after April 2023, reflecting the German phase-out.
- Wind and solar output track wind speed and solar radiation closely.
- Electricity prices spiked sharply in 2022 after the invasion of Ukraine and stabilised at a higher level than before.
- Industrial emissions fell more than electricity-sector emissions during COVID-19; residential emissions rose slightly.

## Policy Implications

The thesis discusses what the results mean for Germany's energy transition:

- **Renewables:** the negative association of wind and solar with emissions supports continued investment in renewable capacity.
- **Pricing:** the inverse relationship between energy prices and consumption suggests carbon pricing and energy taxation can influence demand.
- **Growth vs. demand:** GDP growth continues to push energy demand, so efficiency measures and industrial decarbonisation matter.
- **Demographics:** population trends should feature in climate action plans (urban planning, smart grids, public transport).
- **Resilience:** modelling shocks like COVID-19 and the war helps anticipate and mitigate future supply and demand disruptions.

## Repository Structure

> The structure is kept as-is so that the file paths inside the notebooks keep working.

```
.
├── Thesis report.pdf
└── Data/
    ├── Original data_unprocessed/
    │   ├── (raw Excel / CSV files)
    │   └── weather data extraction through API.ipynb
    ├── Data Pre-processing/
    │   ├── data pre-processing.ipynb
    │   └── (processed output files, Excel files)
    ├── EDA/
    │   ├── EDA.ipynb
    │   ├── plots/
    │   └── (CSV files)
    └── Time-series modelling/
        ├── Time series_SARIMAX_Energy Consumption.ipynb
        ├── Time series_SARIMAX_CO2.ipynb
        ├── train_data.csv
        ├── test_data.csv
        ├── final_data.csv
        ├── plots_energy/
        ├── plots_co2/
        ├── Total Energy Consumption Forecast 2025_2026.csv
        └── forecasted_CO2_emissions_2025_2026.csv
```

## How to Run

Run the notebooks in this order from their own folders (paths are relative to the structure above):

1. `Data/Original data_unprocessed/weather data extraction through API.ipynb`: pulls weather data from Meteostat.
2. `Data/Data Pre-processing/data pre-processing.ipynb`: cleans, merges and engineers features.
3. `Data/EDA/EDA.ipynb`: exploratory analysis and plots.
4. `Data/Time-series modelling/Time series_SARIMAX_Energy Consumption.ipynb`: energy consumption models and forecast.
5. `Data/Time-series modelling/Time series_SARIMAX_CO2.ipynb`: CO₂ emissions models and forecast.

**Setup**

```bash
git clone https://github.com/shreya-umre/Energy-and-CO2-emissions-forecasting.git
cd Energy-and-CO2-emissions-forecasting
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install pandas numpy matplotlib seaborn statsmodels pmdarima meteostat scikit-learn jupyter
jupyter notebook
```

Processed datasets (`train_data.csv`, `test_data.csv`, `final_data.csv`) are included, so you can start directly from step 4 without re-running the earlier notebooks.

## Tech Stack

Python · pandas · NumPy · statsmodels (SARIMAX) · pmdarima (Auto-ARIMA) · Matplotlib · Seaborn · Meteostat API · Jupyter

## Limitations and Future Work

**Limitations**
- **Projected inputs add uncertainty.** SARIMAX needs exogenous values over the forecast horizon, so errors in the extrapolated inputs carry directly into the forecasts.
- **Complexity.** Exogenous variables raise model complexity (higher AIC/BIC for the CO₂ model) and tuning effort, a trade-off for better test-set accuracy.
- **Linearity.** SARIMAX struggles with non-linear dependencies and structural shifts.
- **Widening uncertainty.** Confidence intervals grow substantially over the 2-year horizon.
- **Simple event flags.** COVID-19 and the war are binary indicators rather than graded effects.

**Future work**
- Hybrid statistical + machine learning models (e.g. LSTM, XGBoost)
- Real-time economic and meteorological data integration
- Sector-level analysis of consumption and emissions
- Regional (state-level) forecasting across Germany

## Author

**Shreya Umre**
M.Sc. Data Analytics and Decision Science, RWTH Aachen University

[LinkedIn](www.linkedin.com/in/shreya-umre) · [Email](mailto:shreya.umre@gmail.com)

## Acknowledgements

Data provided by [SMARD](https://www.smard.de), [Meteostat](https://meteostat.net), [Carbon Monitor](https://carbonmonitor.org), [FRED](https://fred.stlouisfed.org) and Statista. Full references are listed in the thesis report.
