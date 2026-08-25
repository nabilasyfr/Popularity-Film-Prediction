# 🎬 CineAnalytics — DVD Rental Intelligence System
### Advanced Database Final Exam Project

---

## 📋 Project Overview

A full-stack Django data analytics system built on the PostgreSQL DVD Rental database.
Features OLAP star schema, ETL pipeline, ML models (recommender, predictor, clustering).

---

## 🗂️ Project Structure

```
dvd_analytics/
├── dvd_analytics/          # Django project config
│   ├── settings.py
│   └── urls.py
├── etl/                    # ETL Pipeline app
│   ├── models.py           # OLAP models (dim_*, fact_*, stats)
│   ├── pipeline.py         # Extract / Transform / Load logic
│   ├── views.py
│   └── urls.py
├── ml/                     # Machine Learning app
│   ├── engine.py           # Recommender, Predictor, Clustering
│   ├── models.py           # MLModelRecord, Recommendation
│   ├── views.py
│   └── urls.py
├── dashboard/              # Analytics Dashboard app
│   ├── views.py
│   └── urls.py
├── templates/
│   ├── base/layout.html    # Shared sidebar + topbar
│   ├── dashboard/index.html
│   ├── etl/index.html
│   └── ml/index.html
├── sql/
│   └── olap_queries.sql    # All SQL: DDL + ETL + OLAP
├── ml_models/              # Saved .pkl files (auto-created)
├── requirements.txt
└── README.md
```

---

## ⚙️ Setup Instructions

### 1. Install PostgreSQL & Restore DVD Rental Database

```bash
# Download the DVD Rental database
# https://www.postgresqltutorial.com/postgresql-getting-started/postgresql-sample-database/

psql -U postgres -c "CREATE DATABASE dvdrental;"
pg_restore -U postgres -d dvdrental dvdrental.tar

# Create the OLAP database
psql -U postgres -c "CREATE DATABASE dvdrental_olap;"
```

### 2. Clone & Set Up Python Environment

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate          # Linux / macOS
venv\Scripts\activate             # Windows

