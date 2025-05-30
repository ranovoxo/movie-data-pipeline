Here's the **README.md** file for your Movie Data ETL Pipeline project:

---

# Movie Data ETL Pipeline with Airflow & PostgreSQL (Dockerized)

This is a fully Dockerized data engineering pipeline that extracts movie data from a public API, stores raw data in PostgreSQL, transforms it into silver and gold tables, and orchestrates the entire process with Apache Airflow. The project also includes structured logging to track ETL stages in detail.

---

## Project Structure
```
movie_data_pipeline/
│
├── dags/
│   ├── init_schema.py              # Creates DB schema on Airflow start
│   └── movie_etl_dag.py            # Airflow DAG definition
│
├── src/
│   ├── extract_movies.py           # Extract movie metadata
│   ├── extract_genres.py           # Extract genre data
│   ├── extract_budget_revenue.py   # Extract budget and revenue data
│   ├── transform_silver_layer.py   # Clean and deduplicate raw data
│   ├── transform_gold_layer.py     # Enrich and finalize analytics-ready data
│   ├── logger.py                   # Custom logger for ETL steps
│   └── tableau/
│       └── hyper_exports/          # CSVs for Tableau dashboards
│           ├── avg_rating_by_lang.csv
│           ├── gold_top_movies.csv
│           └── yearly_counts.csv
│
├── ml/
│   ├── predict_genere.py 
│   ├── preprocess_text.py          # Preprocess text for overview column for machine learning training
│   ├── train_genre_multilabel.py   # Train the model
│   ├── utils.py
├── sql/
│   └── create_table.sql            # PostgreSQL table creation script
│
├── db/
│   └── db_connector.py             # DB connection helper
│
├── logs/                           # ETL & Airflow logs
│   ├── etl/
│   │   ├── extract.log
│   │   ├── transform.log
│   │   └── load.log
│   ├── scheduler/                  # Airflow scheduler logs
│   ├── dag_id=movie_data_etl/     # Logs by DAG run and task
│   └── dag_processor_manager/     # DAG processor logs
│
├── jars/
│   └── postgresql-42.7.1.jar       # JDBC driver for PostgreSQL
│
├── docker-compose.yml             # Docker config for Airflow, Postgres, etc.
├── Dockerfile                     # Custom Airflow image with dependencies
├── requirements.txt               # Python dependencies
├── .env                           # Environment variables (DB, secrets)
├── README.md                      # Project documentation
└── command_line_notes.md          # Helpful CLI commands during development
```

---

## Pipeline Stages

### Extract

* Pulls movie data from a public API or scrape source.

### Load Raw

* Inserts unprocessed data into the `raw_movies` table in PostgreSQL.

### Transform Silver

* Cleans and standardizes the data into `silver_movies`.

### Transform Gold

* Aggregates insights into `gold_movies`.

### Orchestration

* Apache Airflow manages and schedules all tasks.

---

## Technologies Used

* **Python 3.10+**
* **PostgreSQL**
* **Apache Airflow 2.7+**
* **Docker & Docker Compose**
* **BeautifulSoup / Requests** (for scraping)
* **Pandas** (for data wrangling)
* **Custom Logging** (per ETL step)

---

## Requirements

* `apache-airflow==2.7.1`
* `psycopg2-binary`
* `requests`
* `beautifulsoup4`
* `pandas`
* `python-dotenv`

---

## Setup Instructions (Dockerized)

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/movie-data-pipeline.git
cd movie-data-pipeline
```

### 2. Create `.env`

Create an `.env` file in the root of the project directory with the following content:

```plaintext
POSTGRES_USER=airflow
POSTGRES_PASSWORD=airflow
POSTGRES_DB=movies
POSTGRES_HOST=postgres
POSTGRES_PORT=5432
```

### 3. Build and Start Services

Run the following command to build and start the Docker services:

```bash
docker-compose up --build
```

This starts:

## Services Overview

- **Airflow Webserver**  
  Access the Airflow UI at: [http://localhost:8080](http://localhost:8080)

- **Airflow Scheduler**  
  Responsible for scheduling and triggering DAG tasks.

- **PostgreSQL Database**  
  Runs on `localhost:5432`. Used as the main data warehouse for storing:
  - `raw_movies`
  - `raw_genres`
  - `raw_finances`
  - `movies_silver`
  - `movies_gold`

- **pgAdmin**  
  Access pgAdmin UI at: [http://localhost:5050](http://localhost:5050)  
  Default login: `admin@admin.com / [your password in .env or Docker secrets]`


### Default Airflow credentials:

* **Username**: airflow
* **Password**: airflow

### 4. Initialize Airflow

Run the following commands to initialize the Airflow database and create the default user:

```bash
docker-compose exec airflow-webserver airflow db init
docker-compose exec airflow-webserver airflow users create \
  --username airflow --password airflow \
  --firstname Air --lastname Flow --role Admin --email airflow@example.com
```

### 5. Create Database Tables

Run the scripts in the `sql/` directory using a tool like **DBeaver**, **pgAdmin**, or with a DAG task to auto-init the schema in PostgreSQL.

---

## Airflow DAG Tasks

- **`extract_movies`**  
  Fetches raw movie metadata from the TMDB API and stores it in the `raw_movies` table.

- **`extract_genres`**  
  Retrieves the latest genre mappings from TMDB and stores them in the `raw_genres` table.

- **`extract_financials`**  
  Collects budget and revenue details for each movie and saves them to the `raw_finances` table.

- **`transform_to_silver`**  
  Cleans, normalizes, and enriches the raw data (e.g., mapping genre IDs to names) and writes the results to the `movies_silver` table.

- **`transform_to_gold`**  
  Aggregates silver-level data (e.g., total revenue by genre) and creates analytical summaries in the `movies_gold` table.

---

## Logging

Each ETL step uses a shared `logger.py` utility for structured logging into the `logs/etl/` directory:

* `logs/etl/extract.log`
* `logs/etl/transform.log`
* `logs/etl/load.log`

These logs help you monitor step-by-step progress, errors, and runtime metrics. The `logs/` folder is mounted as a Docker volume so logs persist on the host machine even after container shutdown.

### Sample log entry:

```plaintext
2025-05-05 12:00:00 - INFO - Starting extraction task...
```

---

## Improvements & Ideas

* Add **data validation** (using tools like **Great Expectations** or custom validation rules).
* Dockerize for **production** (multi-container deployment).
* Integrate a **dashboard** (using **Streamlit** or **Superset**) to visualize gold-layer insights.
* Schedule with **cron** + Airflow variables.
* Add **unit tests** for `src/` logic.

---

## License

This project is licensed under the **MIT License**.

