# ✈️ Airline Delay Analysis using AWS EMR, Hadoop, Hive & Tableau

This project analyzes large-scale U.S. airline flight data from the years 1987 to 2008 to uncover patterns in delays, cancellations, and flight performance using big data technologies. The goal is to identify major sources of delay by airport, airline carrier, and delay type, and visualize these insights using Tableau dashboards.

---

## Project Overview

- **Data Source:** [Harvard Dataverse – Airline On-Time Performance](https://dataverse.harvard.edu/)
- **Data Format:** CSV (Gigabytes of data across multiple years)
- **Tools Used:**
  - **AWS EMR** – Hadoop Cluster for scalable processing
  - **HDFS** – Distributed storage of flight data
  - **Apache Hive** – SQL-based querying on Hadoop
  - **WinSCP + SSH** – File transfer & remote cluster access
  - **Tableau** – Visualize key findings

---

##  Objectives

1. Identify **Top 3 Airports** with the highest total delay time.
2. Identify **Top 3 Carriers** with the highest delay (in hours).
3. Analyze whether **arrival or departure delays** are more severe across airports.

---

## 📁 Dataset Description

Each record represents a single flight and includes:

- Flight date: Year, Month, Day
- Times: Departure Time, Arrival Time, Taxi In, Taxi Out
- Delays: Departure Delay, Arrival Delay, Carrier Delay, Weather Delay, NAS Delay, Security Delay, Late Aircraft Delay
- Flight status: Cancelled or Diverted
- Origin & Destination airport codes
- Distance traveled

---

## ⚙️ Project Workflow

### 1. Data Extraction
- Downloaded CSV files from [dataverse.harvard.edu](https://dataverse.harvard.edu/)
- Selected `1987.csv` for initial analysis

### 2. AWS EMR Cluster Setup
- Created EMR cluster (Hadoop + Hive) with 1 master and 2 core nodes
- Enabled HDFS and Hive for data processing

### 3. Upload Dataset via WinSCP
- Used WinSCP to transfer large CSV files to the master node
- Verified file integrity before moving to HDFS

### 4. Data Ingestion in HDFS
```bash
hdfs dfs -mkdir -p /user/omkar_airlines
hdfs dfs -put 1987.csv /user/omkar_airlines/
```

### 5.  Hive Table Creation
Connected to Hive and executed:
``` SQL
CREATE DATABASE IF NOT EXISTS omkar_airlines;
USE omkar_airlines;

CREATE EXTERNAL TABLE flight_data_1987 (
  year INT,
  month INT,
  day_of_month INT,
  day_of_week INT,
  dep_time STRING,
  arr_time STRING,
  arr_delay INT,
  dep_delay INT,
  origin STRING,
  dest STRING,
  distance INT,
  taxi_in INT,
  taxi_out INT,
  cancelled INT,
  diverted INT,
  carrier_delay INT,
  weather_delay INT,
  nas_delay INT,
  security_delay INT,
  late_aircraft_delay INT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/omkar_airlines/';
```

### 6. Hive Queries for Analysis
Top 3 Delayed Airports
Top 3 Delayed Carriers (in hours)
Arrival vs Departure Delay Comparison

```
SELECT origin, SUM(arr_delay + dep_delay) AS total_delay
FROM flight_data_1987
GROUP BY origin
ORDER BY total_delay DESC
LIMIT 3;

SELECT dest, SUM(arr_delay + dep_delay)/60.0 AS delay_hours
FROM flight_data_1987
GROUP BY dest
ORDER BY delay_hours DESC
LIMIT 3;

SELECT 'Arrival' AS type, AVG(arr_delay) AS avg_delay FROM flight_data_1987
UNION ALL
SELECT 'Departure' AS type, AVG(dep_delay) AS avg_delay FROM flight_data_1987;
```

