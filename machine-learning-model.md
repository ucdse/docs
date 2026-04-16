# Machine Learning Model

This chapter describes the machine-learning component of the Dublin Bikes project, whose purpose is to predict the number of available bikes at any given station for every weather-forecast entry currently cached in the database (typically the next 1–5 days, depending on the OpenWeatherMap tier in use). The complete training and evaluation code is version-controlled in the main repository under `flask-app/machine_learning/ml.ipynb`, together with the exported artefacts `bike_availability_model.pkl` and `model_features.pkl`.

## 1. Problem Definition and Target Variable

The business objective is to help users decide *when* and *where* to pick up a bike by forecasting future availability. Consequently, the problem is framed as a **supervised regression** task:

*   **Target variable:** `num_bikes_available` – the number of bikes that can be rented from a station at a specific point in time.
*   **Inputs:** A vector of temporal, spatial, and meteorological features that are known in advance or can be observed at prediction time.

Predictions are produced in bulk for every forecast entry stored in the `weather_forecast` table whose `forecast_time` is at or after the current hour, and the results are exposed to the frontend via the `GET /api/stations/<number>/prediction` endpoint, where `<number>` is the JCDecaux station number used as the primary key in the `station` table.

## 2. Dataset and Data-Cleaning Process

### 2.1 Raw Data

The raw training set is the file `final_merged_data.csv` (located in `flask-app/machine_learning/`). It contains **298,946 rows** and **78 columns**, built by joining historical station-availability snapshots with historical weather readings provided by Met Éireann. Each row therefore represents the state of one specific station at one specific hour, augmented with the weather conditions recorded during that hour.

### 2.2 Data-Cleaning and Feature-Selection Rationale

An initial exploratory analysis showed that the dataset contains **no missing values**. Nevertheless, many of the 78 columns are unsuitable for modelling. The following groups were systematically removed:

| Category of removed columns | Reason |
|----------------------------|--------|
| All `*_quality_indicator` columns | These are metadata flags (e.g. data-quality scores from the weather station) and contain no meteorological signal. |
| All `*_std_deviation` columns | Standard-deviation statistics are redundant once the corresponding min/max or mean values are retained. |
| Soil / grass / earth temperature columns (`max_soil_temperature_*`, `min_earth_temperature_*`, etc.) | These readings have very low correlation with urban bike-usage patterns. |
| Text columns (`name`, `address`, `last_reported`) | Not numerical and not required because `station_id`, `lat`, and `lon` already encode location identity. |
| Near-constant booleans (`is_installed`, `is_renting`, `is_returning`) | Almost 100 % `True`; they provide no discriminative power. |

After this pruning step, **15 columns** remained.

### 2.3 Feature Engineering

To capture periodic and meteorological patterns more explicitly, five derived features were created from the reduced set:

*   `day_of_week` – integer 0 (Monday) to 6 (Sunday), extracted from the date.
*   `is_weekend` – binary indicator (1 if Saturday or Sunday, 0 otherwise).
*   `avg_temperature` – arithmetic mean of `max_air_temperature_celsius` and `min_air_temperature_celsius`.
*   `avg_humidity` – arithmetic mean of `max_relative_humidity_percent` and `min_relative_humidity_percent`.
*   `avg_pressure` – arithmetic mean of `max_barometric_pressure_hpa` and `min_barometric_pressure_hpa`.

The averages were preferred over the raw min/max pairs because they reduce collinearity while preserving the same physical signal. A correlation check revealed that `month` and `year` had zero variance (the entire dataset covers December 2024 only); these were dropped because they cannot generalise to other months.

### 2.4 Final Feature Set

The model therefore learns from **11 input features**, in the exact order required by the serialised pipeline:

1.  `station_id` – the JCDecaux station number (stored in the `station.number` column and passed through as `station_id` for historical compatibility with the training data).
2.  `capacity` – total number of docking stands at the station (`station.bike_stands`).
3.  `lat` – station latitude.
4.  `lon` – station longitude.
5.  `hour` – hour of day (0–23).
6.  `day` – day of month (1–31).
7.  `day_of_week` – weekday index (0–6, Monday = 0).
8.  `is_weekend` – weekend flag (0/1).
9.  `avg_temperature` – mean air temperature (°C).
10. `avg_humidity` – mean relative humidity (%).
11. `avg_pressure` – mean barometric pressure (hPa).

