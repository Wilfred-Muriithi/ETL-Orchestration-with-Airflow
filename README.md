# ETL Orchestration with Airflow

A **production-grade ETL pipeline** built with Apache Airflow, demonstrating end-to-end data orchestration, transformation, and warehouse loading with Docker containerization and robust error handling.

## 🎯 Project Overview

This project showcases a complete data engineering workflow that extracts data from multiple sources (APIs and databases), transforms it through cleaning and validation pipelines, and loads it into a PostgreSQL data warehouse. The entire orchestration is managed by Apache Airflow with LocalExecutor, containerized using Docker Compose for easy deployment and scalability.

**Key Highlights:**
- ✅ **Complete ETL Pipeline**: Extract → Transform → Load with parallel processing
- ✅ **Production-Ready**: Error handling, retries with exponential backoff, and email notifications
- ✅ **Data Quality Validation**: Built-in quality checks with configurable thresholds (95% minimum)
- ✅ **Docker Containerization**: Fully containerized environment with PostgreSQL, Redis, and Airflow services
- ✅ **Scalable Architecture**: Easy migration from LocalExecutor to CeleryExecutor or KubernetesExecutor
- ✅ **Monitoring & Logging**: Comprehensive logging and execution reporting

## 🏗️ Architecture

```
┌─────────────────────────────────────────────┐
│      Airflow Orchestration Layer            │
│   (Scheduler, Webserver, LocalExecutor)     │
└──────────────┬──────────────────────────────┘
               │
    ┌──────────┴──────────┐
    │                     │
┌───▼────┐          ┌─────▼─────┐
│  API   │          │ Database  │
│ Source │          │  Source   │
└───┬────┘          └─────┬─────┘
    │                     │
    └──────────┬──────────┘
               │
        ┌──────▼──────┐
        │ Transform:  │
        │ • Clean     │
        │ • Dedupe    │
        │ • Validate  │
        │ • Enrich    │
        └──────┬──────┘
               │
        ┌──────▼─────────┐
        │ Quality Checks │
        │   & Merge      │
        └──────┬─────────┘
               │
     ┌─────────▼──────────┐
     │  Data Warehouse    │
     │   (PostgreSQL)     │
     └────────────────────┘
```

## 🚀 Quick Start

### Prerequisites
- Docker (20.10+)
- Docker Compose (2.0+)
- 4GB RAM minimum

### Setup & Run

```bash
# Clone the repository
git clone https://github.com/Wilfred-Muriithi/ETL-Orchestration-with-Airflow.git
cd ETL-Orchestration-with-Airflow

# Create environment configuration
cat > .env << EOF
POSTGRES_USER=airflow
POSTGRES_PASSWORD=****
POSTGRES_DB=airflow
DATA_WAREHOUSE_DB=data_warehouse
ENVIRONMENT=development
DEBUG=true
EOF

# Build Docker images
docker-compose build

# Initialize Airflow database
docker-compose up airflow-init

# Start all services
docker-compose up -d

# Verify services are running
docker-compose ps
```

### Access Airflow UI

Navigate to `http://localhost:8080`

**Credentials:**
- Username: `airflow`
- Password: `*****`

## 📁 Project Structure

```
ETL-Orchestration-with-Airflow/
├── dags/
│   ├── etl_pipeline_dag.py         # Main DAG definition
│   └── utils/
│       ├── extractors.py           # Data extraction functions
│       ├── transformers.py         # Data transformation logic
│       └── loaders.py              # Data loading utilities
├── config/
│   └── settings.py                 # Configuration management
├── scripts/
│   └── init_db.sql                 # Database initialization
├── tests/
│   └── test_etl_logic.py           # Unit tests
├── docker-compose.yaml             # Service orchestration
├── Dockerfile                      # Custom Airflow image
├── requirements.txt                # Python dependencies
└── README.md                       # This file
```

## 🔄 ETL Pipeline Workflow

### Phase 1: Setup & Validation
- **start_pipeline**: Initialize pipeline execution
- **validate_prerequisites**: Verify configurations and database connections

### Phase 2: Extract (Parallel Execution)
- **extract_from_api**: Fetch data from external REST APIs
- **extract_from_database**: Query internal PostgreSQL database with incremental loading

### Phase 3: Transform (Parallel Execution)
- **transform_api_data**: Clean, deduplicate, and enrich API data
- **transform_db_data**: Clean, deduplicate, and validate database records
- **merge_and_validate**: Combine datasets and perform quality checks

### Phase 4: Load & Verify
- **prepare_warehouse**: Create/update staging tables and indexes
- **load_to_warehouse**: Upsert data with conflict resolution
- **run_data_quality_checks**: Post-load validation (duplicates, nulls, schema)
- **generate_pipeline_report**: Create execution summary with statistics
- **end_pipeline**: Mark successful completion

## 🛠️ Key Features

### Error Handling & Resilience
```python
# Automatic retry configuration
retries=3
retry_delay=timedelta(minutes=5)
retry_exponential_multiplier=2      # 5min, 10min, 20min
max_retry_delay=timedelta(minutes=15)
execution_timeout=timedelta(hours=2)
email_on_failure=True
```

### Data Quality Validation
- **Quality Threshold**: 95% minimum data quality score
- **Checks**: Completeness, accuracy, consistency, uniqueness
- **Validation Results**: Detailed quality reports with issue tracking

### Transformation Pipeline
```python
transformations = [
    'clean',        # Strip whitespace, handle nulls, type casting
    'deduplicate',  # Remove exact duplicates
    'validate',     # Business rule validation
    'enrich'        # Add metadata and computed columns
]
```

## 🗄️ Database Schema

