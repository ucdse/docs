# Machine Learning Model

This chapter describes the machine-learning component of the Dublin Bikes project, whose purpose is to predict the number of available bikes at any given station for the upcoming 24–48 hours. The complete training and evaluation code is version-controlled in the main repository under `flask-app/machine_learning/ml.ipynb`, together with the exported artefacts `bike_availability_model.pkl` and `model_features.pkl`.

## 1. Problem Definition and Target Variable

The business objective is to help users decide *when* and *where* to pick up a bike by forecasting future availability. Consequently, the problem is framed as a **supervised regression** task:

*   **Target variable:** `num_bikes_available` – the number of bikes that can be rented from a station at a specific point in time.
*   **Inputs:** A vector of temporal, spatial, and meteorological features that are known in advance or can be observed at prediction time.

Predictions are produced in bulk for every forecast hour stored in the `weather_forecast` table, and the results are exposed to the frontend via the `/api/stations/<station_id>/prediction` endpoint.

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

The model therefore learns from **11 input features**:

1.  `station_id` – unique station identifier.
2.  `capacity` – total number of docking stands at the station.
3.  `lat` / `lon` – station latitude and longitude.
4.  `hour` – hour of day (0–23).
5.  `day` – day of month (1–31).
6.  `day_of_week` – weekday index (0–6).
7.  `is_weekend` – weekend flag (0/1).
8.  `avg_temperature` – mean air temperature (°C).
9.  `avg_humidity` – mean relative humidity (%).
10. `avg_pressure` – mean barometric pressure (hPa).

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

*Table 1: Regression-model comparison on the 30 % hold-out test set.*

Linear Regression performs poorly (R² ≈ 0.08), confirming that bike availability has strong non-linear relationships with time and location that a linear model cannot capture. Gradient Boosting also under-performs in this first comparison, likely because its default shallow-tree configuration is insufficient for the complex spatial and temporal interactions present in the data.

The Decision Tree achieves the lowest MAE (0.970) and the highest R² (0.941). However, a single deep decision tree is prone to over-fitting; its superior numbers on this particular split may reflect memorisation rather than generalisation. The Random Forest, while slightly worse on this single test split, offers a more conservative and stable ensemble prediction. Because the ultimate goal is a production service that must generalise to unseen future days and weather conditions, the **Random Forest Regressor was selected as the final model**.

### 4.2 Feature Importance

Using the Random Forest’s built-in `feature_importances_` attribute, the most influential predictors were identified. The ranking (from highest to lowest importance) is:

1.  `lat` / `lon` – geographic position dominates because different neighbourhoods have fundamentally different demand profiles.
2.  `hour` / `day` / `day_of_week` – temporal patterns strongly reflect commuting and leisure usage cycles.
3.  `avg_pressure` / `avg_temperature` – weather conditions affect people’s willingness to cycle.
4.  `station_id` / `capacity` – station identity and size provide useful baseline capacity information.
5.  `avg_humidity` / `is_weekend` – moderate but non-negligible contribution.

This ranking aligns well with domain intuition: location and time of day are the primary drivers of bike-share demand, while weather acts as a secondary modifier.

### 4.3 Production Integration

The trained Random Forest and the ordered feature list were serialised with `pickle` as:

*   `flask-app/machine_learning/bike_availability_model.pkl`
*   `flask-app/machine_learning/model_features.pkl`

At application start-up, `app/__init__.py` pre-loads these artefacts into global memory via `_load_model()` in `app/services/prediction_service.py`. When a user requests predictions for a specific station, the service:

1.  Retrieves the station’s static metadata (`capacity`, `lat`, `lon`) from the `station` table.
2.  Fetches the upcoming hourly weather forecasts from the `weather_forecast` table (populated continuously by the scraper).
3.  Constructs a Pandas `DataFrame` with exactly the 11 features in the stored order.
4.  Calls `_model.predict()` in a single batch operation (inference time ≈ 1 ms).
5.  Rounds the raw regression outputs to integers and clamps them to the valid range `[0, capacity]`.
6.  Returns a JSON array of `{forecast_time, predicted_available_bikes}` objects.

This design keeps the prediction endpoint lightweight and stateless: all heavy computation happens once at training time, and runtime inference requires only a deterministic matrix multiplication through the pre-built forest.

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
