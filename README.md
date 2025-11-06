Stock Market Data Warehouse (DBMS Project)
This repository contains the source code for a DBMS project demonstrating the efficient storage, retrieval, and analysis of large-scale financial time-series data using MySQL.

The project builds a stock market data warehouse and benchmarks the performance of "point-in-time" queries and analytical workloads on a non-optimized table versus tables optimized with B-Tree indexing and table partitioning.

Problem Statement
Standard relational databases (like MySQL) are often used as a one-size-fits-all solution, but they are not innately optimized for the massive scale and specific query patterns of time-series data. This leads to severe performance degradation for critical financial queries (e.g., "get the price of AAPL on 2023-10-25"), which in turn makes running analytics or training ML models slow and impractical.

This project proves that by applying fundamental database optimization techniques, a standard MySQL database can be configured to deliver high-performance results comparable to specialized time-series databases.

Project Structure & Workflow
Data Ingestion (data.py): Downloads 20 years of stock data for 20 companies from the yfinance API and saves it as a single CSV file.

Database Loading (load_data.py): Loads the CSV data into three separate MySQL tables:

stock_data_baseline: No optimizations.

stock_data_indexed: With a composite B-Tree index.

stock_data_partitioned: With both the B-Tree index and range partitioning by year.

Data Mining (mine.py): Connects to the fast partitioned table, runs an analytical query (Simple Moving Average), and saves the results into a new patterns_warehouse table.

Prediction (mine_and_predict.py): Connects to the database, fetches all data for a specific stock, trains a scikit-learn Linear Regression model on-the-fly, and predicts the next day's closing price. This script also serves as a benchmark, comparing the data-fetching speed of the baseline vs. partitioned tables.

Requirements
Python 3.x

MySQL Server (8.x recommended)

A MySQL client (like MySQL Workbench or DBeaver)

You can install all necessary Python libraries using the provided requirements.txt file:

Bash

pip install -r requirements.txt
(If you don't have a requirements.txt file, create one and add these lines:)

pandas
yfinance
mysql-connector-python
sqlalchemy
scikit-learn
How to Run This Project
Follow these steps in order.

1. Setup the MySQL Database
Connect to your MySQL server and run the following SQL commands to create the database and the three necessary tables.

SQL

-- 1. Create the database
CREATE DATABASE stock_warehouse;
USE stock_warehouse;

-- 2. Create the Baseline (slow) table
CREATE TABLE stock_data_baseline (
    `date` DATE,
    `open` DECIMAL(10, 2),
    `high` DECIMAL(10, 2),
    `low` DECIMAL(10, 2),
    `close` DECIMAL(10, 2),
    `adj_close` DECIMAL(10, 2),
    `volume` BIGINT,
    `ticker` VARCHAR(10)
);

-- 3. Create the Indexed table
CREATE TABLE stock_data_indexed (
    `date` DATE,
    `open` DECIMAL(10, 2),
    `high` DECIMAL(10, 2),
    `low` DECIMAL(10, 2),
    `close` DECIMAL(10, 2),
    `adj_close` DECIMAL(10, 2),
    `volume` BIGINT,
    `ticker` VARCHAR(10),
    INDEX idx_ticker_date (ticker, `date`)
);

-- 4. Create the fully Optimized (fast) table
CREATE TABLE stock_data_partitioned (
    `date` DATE,
    `open` DECIMAL(10, 2),
    `high` DECIMAL(10, 2),
    `low` DECIMAL(10, 2),
    `close` DECIMAL(10, 2),
    `adj_close` DECIMAL(10, 2),
    `volume` BIGINT,
    `ticker` VARCHAR(10),
    INDEX idx_ticker_date (ticker, `date`)
)
PARTITION BY RANGE ( YEAR(`date`) ) (
    PARTITION p2005 VALUES LESS THAN (2006),
    PARTITION p2006 VALUES LESS THAN (2007),
    PARTITION p2007 VALUES LESS THAN (2008),
    PARTITION p2008 VALUES LESS THAN (2009),
    PARTITION p2009 VALUES LESS THAN (2010),
    PARTITION p2010 VALUES LESS THAN (2011),
    PARTITION p2011 VALUES LESS THAN (2012),
    PARTITION p2012 VALUES LESS THAN (2013),
    PARTITION p2013 VALUES LESS THAN (2014),
    PARTITION p2014 VALUES LESS THAN (2015),
    PARTITION p2015 VALUES LESS THAN (2016),
    PARTITION p2016 VALUES LESS THAN (2017),
    PARTITION p2017 VALUES LESS THAN (2018),
    PARTITION p2018 VALUES LESS THAN (2019),
    PARTITION p2019 VALUES LESS THAN (2020),
    PARTITION p2020 VALUES LESS THAN (2POST-AUTHORIZATION_RULES:),
    PARTITION p2021 VALUES LESS THAN (2022),
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p_max VALUES LESS THAN MAXVALUE
);
2. Download the Stock Data
Run the data.py script. This will contact the yfinance API and create a large all_stock_data_20_years.csv file in your project directory.

Bash

python data.py
3. Load Data into the Database
IMPORTANT: Open the load_data.py script and edit the DB_PASS variable to match your MySQL root password.

After saving your changes, run the script. This will read the CSV and bulk-load the data into all three tables. This may take a few minutes.

Bash

python load_data.py
4. Run the Data Mining Engine (SMA)
IMPORTANT: Open mine.py and edit the DB_PASS variable to match your password.

Run the script. It will connect to the fast stock_data_partitioned table, calculate 20-day and 50-day moving averages, and save them to a new patterns_warehouse table.

Bash

python mine.py
You can verify the result in MySQL Workbench with SELECT * FROM patterns_warehouse LIMIT 10;.

5. Run the Prediction Engine (Linear Regression)
IMPORTANT: Open mine_and_predict.py and edit the DB_PASS variable to match your password.

Run the script. It will ask you for a stock ticker and then which database to use. This allows you to benchmark the performance difference in data fetching.

Bash

python mine_and_predict.py
Example Output (shows the performance difference):

Choose the source table for NFLX:
1. stock_data_partitioned
2. stock_data_baseline
Enter your choice (1 or 2): 1
Fetching data for NFLX from 'stock_data_partitioned'...
Successfully fetched 5032 rows. Time taken: 0.1444 seconds.

---

Choose the source table for NFLX:
1. stock_data_partitioned
2. stock_data_baseline
Enter your choice (1 or 2): 2
Fetching data for NFLX from 'stock_data_baseline'...
Successfully fetched 5032 rows. Time taken: 0.3486 seconds.
License
This project is licensed under the MIT License.
