# Pulse Crop Recommendation Based on Soil and Climate Condition

**System name:** Pulse Crop Recommendation  
**Model:** Random Forest Classifier  
**Project type:** Machine-learning-based multiclass classification  
**Deployment:** FastAPI REST API with Docker  
**Model version:** 1.0.0

**Live Application:** https://pulse-crop-recommendation-demo.streamlit.app/

## 1. Project Overview

Pulse Crop Recommendation is a machine-learning-based decision-support prototype that predicts one of four pulse crop classes from soil nutrient measurements and climate conditions.

The system accepts seven numerical inputs:

- Nitrogen (N)
- Phosphorus (P)
- Potassium (K)
- Temperature (°C)
- Relative humidity (%)
- Soil pH
- Rainfall (dataset unit; treated as mm in the API description)

The trained Random Forest model returns a predicted crop, a model confidence score derived from the highest class probability, and the model version.

### Supported crop classes

- Chickpea
- Kidney beans
- Lentil
- Mungbean

The system is a prototype for educational and preliminary decision-support use. It has not been established as a validated agricultural recommendation service for Ethiopian farms.

## 2. Project Objectives

- Demonstrate how machine learning can classify pulse crop categories using soil and climate features.
- Prepare and analyze a structured agricultural dataset.
- Compare classification algorithms and select a model based on evaluation.
- Package the trained model for API-based use.
- Validate API behavior, including health checks, prediction requests, and invalid inputs.
- Provide a foundation for future evaluation using locally representative agricultural data.

## 3. Dataset

The original dataset contained approximately 2,200 records across multiple crop categories. The project team filtered it to retain four pulse crops, resulting in a working dataset of 400 records.

| Dataset characteristic | Description |
|---|---:|
| Original dataset | Approximately 2,200 records |
| Filtered dataset | 400 records |
| Input features | 7 |
| Target classes | 4 |
| Records per class | 100 |
| Training records | 320 (80%) |
| Testing records | 80 (20%) |
| Split method | Stratified |
| Random state | 42 |

### Data preparation

The notebook documents checks for missing values, duplicate rows, data types, invalid numerical values, outliers, feature distributions, class balance, and correlations. Potential outliers were examined using the Interquartile Range (IQR) method.

The final 400-record dataset was split into training and test sets using an 80:20 stratified split. The dataset source, collection period, license, and representativeness of Ethiopian farming conditions should be confirmed from the original provider before reuse or wider deployment.

## 4. Model Development and Evaluation

Several classification algorithms were explored during model development, including Logistic Regression, K-Nearest Neighbors, Gaussian Naive Bayes, Decision Tree, and Random Forest.

The selected model is a **Random Forest Classifier** with the following hyperparameters:

| Parameter | Value |
|---|---|
| `n_estimators` | 100 |
| `max_depth` | `None` |
| `min_samples_split` | 2 |
| `min_samples_leaf` | 1 |
| `max_features` | `sqrt` |
| `random_state` | 42 |

### Recorded evaluation results

The MLflow tracking record for the final selected model reports:

| Metric | Result |
|---|---:|
| Test accuracy | 1.0000 |
| Test macro precision | 1.0000 |
| Test macro recall | 1.0000 |
| Test macro-F1 | 1.0000 |
| Test weighted-F1 | 1.0000 |
| Five-fold cross-validation macro-F1 | 1.0000 |

These results were calculated using the available 400-record filtered dataset. They should not be interpreted as evidence of equivalent performance on new farms or on data from other regions. Independent external validation is still required.

## 5. System Architecture

The deployed project uses the following components:

- **FastAPI:** exposes the model through HTTP endpoints.
- **Uvicorn:** runs the FastAPI application.
- **scikit-learn:** provides the machine-learning model interface.
- **Joblib:** loads the saved model artifact.
- **Pandas and NumPy:** support numerical data handling.
- **Docker:** packages the application, dependencies, and model artifacts into a container.

The Dockerfile uses Python 3.13 slim, installs the pinned requirements, copies the application and model artifacts, runs the container as a non-root user, and starts the API on port 8000.

## 6. Repository Structure

The deployment project is expected to have the following structure:

```text
Pulse-Crop-Recommendation/
│
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── crop_service.py
│   └── mobile_api.py
│
├── model_artifacts/
│   ├── model.joblib
│   └── model_metadata.json
│
├── Dockerfile
├── requirements.txt
└── README.md
```

`app/main.py` is the application entry point used by the supplied Dockerfile. `app/mobile_api.py` is a separate API module and is not the default Docker entry point.

The notebook used for data preparation, model development, evaluation, and deployment work is maintained separately as `Pulse_Crop_Recommendation.ipynb`.

## 7. Requirements

- Python 3.13 (the version specified by the supplied Dockerfile).
- pip.
- Docker Desktop or Docker Engine (for containerized execution).
- The trained model artifact and metadata in the expected `model_artifacts/` directory.

The Python dependencies and versions are listed in `requirements.txt`.

## 8. Installation and Execution

### Option A: Run locally

