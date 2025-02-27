# User Activity Prediction - Data Science Challenge 🕹️

This repository contains the solution for the Nordeus Data Science Challenge focused on predicting the number of active days a user will have within the first 28 days after re-registering for a game. The project involves data preparation, exploratory analysis, feature engineering, model building, and evaluation.

## Project Overview 🎯

### Goal
To predict the target variable representing the number of days a user was active in the first 28 days after re-registering.

### Datasets
The data consists of two primary datasets:

#### Registration Data
- Captures user activity on the day of re-registration
- Includes features like device information, platform, session statistics, and gameplay metrics

#### Previous Lives Data
- Contains historical activity from the user's past gameplay sessions
- Includes metrics like lifetime active days, purchases, and previous registration details

## Methodology 🛠️

*Note: Detailed explanation of the whole process is in the Jupyter notebook: `notebooks/eda.ipynb`.*

### 1. Data Cleaning and Preprocessing
**Purpose**: Address inconsistencies and prepare datasets for merging and modeling.

**Steps**:
- Handling rare categories in nominal features (e.g., grouping rare registration_country values into "Other")
- Standardizing date and time features for consistency
- Imputing missing values and removing outliers where applicable

### 2. Exploratory Data Analysis (EDA)
**Insights**:
- The target variable has a skewed and zero-inflated distribution, with many users inactive (0 days). It also has a U-shape, indicating two groups of users:
  - Users who are active for only a few days and then become inactive
  - Users who are active for more than ~20 days
- Nominal features like registration_country exhibit many rare values
- The EDA was performed in each step of the process, from data exploration to model building

### 3. Feature Engineering
**Objective**: Enhance predictive power by creating derived features. Feature engineering is done on both previous_lives dataset, as well as the merged dataset (merged dataset is the one obtained by merging previous_lives with registration_data).

**Key Engineered Features**:
- **Recency, Frequency, Age**:
  - Time since last activity
  - Frequency of past activity (e.g., days_active_lifetime_mean)
  - Lifetime activity span (registration_date_min to registration_date_max)
- **Aggregations**:
  - Unique counts (nunique) for temporal features like year, month, week
  - Summarized statistics (sum, mean, std) for user activity and transactions
- **Behavioral Ratios**:
  - Match win rate, spending intensity, etc.

### 4. Model Building
- I have decided to use XGBoost algorithm because it is very powerful and well-suited for columnar data (after merging and feature engineering, I have created 69 features), and it is also flexible when it comes to choosing the objective function (eventually, reg:absoluteerror was used since the performance was measured with MAE metric, but I have also tried Tweedie and Poisson since it seemed suitable for this type of data; results were very similar though). Categorical features were label encoded for this purpose. Also, XGBoost can handle feature importance implicitly.
  
1. **First Approach**: Plain XGBoostRegressor - tried out different objectives and tuned hyperparameters.
2. **Second Approach**: Two-Stage Modeling - this boils down to modeling churn (whether user will be active) first, and then model the activity of users who will not churn separately (my idea here was that this approach will deal with zero-inflated distribution better, however the end results were very similar, perhaps because I did not have enough time to fine-tune the two-stage model since it is more complex and requires more hyperparameters to be tuned).
   - **Stage 1** (Binary Classification):
     - Predict whether a user will be active (days > 0) or inactive (days = 0)
     - Model: XGBoostClassifier
   - **Stage 2** (Regression):
     - Predict the number of active days for users classified as active
     - Model: XGBoostRegressor

### 5. Evaluation
- **Metric**: Mean Absolute Error (MAE)
- **Validation**: K-fold cross-validation with K=5 folds

### 6. Hyperparameter Tuning
Grid search over key parameters:
- Learning rate (eta)
- Maximum depth (max_depth)
- Regularization terms (reg_lambda, reg_alpha)
- Tree-specific parameters

## Results 📊

### First Approach
**Parameters**:
- {'objective': 'reg:absoluteerror', 'eta': 0.08, 'gamma': 0.5, 'max_depth': 6, 'reg_lambda': 0.01, 'min_child_weight': 3}
**Validation MAE**: 5.2774

### Second Approach
**Parameters**:
- For XGBoostRegressor: {'objective': 'reg:absoluteerror', 'eta': 0.08, 'gamma': 0.5, 'max_depth': 6, 'subsample': 1.0, 'min_child_weight': 5}
- For XGBoostClassifier: default parameters, objective='binary:logistic'
**Validation MAE**: 5.3551

*Note: The **first** approach was used for the final submission since it generalizes better. Also, since regression was used, output values which are continuous values are rounded to the closest integer and bouded to the range [0, 28].*

### Feature Importance
- Interesting insights can be gained from the feature importance plots. We can see that playtime on the first day of registration is of high importance. We can also see that some of the engineered features are more important than the raw features (e.g. frequency, mean and max days of active lifetime, session count, etc.
![alt text](feature_importance.png)

## Code 💻
**Important files**:
- `notebooks/eda.ipynb` for the whole process description and eda
- `src/check_values_train_test.py` for checking the values of different features in train and test sets
- `src/data_cleaning.py` for data cleaning
- `src/label_encoding.py` for label encoding
- `src/transform_previous_lives_data.py` for transforming and feature engineering on previous lives data
- `src/feature_engineering.py` for feature engineering and aggregation of both datasets
- `src/stratified_kfold.py` for creating folds stratified by the target variable
- `src/train.py` for the first modeling approach
- `src/train_two_stage.py` for the second modeling approach

## Ideas I did not have time to try out 💡
- Since the distribution of the target variable is U-shaped, it suggests there are two (or more) groups of users. This information can be leveraged in the modeling process, e.g., by using mixture models. I would probably try to incorporate this in the two-stage model, so that non-churned users' activity can be modeled with a mixture model that could capture different types of users.
- Explore ensemble methods to combine predictions for improved accuracy.
