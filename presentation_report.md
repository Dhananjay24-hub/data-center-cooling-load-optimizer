# ThermaOps: Data Center Cooling Optimizer
## Complete Presentation & Viva Preparation Guide

---

# 1. PROJECT OVERVIEW

### What is this project?
Imagine you are managing a massive data center with thousands of servers. These servers generate an immense amount of heat, requiring powerful cooling systems. If you cool too much, you waste electricity and money. If you cool too little, the servers overheat and fail. 

This project, **ThermaOps**, is an AI-powered dashboard and simulation system. It uses machine learning to predict how much cooling power will be used and what the temperatures will be based on specific settings. It then uses optimization algorithms to recommend the best cooling settings to save power while keeping servers safe.

* **Project Title:** ThermaOps — Data Center Cooling Optimizer
* **Domain:** Artificial Intelligence / Machine Learning / Green Computing
* **Problem Statement:** Balancing the trade-off between cooling power consumption and safe server temperatures in data centers.
* **Proposed Solution:** Using ML models to predict thermal responses and power usage, and applying optimization algorithms to find energy-efficient cooling setpoints.
* **Main Objectives:** Minimize cooling power consumption while keeping server inlet and hotspot temperatures below critical limits.
* **Scope:** Dashboard for historical replay, predictive simulation, and setting optimization. (Note: It is a simulation/dashboard, it does not directly control physical equipment).
* **Expected Outcome:** Energy savings, reduced risk of overheating, and a deeper understanding of cooling dynamics.

**Explain the project in 30 seconds:**
"My project is an AI dashboard for data center operators. It takes the current IT workload and weather as input, and uses Machine Learning to predict the temperatures of the servers and the power required to cool them. It then uses a Genetic Algorithm to recommend the optimal fan speeds and temperature setpoints to minimize electricity usage without letting the servers overheat."

---

# 2. REAL-WORLD PROBLEM

### Existing Problem
Data centers consume vast amounts of global electricity, and a huge portion of that goes to cooling systems (CRAC units, chillers, pumps). 

### Why traditional methods fail
Traditionally, operators rely on static, rule-based systems or manual adjustments. They often over-cool the data center "just to be safe," which wastes a massive amount of energy. The relationships between IT load, outdoor weather, fan speeds, and server temperatures are highly complex, nonlinear, and delayed.

### Why AI/ML is useful here
Machine Learning algorithms excel at finding complex, nonlinear patterns in historical data. Instead of guessing how a change in fan speed will affect the hotspot temperature at the back of a server rack, an ML model learns exactly how it behaved in the past and predicts it instantly.

**Real-World Example:**
"Suppose a data center receives a sudden spike in CPU utilization during a hot summer day. Traditionally, operators would manually blast the fans at 100% just to be safe, wasting energy. Our system instead predicts exactly what the temperature will be, and might suggest running fans at 75% and lowering the water temperature slightly—saving power while maintaining the exact same safety margin."

---

# 3. COMPLETE SYSTEM AT A GLANCE

**INPUT**
(Current data center state: Server count, CPU util, Outdoor temp, etc.)
↓
**DATA PREPROCESSING**
(Handling missing values, encoding categorical data)
↓
**PREDICTIVE ML MODELS**
(Four separate models predict Power, Load, Inlet Temp, Hotspot Temp)
↓
**OPTIMIZATION (GA / DE)**
(Testing thousands of combinations to find the lowest power setting)
↓
**VALIDATION & OOD CHECK**
(Ensuring the recommendation is safe and within known data patterns)
↓
**OUTPUT**
(Optimized settings returned to operator dashboard)

### Why is this flow required?
You cannot optimize what you cannot measure or predict. We first must accurately predict temperatures (ML Models). Once we can predict temperatures, we can test different cooling settings (Optimization) to find the one that uses the least power while staying safe.

---

# 4. TECHNOLOGIES USED

