# ⚽ Football Analytics Data Platform — Bootcamp Data Engineering (VS Code Edition)

> Pipeline ELT complet — Python · PostgreSQL (Docker) · PySpark/Delta Lake local · Snowflake · dbt Core · Power BI
> Développé de bout en bout dans **VS Code**, sans dépendance à un service cloud payant.

![Python](https://img.shields.io/badge/Python-3.11%2B-blue)
![dbt](https://img.shields.io/badge/dbt-Core-orange)
![Snowflake](https://img.shields.io/badge/Snowflake-Free%20Trial-29B5E8)
![PySpark](https://img.shields.io/badge/PySpark-Delta%20Lake-E25A1C)
![Power BI](https://img.shields.io/badge/Power%20BI-Desktop-F2C811)
![License](https://img.shields.io/badge/license-MIT-lightgrey)

---

## Sommaire
🗺️ Architecture globale — Vue d'ensemble
Schéma du flux de données
plain
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PHASE 1 — SOURCES & INGESTION                       │
│  ┌──────────────┐  ┌──────────────────┐                                      │
│  │ Kaggle CSV   │  │ API football-data│                                      │
│  │ (historique) │  │ (temps réel)     │                                      │
│  └──────┬───────┘  └────────┬─────────┘                                      │
│         │                   │                                                │
│         ▼                   ▼                                                │
│  ┌─────────────────────────────────────┐                                     │
│  │  SCRIPTS PYTHON (local)            │                                     │
│  │  • scripts/ingest_kaggle.py       │  → écrit dans →  data/raw/kaggle/ │
│  │  • scripts/ingest_api.py            │  → écrit dans →  data/raw/api/    │
│  │  • scripts/utils/                   │     (CSV / Parquet)                 │
│  └─────────────────────────────────────┘                                     │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    PHASE 2 — STAGING RELATIONNEL (PostgreSQL)               │
│                                                                              │
│  sql/postgres/init/01_create_database.sql        →  CREATE DATABASE        │
│  sql/postgres/init/02_create_staging_tables.sql  →  CREATE TABLE staging.* │
│                                                                              │
│  scripts/load_to_postgres.py                     →  COPY CSV → PostgreSQL  │
│                                                                              │
│  sql/postgres/queries/validation.sql             →  SELECT COUNT(*)...   │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼ (export Parquet)
┌─────────────────────────────────────────────────────────────────────────────┐
│              PHASE 3 — LAKEHOUSE (Databricks Community Edition)             │
│                                                                              │
│  notebooks/databricks/01_bronze_ingestion.py     →  DBFS → bronze.matches   │
│  notebooks/databricks/02_silver_cleaning.py    →  bronze → silver.matches  │
│  notebooks/databricks/03_gold_analytics.py     →  silver → gold.*          │
│                                                                              │
│  sql/databricks/validation.sql                 →  requêtes de contrôle     │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼ (export ou connecteur)
┌─────────────────────────────────────────────────────────────────────────────┐
│              PHASE 4 — DATA WAREHOUSE (Snowflake Free Trial)                  │
│                                                                              │
│  sql/snowflake/01_setup.sql          →  WAREHOUSE + DATABASE + SCHEMAS     │
│  sql/snowflake/02_load_raw.sql       →  STAGE + COPY INTO raw.*            │
│  sql/snowflake/03_cleaned_views.sql  →  VUES dans CLEANED                  │
│  sql/snowflake/04_analytics.sql      →  TABLES dans ANALYTICS              │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│              PHASE 5 — TRANSFORMATION (dbt Core)                              │
│                                                                              │
│  dbt-football/dbt_project.yml        →  config projet                       │
│  dbt-football/packages.yml           →  dbt_utils                            │
│  dbt-football/profiles.yml           →  connexion Snowflake                 │
│                                                                              │
│  dbt-football/models/staging/        →  stg_matches.sql, sources.yml...    │
│  dbt-football/models/marts/          →  fact_matches.sql, dim_*.sql       │
│  dbt-football/snapshots/             →  scd2_player_stats.sql               │
│  dbt-football/seeds/                 →  leagues.csv, countries.csv          │
│  dbt-football/macros/                →  fonctions réutilisables             │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│              PHASE 6 — VISUALISATION (Power BI Desktop)                       │
│                                                                              │
│  powerbi/football_analytics.pbix     →  rapport Power BI                    │
│  powerbi/measures.dax               →  fichier texte des mesures DAX      │
└─────────────────────────────────────────────────────────────────────────────┘
📋 Plan de travail — 6 Phases techno par techno
Règle d'or : Tu ne crées un fichier que quand on arrive à la phase qui en a besoin. Pas avant.
Feuilles de calcul
Phase	Techno	Objectif	Fichiers à créer à cette étape
| Phase | Techno     | Objectif                     | Fichiers à créer à cette étape                                                          |
| ----- | ---------- | ---------------------------- | --------------------------------------------------------------------------------------- |
| **0** | Setup      | Préparer l'environnement     | `requirements.txt`, `.gitignore`, `config/.env`                                         |
| **1** | Python     | Ingestion des données        | `scripts/ingest_api.py`, `scripts/ingest_kaggle.py`, `scripts/utils/*`                  |
| **2** | PostgreSQL | Staging relationnel          | `sql/postgres/init/*`, `scripts/load_to_postgres.py`, `sql/postgres/queries/*`          |
| **3** | Databricks | Lakehouse Bronze/Silver/Gold | `notebooks/databricks/01_*.py`, `02_*.py`, `03_*.py`, `sql/databricks/validation.sql`   |
| **4** | Snowflake  | Data Warehouse               | `sql/snowflake/01_setup.sql` → `04_*.sql`                                               |
| **5** | dbt        | Modélisation & tests         | `dbt-football/dbt_project.yml`, `profiles.yml`, `models/**/*`, `snapshots/*`, `seeds/*` |
| **6** | Power BI   | Dashboards                   | `powerbi/measures.dax`, `.pbix`                                                         |

📁 Arborescence progressive — Ce qui existe à la fin de chaque phase
Voici exactement à quoi ressemble ton dossier à la fin de chaque phase. Cela te montre quand chaque fichier apparaît.
🔧 PHASE 0 — Setup (Environnement de travail)
Tu crées maintenant :
plain
football-lab/
├── .gitignore
├── requirements.txt
├── config/
│   └── .env
└── README.md
Pourquoi ces fichiers ?
requirements.txt → liste des librairies Python à installer
config/.env → tes mots de passe et clés API (jamais sur Git)
.gitignore → empêche Git de tracker les données et les secrets
🐍 PHASE 1 — Python & Ingestion
Tu ajoutes à la structure :
plain
football-lab/
├── data/
│   ├── raw/
│   │   ├── kaggle/          ← (dossier vide, les CSV arriveront ici)
│   │   └── api/             ← (dossier vide, les Parquet arriveront ici)
│   └── processed/
├── scripts/
│   ├── __init__.py
│   ├── ingest_api.py        ← appelle l'API football-data.org
│   ├── ingest_kaggle.py     ← télécharge les datasets Kaggle
│   ├── utils/
│   │   ├── __init__.py
│   │   ├── api_client.py    ← classe pour appeler l'API proprement
│   │   ├── data_cleaner.py  ← fonctions de nettoyage pandas
│   │   └── db_connector.py  ← connexion PostgreSQL (préparation Phase 2)
│   └── sql/
│       ├── 01_create_database.sql
│       ├── 02_create_staging_tables.sql
│       └── 03_validation_queries.sql
Résultat attendu : Des fichiers data/raw/api/matches_pl_20250125.parquet et data/raw/kaggle/results.csv.
🐘 PHASE 2 — PostgreSQL (Staging)
Tu ajoutes à la structure :
plain
football-lab/
├── sql/
│   └── postgres/
│       ├── init/
│       │   ├── 01_create_database.sql      ← CREATE DB + SCHEMA + USER
│       │   ├── 02_create_staging_tables.sql ← CREATE TABLE staging.*
│       │   └── 03_create_indexes.sql       ← CREATE INDEX
│       └── queries/
│           ├── validation.sql              ← COUNT, DISTINCT, doublons
│           └── analytics.sql               ← premières requêtes analytiques
└── scripts/
    └── load_to_postgres.py                 ← lit Parquet/CSV → INSERT INTO PostgreSQL
Résultat attendu : Une base football_lab avec des tables staging.matches, staging.teams, peuplées et validées par SQL.
🔥 PHASE 3 — Databricks (Medallion)
Tu ajoutes à la structure :
plain
football-lab/
├── notebooks/
│   ├── databricks/
│   │   ├── 01_bronze_ingestion.py     ← CSV/Parquet → bronze.matches (Delta)
│   │   ├── 02_silver_cleaning.py      ← bronze → silver.matches_clean (Delta)
│   │   └── 03_gold_analytics.py       ← silver → gold.match_stats_by_league (Delta)
│   └── exploratory/
│       └── 00_eda_football.ipynb      ← (optionnel) analyse locale avec Jupyter
└── sql/
    └── databricks/
        └── validation.sql             ← requêtes de contrôle dans Databricks
Résultat attendu : 3 tables Delta Lake (bronze.*, silver.*, gold.*) dans Databricks, avec historique et nettoyage.
❄️ PHASE 4 — Snowflake (Data Warehouse)
Tu ajoutes à la structure :
plain
football-lab/
└── sql/
    └── snowflake/
        ├── 01_setup.sql           ← CREATE WAREHOUSE + DATABASE + SCHEMAS
        ├── 02_load_raw.sql        ← CREATE STAGE + COPY INTO raw.*
        ├── 03_cleaned_views.sql   ← VUES dans CLEANED
        └── 04_analytics_tables.sql ← TABLES dans ANALYTICS (pré-dbt)
Résultat attendu : Database FOOTBALL_LAB avec schémas RAW, CLEANED, ANALYTICS peuplés.
🔄 PHASE 5 — dbt (Transformation & Modélisation)
Tu ajoutes à la structure :
plain
football-lab/
└── dbt-football/
    ├── dbt_project.yml
    ├── packages.yml
    ├── profiles.yml → ../config/profiles.yml  (lien ou copie)
    ├── seeds/
    │   ├── leagues.csv
    │   └── countries.csv
    ├── snapshots/
    │   └── scd2_player_stats.sql
    ├── macros/
    │   ├── generate_schema_name.sql
    │   └── tests/
    │       └── custom_test.sql
    ├── models/
    │   ├── staging/
    │   │   ├── sources.yml
    │   │   ├── staging.yml
    │   │   ├── stg_matches.sql
    │   │   ├── stg_teams.sql
    │   │   ├── stg_players.sql
    │   │   └── stg_competitions.sql
    │   └── marts/
    │       ├── schema.yml
    │       ├── dim_teams.sql
    │       ├── dim_players.sql
    │       ├── dim_competitions.sql
    │       ├── dim_date.sql
    │       └── fact_matches.sql
    └── tests/
        └── generic/
Résultat attendu : Modèle en étoile déployé sur Snowflake, tests passants, documentation générée.
📊 PHASE 6 — Power BI (Visualisation)
Tu ajoutes à la structure :

mkdir football-lab && cd football-lab
python -m venv venv
# Windows : venv\Scripts\activate | Linux/Mac : source venv/bin/activate
pip install pandas requests python-dotenv sqlalchemy psycopg2-binary pyarrow pyspark delta-spark kaggle



football-lab/
└── powerbi/
    ├── football_analytics.pbix    ← (créé via l'interface Power BI)
    ├── measures.dax               ← copie de sauvegarde de tes mesures
    └── themes/
        └── football_theme.json    ← (optionnel) couleurs du dashboard
Résultat attendu : Dashboard Power BI connecté à Snowflake, 3 pages (Vue globale, Équipe, Joueur).
✅ Checklist de démarrage — Que faire MAINTENANT ?
Tu ne dois créer que la PHASE 0 pour l'instant. Voici les 4 actions immédiates :
plain
□ 1. Créer le dossier racine
      mkdir football-lab && cd football-lab

□ 2. Créer les 3 fichiers de la Phase 0
      • requirements.txt
      • .gitignore
      • config/.env

□ 3. Créer l'environnement virtuel
      python -m venv venv
      source venv/bin/activate  (Mac/Linux)
      venv\Scripts\activate     (Windows)

□ 4. Installer les dépendances
      pip install -r requirements.txt

🐍 PHASE 1 — Python & Ingestion
1.1 Créer les dossiers nécessaires
Depuis la racine football-lab/ :
bash
mkdir -p data/raw/kaggle
mkdir -p data/raw/api
mkdir -p data/processed
mkdir -p scripts/utils
mkdir -p scripts/sql
1.2 Créer les fichiers utils
scripts/__init__.py
bash
touch scripts/__init__.py
touch scripts/utils/__init__.py
(Fichier vide, juste pour faire de scripts un package Python)
scripts/utils/api_client.py
Python
"""Client API football-data.org"""

import os
import time
import requests
from typing import List, Dict, Any


class FootballDataClient:
    BASE_URL = "https://api.football-data.org/v4"
    
    def __init__(self, api_key: str = None):
        self.api_key = api_key or os.getenv("FOOTBALL_DATA_API_KEY")
        if not self.api_key:
            raise ValueError("Clé API manquante. Définissez FOOTBALL_DATA_API_KEY dans .env")
        self.headers = {"X-Auth-Token": self.api_key}
    
    def _get(self, endpoint: str, params: dict = None) -> dict:
        url = f"{self.BASE_URL}{endpoint}"
        resp = requests.get(url, headers=self.headers, params=params or {})
        if resp.status_code == 429:
            time.sleep(60)  # Rate limit
            return self._get(endpoint, params)
        resp.raise_for_status()
        return resp.json()
    
    def get_competitions(self) -> List[Dict[str, Any]]:
        return self._get("/competitions").get("competitions", [])
    
    def get_matches(self, competition_code: str = "PL", season: int = 2024) -> List[Dict[str, Any]]:
        """Récupère les matchs d'une compétition (ex: PL = Premier League)"""
        data = self._get(f"/competitions/{competition_code}/matches", {"season": season})
        return data.get("matches", [])
    
    def get_teams(self, competition_code: str = "PL") -> List[Dict[str, Any]]:
        data = self._get(f"/competitions/{competition_code}/teams")
        return data.get("teams", [])
scripts/utils/data_cleaner.py
Python
"""Fonctions de nettoyage des données brutes"""

import pandas as pd
from typing import List, Dict, Any


def clean_matches(raw_matches: List[Dict[str, Any]]) -> pd.DataFrame:
    """Transforme la réponse JSON API en DataFrame structuré."""
    records = []
    for m in raw_matches:
        score = m.get("score", {})
        full_time = score.get("fullTime", {})
        records.append({
            "match_id": m.get("id"),
            "season": str(m.get("season", {}).get("startDate", ""))[:4],
            "competition": m.get("competition", {}).get("name"),
            "competition_code": m.get("competition", {}).get("code"),
            "matchday": m.get("matchday"),
            "home_team": m.get("homeTeam", {}).get("name"),
            "home_team_id": m.get("homeTeam", {}).get("id"),
            "away_team": m.get("awayTeam", {}).get("name"),
            "away_team_id": m.get("awayTeam", {}).get("id"),
            "home_score": full_time.get("home"),
            "away_score": full_time.get("away"),
            "winner": score.get("winner"),
            "status": m.get("status"),
            "utc_date": m.get("utcDate"),
            "stage": m.get("stage"),
        })
    df = pd.DataFrame(records)
    # Typage
    numeric_cols = ["match_id", "matchday", "home_team_id", "away_team_id", "home_score", "away_score"]
    for col in numeric_cols:
        df[col] = pd.to_numeric(df[col], errors="coerce")
    df["utc_date"] = pd.to_datetime(df["utc_date"], errors="coerce")
    return df


def clean_teams(raw_teams: List[Dict[str, Any]]) -> pd.DataFrame:
    records = []
    for t in raw_teams:
        records.append({
            "team_id": t.get("id"),
            "team_name": t.get("name"),
            "short_name": t.get("shortName"),
            "tla": t.get("tla"),
            "area": t.get("area", {}).get("name"),
            "founded": t.get("founded"),
            "venue": t.get("venue"),
            "website": t.get("website"),
        })
    df = pd.DataFrame(records)
    df["team_id"] = pd.to_numeric(df["team_id"], errors="coerce")
    df["founded"] = pd.to_numeric(df["founded"], errors="coerce")
    return df
scripts/utils/db_connector.py
Python
"""Connexion PostgreSQL (préparation Phase 2)"""

import os
from sqlalchemy import create_engine


def get_postgres_engine():
    """Retourne un engine SQLAlchemy vers PostgreSQL local."""
    host = os.getenv("POSTGRES_HOST", "localhost")
    port = os.getenv("POSTGRES_PORT", "5432")
    db = os.getenv("POSTGRES_DB", "football_lab")
    user = os.getenv("POSTGRES_USER", "football_user")
    pwd = os.getenv("POSTGRES_PASSWORD", "secure_password")
    
    url = f"postgresql+psycopg2://{user}:{pwd}@{host}:{port}/{db}"
    return create_engine(url)
1.3 Créer le script d'ingestion API
scripts/ingest_api.py
Python
#!/usr/bin/env python3
"""Script d'ingestion des données via l'API football-data.org"""

import os
import sys
from datetime import datetime

# Ajouter la racine au PYTHONPATH
sys.path.insert(0, os.path.dirname(os.path.dirname(os.path.abspath(__file__))))

import pandas as pd
from dotenv import load_dotenv

from scripts.utils.api_client import FootballDataClient
from scripts.utils.data_cleaner import clean_matches, clean_teams


load_dotenv()


def save_raw(df: pd.DataFrame, name: str, subfolder: str = "api"):
    """Sauvegarde en Parquet dans data/raw/{subfolder}/"""
    out_dir = os.path.join("data", "raw", subfolder)
    os.makedirs(out_dir, exist_ok=True)
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    filepath = os.path.join(out_dir, f"{name}_{timestamp}.parquet")
    df.to_parquet(filepath, index=False)
    print(f"✅ Sauvegardé : {filepath} ({len(df)} lignes, {len(df.columns)} colonnes)")
    return filepath


def main():
    client = FootballDataClient()
    
    # --- Ingestion des compétitions disponibles ---
    print("📡 Récupération des compétitions...")
    competitions = client.get_competitions()
    print(f"   {len(competitions)} compétitions trouvées")
    
    # --- Ingestion des matchs Premier League 2024 ---
    print("📡 Récupération des matchs PL 2024...")
    raw_matches = client.get_matches(competition_code="PL", season=2024)
    df_matches = clean_matches(raw_matches)
    save_raw(df_matches, "matches_pl")
    
    # --- Ingestion des équipes PL ---
    print("📡 Récupération des équipes PL...")
    raw_teams = client.get_teams(competition_code="PL")
    df_teams = clean_teams(raw_teams)
    save_raw(df_teams, "teams_pl")
    
    # Optionnel : autres compétitions
    # for code in ["BL1", "SA", "PD", "FL1"]:
    #     raw = client.get_matches(competition_code=code, season=2024)
    #     save_raw(clean_matches(raw), f"matches_{code.lower()}")
    
    print("\n🎉 Ingestion terminée. Fichiers dans data/raw/api/")


if __name__ == "__main__":
    main()
1.4 Créer le script d'ingestion Kaggle
scripts/ingest_kaggle.py
Python
#!/usr/bin/env python3
"""Téléchargement et préparation des datasets Kaggle"""

import os
import zipfile
import pandas as pd
from kaggle.api.kaggle_api_extended import KaggleApi


DATASETS = {
    "results": "martj42/international-football-results-from-1872-to-2017",
    "shootouts": "martj42/international-football-results-from-1872-to-2017",  # même dataset
}


def download_dataset(kaggle_name: str, path: str = "data/raw/kaggle"):
    api = KaggleApi()
    api.authenticate()
    os.makedirs(path, exist_ok=True)
    print(f"⬇️  Téléchargement de {kaggle_name}...")
    api.dataset_download_files(kaggle_name, path=path, unzip=True)
    print(f"✅ Dataset extrait dans {path}")


def preview_csv(filepath: str) -> pd.DataFrame:
    df = pd.read_csv(filepath)
    print(f"📊 {os.path.basename(filepath)}: {len(df)} lignes × {len(df.columns)} colonnes")
    print(df.head(3))
    return df


def main():
    # Téléchargement
    download_dataset(DATASETS["results"])
    
    # Vérification des fichiers extraits
    kaggle_dir = "data/raw/kaggle"
    files = os.listdir(kaggle_dir)
    print(f"\n📁 Fichiers dans {kaggle_dir}: {files}")
    
    # Preview
    for f in files:
        if f.endswith(".csv"):
            preview_csv(os.path.join(kaggle_dir, f))


if __name__ == "__main__":
    main()
1.5 Exécuter la Phase 1
bash
# 1. S'assurer que l'environnement est activé
source venv/bin/activate   # Mac/Linux
# venv\Scripts\activate  # Windows

# 2. Lancer l'ingestion API
python scripts/ingest_api.py

# 3. Lancer l'ingestion Kaggle
python scripts/ingest_kaggle.py

# 4. Vérifier que les fichiers existent
ls -lah data/raw/api/
ls -lah data/raw/kaggle/
Résultat attendu :
data/raw/api/matches_pl_20260125_143022.parquet
data/raw/api/teams_pl_20260125_143025.parquet
data/raw/kaggle/results.csv
data/raw/kaggle/shootouts.csv (si présent dans le dataset)

🐘 PHASE 2 — PostgreSQL Staging
2.1 Créer les dossiers et fichiers SQL
bash
mkdir -p sql/postgres/init
mkdir -p sql/postgres/queries
sql/postgres/init/01_create_database.sql
sql
-- Se connecter d'abord à postgres (base système) pour créer la DB
-- psql -U postgres
CREATE DATABASE football_lab;

-- Se connecter ensuite à football_lab
\c football_lab;

-- Schéma staging
CREATE SCHEMA IF NOT EXISTS staging;

-- Utilisateur dédié
CREATE USER football_user WITH PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON SCHEMA staging TO football_user;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA staging TO football_user;
ALTER DEFAULT PRIVILEGES IN SCHEMA staging GRANT ALL ON TABLES TO football_user;
sql/postgres/init/02_create_staging_tables.sql
sql
\c football_lab;

CREATE TABLE IF NOT EXISTS staging.matches (
    match_id BIGINT PRIMARY KEY,
    season VARCHAR(10),
    competition VARCHAR(100),
    competition_code VARCHAR(10),
    matchday INT,
    home_team VARCHAR(100),
    home_team_id BIGINT,
    away_team VARCHAR(100),
    away_team_id BIGINT,
    home_score INT,
    away_score INT,
    winner VARCHAR(20),
    status VARCHAR(20),
    utc_date TIMESTAMP,
    stage VARCHAR(50),
    loaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS staging.teams (
    team_id BIGINT PRIMARY KEY,
    team_name VARCHAR(100) NOT NULL,
    short_name VARCHAR(50),
    tla VARCHAR(10),
    area VARCHAR(100),
    founded INT,
    venue VARCHAR(100),
    website VARCHAR(200),
    loaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS staging.players (
    player_id BIGINT PRIMARY KEY,
    player_name VARCHAR(100) NOT NULL,
    position VARCHAR(50),
    date_of_birth DATE,
    nationality VARCHAR(100),
    current_team VARCHAR(100),
    loaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
sql/postgres/init/03_create_indexes.sql
sql
\c football_lab;

CREATE INDEX IF NOT EXISTS idx_matches_season ON staging.matches(season);
CREATE INDEX IF NOT EXISTS idx_matches_competition ON staging.matches(competition);
CREATE INDEX IF NOT EXISTS idx_matches_matchday ON staging.matches(matchday);
CREATE INDEX IF NOT EXISTS idx_matches_home_team ON staging.matches(home_team);
CREATE INDEX IF NOT EXISTS idx_matches_away_team ON staging.matches(away_team);
CREATE INDEX IF NOT EXISTS idx_matches_date ON staging.matches(utc_date);
CREATE INDEX IF NOT EXISTS idx_teams_name ON staging.teams(team_name);
CREATE INDEX IF NOT EXISTS idx_teams_area ON staging.teams(area);
sql/postgres/queries/validation.sql
sql
\c football_lab;

-- Comptage global
SELECT 'matches' AS table_name, COUNT(*) AS row_count FROM staging.matches
UNION ALL
SELECT 'teams', COUNT(*) FROM staging.teams
UNION ALL
SELECT 'players', COUNT(*) FROM staging.players;

-- Compétitions distinctes
SELECT DISTINCT competition, competition_code 
FROM staging.matches 
ORDER BY competition;

-- Buts moyens par compétition
SELECT 
    competition,
    COUNT(*) AS total_matches,
    ROUND(AVG(COALESCE(home_score, 0) + COALESCE(away_score, 0)), 2) AS avg_goals
FROM staging.matches
GROUP BY competition
ORDER BY avg_goals DESC;

-- Doublons
SELECT match_id, COUNT(*) 
FROM staging.matches 
GROUP BY match_id 
HAVING COUNT(*) > 1;

-- Forme récente d'une équipe (ex: Arsenal)
SELECT 
    utc_date,
    home_team, away_team,
    home_score, away_score,
    CASE 
        WHEN home_score > away_score THEN 'W'
        WHEN home_score < away_score THEN 'L'
        ELSE 'D'
    END AS result
FROM staging.matches
WHERE home_team = 'Arsenal FC' OR away_team = 'Arsenal FC'
ORDER BY utc_date DESC
LIMIT 10;
2.2 Créer le script de chargement PostgreSQL
scripts/load_to_postgres.py
Python
#!/usr/bin/env python3
"""Charge les fichiers Parquet/CSV bruts vers PostgreSQL staging."""

import os
import sys
import glob
import pandas as pd

sys.path.insert(0, os.path.dirname(os.path.dirname(os.path.abspath(__file__))))

from scripts.utils.db_connector import get_postgres_engine


def load_parquet_to_postgres(pattern: str, table_name: str, schema: str = "staging"):
    """Charge le dernier fichier Parquet correspondant au pattern."""
    files = glob.glob(pattern)
    if not files:
        print(f"❌ Aucun fichier trouvé pour {pattern}")
        return
    
    # Prend le plus récent
    latest = max(files, key=os.path.getctime)
    print(f"📂 Chargement de {latest} vers {schema}.{table_name}...")
    
    df = pd.read_parquet(latest)
    engine = get_postgres_engine()
    
    # Truncate & load pour rechargement idempotent
    df.to_sql(table_name, engine, schema=schema, if_exists="replace", index=False)
    print(f"✅ {len(df)} lignes chargées dans {schema}.{table_name}")


def load_csv_to_postgres(filepath: str, table_name: str, schema: str = "staging"):
    if not os.path.exists(filepath):
        print(f"❌ Fichier introuvable : {filepath}")
        return
    
    print(f"📂 Chargement de {filepath} vers {schema}.{table_name}...")
    df = pd.read_csv(filepath)
    engine = get_postgres_engine()
    df.to_sql(table_name, engine, schema=schema, if_exists="replace", index=False)
    print(f"✅ {len(df)} lignes chargées dans {schema}.{table_name}")


def main():
    # Charger les matchs API (dernier fichier parquet)
    load_parquet_to_postgres("data/raw/api/matches_pl_*.parquet", "matches")
    
    # Charger les équipes API
    load_parquet_to_postgres("data/raw/api/teams_pl_*.parquet", "teams")
    
    # Charger les résultats historiques Kaggle
    load_csv_to_postgres("data/raw/kaggle/results.csv", "results_historical")
    
    print("\n🎉 Chargement PostgreSQL terminé.")


if __name__ == "__main__":
    main()
2.3 Exécuter la Phase 2
bash
# 1. Créer la base (en tant qu'utilisateur postgres)
psql -U postgres -f sql/postgres/init/01_create_database.sql

# 2. Créer les tables
psql -U postgres -f sql/postgres/init/02_create_staging_tables.sql

# 3. Créer les index
psql -U postgres -f sql/postgres/init/03_create_indexes.sql

# 4. Charger les données
python scripts/load_to_postgres.py

# 5. Valider
psql -U football_user -d football_lab -f sql/postgres/queries/validation.sql


🔥 PHASE 3 — Databricks Medallion
3.1 Créer les dossiers
bash
mkdir -p notebooks/databricks
mkdir -p notebooks/exploratory
mkdir -p sql/databricks
3.2 Notebook Bronze
notebooks/databricks/01_bronze_ingestion.py
Python
# Databricks notebook source
# MAGIC %md
# MAGIC # 🥉 Bronze Layer — Ingestion Raw vers Delta Lake
# MAGIC Objectif : Charger les fichiers Parquet bruts depuis DBFS vers des tables Delta sans transformation.

# COMMAND ----------

# CONFIGURATION
bronze_db = "bronze"
raw_path = "/FileStore/football/raw/api/"

# Créer la base
spark.sql(f"CREATE DATABASE IF NOT EXISTS {bronze_db}")

# COMMAND ----------

# MAGIC %md
# MAGIC ## 1. Ingestion des matchs

# COMMAND ----------

# Lire tous les Parquet de matchs (on prend le plus récent ou on fait un union)
from pyspark.sql.functions import input_file_name, current_timestamp

df_matches = (spark.read.parquet(f"{raw_path}matches_pl_*.parquet")
              .withColumn("source_file", input_file_name())
              .withColumn("ingested_at", current_timestamp()))

(df_matches.write
   .format("delta")
   .mode("overwrite")
   .option("overwriteSchema", "true")
   .saveAsTable(f"{bronze_db}.matches"))

print(f"✅ bronze.matches : {df_matches.count()} lignes")

# COMMAND ----------

# MAGIC %md
# MAGIC ## 2. Ingestion des équipes

# COMMAND ----------

df_teams = (spark.read.parquet(f"{raw_path}teams_pl_*.parquet")
            .withColumn("source_file", input_file_name())
            .withColumn("ingested_at", current_timestamp()))

(df_teams.write
   .format("delta")
   .mode("overwrite")
   .option("overwriteSchema", "true")
   .saveAsTable(f"{bronze_db}.teams"))

print(f"✅ bronze.teams : {df_teams.count()} lignes")

# COMMAND ----------

# MAGIC %md
# MAGIC ## 3. Validation

# COMMAND ----------

display(spark.sql(f"DESCRIBE HISTORY {bronze_db}.matches"))
display(spark.sql(f"SELECT * FROM {bronze_db}.matches LIMIT 10"))
3.3 Notebook Silver
notebooks/databricks/02_silver_cleaning.py
Python
# Databricks notebook source
# MAGIC %md
# MAGIC # 🥈 Silver Layer — Nettoyage, Dédoublonnage, Typage

# COMMAND ----------

silver_db = "silver"
spark.sql(f"CREATE DATABASE IF NOT EXISTS {silver_db}")

# COMMAND ----------

# MAGIC %md
# MAGIC ## 1. Nettoyage des matchs

# COMMAND ----------

from pyspark.sql.functions import col, trim, upper, when, coalesce, regexp_replace

df_bronze_matches = spark.table("bronze.matches")

df_silver_matches = (df_bronze_matches
    .dropDuplicates(["match_id"])
    .filter(col("status") == "FINISHED")
    .filter(col("home_score").isNotNull())
    .filter(col("away_score").isNotNull())
    .withColumn("home_team", trim(col("home_team")))
    .withColumn("away_team", trim(col("away_team")))
    .withColumn("competition", upper(trim(col("competition"))))
    .withColumn("season", col("season").cast("int"))
    .withColumn("home_score", col("home_score").cast("int"))
    .withColumn("away_score", col("away_score").cast("int"))
    .withColumn("total_goals", col("home_score") + col("away_score"))
    .withColumn("result", 
        when(col("home_score") > col("away_score"), "HOME_WIN")
        .when(col("home_score") < col("away_score"), "AWAY_WIN")
        .otherwise("DRAW"))
    .withColumn("match_date", col("utc_date").cast("date"))
    .select(
        "match_id", "season", "competition", "competition_code",
        "matchday", "stage", "match_date",
        "home_team_id", "home_team", "away_team_id", "away_team",
        "home_score", "away_score", "total_goals", "result", "winner",
        "ingested_at"
    ))

(df_silver_matches.write
   .format("delta")
   .mode("overwrite")
   .option("overwriteSchema", "true")
   .saveAsTable(f"{silver_db}.matches_clean"))

print(f"✅ silver.matches_clean : {df_silver_matches.count()} lignes")

# COMMAND ----------

# MAGIC %md
# MAGIC ## 2. Nettoyage des équipes

# COMMAND ----------

df_bronze_teams = spark.table("bronze.teams")

df_silver_teams = (df_bronze_teams
    .dropDuplicates(["team_id"])
    .withColumn("team_name", trim(col("team_name")))
    .withColumn("area", trim(col("area")))
    .withColumn("founded", col("founded").cast("int"))
    .select("team_id", "team_name", "short_name", "tla", "area", "founded", "venue"))

(df_silver_teams.write
   .format("delta")
   .mode("overwrite")
   .option("overwriteSchema", "true")
   .saveAsTable(f"{silver_db}.teams_clean"))

# COMMAND ----------

# MAGIC %md
# MAGIC ## 3. Qualité des données

# COMMAND ----------

display(spark.sql(f"""
    SELECT 
        COUNT(*) AS total_rows,
        COUNT(DISTINCT match_id) AS distinct_matches,
        SUM(CASE WHEN home_score IS NULL THEN 1 ELSE 0 END) AS null_scores,
        SUM(CASE WHEN result = 'DRAW' THEN 1 ELSE 0 END) AS draws
    FROM {silver_db}.matches_clean
"""))
3.4 Notebook Gold
notebooks/databricks/03_gold_analytics.py
Python
# Databricks notebook source
# MAGIC %md
# MAGIC # 🥇 Gold Layer — Tables Analytiques

# COMMAND ----------

gold_db = "gold"
spark.sql(f"CREATE DATABASE IF NOT EXISTS {gold_db}")

# COMMAND ----------

# MAGIC %md
# MAGIC ## 1. Stats par ligue et saison

# COMMAND ----------

spark.sql(f"""
    CREATE OR REPLACE TABLE {gold_db}.match_stats_by_league AS
    SELECT 
        competition,
        season,
        COUNT(*) AS total_matches,
        ROUND(AVG(total_goals), 2) AS avg_goals_per_match,
        ROUND(AVG(home_score), 2) AS avg_home_goals,
        ROUND(AVG(away_score), 2) AS avg_away_goals,
        SUM(CASE WHEN result = 'HOME_WIN' THEN 1 ELSE 0 END) AS home_wins,
        SUM(CASE WHEN result = 'AWAY_WIN' THEN 1 ELSE 0 END) AS away_wins,
        SUM(CASE WHEN result = 'DRAW' THEN 1 ELSE 0 END) AS draws,
        ROUND(100.0 * SUM(CASE WHEN result = 'HOME_WIN' THEN 1 ELSE 0 END) / COUNT(*), 1) AS home_win_pct
    FROM silver.matches_clean
    GROUP BY competition, season
    ORDER BY competition, season
""")

# COMMAND ----------

# MAGIC %md
# MAGIC ## 2. Performance équipe par saison

# COMMAND ----------

spark.sql(f"""
    CREATE OR REPLACE TABLE {gold_db}.team_season_performance AS
    WITH home_stats AS (
        SELECT 
            home_team AS team, season, competition,
            COUNT(*) AS played,
            SUM(CASE WHEN result = 'HOME_WIN' THEN 1 ELSE 0 END) AS wins,
            SUM(CASE WHEN result = 'DRAW' THEN 1 ELSE 0 END) AS draws,
            SUM(CASE WHEN result = 'AWAY_WIN' THEN 1 ELSE 0 END) AS losses,
            SUM(home_score) AS goals_for,
            SUM(away_score) AS goals_against
        FROM silver.matches_clean
        GROUP BY home_team, season, competition
    ),
    away_stats AS (
        SELECT 
            away_team AS team, season, competition,
            COUNT(*) AS played,
            SUM(CASE WHEN result = 'AWAY_WIN' THEN 1 ELSE 0 END) AS wins,
            SUM(CASE WHEN result = 'DRAW' THEN 1 ELSE 0 END) AS draws,
            SUM(CASE WHEN result = 'HOME_WIN' THEN 1 ELSE 0 END) AS losses,
            SUM(away_score) AS goals_for,
            SUM(home_score) AS goals_against
        FROM silver.matches_clean
        GROUP BY away_team, season, competition
    )
    SELECT 
        COALESCE(h.team, a.team) AS team,
        COALESCE(h.season, a.season) AS season,
        COALESCE(h.competition, a.competition) AS competition,
        COALESCE(h.played, 0) + COALESCE(a.played, 0) AS total_played,
        COALESCE(h.wins, 0) + COALESCE(a.wins, 0) AS total_wins,
        COALESCE(h.draws, 0) + COALESCE(a.draws, 0) AS total_draws,
        COALESCE(h.losses, 0) + COALESCE(a.losses, 0) AS total_losses,
        COALESCE(h.goals_for, 0) + COALESCE(a.goals_for, 0) AS goals_for,
        COALESCE(h.goals_against, 0) + COALESCE(a.goals_against, 0) AS goals_against,
        (COALESCE(h.wins, 0) + COALESCE(a.wins, 0)) * 3 
            + (COALESCE(h.draws, 0) + COALESCE(a.draws, 0)) AS points
    FROM home_stats h
    FULL OUTER JOIN away_stats a 
        ON h.team = a.team AND h.season = a.season AND h.competition = a.competition
""")

# COMMAND ----------

# MAGIC %md
# MAGIC ## 3. Validation

# COMMAND ----------

display(spark.sql(f"SELECT * FROM {gold_db}.match_stats_by_league"))
display(spark.sql(f"SELECT * FROM {gold_db}.team_season_performance ORDER BY points DESC LIMIT 20"))
3.5 Validation SQL Databricks
sql/databricks/validation.sql
sql
-- Vérifier l'historique Delta
DESCRIBE HISTORY bronze.matches;

-- Comparer bronze vs silver
SELECT 'bronze' AS layer, COUNT(*) FROM bronze.matches
UNION ALL
SELECT 'silver', COUNT(*) FROM silver.matches_clean;

-- Top 10 des matchs avec le plus de buts
SELECT match_date, home_team, away_team, total_goals
FROM silver.matches_clean
ORDER BY total_goals DESC
LIMIT 10;

-- Classement des équipes par points (saison 2024)
SELECT team, total_played, total_wins, goals_for, goals_against, points
FROM gold.team_season_performance
WHERE season = 2024
ORDER BY points DESC
LIMIT 10;
3.6 Exécuter la Phase 3
Uploader les fichiers Parquet de data/raw/api/ dans Databricks DBFS (/FileStore/football/raw/api/)
Créer 3 notebooks dans Databricks, coller le contenu ci-dessus
Exécuter dans l'ordre : 01_ → 02_ → 03_
Valider avec les requêtes SQL dans un notebook ou sql/databricks/validation.sql

❄️ PHASE 4 — Snowflake Data Warehouse
4.1 Créer les fichiers SQL
bash
mkdir -p sql/snowflake
sql/snowflake/01_setup.sql
sql
-- Créer le warehouse (compute)
CREATE WAREHOUSE IF NOT EXISTS FOOTBALL_WH
    WITH WAREHOUSE_SIZE = 'XSMALL'
    AUTO_SUSPEND = 300
    AUTO_RESUME = TRUE;

USE WAREHOUSE FOOTBALL_WH;

-- Database et schémas
CREATE DATABASE IF NOT EXISTS FOOTBALL_LAB;
USE DATABASE FOOTBALL_LAB;

CREATE SCHEMA IF NOT EXISTS RAW;
CREATE SCHEMA IF NOT EXISTS CLEANED;
CREATE SCHEMA IF NOT EXISTS ANALYTICS;

-- Rôle (optionnel)
CREATE ROLE IF NOT EXISTS FOOTBALL_ROLE;
GRANT USAGE ON WAREHOUSE FOOTBALL_WH TO ROLE FOOTBALL_ROLE;
GRANT ALL ON DATABASE FOOTBALL_LAB TO ROLE FOOTBALL_ROLE;
sql/snowflake/02_load_raw.sql
sql
USE DATABASE FOOTBALL_LAB;
USE SCHEMA RAW;

-- File format Parquet
CREATE OR REPLACE FILE FORMAT parquet_format
    TYPE = 'PARQUET'
    COMPRESSION = 'SNAPPY';

-- Stage interne
CREATE OR REPLACE STAGE raw_stage
    FILE_FORMAT = parquet_format;

-- PUT file://data/raw/api/matches_pl_*.parquet @raw_stage;  -- via SnowSQL ou UI

-- Table raw matchs
CREATE OR REPLACE TABLE RAW.MATCHES (
    match_id BIGINT,
    season VARCHAR(10),
    competition VARCHAR(100),
    competition_code VARCHAR(10),
    matchday INT,
    home_team VARCHAR(100),
    home_team_id BIGINT,
    away_team VARCHAR(100),
    away_team_id BIGINT,
    home_score INT,
    away_score INT,
    winner VARCHAR(20),
    status VARCHAR(20),
    utc_date TIMESTAMP_NTZ,
    stage VARCHAR(50),
    ingested_at TIMESTAMP_NTZ
);

-- Chargement (adapter le nom de fichier)
COPY INTO RAW.MATCHES
    FROM @raw_stage
    FILE_FORMAT = (FORMAT_NAME = 'parquet_format')
    PATTERN = '.*matches_pl_.*\\.parquet'
    MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE;

-- Vérification
SELECT COUNT(*) FROM RAW.MATCHES;
SELECT DISTINCT competition FROM RAW.MATCHES;
sql/snowflake/03_cleaned_views.sql
sql
USE DATABASE FOOTBALL_LAB;
USE SCHEMA CLEANED;

CREATE OR REPLACE VIEW CLEANED.MATCHES AS
SELECT
    MATCH_ID,
    CAST(SEASON AS INT) AS SEASON,
    TRIM(UPPER(COMPETITION)) AS COMPETITION,
    COMPETITION_CODE,
    MATCHDAY,
    TRIM(HOME_TEAM) AS HOME_TEAM,
    HOME_TEAM_ID,
    TRIM(AWAY_TEAM) AS AWAY_TEAM,
    AWAY_TEAM_ID,
    HOME_SCORE,
    AWAY_SCORE,
    CASE 
        WHEN HOME_SCORE > AWAY_SCORE THEN 'HOME_WIN'
        WHEN HOME_SCORE < AWAY_SCORE THEN 'AWAY_WIN'
        ELSE 'DRAW'
    END AS RESULT,
    HOME_SCORE + AWAY_SCORE AS TOTAL_GOALS,
    STATUS,
    CAST(UTC_DATE AS DATE) AS MATCH_DATE,
    STAGE
FROM RAW.MATCHES
WHERE STATUS = 'FINISHED'
    AND HOME_SCORE IS NOT NULL
    AND AWAY_SCORE IS NOT NULL;
sql/snowflake/04_analytics_tables.sql
sql
USE DATABASE FOOTBALL_LAB;
USE SCHEMA ANALYTICS;

-- Table de faits : matchs
CREATE OR REPLACE TABLE ANALYTICS.FACT_MATCHES AS
SELECT
    MATCH_ID,
    SEASON,
    COMPETITION,
    COMPETITION_CODE,
    MATCHDAY,
    MATCH_DATE,
    HOME_TEAM_ID,
    HOME_TEAM,
    AWAY_TEAM_ID,
    AWAY_TEAM,
    HOME_SCORE,
    AWAY_SCORE,
    TOTAL_GOALS,
    RESULT,
    CASE WHEN RESULT = 'HOME_WIN' THEN 1 ELSE 0 END AS IS_HOME_WIN,
    CASE WHEN RESULT = 'AWAY_WIN' THEN 1 ELSE 0 END AS IS_AWAY_WIN,
    CASE WHEN RESULT = 'DRAW' THEN 1 ELSE 0 END AS IS_DRAW
FROM CLEANED.MATCHES;

-- Dimension équipes (simplifiée depuis les données API)
CREATE OR REPLACE TABLE ANALYTICS.DIM_TEAMS AS
SELECT DISTINCT
    HOME_TEAM_ID AS TEAM_ID,
    HOME_TEAM AS TEAM_NAME
FROM CLEANED.MATCHES
WHERE HOME_TEAM_ID IS NOT NULL
UNION
SELECT DISTINCT
    AWAY_TEAM_ID AS TEAM_ID,
    AWAY_TEAM AS TEAM_NAME
FROM CLEANED.MATCHES
WHERE AWAY_TEAM_ID IS NOT NULL;

-- Dimension temps
CREATE OR REPLACE TABLE ANALYTICS.DIM_DATE AS
SELECT DISTINCT
    MATCH_DATE AS DATE_KEY,
    YEAR(MATCH_DATE) AS YEAR,
    MONTH(MATCH_DATE) AS MONTH,
    MONTHNAME(MATCH_DATE) AS MONTH_NAME,
    DAY(MATCH_DATE) AS DAY,
    DAYOFWEEK(MATCH_DATE) AS DAY_OF_WEEK,
    DAYOFYEAR(MATCH_DATE) AS DAY_OF_YEAR,
    WEEKOFYEAR(MATCH_DATE) AS WEEK_OF_YEAR,
    QUARTER(MATCH_DATE) AS QUARTER
FROM CLEANED.MATCHES
WHERE MATCH_DATE IS NOT NULL;
4.2 Exécuter la Phase 4
Se connecter à Snowflake (UI web ou SnowSQL)
Exécuter 01_setup.sql
Uploader les fichiers Parquet dans le stage @raw_stage
Exécuter 02_load_raw.sql
Exécuter 03_cleaned_views.sql
Exécuter 04_analytics_tables.sql

🔄 PHASE 5 — dbt (Transformation & Modélisation)
5.1 Créer la structure dbt
bash
mkdir -p dbt-football/models/staging
mkdir -p dbt-football/models/marts
mkdir -p dbt-football/seeds
mkdir -p dbt-football/snapshots
mkdir -p dbt-football/macros
mkdir -p dbt-football/tests/generic
mkdir -p dbt-football/analyses
5.2 Fichiers de configuration
dbt-football/dbt_project.yml
yaml
name: 'football_dbt'
version: '1.0.0'
config-version: 2

profile: 'football_dbt'

model-paths: ["models"]
analysis-paths: ["analyses"]
test-paths: ["tests"]
seed-paths: ["seeds"]
macro-paths: ["macros"]
snapshot-paths: ["snapshots"]

target-path: "target"
clean-targets:
  - "target"
  - "dbt_packages"

models:
  football_dbt:
    staging:
      +materialized: view
      +schema: staging
    marts:
      +materialized: table
      +schema: analytics

seeds:
  football_dbt:
    +schema: staging
dbt-football/packages.yml
yaml
packages:
  - package: dbt-labs/dbt_utils
    version: 1.2.0
config/profiles.yml (à la racine du projet, pas dans dbt-football/)
yaml
football_dbt:
  target: dev
  outputs:
    dev:
      type: snowflake
      account: your_account.eu-central-1
      user: your_username
      password: your_password
      role: FOOTBALL_ROLE
      database: FOOTBALL_LAB
      warehouse: FOOTBALL_WH
      schema: DBT_DEV
      threads: 4
      client_session_keep_alive: False
Crée un lien symbolique ou copie ce fichier vers dbt-football/profiles.yml :
bash
cp config/profiles.yml dbt-football/profiles.yml
5.3 Seeds
dbt-football/seeds/leagues.csv
csv
league_code,league_name,country,confederation
PL,Premier League,England,UEFA
BL1,Bundesliga,Germany,UEFA
SA,Serie A,Italy,UEFA
PD,La Liga,Spain,UEFA
FL1,Ligue 1,France,UEFA
CL,UEFA Champions League,Europe,UEFA
dbt-football/seeds/countries.csv
csv
country_code,country_name,confederation
ENG,England,UEFA
FRA,France,UEFA
GER,Germany,UEFA
ITA,Italy,UEFA
ESP,Spain,UEFA
BRA,Brazil,CONMEBOL
ARG,Argentina,CONMEBOL
5.4 Modèles Staging
dbt-football/models/staging/sources.yml
yaml
version: 2

sources:
  - name: raw_football
    database: FOOTBALL_LAB
    schema: RAW
    tables:
      - name: matches
        description: "Données brutes des matchs depuis l'API football-data.org"
      - name: teams
        description: "Données brutes des équipes"

  - name: cleaned_football
    database: FOOTBALL_LAB
    schema: CLEANED
    tables:
      - name: matches
        description: "Vue nettoyée des matchs"
dbt-football/models/staging/staging.yml
yaml
version: 2

models:
  - name: stg_matches
    description: "Matchs nettoyés et validés"
    columns:
      - name: match_id
        description: "Clé primaire du match"
        tests:
          - unique
          - not_null
      - name: season
        tests:
          - not_null
      - name: home_team
        tests:
          - not_null
      - name: away_team
        tests:
          - not_null
      - name: result
        tests:
          - accepted_values:
              values: ['HOME_WIN', 'AWAY_WIN', 'DRAW']
      - name: total_goals
        tests:
          - not_null

  - name: stg_teams
    columns:
      - name: team_id
        tests:
          - unique
          - not_null
      - name: team_name
        tests:
          - not_null
dbt-football/models/staging/stg_matches.sql
sql
WITH source AS (
    SELECT * FROM {{ source('cleaned_football', 'matches') }}
),

cleaned AS (
    SELECT
        MATCH_ID AS match_id,
        CAST(SEASON AS INT) AS season,
        COMPETITION AS competition,
        COMPETITION_CODE AS competition_code,
        MATCHDAY AS matchday,
        TRIM(HOME_TEAM) AS home_team,
        HOME_TEAM_ID AS home_team_id,
        TRIM(AWAY_TEAM) AS away_team,
        AWAY_TEAM_ID AS away_team_id,
        HOME_SCORE AS home_score,
        AWAY_SCORE AS away_score,
        RESULT AS result,
        TOTAL_GOALS AS total_goals,
        MATCH_DATE AS match_date,
        STAGE AS stage
    FROM source
    WHERE MATCH_DATE IS NOT NULL
)

SELECT * FROM cleaned
dbt-football/models/staging/stg_teams.sql
sql
WITH source AS (
    SELECT * FROM {{ source('raw_football', 'teams') }}
),

cleaned AS (
    SELECT
        TEAM_ID AS team_id,
        TRIM(TEAM_NAME) AS team_name,
        SHORT_NAME AS short_name,
        TLA AS tla,
        AREA AS area,
        FOUNDED AS founded,
        VENUE AS venue
    FROM source
    WHERE TEAM_ID IS NOT NULL
)

SELECT * FROM cleaned
dbt-football/models/staging/stg_competitions.sql
sql
SELECT
    LEAGUE_CODE AS competition_code,
    LEAGUE_NAME AS competition_name,
    COUNTRY AS country,
    CONFEDERATION AS confederation
FROM {{ ref('leagues') }}
5.5 Modèles Marts (Schéma en Étoile)
dbt-football/models/marts/schema.yml
yaml
version: 2

models:
  - name: fact_matches
    description: "Table de faits : 1 ligne = 1 match joué"
    tests:
      - dbt_utils.unique_combination_of_columns:
          combination_of_columns:
            - match_id
    columns:
      - name: match_id
        tests:
          - unique
          - not_null
          - relationships:
              to: ref('stg_matches')
              field: match_id
      - name: home_team_sk
        tests:
          - relationships:
              to: ref('dim_teams')
              field: team_sk
      - name: away_team_sk
        tests:
          - relationships:
              to: ref('dim_teams')
              field: team_sk

  - name: dim_teams
    columns:
      - name: team_sk
        tests:
          - unique
          - not_null

  - name: dim_date
    columns:
      - name: date_key
        tests:
          - unique
          - not_null
dbt-football/models/marts/dim_teams.sql
sql
WITH all_teams AS (
    SELECT DISTINCT home_team_id AS team_id, home_team AS team_name
    FROM {{ ref('stg_matches') }}
    WHERE home_team_id IS NOT NULL
    UNION
    SELECT DISTINCT away_team_id AS team_id, away_team AS team_name
    FROM {{ ref('stg_matches') }}
    WHERE away_team_id IS NOT NULL
),

ranked AS (
    SELECT 
        team_id,
        team_name,
        ROW_NUMBER() OVER (ORDER BY team_name) AS team_sk
    FROM all_teams
)

SELECT 
    team_sk,
    team_id,
    team_name,
    CURRENT_TIMESTAMP() AS loaded_at
FROM ranked
dbt-football/models/marts/dim_date.sql
sql
WITH date_spine AS (
    {{ dbt_utils.date_spine(
        datepart="day",
        start_date="cast('2020-01-01' as date)",
        end_date="cast('2026-12-31' as date)"
    ) }}
)

SELECT
    date_day AS date_key,
    EXTRACT(YEAR FROM date_day) AS year,
    EXTRACT(MONTH FROM date_day) AS month,
    EXTRACT(DAY FROM date_day) AS day,
    EXTRACT(DOW FROM date_day) AS day_of_week,
    EXTRACT(QUARTER FROM date_day) AS quarter
FROM date_spine
dbt-football/models/marts/dim_competitions.sql
sql
SELECT
    competition_code,
    competition_name,
    country,
    confederation
FROM {{ ref('stg_competitions') }}
dbt-football/models/marts/fact_matches.sql
sql
WITH matches AS (
    SELECT * FROM {{ ref('stg_matches') }}
),

teams_home AS (
    SELECT team_sk, team_id FROM {{ ref('dim_teams') }}
),

teams_away AS (
    SELECT team_sk, team_id FROM {{ ref('dim_teams') }}
)

SELECT
    m.match_id,
    m.season,
    m.competition,
    m.competition_code,
    m.matchday,
    m.match_date,
    h.team_sk AS home_team_sk,
    a.team_sk AS away_team_sk,
    m.home_score,
    m.away_score,
    m.result,
    m.total_goals,
    CASE WHEN m.result = 'HOME_WIN' THEN 1 ELSE 0 END AS is_home_win,
    CASE WHEN m.result = 'AWAY_WIN' THEN 1 ELSE 0 END AS is_away_win,
    CASE WHEN m.result = 'DRAW' THEN 1 ELSE 0 END AS is_draw,
    CURRENT_TIMESTAMP() AS loaded_at
FROM matches m
LEFT JOIN teams_home h ON m.home_team_id = h.team_id
LEFT JOIN teams_away a ON m.away_team_id = a.team_id
5.6 Snapshot SCD2
dbt-football/snapshots/scd2_player_stats.sql
sql
{% snapshot scd2_player_stats %}

{{
    config(
      target_database='FOOTBALL_LAB',
      target_schema='ANALYTICS',
      unique_key='player_id',
      strategy='check',
      check_cols=['goals', 'assists', 'minutes_played'],
    )
}}

SELECT 
    player_id,
    player_name,
    season,
    competition,
    goals,
    assists,
    minutes_played,
    CURRENT_TIMESTAMP() AS loaded_at
FROM {{ source('raw_football', 'players') }}
WHERE player_id IS NOT NULL

{% endsnapshot %}
5.7 Exécuter la Phase 5
bash
# 1. Aller dans le projet dbt
cd dbt-football

# 2. Installer les packages
dbt deps

# 3. Vérifier la connexion
dbt debug

# 4. Charger les seeds
dbt seed

# 5. Exécuter les modèles
dbt run

# 6. Exécuter les tests
dbt test

# 7. Générer la documentation
dbt docs generate
dbt docs serve  # http://localhost:8080

🔄 PHASE 5 — dbt (Transformation & Modélisation)
5.1 Créer la structure dbt
bash
mkdir -p dbt-football/models/staging
mkdir -p dbt-football/models/marts
mkdir -p dbt-football/seeds
mkdir -p dbt-football/snapshots
mkdir -p dbt-football/macros
mkdir -p dbt-football/tests/generic
mkdir -p dbt-football/analyses
5.2 Fichiers de configuration
dbt-football/dbt_project.yml
yaml
name: 'football_dbt'
version: '1.0.0'
config-version: 2

profile: 'football_dbt'

model-paths: ["models"]
analysis-paths: ["analyses"]
test-paths: ["tests"]
seed-paths: ["seeds"]
macro-paths: ["macros"]
snapshot-paths: ["snapshots"]

target-path: "target"
clean-targets:
  - "target"
  - "dbt_packages"

models:
  football_dbt:
    staging:
      +materialized: view
      +schema: staging
    marts:
      +materialized: table
      +schema: analytics

seeds:
  football_dbt:
    +schema: staging
dbt-football/packages.yml
yaml
packages:
  - package: dbt-labs/dbt_utils
    version: 1.2.0
config/profiles.yml (à la racine du projet, pas dans dbt-football/)
yaml
football_dbt:
  target: dev
  outputs:
    dev:
      type: snowflake
      account: your_account.eu-central-1
      user: your_username
      password: your_password
      role: FOOTBALL_ROLE
      database: FOOTBALL_LAB
      warehouse: FOOTBALL_WH
      schema: DBT_DEV
      threads: 4
      client_session_keep_alive: False
Crée un lien symbolique ou copie ce fichier vers dbt-football/profiles.yml :
bash
cp config/profiles.yml dbt-football/profiles.yml
5.3 Seeds
dbt-football/seeds/leagues.csv
csv
league_code,league_name,country,confederation
PL,Premier League,England,UEFA
BL1,Bundesliga,Germany,UEFA
SA,Serie A,Italy,UEFA
PD,La Liga,Spain,UEFA
FL1,Ligue 1,France,UEFA
CL,UEFA Champions League,Europe,UEFA
dbt-football/seeds/countries.csv
csv
country_code,country_name,confederation
ENG,England,UEFA
FRA,France,UEFA
GER,Germany,UEFA
ITA,Italy,UEFA
ESP,Spain,UEFA
BRA,Brazil,CONMEBOL
ARG,Argentina,CONMEBOL
5.4 Modèles Staging
dbt-football/models/staging/sources.yml
yaml
version: 2

sources:
  - name: raw_football
    database: FOOTBALL_LAB
    schema: RAW
    tables:
      - name: matches
        description: "Données brutes des matchs depuis l'API football-data.org"
      - name: teams
        description: "Données brutes des équipes"

  - name: cleaned_football
    database: FOOTBALL_LAB
    schema: CLEANED
    tables:
      - name: matches
        description: "Vue nettoyée des matchs"
dbt-football/models/staging/staging.yml
yaml
version: 2

models:
  - name: stg_matches
    description: "Matchs nettoyés et validés"
    columns:
      - name: match_id
        description: "Clé primaire du match"
        tests:
          - unique
          - not_null
      - name: season
        tests:
          - not_null
      - name: home_team
        tests:
          - not_null
      - name: away_team
        tests:
          - not_null
      - name: result
        tests:
          - accepted_values:
              values: ['HOME_WIN', 'AWAY_WIN', 'DRAW']
      - name: total_goals
        tests:
          - not_null

  - name: stg_teams
    columns:
      - name: team_id
        tests:
          - unique
          - not_null
      - name: team_name
        tests:
          - not_null
dbt-football/models/staging/stg_matches.sql
sql
WITH source AS (
    SELECT * FROM {{ source('cleaned_football', 'matches') }}
),

cleaned AS (
    SELECT
        MATCH_ID AS match_id,
        CAST(SEASON AS INT) AS season,
        COMPETITION AS competition,
        COMPETITION_CODE AS competition_code,
        MATCHDAY AS matchday,
        TRIM(HOME_TEAM) AS home_team,
        HOME_TEAM_ID AS home_team_id,
        TRIM(AWAY_TEAM) AS away_team,
        AWAY_TEAM_ID AS away_team_id,
        HOME_SCORE AS home_score,
        AWAY_SCORE AS away_score,
        RESULT AS result,
        TOTAL_GOALS AS total_goals,
        MATCH_DATE AS match_date,
        STAGE AS stage
    FROM source
    WHERE MATCH_DATE IS NOT NULL
)

SELECT * FROM cleaned
dbt-football/models/staging/stg_teams.sql
sql
WITH source AS (
    SELECT * FROM {{ source('raw_football', 'teams') }}
),

cleaned AS (
    SELECT
        TEAM_ID AS team_id,
        TRIM(TEAM_NAME) AS team_name,
        SHORT_NAME AS short_name,
        TLA AS tla,
        AREA AS area,
        FOUNDED AS founded,
        VENUE AS venue
    FROM source
    WHERE TEAM_ID IS NOT NULL
)

SELECT * FROM cleaned
dbt-football/models/staging/stg_competitions.sql
sql
SELECT
    LEAGUE_CODE AS competition_code,
    LEAGUE_NAME AS competition_name,
    COUNTRY AS country,
    CONFEDERATION AS confederation
FROM {{ ref('leagues') }}
5.5 Modèles Marts (Schéma en Étoile)
dbt-football/models/marts/schema.yml
yaml
version: 2

models:
  - name: fact_matches
    description: "Table de faits : 1 ligne = 1 match joué"
    tests:
      - dbt_utils.unique_combination_of_columns:
          combination_of_columns:
            - match_id
    columns:
      - name: match_id
        tests:
          - unique
          - not_null
          - relationships:
              to: ref('stg_matches')
              field: match_id
      - name: home_team_sk
        tests:
          - relationships:
              to: ref('dim_teams')
              field: team_sk
      - name: away_team_sk
        tests:
          - relationships:
              to: ref('dim_teams')
              field: team_sk

  - name: dim_teams
    columns:
      - name: team_sk
        tests:
          - unique
          - not_null

  - name: dim_date
    columns:
      - name: date_key
        tests:
          - unique
          - not_null
dbt-football/models/marts/dim_teams.sql
sql
WITH all_teams AS (
    SELECT DISTINCT home_team_id AS team_id, home_team AS team_name
    FROM {{ ref('stg_matches') }}
    WHERE home_team_id IS NOT NULL
    UNION
    SELECT DISTINCT away_team_id AS team_id, away_team AS team_name
    FROM {{ ref('stg_matches') }}
    WHERE away_team_id IS NOT NULL
),

ranked AS (
    SELECT 
        team_id,
        team_name,
        ROW_NUMBER() OVER (ORDER BY team_name) AS team_sk
    FROM all_teams
)

SELECT 
    team_sk,
    team_id,
    team_name,
    CURRENT_TIMESTAMP() AS loaded_at
FROM ranked
dbt-football/models/marts/dim_date.sql
sql
WITH date_spine AS (
    {{ dbt_utils.date_spine(
        datepart="day",
        start_date="cast('2020-01-01' as date)",
        end_date="cast('2026-12-31' as date)"
    ) }}
)

SELECT
    date_day AS date_key,
    EXTRACT(YEAR FROM date_day) AS year,
    EXTRACT(MONTH FROM date_day) AS month,
    EXTRACT(DAY FROM date_day) AS day,
    EXTRACT(DOW FROM date_day) AS day_of_week,
    EXTRACT(QUARTER FROM date_day) AS quarter
FROM date_spine
dbt-football/models/marts/dim_competitions.sql
sql
SELECT
    competition_code,
    competition_name,
    country,
    confederation
FROM {{ ref('stg_competitions') }}
dbt-football/models/marts/fact_matches.sql
sql
WITH matches AS (
    SELECT * FROM {{ ref('stg_matches') }}
),

teams_home AS (
    SELECT team_sk, team_id FROM {{ ref('dim_teams') }}
),

teams_away AS (
    SELECT team_sk, team_id FROM {{ ref('dim_teams') }}
)

SELECT
    m.match_id,
    m.season,
    m.competition,
    m.competition_code,
    m.matchday,
    m.match_date,
    h.team_sk AS home_team_sk,
    a.team_sk AS away_team_sk,
    m.home_score,
    m.away_score,
    m.result,
    m.total_goals,
    CASE WHEN m.result = 'HOME_WIN' THEN 1 ELSE 0 END AS is_home_win,
    CASE WHEN m.result = 'AWAY_WIN' THEN 1 ELSE 0 END AS is_away_win,
    CASE WHEN m.result = 'DRAW' THEN 1 ELSE 0 END AS is_draw,
    CURRENT_TIMESTAMP() AS loaded_at
FROM matches m
LEFT JOIN teams_home h ON m.home_team_id = h.team_id
LEFT JOIN teams_away a ON m.away_team_id = a.team_id
5.6 Snapshot SCD2
dbt-football/snapshots/scd2_player_stats.sql
sql
{% snapshot scd2_player_stats %}

{{
    config(
      target_database='FOOTBALL_LAB',
      target_schema='ANALYTICS',
      unique_key='player_id',
      strategy='check',
      check_cols=['goals', 'assists', 'minutes_played'],
    )
}}

SELECT 
    player_id,
    player_name,
    season,
    competition,
    goals,
    assists,
    minutes_played,
    CURRENT_TIMESTAMP() AS loaded_at
FROM {{ source('raw_football', 'players') }}
WHERE player_id IS NOT NULL

{% endsnapshot %}
5.7 Exécuter la Phase 5
bash
# 1. Aller dans le projet dbt
cd dbt-football

# 2. Installer les packages
dbt deps

# 3. Vérifier la connexion
dbt debug

# 4. Charger les seeds
dbt seed

# 5. Exécuter les modèles
dbt run

# 6. Exécuter les tests
dbt test

# 7. Générer la documentation
dbt docs generate
dbt docs serve  # http://localhost:8080