1. Clone or download the project repository.
2. Open a terminal in the project root directory.
3. Create and activate a virtual environment.

**Windows PowerShell:**

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

**Windows Command Prompt:**

```bat
python -m venv .venv
.venv\Scripts\activate.bat
```

**Linux/macOS:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

4. Install the dependencies:

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

5. Confirm that `model_artifacts/model.joblib` exists.
6. Start the API from the project root:

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

The API will be available at:

```text
http://127.0.0.1:8000
```

### Option B: Run with Docker

Make sure Docker Desktop or Docker Engine is installed and running.

1. Open a terminal in the directory containing the Dockerfile.
2. Build the image:

```bash
docker build -t pulse-crop-recommendation:1.0 .
```

3. Run the container:

```bash
docker run --name pulse-crop-api -p 8000:8000 pulse-crop-recommendation:1.0
```

4. Open the API documentation:

```text
http://127.0.0.1:8000/docs
```

To stop the container, press `Ctrl+C` if running in the foreground. To remove a stopped container:

```bash
docker rm pulse-crop-api
```

## 9. API Usage

The supplied Docker application exposes two endpoints.

### Health check

**Method:** `GET`  
**Endpoint:** `/health`

Example:

```bash
curl http://127.0.0.1:8000/health
```

A successful response reports the service status and model version:

```json
{
  "status": "healthy",
  "model_version": "1.0.0"
}
```

### Crop prediction

**Method:** `POST`  
**Endpoint:** `/predict`  
**Content-Type:** `application/json`

Example request:

```json
{
  "N": 40,
  "P": 60,
  "K": 40,
  "temperature": 22.0,
  "humidity": 60.0,
  "ph": 6.5,
  "rainfall": 100.0
}
```

Example response structure:

```json
{
  "recommended_crop": "predicted crop label",
  "confidence": 0.95,
  "model_version": "1.0.0"
}
```

The crop label and confidence value shown above are illustrative response values, not a guaranteed prediction for the example input. The actual result is generated by the saved model.

The confidence field is the highest value returned by `predict_proba`. It is a model class score and has not been independently calibrated as a probability of real-world crop suitability.

### Input validation

The API validates the request using Pydantic:

- `N`, `P`, and `K` must be non-negative.
- `humidity` must be between 0 and 100.
- `ph` must be between 0 and 14.
- `rainfall` must be non-negative.
- All seven fields are required and must be numerical.

Invalid requests are rejected with an HTTP 422 validation response. Unexpected prediction errors return an HTTP 500 response.

## 10. Testing and Monitoring

The project submission includes API test evidence covering health checks, prediction requests, invalid input types, and out-of-range values. The submission also includes automated test evidence showing six tests passed.

The portfolio package contains monitoring evidence for API health and prediction responses. These artifacts demonstrate prototype-level API checks; they do not establish continuous production monitoring or long-term model performance in real agricultural settings.

For further monitoring, review API availability, failed requests, input validation errors, prediction behavior, and feedback from qualified agricultural users. Avoid treating a successful health check as proof that predictions are agronomically correct.

## 11. Limitations and Responsible Use

- The final working dataset contains only 400 records.
- The dataset's representativeness of Ethiopian soil, climate, and farming conditions has not been independently established.
- The model supports only four pulse crop labels.
- The model does not predict yield, profitability, disease, pest risk, or fertilizer requirements.
- Predictions may be less reliable for inputs outside the ranges observed in the training data.
- A high model confidence score is not a calibrated measure of crop suitability.
- Perfect evaluation results on this small dataset do not guarantee performance on independent data.

**Disclaimer:** Pulse Crop Recommendation is an educational and preliminary decision-support prototype. Do not use its output as the sole basis for farming decisions. Consult local agronomists, agricultural extension professionals, and soil-testing services before selecting crops.

## 12. Future Improvements

- Collect and validate larger datasets from Ethiopian pulse-producing regions.
- Include multiple locations, soil types, and growing seasons.
- Work with agronomists and agricultural extension professionals to validate crop suitability.
- Evaluate the model using independent external datasets.
- Monitor prediction performance as new labeled data becomes available.
- Improve input guidance and communicate uncertainty more clearly.
- Expand supported pulse crops only after obtaining suitable data and validation evidence.

## 13. Project Materials

The wider project submission includes:

- `Pulse_Crop_Recommendation.ipynb` — model development and evaluation notebook.
- `Model Card.pdf` — model purpose, performance, limitations, and technical details.
- `Data Statement.pdf` — dataset description, preparation, and limitations.
- `Support and Improvement Plan.pdf` — proposed maintenance and improvement activities.
- API and Docker test evidence.

## 14. Team and Acknowledgement

This project was developed collaboratively by Group 4. Team member names, role assignments, and individual contributions should be maintained in the portfolio package or repository documentation.

---

**Project title:** Pulse Crop Recommendation Based on Soil and Climate Condition  
**System name:** Pulse Crop Recommendation  
**Model version:** 1.0.0  
**Project status:** Educational prototype