| Technology | Where Used | Why Used | What It Does |
| ---------- | ---------- | -------- | ------------ |
| **Python** | Entire Backend & ML | Industry standard for ML | Core programming language |
| **Scikit-Learn** | ML Pipeline | Reliable standard ML algorithms | Preprocessing, Random Forest, Dummy, Linear Regression, Metrics |
| **XGBoost** | ML Models | High performance gradient boosting | Predicts Cooling Load & Server Inlet Temperature |
| **LightGBM** | ML Models | Fast, efficient gradient boosting | Predicts Hotspot Temperature |
| **Pandas / NumPy** | Data Processing | Standard data manipulation | Loads CSVs, handles arrays, cleans data |
| **Flask** | Backend API | Lightweight web framework | Serves the UI and handles HTTP requests from dashboard |
| **HTML/CSS/JS** | Frontend | Standard web technologies | Renders the operator dashboard |
| **SQLite3** | Database | Zero-configuration local database | Stores operator history, saved limits, and optimization runs |

---

# 5. PROJECT ARCHITECTURE

**User** (Data Center Operator)
↓ *(Interacts via Web Browser)*
**Frontend** (HTML/JS/CSS Dashboard)
↓ *(Makes REST API Calls)*
**Backend API** (Flask Application - `api/app.py`, `api/service.py`)
↓ *(Processes State, Requests Predictions)*
**ML Pipeline** (Preprocessing pipelines and Trained Models)
↓ *(Calculates predictions and OOD checks)*
**Optimizer** (Genetic Algorithm or Differential Evolution)
↓ *(Finds best settings)*
**Database** (SQLite3 - saves results)
↓
**Result** (Displayed on UI to user)

**Component Breakdown:**
*   **API Service:** The brain connecting the UI to the ML models. It receives state inputs, enforces safety limits, and calls the models.
*   **OOD Detector (Out of Distribution):** A safety check (using Mahalanobis distance) to ensure the AI doesn't recommend settings it has never seen in training data.

---

# 6. COMPLETE DATA FLOW

Let's trace a single prediction:
1. **Raw Input:** The system receives state `state_id` from the dashboard (e.g., `cpu_utilization_pct = 56.75`, `season = Winter`).
2. **Data Extraction:** The exact row is fetched from `test.csv` in memory.
3. **Preprocessing:** 
   * `SimpleImputer` fills any missing numerical values with the median.
   * `OneHotEncoder` turns categorical variables (like `season`) into 1s and 0s.
4. **Model Prediction:** 
   * The processed array is passed to **four different models**.
   * Model 1 (Random Forest) predicts Cooling Power.
   * Model 2 (XGBoost) predicts Cooling Load.
   * Model 3 (XGBoost) predicts Inlet Temp.
   * Model 4 (LightGBM) predicts Hotspot Temp.
5. **Optimization (If requested):** The optimizer loops through steps 3-4 thousands of times, tweaking `fan_speed_pct` and `cooling_setpoint_c`, searching for the lowest power that keeps temps safe.
6. **Final Output:** The optimal controls, predicted temperatures, and estimated power savings are returned to the user.

---

# 7. DATASET

* **Dataset Name:** Data Center Cooling Load (Custom Research Data)
* **Total Rows:** 10,000 observations
* **Splits:** 7,000 (Training) / 1,500 (Validation) / 1,500 (Held-out Test)
* **Missing Values:** Handled via Median/Mode imputation in the pipeline.

| Feature | Meaning | Data Type | Role |
| ------- | ------- | --------- | ---- |
| `cpu_utilization_pct` | Average CPU load | Float | Independent Feature |
| `outdoor_temperature_c` | Weather outside | Float | Independent Feature |
| `season` | Winter/Summer/etc. | Categorical | Independent Feature |
| `fan_speed_pct` | Speed of cooling fans | Float | Control Variable (Optimized) |
| `cooling_setpoint_c` | Target cooling temp | Float | Control Variable (Optimized) |
| `cooling_power_kw` | Electricity used by cooling | Float | Target 1 |
| `server_inlet_temperature_c`| Temp of air entering server| Float | Target 2 |
| `hotspot_temperature_c` | Highest temp at server back | Float | Target 3 |

