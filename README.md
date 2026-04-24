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
Install dependencies:%pip install pandas, numpy, xgboost, optuna, openpyxl, matplotlib.


An ideal output of the pipeline cell(without ensemble):
[I 2026-04-24 11:42:18,932] A new study created in memory with name: no-name-85652967-d7c7-422e-991c-483617263ef3
--- Optimizing RF ---
[I 2026-04-24 11:47:14,967] Trial 0 finished with value: 0.05397882772997776 and parameters: {'n_estimators': 139, 'max_depth': 15, 'min_samples_split': 9}. Best is trial 0 with value: 0.05397882772997776.
[I 2026-04-24 11:48:49,750] Trial 1 finished with value: 0.0623841502783866 and parameters: {'n_estimators': 143, 'max_depth': 5, 'min_samples_split': 7}. Best is trial 0 with value: 0.05397882772997776.
[I 2026-04-24 11:53:29,771] Trial 2 finished with value: 0.0570691735662773 and parameters: {'n_estimators': 291, 'max_depth': 7, 'min_samples_split': 4}. Best is trial 0 with value: 0.05397882772997776.
[I 2026-04-24 12:05:09,339] Trial 3 finished with value: 0.05386241673597286 and parameters: {'n_estimators': 321, 'max_depth': 15, 'min_samples_split': 8}. Best is trial 3 with value: 0.05386241673597286.
[I 2026-04-24 12:21:55,138] Trial 4 finished with value: 0.053859701109363646 and parameters: {'n_estimators': 386, 'max_depth': 28, 'min_samples_split': 6}. Best is trial 4 with value: 0.053859701109363646.
[I 2026-04-24 12:29:03,321] Trial 5 finished with value: 0.05380968566464479 and parameters: {'n_estimators': 183, 'max_depth': 17, 'min_samples_split': 7}. Best is trial 5 with value: 0.05380968566464479.
[I 2026-04-24 12:33:47,702] Trial 6 finished with value: 0.05385192410791787 and parameters: {'n_estimators': 108, 'max_depth': 23, 'min_samples_split': 6}. Best is trial 5 with value: 0.05380968566464479.
[I 2026-04-24 12:44:36,912] Trial 7 finished with value: 0.05379074916879877 and parameters: {'n_estimators': 268, 'max_depth': 28, 'min_samples_split': 10}. Best is trial 7 with value: 0.05379074916879877.
[I 2026-04-24 12:49:08,125] Trial 8 finished with value: 0.05379615808390853 and parameters: {'n_estimators': 109, 'max_depth': 29, 'min_samples_split': 9}. Best is trial 7 with value: 0.05379074916879877.
[I 2026-04-24 12:52:47,110] Trial 9 finished with value: 0.05939639454189421 and parameters: {'n_estimators': 271, 'max_depth': 6, 'min_samples_split': 5}. Best is trial 7 with value: 0.05379074916879877.
[I 2026-04-24 13:03:33,704] A new study created in memory with name: no-name-f35285f2-172e-4dfb-811c-4278dd23764f
RF Test MAPE: 1.99%
--- Optimizing XGBOOST ---
[I 2026-04-24 13:04:50,628] Trial 0 finished with value: 0.05307254456819226 and parameters: {'n_estimators': 1358, 'max_depth': 7, 'learning_rate': 0.011660297411038485, 'subsample': 0.5981902301351034, 'colsample_bytree': 0.8725600816400696}. Best is trial 0 with value: 0.05307254456819226.
[I 2026-04-24 13:06:43,895] Trial 1 finished with value: 0.0549296393859118 and parameters: {'n_estimators': 893, 'max_depth': 9, 'learning_rate': 0.01667323420002943, 'subsample': 0.9138705310636654, 'colsample_bytree': 0.8232921944760481}. Best is trial 0 with value: 0.05307254456819226.
[I 2026-04-24 13:07:14,370] Trial 2 finished with value: 0.05166674096072168 and parameters: {'n_estimators': 1098, 'max_depth': 5, 'learning_rate': 0.015329348582954472, 'subsample': 0.649111168415309, 'colsample_bytree': 0.9590619463692684}. Best is trial 2 with value: 0.05166674096072168.
[I 2026-04-24 13:07:39,774] Trial 3 finished with value: 0.05345611313287949 and parameters: {'n_estimators': 706, 'max_depth': 6, 'learning_rate': 0.06950743586571277, 'subsample': 0.6500835637014128, 'colsample_bytree': 0.7876489366285337}. Best is trial 2 with value: 0.05166674096072168.
[I 2026-04-24 13:10:23,123] Trial 4 finished with value: 0.056011388987780165 and parameters: {'n_estimators': 1343, 'max_depth': 9, 'learning_rate': 0.10579066027148715, 'subsample': 0.7063847269322615, 'colsample_bytree': 0.79043438229145}. Best is trial 2 with value: 0.05166674096072168.
[I 2026-04-24 13:10:53,496] Trial 5 finished with value: 0.05258023762646081 and parameters: {'n_estimators': 1432, 'max_depth': 5, 'learning_rate': 0.060944992763279755, 'subsample': 0.8495860880293364, 'colsample_bytree': 0.5725913815182266}. Best is trial 2 with value: 0.05166674096072168.
[I 2026-04-24 13:13:59,080] Trial 6 finished with value: 0.05985780435857416 and parameters: {'n_estimators': 1299, 'max_depth': 10, 'learning_rate': 0.22627607125808832, 'subsample': 0.6922604888007533, 'colsample_bytree': 0.5925782028329838}. Best is trial 2 with value: 0.05166674096072168.
[I 2026-04-24 13:15:40,814] Trial 7 finished with value: 0.05686730097935253 and parameters: {'n_estimators': 1434, 'max_depth': 8, 'learning_rate': 0.2889257163487186, 'subsample': 0.7671912863777891, 'colsample_bytree': 0.6440060094580353}. Best is trial 2 with value: 0.05166674096072168.
[I 2026-04-24 13:15:55,530] Trial 8 finished with value: 0.05249539926646968 and parameters: {'n_estimators': 1031, 'max_depth': 3, 'learning_rate': 0.028022014654716695, 'subsample': 0.5350512285195602, 'colsample_bytree': 0.5738962128677156}. Best is trial 2 with value: 0.05166674096072168.
[I 2026-04-24 13:16:23,593] Trial 9 finished with value: 0.051685461823180616 and parameters: {'n_estimators': 1042, 'max_depth': 5, 'learning_rate': 0.022094524587642963, 'subsample': 0.588011523659327, 'colsample_bytree': 0.8826957835417608}. Best is trial 2 with value: 0.05166674096072168.
XGBOOST Test MAPE: 1.93%

Pipeline Complete. Best Model: XGBOOST


Execute the Jupyter Notebook to run the full pipeline from cleaning to evaluation.
