# Big Data Pipeline with Apache Superset

## Overview

This project implements a complete big data pipeline using modern technologies in a containerized environment with Docker. The pipeline extracts data from various APIs, sends it through a Kafka message bus, processes it with Spark, stores it in a PostgreSQL database, and visualizes it with Apache Superset.
![WhatsApp Image 2025-05-20 à 14 04 26_0805845d](https://github.com/user-attachments/assets/e26b16fa-9d68-492b-b1fb-649be61b390b)

## Table of Contents

- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Setup Instructions](#setup-instructions)
- [Running the Pipeline](#running-the-pipeline)
- [Monitoring and Logs](#monitoring-and-logs)
- [Visualizing Data with Superset](#visualizing-data-with-superset)
- [Useful Commands](#useful-commands)
- [Stopping the Environment](#stopping-the-environment)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## Prerequisites

Before starting, ensure you have the following installed on your machine:

- [Docker](https://www.docker.com/products/docker-desktop)
- [Docker Compose](https://docs.docker.com/compose/install/) (usually included with Docker Desktop)

## Project Structure

```
data-pipeline/
├── docker-compose.yml          # Docker services configuration
├── Dockerfile                  # Image configuration
├── requirements.txt            # Python dependencies
├── config/
│   ├── init-db.sql             # PostgreSQL initialization script
│   └── superset_config.py      # Apache Superset configuration
├── scripts/
│   ├── start_pipeline.sh       # Pipeline startup automation
│   └── python/
│       ├── multi_api_producer.py       # Data producer for Kafka
│       ├── spark_products_processor.py # Spark processor for products
│       ├── spark_carts_processor.py    # Spark processor for carts
│       └── process_users.py            # User data processor
├── data/                       # Data storage directory
└── logs/                       # Log files directory
```

## Setup Instructions

1. **Clone the repository**
   ```bash
   git clone https://github.com/CHOUAY15/Superset-visualisation.git
   ```

2. **Start the Docker containers**
   ```bash
   docker-compose up -d
   ```

3. **Verify all services are running**
   ```bash
   docker-compose ps
   ```
   
   > Note: Initial startup of all services may take several minutes. Ensure all services show "Up" status before proceeding.

4. **Install Python dependencies in the Spark master container**
   ```bash
   docker-compose exec spark-master pip install requests kafka-python psycopg2-binary
   docker-compose exec spark-master pip install py4j==0.10.9.5
   ```

## Running the Pipeline

Execute each component of the pipeline sequentially:

1. **Start the API data producer**
   ```bash
   docker-compose exec -d spark-master bash -c "cd /scripts/python && python multi_api_producer.py > /logs/producer.log 2>&1"
   ```

2. **Wait a few seconds for data to be available**

3. **Start the Spark product processor**
   ```bash
   docker-compose exec -d spark-master bash -c "cd /scripts/python && python spark_products_processor.py > /logs/spark_products.log 2>&1"
   ```

4. **Start the user data processor**
   ```bash
   docker-compose exec -d spark-master bash -c "cd /scripts/python && python process_users.py > /logs/users.log 2>&1"
   ```

5. **Start the Spark cart processor**
   ```bash
   docker-compose exec -d spark-master bash -c "cd /scripts/python && python spark_carts_processor.py > /logs/spark_carts.log 2>&1"
   ```

## Monitoring and Logs

Monitor the pipeline execution by checking the log files:

```bash
# View producer logs
cat logs/producer.log

# View product processor logs
cat logs/spark_products.log

# View user processor logs
cat logs/users.log

# View cart processor logs
cat logs/spark_carts.log
```

## Verifying Data in PostgreSQL

Connect to PostgreSQL and query the tables to verify data loading:

```bash
# Connect to PostgreSQL
docker-compose exec postgres psql -U postgres_user -d data_fakestore_db

# Check record counts
SELECT COUNT(*) FROM products;
SELECT COUNT(*) FROM cart_items;
SELECT COUNT(*) FROM users;

# Explore views
SELECT * FROM vw_product_ratings LIMIT 10;
SELECT * FROM vw_cart_analysis LIMIT 10;
SELECT * FROM vw_cart_products LIMIT 10;

# Exit PostgreSQL
\q
```

## Visualizing Data with Superset

1. **Create an admin account**
   ```bash
   docker-compose exec superset superset fab create-admin \
   --username admin \
   --firstname Admin \
   --lastname User \
   --email admin@example.com \
   --password admin
   ```

2. **Access Superset UI**
   - Open your browser and navigate to: http://localhost:8088
   - Log in with:
     - Username: admin
     - Password: admin

3. **Configure PostgreSQL Data Source**
   - Add a new database connection with these parameters:
     - Database Type: PostgreSQL
     - Host: postgres
     - Port: 5432
     - Database Name: data_fakestore_db
     - Username: postgres_user
     - Password: postgres_password
   - Test the connection and save

4. **Explore Data with SQL Lab**
   - Navigate to SQL Lab → SQL Editor
   - Write and execute queries, for example:
     ```sql
     SELECT category, AVG(price) as average_price, COUNT(*) as product_count
     FROM products
     GROUP BY category
     ORDER BY product_count;
     ```

5. **Create Visualizations**
   - Create charts from your queries
   - Add charts to dashboards
   - Configure filters for interactive analysis

## Useful Commands

```bash
# List all running containers
docker ps

# View logs of a specific service
docker-compose logs [service_name]

# Enter a bash shell in a container
docker-compose exec [service_name] bash
```

## Stopping the Environment

When you're done working with the pipeline:

```bash
# Stop containers but keep data
docker-compose down

# Stop containers and remove volumes (deletes all data)
docker-compose down -v
```

## Troubleshooting

- **Services not starting**: Check Docker logs for error messages
  ```bash
  docker-compose logs [service_name]
  ```

- **Data not appearing in PostgreSQL**: Verify Kafka topics and consumer logs
  ```bash
  docker-compose exec kafka kafka-topics.sh --list --bootstrap-server kafka:9092
  ```

- **Superset connection issues**: Ensure the database information is correctly configured

## Contributing
AMERGA Younes

Contributions are welcome! Please feel free to submit a Pull Request.