# Install dependencies
pip install -r requirements.txt
```

### 3. Configure Database Credentials

Edit `dvd_analytics/settings.py`:
```python
DATABASES = {
    'default': {          # OLTP source (dvdrental)
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'dvdrental',
        'USER': 'postgres',
        'PASSWORD': 'your_password',
        'HOST': 'localhost',
        'PORT': '5432',
    },
    'olap': {             # OLAP target (dvdrental_olap)
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'dvdrental_olap',
        'USER': 'postgres',
        'PASSWORD': 'your_password',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```

### 4. Run Django Migrations

```bash
python manage.py makemigrations etl ml
python manage.py migrate
```

### 5. Run the Development Server

```bash
python manage.py runserver
```

Open: **http://127.0.0.1:8000/**

---

## 🚀 Usage Guide

### Step 1 — Run ETL Pipeline
<img width="1911" height="853" alt="Screenshot 2026-04-05 221255" src="https://github.com/user-attachments/assets/a65d8f04-2305-4817-9c19-0b52c94d8289" />

Navigate to **ETL Pipeline** → Click **🚀 Run Full ETL Pipeline**
- Extracts from OLTP `dvdrental`
- Transforms and aggregates data (film stats, customer stats, genre stats)
- Loads into OLAP star schema tables

### Step 2 — Train ML Models
<img width="1911" height="911" alt="Screenshot 2026-04-05 162252" src="https://github.com/user-attachments/assets/6991dde8-f836-4ccc-8c98-347ce1e84836" />

Navigate to **ML Models** → Train each model:
1. **🎬 Film Recommender** — Click "Train Model" to build cosine-similarity matrix
2. **📈 Popularity Predictor** — Click "Train Model" to fit GradientBoosting regressor
3. **👥 Customer Clustering** — Set k value, click "Train Model" for KMeans

### Step 3 — View Dashboard
<img width="1910" height="951" alt="Screenshot 2026-04-04 164706" src="https://github.com/user-attachments/assets/a4ada022-540c-4d54-bd58-363e2c55ce73" />

Navigate to **Dashboard** to see:
- KPI cards (rentals, revenue, customers, top film)
- Rental trend line chart
- Genre doughnut chart
- Top 10 films bar chart
- Popularity radar chart
- Revenue by genre chart
- Film leaderboard table

### Step 4 — Run Inference
In **ML Models** → Live Inference section:
- **Recommendations**: Select a customer → "Get Recommendations"
- **Popularity Prediction**: Select films → "Predict Rentals"
- **Customer Segment**: Select a customer → "Identify Segment"

---

## 🧱 Tech Stack

| Layer         | Technology                            |
|---------------|---------------------------------------|
| Backend       | Django 4.2                            |
| Database      | PostgreSQL (OLTP + OLAP)              |
| Data Layer    | Pandas, NumPy                         |
| ML            | Scikit-learn, Joblib                  |
| Frontend      | HTML/CSS/JS, Chart.js 4               |
| Fonts         | Space Grotesk, DM Mono                |

---

## 🤖 ML Models

### 1. Film Recommender (Collaborative Filtering)
- **Algorithm**: Item-based cosine similarity
- **Input**: Customer–film rental matrix (binary)
- **Output**: Top-N unseen films ranked by similarity to watched films
- **Cold Start**: Falls back to global popularity ranking

### 2. Film Popularity Predictor (Regression)
- **Algorithm**: GradientBoostingRegressor (200 estimators)
- **Features**: rental_rate, length, replacement_cost, category (one-hot)
- **Target**: total_rentals
- **Metrics**: MAE, R² Score

### 3. Customer Preference Clustering (KMeans)
- **Algorithm**: KMeans (default k=5)
- **Input**: Customer × Genre rental frequency matrix
- **Output**: Segment labels (Action Lovers, Drama Fans, Comedy Seekers, etc.)
- **Metric**: Silhouette Score

---

## 🗄️ OLAP Star Schema

```
        dim_customer ─────┐
        dim_film      ─────┤──── fact_rental ────┬──── dim_date
        dim_category  ─────┘                     │
                                                  └──── (all FKs)

Aggregated tables:
  film_stats       — per-film rollup
  customer_stats   — per-customer rollup
  genre_stats      — per-genre rollup
```

---

## 📊 ETL Transformation Logic

| Transform          | Description                                          |
|--------------------|------------------------------------------------------|
| total_rentals      | COUNT(rental_id) GROUP BY film_id                    |
| rentals_per_cust   | COUNT(rental_id) GROUP BY customer_id                |
| genre_frequency    | COUNT per customer per category → favorite genre     |
| last_rental_date   | MAX(rental_date) per customer                        |
| popularity_score   | 0.7 × normalized_rentals + 0.3 × normalized_revenue  |
| rental_duration    | (return_date − rental_date) in days                  |

---
dvd_analytics/                    ← Folder utama project
│
├── dvd_analytics/                ← Pengaturan Django
│   ├── settings.py               ← Konfigurasi database, app, dll
│   └── urls.py                   ← Peta URL seluruh project
│
├── etl/                          ← App ETL Pipeline
│   ├── models.py                 ← Definisi tabel OLAP 
│   ├── pipeline.py               ← Logika Extract Transform Load
│   ├── views.py                  ← Handler tombol ETL
│   └── urls.py                   ← URL untuk ETL
│
├── ml/                           ← App Machine Learning
│   ├── engine.py                 ← Logika 3 model ML
│   ├── models.py                 ← Tabel metadata model
│   ├── views.py                  ← Handler train & predict
│   └── urls.py                   ← URL untuk ML
│
├── dashboard/                    ← App Dashboard
│   ├── views.py                  ← Query data + kirim ke chart
│   └── urls.py                   ← URL untuk dashboard
│
├── templates/                    ← Semua file HTML
│   ├── base/layout.html          ← Template induk (sidebar, navbar)
│   ├── etl/index.html            ← Halaman ETL
│   ├── ml/index.html             ← Halaman ML
│   └── dashboard/index.html      ← Halaman Dashboard
│
├── ml_models/                    ← File model tersimpan
│   ├── recommender.pkl
│   ├── popularity.pkl
│   └── clustering.pkl
│
└── manage.py                     ← Perintah Django
## 👥 Team Roles (Suggestion)

| Member | Focus Area                                  |
|--------|---------------------------------------------|
| A      | ETL Pipeline + Star Schema SQL              |
| B      | Film Recommender + Dashboard Charts         |
| C      | Popularity Prediction + KPI Cards           |
| D      | Customer Clustering + Segment Analysis      |
| E      | Frontend UI + Integration                   |

---

## 📝 Notes

- All models are saved as `.pkl` files in the `ml_models/` directory
- Re-running ETL will clear and reload all OLAP tables
- Re-training a model will overwrite the existing `.pkl` file
- The recommender falls back to popularity ranking for new customers (cold-start)