All features are numeric, which avoids the need for categorical encoding and keeps the production inference pipeline simple.

## 3. Training and Testing Process

### 3.1 Train–Test Split

The cleaned dataset was split with scikit-learn’s `train_test_split` using a **70 % training / 30 % testing** ratio and `random_state=42` for reproducibility. This yielded **209,262 training rows** and **89,684 test rows**.

### 3.2 Model Candidates

Four regression algorithms were trained and evaluated under identical conditions:

1.  **Linear Regression** – chosen as a simple baseline.
2.  **Decision Tree Regressor** – captures non-linear interactions without feature scaling.
3.  **Random Forest Regressor** – an ensemble of decision trees; generally more robust and less prone to over-fitting than a single tree.
4.  **Gradient Boosting Regressor** – another ensemble method that builds trees sequentially to correct residuals.

The Random Forest was configured with modest regularisation (`n_estimators=100`, `max_depth=15`, `min_samples_leaf=10`) to prevent over-fitting on the large training set, while Linear Regression, Decision Tree, and Gradient Boosting used their default or standard hyper-parameter sets for a fair first comparison.

### 3.3 Evaluation Metrics

Predictive performance was quantified with three standard regression metrics:

*   **Mean Absolute Error (MAE)** – average absolute difference between predicted and actual bike counts.
*   **Root Mean Squared Error (RMSE)** – penalises larger errors more heavily than MAE.
*   **Coefficient of Determination (R²)** – proportion of variance in the target explained by the model.

## 4. Results

### 4.1 Model Comparison

Table 1 summarises the out-of-sample performance of each candidate.

| Model | MAE | RMSE | R² |
|-------|-----|------|----|
| Linear Regression | 7.829 | 9.366 | 0.075 |
| Decision Tree | 0.970 | 2.376 | 0.941 |
| **Random Forest** | **2.325** | **3.393** | **0.879** |
| Gradient Boosting | 6.120 | 7.537 | 0.401 |

*Table 1: Regression-model comparison on the 30 % hold-out test set. The values are taken directly from the executed notebook cells; the final markdown summary cell in `ml.ipynb` accidentally lists a different (erroneous) set of Random Forest metrics, which should be disregarded.*

Linear Regression performs poorly (R² ≈ 0.08), confirming that bike availability has strong non-linear relationships with time and location that a linear model cannot capture. Gradient Boosting also under-performs in this first comparison, likely because its default shallow-tree configuration is insufficient for the complex spatial and temporal interactions present in the data.

The Decision Tree achieves the lowest MAE (0.970) and the highest R² (0.941) on this single hold-out split. However, a single deep decision tree is prone to over-fitting; its superior numbers here may reflect memorisation rather than generalisation. The Random Forest, while worse on this particular test split, offers a more conservative and stable ensemble prediction. Because the ultimate goal is a production service that must generalise to unseen future days and weather conditions, the **Random Forest Regressor was selected as the final model** and exported for the Flask backend.

### 4.2 Feature Importance

Using the Random Forest’s built-in `feature_importances_` attribute, the most influential predictors were identified. The exact ranking observed in the training run (from highest to lowest importance) is:

1.  `lat` / `lon` – geographic position dominates because different neighbourhoods have fundamentally different demand profiles.
2.  `day` / `hour` – the specific calendar day and hour of day capture strong commuting and usage cycles.
3.  `station_id` – the station identifier provides additional location-specific baseline information beyond pure coordinates.
4.  `avg_pressure` – barometric pressure emerges as the most influential weather signal in this dataset.
5.  `capacity` – the total number of stands sets an upper bound and influences baseline availability.
6.  `day_of_week` / `avg_temperature` – weekday effects and temperature contribute moderate predictive power.
7.  `avg_humidity` / `is_weekend` – humidity and the weekend flag show smaller but non-negligible contributions.

