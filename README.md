Here’s an **in-depth, step-by-step guide** to your AWS Big Data Airline Delay Analysis project. This version is structured for sharing with others (e.g., on GitHub), with clear sections and instructions so anyone can follow and replicate the work.

---

## ✈️ Airline Delay Analysis Using AWS EMR, Hadoop, and Tableau

### 🔍 Project Summary

We analyze flight delay data (1987–2008) using a Hadoop-based architecture on AWS. The dataset contains key flight attributes (arrival/departure delays, origin/destination airports, carriers, delay causes, etc.). Our objectives:

* Identify top 3 airports with highest delay time
* Identify top 3 carriers with highest delay time (in hours)
* Analyze whether arrival or departure delays are greater across airports

---

## 📁 Dataset Source

**URL**: [Harvard Dataverse](https://dataverse.harvard.edu)

* Dataset: \[Airline On-Time Performance Data (1987–2008)]
* Format: `.csv`
* File Size: Large (multi-GB; ideal for distributed processing)

---

## 🧰 Tools & Technologies Used

| Tool             | Purpose                                 |
| ---------------- | --------------------------------------- |
| **AWS EMR**      | Managed Hadoop Cluster                  |
| **HDFS**         | Distributed file storage                |
| **Hive**         | Querying structured data on Hadoop      |
| **WinSCP**       | File transfer to EC2                    |
| **SSH Terminal** | Cluster access for HDFS & Hive commands |
| **Tableau**      | Visualizing final results               |

---

## 🚀 Step-by-Step Instructions

### Step 1: 🔽 Download the Dataset

1. Visit: [https://dataverse.harvard.edu/](https://dataverse.harvard.edu/)
2. Search: `Airline On-Time Performance`
3. Filter year-wise or download the entire dataset as CSV
4. For demo purposes, use only `1987.csv` (reduce resource usage)

---

### Step 2: 🖥️ Set Up AWS EMR Cluster

1. Log into [AWS Console](https://console.aws.amazon.com/)
2. Navigate to **EMR > Create Cluster**
3. Select:

   * **Software Configuration**: Hadoop + Hive
   * **Hardware**: 1 master, 2 core nodes (can change per budget)
   * **EC2 Key Pair**: Create/Use for SSH access
4. Enable HDFS and Hive
5. Launch the cluster

---

### Step 3: 📂 Transfer CSV File to Cluster (via WinSCP)

1. Open **WinSCP**
2. Connect using:

   * Hostname: Master node DNS (from AWS EMR)
   * Username: `hadoop`
   * Key file: `.pem` file from EC2 Key Pair
3. Transfer the `1987.csv` file to `/home/hadoop/`

---

### Step 4: 🔐 SSH into Cluster

Use terminal (or PuTTY):

```bash
ssh -i your-key.pem hadoop@<master-node-public-dns>
```

---

### Step 5: 🗃️ Create HDFS Directory and Move File

```bash
# Make directory in HDFS
hdfs dfs -mkdir -p /user/omkar_airlines

# Move CSV file into HDFS
hdfs dfs -put /home/hadoop/1987.csv /user/omkar_airlines/
```

---

### Step 6: 🐝 Create Hive Table

Start Hive shell:

```bash
hive
```

Then create a Hive table:

```sql
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

*Note: Adjust column data types if necessary based on actual data formatting.*

---

### Step 7: 🔎 Run Hive Queries

**1. Top 3 Airports by Delay Time**

```sql
SELECT origin, SUM(arr_delay + dep_delay) AS total_delay
FROM flight_data_1987
GROUP BY origin
ORDER BY total_delay DESC
LIMIT 3;
```

**2. Top 3 Carriers by Delay Time in Hours**

```sql
SELECT dest, SUM(arr_delay + dep_delay)/60.0 AS delay_hours
FROM flight_data_1987
GROUP BY dest
ORDER BY delay_hours DESC
LIMIT 3;
```

**3. Greater Delay: Arrival or Departure?**

```sql
SELECT 
  'Arrival' AS type, AVG(arr_delay) AS avg_delay
FROM flight_data_1987
UNION ALL
SELECT 
  'Departure' AS type, AVG(dep_delay) AS avg_delay
FROM flight_data_1987;
```

---

### Step 8: 📊 Export and Visualize in Tableau

1. Export Hive query outputs to CSV using:

   ```sql
   INSERT OVERWRITE DIRECTORY '/user/omkar_airlines/results'
   ROW FORMAT DELIMITED
   FIELDS TERMINATED BY ','
   SELECT ...;
   ```

2. Use WinSCP or `hdfs dfs -get` to copy result CSV to local

3. Open Tableau:

   * Connect to CSV
   * Create bar charts, comparison charts, delay distributions
   * Add annotations explaining key insights

---

## 📁 Sample Folder Structure for GitHub

```
airline-delay-analysis-aws/
│
├── README.md
├── data/
│   └── 1987_sample.csv
├── scripts/
│   └── hive_queries.sql
├── screenshots/
│   └── emr_cluster_setup.png
│   └── query_results.png
├── tableau/
│   └── dashboard.twb
```

---

## 📘 README Snippet (for GitHub)

```md
# ✈️ Airline Delay Analysis on AWS using Hadoop & Hive

This project analyzes large-scale flight delay data using Hadoop on AWS EMR. It demonstrates end-to-end data engineering: ingestion, querying, and visualization of airline performance metrics from 1987 onward.

### Key Features
- Uses AWS EMR to handle large airline datasets
- Performs distributed querying via Hive
- Identifies airports & carriers with maximum delays
- Tableau dashboards to visualize results

### To Reproduce
1. Set up EMR cluster (see EMR setup guide)
2. Upload dataset to HDFS
3. Create Hive table and run queries
4. Export results and visualize in Tableau

Dataset Source: [Harvard Dataverse](https://dataverse.harvard.edu/)
```

---

## ✅ Final Tips

* **Use small-year subsets** first (`1987.csv`) before full dataset
* Check for **data type issues** (e.g., missing values in delay columns)
* For Tableau, create filters to compare delays over months, airports, or carriers
* You can automate Hive script execution via `.hql` files and `hive -f` command

---