*(Note: Features like PUE, optimal settings, and thermal safety margins are explicitly excluded to prevent target leakage).*

---

# 8. DATA PREPROCESSING

### What? & How?
1. **Missing Value Imputation:** 
   * `SimpleImputer(strategy='median')` for numerical data.
   * `SimpleImputer(strategy='most_frequent')` for categorical data.
2. **Encoding:** 
   * `OneHotEncoder(handle_unknown='ignore')` for categories.
3. **Target Leakage Removal:**
   * Features that are technically "outcomes" (like energy savings) are strictly dropped before training to prevent the model from cheating.

### Why?
ML models only understand numbers. They cannot process "Winter" or `NaN` (blank data). 

### What if we don't do it?
The code would crash, or the model would memorize "cheating" variables (like using `optimized_cooling_power` to predict `cooling_power`), resulting in 100% accuracy in training but total failure in the real world.

---

# 9. MACHINE LEARNING PART

### What is Machine Learning?
Instead of hardcoding rules like `if cpu > 80 then fans = 100`, we feed past data to an algorithm. It uses math to figure out the complex relationships between the inputs (CPU, weather) and the outputs (temperatures, power) on its own.

*   **Category:** Supervised Learning (Specifically, Regression).
*   **Why?** Because we have historical data with the "answers" (labeled data), and we are trying to predict a continuous numerical value (temperature, kilowatts), not a category (like cat vs. dog).

---

# 10. EVERY MODEL USED

*Note: The project actually trains multiple models and selects the best one for each target based on Validation MAE. The final active architecture is:*

## Model 1 — Random Forest Regressor (Predicts: Cooling Power)
*   **What is it?** An ensemble model that creates hundreds of Decision Trees.
*   **How it works?** It asks a series of yes/no questions based on features. Each tree makes a prediction, and the forest averages them out.
*   **Hyperparameters used:** `n_estimators=100` (100 trees), `max_depth=20`, `min_samples_split=2`.
*   **Why it fits:** Handles non-linear relationships well without complex scaling.

## Model 2 — XGBoost (Predicts: Server Inlet Temp & Cooling Load)
*   **What is it?** Extreme Gradient Boosting.
*   **How it works?** It builds trees sequentially. The first tree makes predictions. The second tree looks at the *errors* of the first tree and tries to fix them. The third fixes the second, and so on.
*   **Hyperparameters used:** `n_estimators=100`, `max_depth=5`, `learning_rate=0.1`.
*   **Why it fits:** Extremely accurate for tabular data and handles complex feature interactions perfectly.

## Model 3 — LightGBM (Predicts: Hotspot Temp)
*   **What is it?** Light Gradient Boosting Machine.
*   **How it works?** Similar to XGBoost, but it grows trees leaf-wise (vertically) instead of level-wise (horizontally). 
*   **Hyperparameters used:** `num_leaves=31`, `n_estimators=100`, `learning_rate=0.1`.
*   **Why it fits:** Extremely fast to train and predict, highly accurate.

---

# 11. MODEL TRAINING & EVALUATION

### Training Process (`train.py`)
1. Data split into 8000 rows (train) / 500 rows (validation) / 1500 rows (test).
2. The pipeline trains 5 different families (Dummy, Linear, RF, XGB, LGBM) for *each* of the 4 targets.
3. It evaluates them on the 500-row validation set.
4. It selects the single model for each target that has the lowest **MAE (Mean Absolute Error)**.
5. It saves ONLY the 4 best models as `.joblib` files to `ml/models/artifacts`.

### Evaluation Metrics
*   **MAE (Mean Absolute Error):** The average difference between the predicted value and the actual value. If MAE is 0.5 for temperature, the model is off by 0.5 degrees on average.
*   **RMSE (Root Mean Squared Error):** Punishes large errors heavily. 
*   **R² (R-Squared):** Explains how much of the variance the model captured (closer to 1.0 is better).
*   **95th Percentile Upper Residual:** A safety margin calculated on validation data, added to predictions to ensure 95% confidence that the real temperature won't exceed the prediction.

