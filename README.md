**#Binance CDC Pipeline: Real-Time Data Capture and Dashboards**

This project implements a Change Data Capture (CDC) pipeline for real-time market data from the Binance API. It extracts data using specific endpoints, loads it into PostgreSQL with proper timestamps and indexing, captures changes via Debezium, and enables dashboard visualization on Cassandra. The setup uses Docker Compose for containerized services (Zookeeper, Kafka, Postgres, Debezium Connect).
The primary goal is to monitor and analyze cryptocurrency market changes in real-time, ensuring data integrity and efficient querying.
Features

Real-Time Data Extraction: Fetches latest prices, order book, recent trades, klines (candlesticks), and 24hr ticker stats from Binance API endpoints.
Data Loading to PostgreSQL: Structures incoming data in PostgreSQL with timestamps for reliable querying and indexing.
Change Data Capture (CDC): Uses Debezium to capture database changes from PostgreSQL and stream them via Kafka.
Dashboard Visualization: Integrates with Cassandra for storing captured changes and viewing interactive dashboards (e.g., via Apache Superset or Grafana).
Containerized Setup: Docker Compose for easy deployment of Zookeeper, Kafka, Postgres, and Debezium Connect.
Data Processing: Jupyter Notebook for API calls, data flattening, and ETL processes using Pandas and SQLAlchemy.

Note: This project uses Debezium 2.7.3 and Kafka 2.7.3 (pre-KRaft mode; consider migrating to KRaft for newer versions as noted in docker_compose.yml).
Architecture Overview

Binance API Extraction: Python script in Jupyter Notebook fetches data from endpoints like /api/v3/ticker/price, /api/v3/depth, etc.
PostgreSQL Storage: Data is flattened and loaded with timestamps for efficient storage and querying.
Debezium CDC: Monitors PostgreSQL for changes and publishes them to Kafka topics.
Cassandra Integration: Captured changes are sunk to Cassandra for long-term storage and dashboarding.
Dashboards: Visualize trends, trades, and stats (e.g., price changes over time).

[High-level diagram here – add an image like architecture.png if you create one]
Prerequisites

Docker and Docker Compose installed.
Python 3.11+ with libraries: pandas, sqlalchemy, requests, python-dotenv.
PostgreSQL credentials (set in .env file).
Binance API access (no API key needed for public endpoints used here).

Setup and Installation

Clone the Repository:
git clone https://github.com/yourusername/binance-cdc-pipeline.git
cd binance-cdc-pipeline


Set Up Environment Variables:Create a .env file in the root:
db_user=your_pg_user
db_password=your_pg_password
db_host=localhost
db_port=5432
db_name=your_db_name


Start Services with Docker Compose:
docker-compose up -d

This spins up:

Zookeeper (port 2181)
Kafka (port 9092)
Postgres (port 5432, user/password: postgres/postgres)
Debezium Connect (port 8083)

Note: For testing newer Debezium connectors, uncomment the volume mount in docker_compose.yml and add your connector archive.

Run the Jupyter Notebook:Install dependencies:
pip install -r requirements.txt  # Add a requirements.txt if not present

Launch Jupyter:
jupyter notebook Binance\ API\ Project.ipynb

Execute cells to fetch Binance data, flatten it, and load to PostgreSQL.

Configure Debezium Connector:

Use the Debezium UI or API to create a PostgreSQL connector (e.g., via curl to http://localhost:8083/connectors).
Example connector config (save as pg-connector.json):{
  "name": "pg-connector",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "database.hostname": "postgres",
    "database.port": "5432",
    "database.user": "postgres",
    "database.password": "postgres",
    "database.dbname": "your_db_name",
    "database.server.name": "binance_server",
    "topic.prefix": "binance",
    "slot.name": "debezium"
  }
}


Deploy: curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d @pg-connector.json


Sink to Cassandra:

Use Kafka Connect with a Cassandra sink connector (e.g., from Confluent Hub).
Configure topics like binance.public.binance_data to stream to Cassandra tables.


Set Up Dashboards:

Install Apache Superset or Grafana.
Connect to Cassandra as the data source.
Create dashboards for metrics like price changes, trade volumes, and klines.



Usage

Extract and Load Data: Run the notebook to fetch top 5 coins (by 24hr volume) and load to PostgreSQL table binance_data.
Monitor CDC: Use Kafka tools (e.g., kafka-console-consumer) to view changes: kafka-console-consumer --bootstrap-server localhost:9092 --topic binance.public.binance_data.
View Dashboards: Query Cassandra for visualized data (e.g., time-series charts of price/volume).

Documentation
Detailed docs are in the /docs folder:

Architecture
API Endpoints
Debezium Setup
Troubleshooting

Contributing
Contributions welcome! Fork the repo, create a branch, and submit a PR. See CONTRIBUTING.md for guidelines.
License
MIT License – see LICENSE for details.
Acknowledgments

Built with Debezium, Kafka, PostgreSQL, and Binance API.
Inspired by real-time data pipelines for crypto analysis.
