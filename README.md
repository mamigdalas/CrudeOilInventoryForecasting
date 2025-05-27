# Crude Oil Inventory Forecasting & Dashboard

## Project Overview

This project focuses on forecasting U.S. Crude Oil Inventory levels and visualizing the actuals versus forecasts in an interactive dashboard. The goal is to apply time series forecasting techniques to real-world energy data and build a professional Power BI dashboard for analysis.

## Motivation & Problem

Crude oil inventory levels are a critical indicator for the global energy market, influencing oil prices and providing insights into supply-demand dynamics. Accurate forecasting can aid in strategic planning and market analysis. This project tackles the challenge of predicting inventory fluctuations, especially during periods of significant market shifts.

## Data Source

The primary data used for this project is the U.S. Crude Oil, Distillate Fuel Oil, and Motor Gasoline Stocks from the **Energy Information Administration (EIA)**.
* [Link to EIA data source (if available, e.g., series PET.RWCK_1.W)](https://www.eia.gov/dnav/pet/hist/LeafHandler.ashx?n=PET&s=RWCK_1&f=W)

## Methodology: Forecasting (Python in Google Colab)

The forecasting component of this project was developed using Python in Google Colab.

### 1. Data Acquisition and Cleaning

* Raw weekly U.S. inventory data was downloaded from the EIA.
* Data was loaded, inspected, and filtered to focus on specific product types (Crude Oil, Distillate Fuel Oil, Motor Gasoline).
* Date columns were converted to datetime objects, and the data was reshaped for time series analysis.
* Missing values were handled (details on method, e.g., interpolation/forward-fill).
* **Output:** `data/processed/cleaned_eia_inventory.csv`

### 2. Time Series Modeling

#### SARIMA Model
* **Approach:** Initial exploration with the Seasonal Autoregressive Integrated Moving Average (SARIMA) model.
* **Outcome:** While providing a baseline, SARIMA found it challenging to accurately capture the sustained structural break in Crude Oil inventory after 2020 (e.g., sharp declines). The model often struggled to adapt to the new trend.

#### Facebook Prophet Model
* **Approach:** Utilized Facebook Prophet for its ability to handle seasonality, holidays, and trend changes automatically.
* **Tuning:** Extensive experimentation with Prophet parameters was conducted to improve performance, particularly in capturing the post-2020 trend. Key parameters adjusted included:
    * `changepoint_prior_scale`: Adjusted to control trend flexibility.
    * `n_changepoints`: Increased the number of potential trend change points.
    * `changepoint_range`: Adjusted to allow changepoints closer to the end of the data.
* **Outcome:** Despite tuning, Prophet also faced challenges in accurately predicting the sharp and sustained decline in Crude Oil inventory post-2020. It often showed a tendency to revert or level off, highlighting the inherent difficulty of forecasting such a significant structural break without additional exogenous variables.
* **Output:** `data/forecasts/crude_oil_future_forecast_tuned_higher.csv` (contains the final Prophet forecast).

#### Automated SARIMA (`pmdarima`)
* **Attempt:** An attempt was made to use `pmdarima.auto_arima` for automated model selection.
* **Outcome:** This was blocked by `numpy` environment errors within Google Colab, preventing execution.

## Methodology: Dashboarding (Power BI)

The processed historical data and the Prophet forecast were integrated into an interactive Power BI dashboard for comprehensive visualization and analysis.

* **Data Import:** Both `cleaned_eia_inventory.csv` (Actuals) and `crude_oil_future_forecast_tuned_higher.csv` (Forecasts) were imported.
* **Data Modeling:**
    * Creation of a `Date Table` using DAX (`CALENDARAUTO()`).
    * Establishment of a **many-to-one relationship** between `Date Table[Date]` and `Crude_Oil_Combined[Date]` (which merges actuals and forecasts based on the `Type` column).
* **Key DAX Measures Implemented:**
    * `Actual Value (Measure)`: Filters `Crude_Oil_Combined[Value]` for "Actual" type.
    * `Forecast Value (Measure)`: Filters `Crude_Oil_Combined[Value]` for "Forecast" type.
    * `Residual (Actual - Forecast)`: Calculates the difference between Actual and Forecast, returning BLANK where Actuals do not exist.
    * `Percentage Residual`: Calculates percentage difference.
    * `Average Absolute Percentage Error (MAPE)`: Provides an overall accuracy metric for the overlap period.
    * `Latest Actual Inventory`, `Latest Forecasted Inventory`, `Forecast 6 Months Out`, `YoY Actual Inventory Change %`.
* **Visualizations:**
    * A combined line chart displaying Actuals, Forecasts, and Residuals over time (with Residuals on a secondary Y-axis).
    * Key Performance Indicator (KPI) cards for latest inventory figures and forecast accuracy.
    * Interactive slicers (e.g., for `Product`).
* **Output:** `powerbi/Crude_Oil_Inventory_Dashboard.pbix`

## Results & Learnings

* The project successfully integrated Python-based forecasting with Power BI visualization.
* It highlighted the challenges of forecasting significant structural breaks in time series data (like the post-2020 crude oil inventory decline) without external exogenous variables, even with advanced models like Prophet.
* The Power BI dashboard provides an intuitive way to track inventory trends, compare actuals to forecasts, and assess forecast accuracy.
* **Challenges Encountered:** Debugging specific Power BI behaviors, such as ensuring correct date relationships and line chart rendering, provided valuable learning experiences in data modeling and visualization best practices.

## How to Use/Reproduce

1.  **Clone the repository:**
    ```bash
    git clone [https://github.com/YOUR_USERNAME/CrudeOilInventoryForecasting.git](https://github.com/YOUR_USERNAME/CrudeOilInventoryForecasting.git)
    ```
    (Replace `YOUR_USERNAME` with your actual GitHub username)
2.  **Open the Power BI Dashboard:**
    * Navigate to the `powerbi/` folder.
    * Open `Crude_Oil_Inventory_Dashboard.pbix` with Power BI Desktop.
3.  **Run the Forecasting Notebook:**
    * Upload `notebooks/Crude_Oil_Forecasting_Colab.ipynb` to Google Colab.
    * Run all cells to re-generate the `cleaned_eia_inventory.csv` and `crude_oil_future_forecast_tuned_higher.csv` files. Ensure your Google Drive is mounted for saving outputs.
    * Place your raw EIA data (e.g., `PET_RWCK_1.csv` or similar) in the `data/raw/` folder, as referenced by the notebook.

## Future Enhancements (Optional)

* Incorporate exogenous variables (e.g., oil prices, economic indicators) into the forecasting models for improved accuracy, especially during periods of market disruption.
* Explore more advanced time series models or ensemble methods.
* Add more interactive elements and drill-through capabilities to the Power BI dashboard.