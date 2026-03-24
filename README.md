# Spark + Kafka + MinIO DataLake

Environnement de développement Big Data complet pour l'apprentissage et le prototypage de pipelines de données modernes.

## Description

Cette stack Docker fournit une infrastructure **Data Lake** prête à l'emploi combinant :

- **Apache Spark 4.0** via JupyterLab pour le traitement distribué
- **Apache Kafka** pour le streaming temps réel
- **MinIO** comme stockage objet compatible S3
- **PostgreSQL** avec la base Northwind pour les exercices SQL

Idéal pour apprendre le Big Data, tester des pipelines ETL/ELT ou prototyper des architectures data.

---

## Démarrage rapide

```bash
# Démarrer tous les services
docker compose up -d

# Vérifier l'état
docker compose ps

# Suivre les logs
docker compose logs -f
```

## Arrêt

```bash
# Arrêter tous les services (conserve les données)
docker compose down

# Arrêter et supprimer les volumes (ATTENTION: perte de données)
docker compose down -v
```

---

## Architecture

```ascii
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         SPARK + KAFKA + MINIO DATALAKE                          │
└─────────────────────────────────────────────────────────────────────────────────┘

    ┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
    │   POSTGRESQL    │      │  KAFKA BROKER   │      │     MINIO       │
    │    :5433        │      │    :9092        │      │  :9000 / :9001  │
    │  ┌───────────┐  │      │  (Confluent)    │      │   (S3 API)      │
    │  │ Northwind │  │      └────────┬────────┘      └────────┬────────┘
    │  │    DB     │  │               │                        │
    │  └───────────┘  │               │                        │
    └────────┬────────┘               │                        │
             │                        │                        │
    ┌────────▼────────┐      ┌────────▼────────┐               │
    │    ADMINER      │      │    KAFKA-UI     │               │
    │     :9080       │      │     :7080       │               │
    │  (SQL Client)   │      │  (Monitoring)   │               │
    └─────────────────┘      └─────────────────┘               │
                                                               │
    ┌──────────────────────────────────────────────────────────▼──────────────────┐
    │                           JUPYTER-SPARK                                     │
    │                              :8888                                          │
    │  ┌──────────────────────────────────────────────────────────────────────┐   │
    │  │  • Apache Spark 4.0.1        • PySpark                               │   │
    │  │  • Hadoop 3.4                • Delta Lake                            │   │
    │  │  • Python / Scala            • Kafka Integration                     │   │
    │  └──────────────────────────────────────────────────────────────────────┘   │
    └─────────────────────────────────────────────────────────────────────────────┘
```

---

## URLs & Services

| Service | URL | Credentials |
|---------|-----|-------------|
| **JupyterLab** | <http://localhost:8888> | Token: aucun (accès direct) |
| **Kafka UI** | <http://localhost:7080> | - |
| **Adminer** (PostgreSQL) | <http://localhost:9080> | `postgres` / `postgres` |
| **MinIO Console** | <http://localhost:9001> | `minioadmin` / `minioadmin123` |
| **MinIO API (S3)** | <http://localhost:9000> | - |
| **Kafka Broker** | localhost:9092 | - |
| **PostgreSQL** | localhost:5433 | `postgres` / `postgres` / DB: `app` |

---

## Structure du projet

```ascii
├── docker-compose.yml          # Configuration des services
├── notebooks/                  # Notebooks Jupyter
│   ├── exemples/              # Exemples de démarrage
│   ├── exercices/             # Exercices pratiques
│   └── elk/                   # Intégration ELK
├── vol/
│   ├── jupyter/               # Config Jupyter
│   └── postgresql/            # Scripts SQL (Northwind)
└── .docker/
    └── jupyter-spark/         # Dockerfile Spark custom
```

---

## Notebooks disponibles

| Notebook | Description |
|----------|-------------|
| `1_test_demarrage.ipynb` | Vérification de l'environnement Spark |
| `2_simple_python_producer.ipynb` | Producer Kafka en Python |
| `3_pyspark_consumer.ipynb` | Consumer Kafka avec PySpark |
| `4_pyspark_stream_consumer.ipynb` | Streaming Spark + Kafka |
| `5_gen_data.ipynb` | Génération de données de test |

---

## Configuration

### Variables d'environnement (optionnel)

Créer un fichier `.env` pour personnaliser :

```env
MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=minioadmin123
MINIO_VERSION=latest
```

### Connexion Spark → MinIO

```python
spark = SparkSession.builder \
    .appName("MinIO") \
    .config("spark.hadoop.fs.s3a.endpoint", "http://minio:9000") \
    .config("spark.hadoop.fs.s3a.access.key", "minioadmin") \
    .config("spark.hadoop.fs.s3a.secret.key", "minioadmin123") \
    .config("spark.hadoop.fs.s3a.path.style.access", "true") \
    .getOrCreate()
```

### Connexion Spark → Kafka

```python
df = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "broker:29092") \
    .option("subscribe", "mon-topic") \
    .load()
```

### Connexion Spark → PostgreSQL

```python
df = spark.read \
    .format("jdbc") \
    .option("url", "jdbc:postgresql://postgres:5432/app") \
    .option("user", "postgres") \
    .option("password", "postgres") \
    .option("dbtable", "customers") \
    .load()
```

---

## Dépannage

### Problème : Container qui ne démarre pas

```bash
# Voir les logs d'un service
docker compose logs postgres

# Reconstruire les images
docker compose build --no-cache jupyter-spark
```

### Problème : Port déjà utilisé

```bash
# Trouver le processus utilisant le port
netstat -ano | findstr :8888
```

### Réinitialisation complète

```bash
docker compose down -v
docker system prune -f
docker compose up -d
```

---

## Documentation

- [Apache Spark](https://spark.apache.org/docs/latest/)
- [Apache Kafka](https://kafka.apache.org/documentation/)
- [MinIO](https://min.io/docs/minio/container/index.html)
- [PySpark](https://spark.apache.org/docs/latest/api/python/)
