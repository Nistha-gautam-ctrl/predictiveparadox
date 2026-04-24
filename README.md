#Predictive Paradox – Short-Term Power Demand Forecasting
This repository contains a robust machine learning pipeline designed to predict the next hour's electricity demand (demand_mw) on the national grid. 
Accurate electricity demand forecasting is critical for power grid stability and efficient energy management. 
Overestimating demand leads to wasted generation and financial losses, while underestimating demand causes severe grid instability and load shedding. 
The goal is to build a predictive model using classical machine learning architectures, specifically avoiding deep learning or specialized autoregressive packages.

1.Rationale for Data Handling
The provided datasets are unaltered extractions containing real-world anomalies, irregular timestamp frequencies, and undocumented spikes.

A.Missing Data and Outlier ResolutionDuplicates & Frequency: Exact duplicates were removed, and irregular timestamps were resolved by taking the mean of numeric columns. The dataset was then resampled to a consistent 1-hour frequency.
~Anomaly Detection: A Rolling Hampel Filter approach was implemented. Using a 24-hour window, any point exceeding 3 standard deviations from the local median—or values less than 0 were flagged as physical impossibilities.
~Imputation: Flagged anomalies and missing gaps were filled using linear interpolation to maintain time-series continuity, followed by forward/backward filling for edge cases.

B. Macroeconomic Integration
~Annual indicators (GDP, Population) were merged into the hourly dataset via a Broadcasting Method.
~Log Transformation: To handle extreme scales in GDP and Population and reduce skewness, a log(1+x) transformation was applied to improve model stability.
2. Feature Engineering:
Since the models used (XGBoost, Random Forest) treat observations independently, the concept of "time" was engineered manually to establish trends and seasonality.

Temporal Features
~Calendar Encoding: Hour of day, day of week, and weekend flags were extracted to capture daily and weekly seasonality.
~Historical Lags: Features were created for 1-hour, 2-hour, and 24-hour lags to allow the model to "look back" at recent behavior without violating supervised tabular rules.
~Rolling Aggregates: A 24-hour rolling mean was calculated to capture recent averages and variability.
~Demand Delta: A "Rate of Change" feature was calculated (difference between t and (t-1)) to capture sudden momentum shifts.

3. Validation & Modeling Strategy
To prevent data leakage, a strict chronological separation was maintained.

Training Set: Historical data prior to the test period.
Test Set: A strict chronological hold-out set (e.g., all of 2021/2022/2023/ 2024) was used to evaluate real-world performance.
Hyperparameter Tuning: Optuna was utilized with TimeSeriesSplit (3-fold cross-validation) to optimize parameters for both XGBoost and Random Forest.
Ensemble: A VotingRegressor was implemented to combine the strengths of multiple tree-based models.

4. Evaluation & Results
The primary evaluation metric is Mean Absolute Percentage Error (MAPE).
ModelTest           MAPE
XGBoost	            Calculated during execution
Random Forest	      Calculated during execution
Ensemble(Voting)	  Calculated during execution

Key Insights from Feature Importance
~Lagged Demand: Typically the strongest predictor, as power demand is highly autocorrelated.
~Hour of Day: Captures the consistent peaks and troughs of the human activity cycle.
~Temperature: Shows a strong correlation with demand due to cooling and heating loads.
6. How to Run
Ensure you have the required files: PGCB_date_power_demand.xlsx , weather_data.xlsx , and economic_full_1.csv.
Install dependencies:Bashpip install pandas, numpy, xgboost, optuna, openpyxl, matplotlib.

Execute the Jupyter Notebook to run the full pipeline from cleaning to evaluation.
