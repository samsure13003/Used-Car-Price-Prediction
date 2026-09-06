# 🚗 Used Car Price Prediction

An end-to-end machine learning project that predicts the resale (selling) price of a used car from its specifications — model, age, mileage, fuel type, transmission, and more. Built as a modular ML pipeline (ingestion → transformation → training → prediction) and served through a Flask web app, containerized with Docker and deployable to AWS Elastic Beanstalk.

## 🔗 Live Demo / Repo

- Repository: https://github.com/samsure13003/Used-Car-Price-Prediction

## ✨ Features

- **Modular ML pipeline** — separate components for data ingestion, preprocessing/transformation, and model training, orchestrated via `src/pipeline`.
- **Automated model selection** — trains multiple regressors (Linear, Ridge, Lasso, KNN, Decision Tree, Random Forest, SVR, AdaBoost) with `RandomizedSearchCV` hyperparameter tuning, and automatically picks the best performer by R² score.
- **Web UI for predictions** — a Flask app with an HTML form (`templates/index.html`) where users enter car details and get an instant price estimate.
- **Custom logging & exception handling** — every pipeline stage logs to timestamped log files and raises detailed, traceable custom exceptions.
- **Production-ready packaging** — `setup.py` for installable packaging, `Dockerfile` for containerized deployment, and `.ebextensions` config for AWS Elastic Beanstalk.

## 📚 Libraries Used

From `requirements.txt`:

| Library | Purpose |
|---|---|
| `pandas` | Data loading, manipulation, and DataFrame operations |
| `numpy` | Numerical operations and array handling |
| `matplotlib` | Data visualization / EDA plots |
| `seaborn` | Statistical data visualization for EDA |
| `scikit-learn` | Preprocessing (scaling, encoding, imputation), regression models, hyperparameter search, and evaluation metrics |
| `flask` | Web framework serving the prediction UI and API routes |
| `gunicorn` | Production WSGI server for running the Flask app |
| `dill` | Extended object serialization (pickling) |
| `ipykernel` | Jupyter kernel support for running the EDA/training notebooks |

Also used in the codebase (installed as dependencies of the above):

| Library | Purpose |
|---|---|
| `joblib` | Saving/loading the trained model and preprocessor (`artifacts/model.pkl`, `artifacts/preprocessor.pkl`) |

Install everything with:

```bash
pip install -r requirements.txt
```

## 🏗️ Project Structure

```
Used_car_CI_CD/
├── application.py                     # Flask app entry point (routes: / and /predict)
├── artifacts/                         # Generated at runtime: train/test CSVs, preprocessor.pkl, model.pkl
├── notebook/
│   ├── data/cardekho_imputated.csv    # Source dataset
│   ├── EDA_Used_car_price.ipynb       # Exploratory data analysis
│   └── Used_car_price_model_training.ipynb
├── src/
│   ├── components/
│   │   ├── data_injestion.py          # Reads raw data, splits train/test
│   │   ├── data_transformation.py     # Builds preprocessing pipeline (scaling, encoding)
│   │   └── model_trainer.py           # Trains & selects the best regression model
│   ├── pipeline/
│   │   ├── train_pipeline.py
│   │   └── predict_pipeline.py        # Loads saved model/preprocessor and serves predictions
│   ├── exception.py                   # Custom exception handling
│   ├── logger.py                      # Logging configuration
│   └── utils.py                       # save_object / load_object / evaluate_model helpers
├── templates/
│   └── index.html                     # Frontend form for predictions
├── Dockerfile
├── .ebextensions/python.config        # AWS Elastic Beanstalk WSGI config
├── requirements.txt
└── setup.py
```

## 🧠 Model Pipeline

1. **Data Ingestion** (`data_injestion.py`) — loads `cardekho_imputated.csv`, saves a raw copy, and splits it into train/test sets (80/20).
2. **Data Transformation** (`data_transformation.py`) — builds a `ColumnTransformer` that:
   - Scales numerical features (`vehicle_age`, `km_driven`, `mileage`, `engine`, `max_power`, `seats`) with median imputation + `StandardScaler`.
   - One-hot encodes categorical features (`seller_type`, `fuel_type`, `transmission_type`).
   - Ordinal-encodes the `model` column, then scales it.
   - Saves the fitted preprocessor to `artifacts/preprocessor.pkl`.
3. **Model Training** (`model_trainer.py`) — trains 8 candidate regressors with hyperparameter search, evaluates each on the test set, and persists the best one to `artifacts/model.pkl`.
4. **Prediction** (`predict_pipeline.py`) — wraps user input into a DataFrame (`CustomData`), applies the saved preprocessor, and runs it through the saved model.

## 🚀 Getting Started

### Prerequisites

- Python 3.11+
- pip

### Installation

```bash
git clone https://github.com/samsure13003/Used-Car-Price-Prediction.git
cd Used-Car-Price-Prediction
pip install -r requirements.txt
```

### Train the model

This runs ingestion → transformation → training end to end, and populates `artifacts/`:

```bash
python src/components/data_injestion.py
```

### Run the web app

```bash
python application.py
```

Then open `http://localhost:5000` in your browser, fill in the car details, and get a predicted price.

## 🐳 Docker

Build and run the app in a container:

```bash
docker build -t used-car-price-prediction .
docker run -p 5000:5000 used-car-price-prediction
```

## ☁️ Deployment

The project is set up for deployment on **AWS Elastic Beanstalk**:

- `.ebextensions/python.config` points EB's WSGI server at `application:application`.
- The Docker image can alternatively be deployed to any container platform (ECS, EC2, etc.) via the included `Dockerfile`.

## 🛠️ Tech Stack

| Layer | Tools |
|---|---|
| Language | Python 3.11 |
| ML | scikit-learn, pandas, numpy |
| Serving | Flask, Gunicorn |
| Packaging | Docker, setuptools |
| Deployment | AWS Elastic Beanstalk |
| Serialization | joblib / dill |

## 📊 Input Features

| Feature | Description |
|---|---|
| `model` | Car model name |
| `vehicle_age` | Age of the vehicle (years) |
| `km_driven` | Total kilometers driven |
| `seller_type` | Individual / Dealer / Trustmark Dealer |
| `fuel_type` | Petrol / Diesel / CNG / Electric / LPG |
| `transmission_type` | Manual / Automatic |
| `mileage` | Fuel efficiency (kmpl) |
| `engine` | Engine displacement (cc) |
| `max_power` | Max power output (bhp) |
| `seats` | Number of seats |

**Target:** `selling_price`

## 👤 Author

**Samsur Rahman**
📧 samsurerahman13003@gmail.com

## 📄 License

No license file is currently included — add one (e.g. MIT) if you plan to open this project up for reuse.