---

# 12. MODEL PREDICTION — VERY IMPORTANT

**"Sir, what exactly happens when you enter a new input into your project?"**

1.  **Input:** The user clicks "Simulate" on the dashboard. The frontend sends the `state_id` and the user's chosen `fan_speed_pct` and `cooling_setpoint_c`.
2.  **Lookup:** The backend fetches the exact weather and IT load data for that `state_id`.
3.  **Combination:** It combines the real weather/IT data with the *user's requested* fan speed.
4.  **Preprocessing:** This data array passes through the saved `ColumnTransformer` (missing values filled, one-hot encoded).
5.  **Prediction:** 
    * The RF model predicts power.
    * The XGB models predict inlet temp.
    * The LGBM model predicts hotspot temp.
6.  **Safety Check (OOD):** The system checks if this data point is too weird (Mahalanobis distance). If the user asked for 10% fan speed on a 100% CPU load day, the system might reject it as "Outside learned operating region".
7.  **Output:** The predicted temperatures and power are returned and graphed on the dashboard.

---

# 13. ACTUAL CODE EXPLANATION

### `api/service.py` (The Core Logic)
*   **`predict()`**: Fetches state, applies limits, calls `prediction()`.
*   **`prediction()`**: Takes a row of data and control settings, runs the ML model `_evaluate()`, applies the 95% safety margin, checks against user limits, and checks OOD status. Returns feasibility.
*   **`optimize()`**: Runs the Genetic Algorithm (GA) or Differential Evolution (DE) via `run_optimizer()`. It tests random populations of fan/pump speeds, uses the ML model as the "fitness function" to score them (lower power is better, high temps are penalized), and returns the best safe setting.

### `ml/models/train.py` (The Training Pipeline)
*   **`train_bundle()`**: Loops through `TARGET_KEYS`. For each target, it loops through `FACTORIES` (RF, XGB, etc.), fits the model, checks `validation_mae`, and saves the best one using `joblib.dump()`.

---

# 14. END-TO-END WORKFLOW (The "Most Important Section")

1.  User opens application (`http://127.0.0.1:8501`).
2.  Frontend (HTML/JS) loads and requests past historical data via API.
3.  User selects a specific historical timestamp and clicks "Find Optimal Settings".
4.  Frontend sends API request (`POST /api/optimize`) with `state_id`.
5.  Backend `CoolingService.optimize()` retrieves the IT and weather data for that moment.
6.  Backend starts the Genetic Algorithm.
7.  GA generates 40 random combinations of fan speeds and temperature setpoints.
8.  GA asks the ML models: "If I use these settings, what will the temperatures and power be?"
9.  ML Models preprocess the data and return predictions.
10. GA discards combinations that cause overheating and keeps the ones that save power. It mutates and breeds them for 35 generations.
11. GA finds the absolute best safe setting.
12. Backend saves the run to SQLite database.
13. Backend returns the optimal settings, predicted temps, and estimated power savings (in kW) to frontend.
14. Frontend updates the UI to show the savings!

---

# 15. WHAT THE MODEL ACTUALLY LEARNS

The model does **not** just memorize the spreadsheet.
It learns **patterns and physics approximations**:
*   *It learns* that as `cpu_utilization_pct` goes up, `cooling_load_kw` goes up.
*   *It learns* that if `outdoor_temperature_c` is high, lowering the `cooling_setpoint_c` requires exponentially more `cooling_power_kw`.
*   *It learns* the complex interaction: high fan speeds lower the `hotspot_temperature_c`, but increase the `cooling_power_kw`. 

---

# 16. LIMITATIONS (What the project is NOT doing)

*   **IMPLEMENTED:** It simulates recommendations and calculates power savings based on past data.
*   **NOT DOING:** It does **not** connect to real HVAC equipment or directly control physical data center CRAC units.
*   **Dataset Limitation:** It cannot predict accurately if the data center conditions change drastically (e.g., adding 5,000 new servers). The OOD detector will flag this, but the model cannot adapt without retraining.
*   **Time-Series Dynamics:** The current models predict snapshots in time. They don't perfectly model thermal inertia (how long it takes a room to heat up over 30 minutes).

