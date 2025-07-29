````markdown
# Network Security Project

This project aims to build a machine learning pipeline for network security, specifically for detecting intrusions or anomalies within network traffic. The pipeline includes data ingestion, data validation, data transformation, and model training.

## Table of Contents
- [Project Overview](#project-overview)
- [Installation](#installation)
- [Usage](#usage)
- [Pipeline Stages](#pipeline-stages)
  - [Data Ingestion](#data-ingestion)
  - [Data Validation](#data-validation)
  - [Data Transformation](#data-transformation)
  - [Model Training](#model-training)
- [MLflow Tracking](#mlflow-tracking)
- [Project Structure](#project-structure)
- [Contributing](#contributing)
- [License](#license)

## Project Overview

The core idea behind this project is to automate the process of building and deploying a machine learning model that can identify malicious activities in network data. The pipeline is designed to be robust, handling data inconsistencies, drifts, and leveraging various machine learning models for optimal performance.

## Installation

1.  **Clone the repository:**
    ```bash
    git clone [https://github.com/your_username/NetworkSecurity.git](https://github.com/your_username/NetworkSecurity.git)
    cd NetworkSecurity
    ```

2.  **Create a virtual environment:**
    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows: `venv\Scripts\activate`
    ```

3.  **Install dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

4.  **Set up environment variables:**
    Create a `.env` file in the root directory and add your MongoDB URL:
    ```
    MONGO_DB_URL="your_mongodb_connection_string"
    ```
    If you plan to use DagsHub for MLflow tracking, also set up your DagsHub credentials:
    ```
    DAGSHUB_USERNAME="your_dagshub_username"
    DAGSHUB_USER_TOKEN="your_dagshub_token"
    ```

## Usage

To run the entire machine learning pipeline, execute the `main.py` file:

```bash
python main.py
````

This will trigger the sequence of operations: data ingestion, data validation, data transformation, and model training.

## Pipeline Stages

The project is structured into several distinct stages, each handled by a dedicated component.

### Data Ingestion

The `DataIngestion` class (from `networksecurity.components.data_ingestion`) is responsible for:

  - Connecting to a MongoDB database to retrieve network security data.
  - Exporting the collection data into a Pandas DataFrame.
  - Handling missing values by replacing "na" with `np.nan`.
  - Saving the raw data to a feature store (a CSV file).
  - Splitting the data into training and testing sets and saving them as CSV files.

### Data Validation

The `DataValidation` class (from `networksecurity.components.data_validation`) performs the following checks:

  - Reads the training and testing datasets.
  - Validates the number of columns in both train and test data against a predefined schema.
  - Detects dataset drift between the base (training) and current (testing) datasets using the Kolmogorov-Smirnov (KS) 2-sample test. A drift report is generated and saved as a YAML file.
  - If validation passes, the valid train and test data are saved to specified paths.

### Data Transformation

The `DataTransformation` class (from `networksecurity.components.data_transformation`) handles:

  - Reading the validated training and testing datasets.
  - Separating input features from the target column.
  - Replacing `-1` with `0` in the target feature.
  - Initializing a `KNNImputer` (with parameters from `DATA_TRANSFORMATION_IMPUTER_PARAMS`) within a scikit-learn Pipeline to handle missing values.
  - Fitting the preprocessor on the training input features and transforming both training and testing input features.
  - Combining transformed features with their respective target features into NumPy arrays.
  - Saving the transformed training and testing arrays.
  - Saving the fitted preprocessor object.

### Model Training

The `ModelTrainer` class (from `networksecurity.components.model_trainer`) is responsible for:

  - Loading the transformed training and testing NumPy arrays.
  - Defining a dictionary of machine learning models (Random Forest, Decision Tree, Gradient Boosting, Logistic Regression, AdaBoost) and their respective hyperparameters for tuning.
  - Evaluating these models using `evaluate_models` function to find the best performing model based on classification metrics.
  - Calculating classification metrics (F1-score, precision, recall) for both training and testing predictions.
  - **MLflow Tracking**: Logs metrics and potentially the best model to MLflow, integrated with DagsHub for remote tracking and versioning of experiments.
  - Saving the best trained model (encapsulated within a `NetworkModel` object, which includes the preprocessor and the trained model) as a pickle file.

## MLflow Tracking

This project utilizes MLflow for experiment tracking, allowing for easy comparison of different model runs, metrics, and parameters. It is integrated with DagsHub for remote storage and visualization of MLflow experiments.

  - **To view MLflow experiments:**
    If DagsHub is configured correctly, you can view your experiments directly on your DagsHub repository under the "Experiments" tab.
    Alternatively, if running locally, you can start the MLflow UI:
    ```bash
    mlflow ui
    ```
    Then, navigate to `http://localhost:5000` in your web browser.

## Project Structure

```
.
├── networksecurity/
│   ├── components/
│   │   ├── data_ingestion.py
│   │   ├── data_validation.py
│   │   ├── data_transformation.py
│   │   └── model_trainer.py
│   ├── constant/
│   │   └── training_pipeline.py
│   ├── entity/
│   │   ├── artifact_entity.py
│   │   └── config_entity.py
│   ├── exception/
│   │   ├── exception.py
│   │   └── __init__.py
│   ├── logging/
│   │   ├── logger.py
│   │   └── __init__.py
│   └── utils/
│       ├── main_utils/
│       │   ├── utils.py
│       │   └── __init__.py
│       └── ml_utils/
│           ├── model/
│           │   ├── estimator.py
│           │   └── __init__.py
│           └── metric/
│               ├── classification_metric.py
│               └── __init__.py
├── config/
│   └── schema.yaml
├── .env
├── main.py
├── README.md
├── requirements.txt
└── setup.py
```