This ranking aligns well with domain intuition: location and time are the primary drivers of bike-share demand, while weather acts as a secondary modifier.

### 4.3 Production Integration

The trained Random Forest and the ordered feature list were serialised with `pickle` as:

*   `flask-app/machine_learning/bike_availability_model.pkl`
*   `flask-app/machine_learning/model_features.pkl`

At application start-up, `app/__init__.py` invokes `_load_model()` from `app/services/prediction_service.py` inside the Flask app context to pre-warm the model and feature list into module-level globals, so that no disk I/O is required on the request path. When a user requests predictions for a specific station, the service:

1.  Retrieves the station’s static metadata (`number`, `bike_stands`, `latitude`, `longitude`) from the `station` table.
2.  Fetches every entry in the `weather_forecast` table whose `forecast_time` is at or after the current UTC hour, ordered ascending. The table is populated continuously by the scraper, so the forecast horizon depends on the active OpenWeatherMap tier (free 5-day/3-hour, or paid hourly).
3.  Constructs a Pandas `DataFrame` with exactly the 11 features above and reorders the columns to match the persisted `model_features.pkl` list before calling `predict`.
4.  Calls `_model.predict(df_input)` in a single batch operation.
5.  Rounds the raw regression outputs to integers and clamps them to the valid range `[0, station.bike_stands]`.
6.  Returns the predictions inside the standard API response envelope. On success the route emits `{"code": 0, "msg": "ok", "data": [...]}` with HTTP 200, where each item in `data` is `{"forecast_time": <ISO 8601 string>, "predicted_available_bikes": <int>}`. Domain errors (unknown station, no forecast available) yield `{"code": 1, "msg": <reason>, "data": null}` with HTTP 400, while unexpected failures yield `{"code": 1, "msg": "Prediction service unavailable", "data": null, "error": <exception text>}` with HTTP 500.

This design keeps the prediction endpoint lightweight and stateless: all heavy computation happens once at training time, and runtime inference is a single batched call into the cached Random Forest with no per-request I/O beyond the station and forecast queries.

## 5. Reflection and Future Work

### 5.1 Strengths

*   **Simplicity and speed:** The model uses only 11 easily obtainable numeric features, making both training and production inference extremely fast.
*   **No missing-value imputation:** The raw merged dataset had no gaps, which eliminated a common source of pipeline complexity.
*   **Direct business value:** Users can now see not only the *current* availability on the map, but also a 24–48 hour forecast, helping them plan trips in advance.

### 5.2 Limitations

*   **Temporal scope:** The training data span a single month (December 2024). Consequently, the model has not learned seasonal effects such as summer tourism peaks or university semester schedules.
*   **Weather-source mismatch:** The model was trained on historical Met Éireann observations, yet production inference uses OpenWeatherMap forecasts. Although the variables are conceptually the same (temperature, humidity, pressure), small distributional differences between the two providers may introduce slight prediction drift.
*   **Decision Tree anomaly:** The single Decision Tree outperformed the Random Forest on the test split, which is unusual. This suggests that the ensemble may benefit from additional hyper-parameter tuning (e.g. increasing `max_depth` or `n_estimators`, or adding more regularisation to the Decision Tree baseline) to squeeze out extra performance without sacrificing stability.

### 5.3 Future Improvements

1.  **Expand training history:** Collect data across multiple seasons (at least 6–12 months) so the model can learn long-term trends and holiday effects.
2.  **Add lag features:** Include the availability count from 1 hour, 24 hours, and 7 days ago as additional features. Bike usage is highly auto-correlated in time, and lag variables usually improve forecast accuracy significantly.
3.  **Hyper-parameter optimisation:** Run a grid search or random search on the Random Forest (and potentially XGBoost/LightGBM) to find a better bias-variance trade-off.
4.  **Confidence intervals:** Instead of returning a single point estimate, use the forest’s distribution of tree predictions to provide a prediction interval (e.g. 80 % confidence), which would help users assess forecast uncertainty.

Overall, the current machine-learning pipeline provides a robust, production-ready forecasting capability that directly supports the core user journey of the Dublin Bikes application.