### Schemas
- **staging**: Temporary ETL data and intermediate results
- **warehouse**: Production-cleaned data for analytics
- **logs**: Pipeline execution audit trail

### Key Tables

**staging.etl_data**
```sql
CREATE TABLE staging.etl_data (
    id SERIAL PRIMARY KEY,
    source VARCHAR(50),
    data JSONB NOT NULL,
    processed_at TIMESTAMP DEFAULT NOW(),
    created_at TIMESTAMP DEFAULT NOW()
);
```

**warehouse.job_listings**
```sql
CREATE TABLE warehouse.job_listings (
    job_id SERIAL PRIMARY KEY,
    source VARCHAR(100),
    title VARCHAR(500),
    company VARCHAR(300),
    location VARCHAR(200),
    salary_range VARCHAR(100),
    description TEXT,
    url TEXT,
    extracted_at TIMESTAMP,
    processed_at TIMESTAMP DEFAULT NOW()
);
```

## 📊 Monitoring & Debugging

### View Logs
```bash
# Scheduler logs
docker-compose logs -f airflow-scheduler

# Webserver logs
docker-compose logs -f airflow-webserver

# PostgreSQL logs
docker-compose logs -f postgres
```

### Database Access
```bash
# Connect to PostgreSQL
docker-compose exec postgres psql -U airflow -d airflow

# Query staging data
SELECT * FROM staging.etl_data LIMIT 10;

# Check execution logs
SELECT * FROM logs.etl_logs ORDER BY created_at DESC LIMIT 5;
```

### Airflow UI Features
- **Grid View**: Task execution history and status tracking
- **Gantt Chart**: Timeline visualization of task dependencies
- **Graph View**: Visual DAG structure and relationships
- **Calendar**: Historical success/failure patterns
- **Logs**: Real-time task execution logs

## 🔧 Configuration

### Environment Variables

Key configurations in `.env`:

```bash
# Airflow Core
AIRFLOW_HOME=/opt/airflow
AIRFLOW__CORE__EXECUTOR=LocalExecutor
AIRFLOW__CORE__PARALLELISM=4
AIRFLOW__CORE__MAX_ACTIVE_TASKS_PER_DAG=3

# Database
POSTGRES_USER=airflow
POSTGRES_PASSWORD=****
POSTGRES_HOST=postgres
POSTGRES_PORT=5432

# Data Warehouse
DATA_WAREHOUSE_DB=data_warehouse
DATA_WAREHOUSE_HOST=postgres

# Pipeline Settings
BATCH_SIZE=1000
DATA_QUALITY_THRESHOLD=0.95
ENVIRONMENT=development
```

### Airflow Connections

Configure database connections in the UI:
1. Navigate to **Admin → Connections**
2. Add **postgres_default** connection
3. Set connection parameters (host, port, database, credentials)

## 📦 Dependencies

### Core Libraries
```
apache-airflow==2.7.3
apache-airflow-providers-postgres==5.10.0
apache-airflow-providers-http==4.7.0
```

### Data Processing
```
pandas==2.1.4
numpy==1.26.3
python-dateutil==2.8.2
```

### Database
```
psycopg2-binary==2.9.9
sqlalchemy==2.0.23
```

See `requirements.txt` for complete dependency list.

## 🧪 Testing

```bash
# Run all tests
pytest tests/ -v

# Run with coverage
pytest tests/ --cov=dags --cov-report=html

# Validate DAG syntax
airflow dags validate

# Test DAG parsing
airflow dags list

# Trigger test run
airflow dags test etl_pipeline_dag 2024-01-01
```

## 🚀 Scaling Strategy

### Current: LocalExecutor
- Single-machine execution
- Suitable for development and testing
- 1-4 parallel tasks

### Scale to CeleryExecutor
- Distributed task execution
- Multiple worker machines
- Redis broker (already configured)
- Horizontal scaling capability

### Scale to KubernetesExecutor
- Each task runs in isolated pod
- Auto-scaling based on load
- Cloud-native deployment
- Use Helm charts for management

### Production Deployment
- **AWS**: Amazon MWAA (Managed Workflows for Apache Airflow)
- **GCP**: Cloud Composer
- **Azure**: Azure Synapse Analytics + Airflow

## 🔐 Security Best Practices

### Secrets Management
✅ Store secrets in Airflow Connections (encrypted)  
✅ Use environment variables for configuration  
✅ Rotate credentials regularly  
✅ Never commit secrets to version control

### Access Control
```python
# Enable RBAC in production
AIRFLOW__WEBSERVER__RBAC = True
AIRFLOW__WEBSERVER__AUTHENTICATE = True
```

## 🐛 Troubleshooting

### Common Issues

**Issue: DAG not appearing in UI**
```bash
# Wait 2-3 minutes for DAG parsing
# Check scheduler logs
docker-compose logs airflow-scheduler | grep "Parsing"
```

**Issue: Database connection error**
```bash
# Verify PostgreSQL health
docker-compose ps postgres

# Check connection string
docker-compose logs postgres
```

**Issue: Task timeout**
```python
# Increase execution timeout in DAG
execution_timeout=timedelta(hours=4)
```

## 📚 Resources

- [Apache Airflow Documentation](https://airflow.apache.org/docs/)
- [Airflow Best Practices](https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html)
- [Provider Packages](https://registry.astronomer.io/)
- [Community Slack](https://apache-airflow.slack.com/)

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is available for educational and portfolio purposes.

## 👤 Author

**Wilfred Muriithi**
- GitHub: [@Wilfred-Muriithi](https://github.com/Wilfred-Muriithi)
- Portfolio: [wilfred-muriithi.github.io](https://wilfred-muriithi.github.io)

***

**Built with ❤️ for data engineering excellence**