---

# 17. FUTURE SCOPE

*   **Immediate:** Implement Time-Series models (like LSTMs) to account for thermal inertia and delayed cooling effects.
*   **Advanced:** Connect the optimization API via MQTT or Modbus to actual physical CRAC unit controllers in a lab environment for closed-loop control.
*   **Long-term:** Implement Deep Reinforcement Learning (RL) where an AI agent learns cooling strategies by interacting continuously with a simulated data center environment.

---

# 18. VIVA / PRESENTATION QUESTIONS

### Basic Questions
*   **Q: What is your project?**
    *   *Short:* An AI dashboard that predicts data center temperatures and optimizes cooling settings to save power.
    *   *Detail:* (See 30-second explanation).
*   **Q: Why did you use Machine Learning?**
    *   *Short:* Because physical thermodynamics in a data center are too complex for simple `if-else` rules.
    *   *Detail:* The relationship between 25 features (IT load, weather, fans) and temperatures is highly nonlinear. ML captures these complex interactions accurately from historical data.

### ML Questions
*   **Q: Which algorithms did you use and why?**
    *   *Detail:* I trained 5 families of models (Dummy, Linear, RF, XGB, LGBM). The system automatically selected Random Forest for power, XGBoost for inlet/load, and LightGBM for hotspots because they had the lowest Validation MAE. Tree-based ensembles are best for tabular data with non-linear relationships.
*   **Q: What is the difference between your targets and features?**
    *   *Detail:* Features (X) are inputs we know (CPU load, weather, fan speed). Targets (y) are what we want to predict (Power used, Hotspot Temperature).

### Code & Architecture Questions
*   **Q: How does the Genetic Algorithm use the ML Model?**
    *   *Detail:* The GA generates possible cooling settings. To know if a setting is "good", it must know the power and temperatures. It feeds the settings into the ML model, gets the predictions, and uses the power prediction as its "fitness score" (lower is better), while strictly rejecting any settings where predicted temps exceed the safety limits.
*   **Q: What is the OOD detector in your code?**
    *   *Detail:* It stands for Out-Of-Distribution. It calculates the Mahalanobis distance of a new input compared to the training data. If the GA suggests a combination of weather and fan speeds the model has never seen, the OOD detector blocks it, because ML models hallucinate when given data completely outside their training bounds.

---

# 19. FINAL ONE-PAGE REVISION SHEET

*   **Project Name:** ThermaOps
*   **Problem:** Data centers waste energy by over-cooling to prevent server overheating.
*   **Solution:** ML predicts temperatures; GA optimizes cooling settings to save power safely.
*   **Dataset:** 10,000 rows, 5 data centers.
*   **Features:** CPU util, outdoor temp, server count, season, fan speed, etc.
*   **Targets:** Cooling Power, Cooling Load, Inlet Temp, Hotspot Temp.
*   **ML Type:** Supervised Learning (Regression)
*   **Algorithms:** Random Forest, XGBoost, LightGBM
*   **Evaluation:** Mean Absolute Error (MAE)
*   **Technology Stack:** Python, Scikit-learn, XGBoost, Flask, HTML/JS, SQLite.
*   **Key Advantage:** Replaces guesswork with data-driven optimization, proving safe energy savings.
*   **Limitation:** It is a predictive simulation, not a real-time physical controller.

### 5 Things I MUST remember:
1. I have **four separate models** predicting four separate things, not one giant model.
2. The ML predicts; the GA (Genetic Algorithm) optimizes based on those predictions.
3. Features that represent "outcomes" (like energy savings) were dropped to prevent Target Leakage.
4. I use a **95th percentile safety margin** added to temperature predictions to ensure servers don't overheat.
5. The **OOD detector** stops the model from predicting on scenarios it was never trained on.
