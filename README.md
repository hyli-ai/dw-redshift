# Data Warehouse (Amazon Redshift)
>
In this project, an ETL pipeline is built using python to extracts data from S3, stages them in Redshift, and transforms data into a set of dimensional tables for the analytics team.

## Table of contents

* [Introduction](#introduction)
* [Project Description](#project-description)
* [Dataset](#dataset)
* [Database and the Star Schema Design](#database-and-the-star-schema-design)
* [Files in this Project](#files-in-this-project)
* [Project Instructions](#project-instructions)

## Introduction

### Scenario
A music streaming startup, Sparkify, has grown their user base and song database and want to move their processes and data onto the cloud. Their data resides in S3, in a directory of JSON logs on user activity on the app, as well as a directory with JSON metadata on the songs in their app.

### Goal
The goal of this project, as a data engineer, is to build an ETL pipeline that extracts data from S3, stages them in Redshift, and transforms data into a set of dimensional tables for the analytics team to continue finding insights in what songs their users are listening to. It is also expected to test the database and ETL pipeline by running queries given by the analytics team from Sparkify and compare the results with their expected results.

## Project Description
* Build an ETL pipeline for a database hosted on Redshift based on the knowledge of data warehouses and AWS.

* Load data from S3 to staging tables on Redshift and execute SQL statements that create the analytics tables from these staging tables.

## Dataset
Two sets of data are presented: the **Song Dataset** and the **Log Dataset**. These datasets reside in S3, and here are the S3 links for each:

* Song data: `s3://udacity-dend/song_data`
* Log data: `s3://udacity-dend/log_data`
* Log data json path: `s3://udacity-dend/log_json_path.json`

### Song Dataset
The first dataset is a subset of real data from the [Million Song Dataset](https://labrosa.ee.columbia.edu/millionsong/). Each file is in JSON format and contains metadata about a song and the artist of that song. The files are partitioned by the first three letters of each song's track ID. For example, here are filepaths to two files in this dataset.

```
song_data/A/B/C/TRABCEI128F424C983.json
song_data/A/A/B/TRAABJL12903CDCF1A.json
```

And below is an example of what a single song file, **TRAABJL12903CDCF1A.json**, looks like.

```json
{"num_songs": 1, "artist_id": "ARJIE2Y1187B994AB7", "artist_latitude": null, "artist_longitude": null, "artist_location": "", "artist_name": "Line Renaud", "song_id": "SOUPIRU12A6D4FA1E1", "title": "Der Kleine Dompfaff", "duration": 152.92036, "year": 0}
```

### Log Dataset
The second dataset consists of log files in JSON format generated by this [event simulator](https://github.com/Interana/eventsim) based on the songs in the dataset above. These simulate activity logs from a music streaming app based on specified configurations.

The log files in the dataset are partitioned by year and month. For example, here are filepaths to two files in this dataset.

```
log_data/2018/11/2018-11-12-events.json
log_data/2018/11/2018-11-13-events.json
```

And below is an example of what the data in a log file, **2018-11-12-events.json**, looks like.

![Log-data image](/log-data.png)

## Database and the Star Schema Design
Using the song and log datasets, a star schema optimized for queries on song play analysis was created based on the following entity-relationship (ER) diagram:
![ER Diagram](/ERD.png)

This includes the following tables:
### Fact Table
1. **songplays** - records in log data associated with song plays i.e. records with page `NextSong`
    * *songplay_id, start_time, user_id, level, song_id, artist_id, session_id, location, user_agent*
### Dimension Tables
1. **users** - users in the app
    * *user_id, first_name, last_name, gender, level*
2. **songs** - songs in music database
    * *song_id, title, artist_id, year, duration*
3. **artists** - artists in music database
    * *artist_id, name, location, latitude, longitude*
4. **time** - timestamps of records in **songplays** broken down into specific units
    * *start_time, hour, day, week, month, year, weekday*


## Files in this Project
This project workspace includes four files:

1. `create_tables.py` is where the fact and dimension tables are created for the star schema in Redshift.

2. `etl.py` is will load data from S3 into staging tables on Redshift and then process that data into the analytics tables on Redshift.

3. `sql_queries.py` is where the SQL statements are defined, which will be imported into the two other files above.

4. `dl.cfg` contains the AWS credentials.

5. `README.md` provide discussion on your process and decisions for this ETL pipeline.

## Project Instructions

### Create Table Schema

1. Design schemas for the fact and dimension tables (described as above).
2. Write a SQL `CREATE` statement for each of these tables in `sql_queries.py`.
3. Complete the logic in `create_tables.py` to connect to the database and create these tables.
4. Write SQL `DROP` statements to drop tables in the beginning of `create_tables.py` if the tables already exist. This way, one can run `create_tables.py` whenever he/she wants to reset the database and test the ETL pipeline.
5. Launch a redshift cluster and create an IAM role that has read access to S3.
6. Add redshift database and IAM role info to `dwh.cfg`.
7. Test by running `create_tables.py` and checking the table schemas in the redshift database. Use Query Editor in the AWS Redshift console for this.

### Build ETL Pipeline

1. Implement the logic in `etl.py` to load data from S3 to staging tables on Redshift.
2. Implement the logic in `etl.py` to load data from staging tables to analytics tables on Redshift.
3. Test by running `etl.py` after running `create_tables.py` and running the analytic queries on your Redshift database to compare the results with the expected results.
4. Delete the redshift cluster when finished.

### Notes

1. File dwh.cfg does not contain HOST and IAM Role for security reasons.

## Notes on Data Warehouses

### Data Warehouses from a business perspective

Let's take a closer look at details that may affect the data infrastructure. Maybe one single relational database won’t suffice.

* Retailer has a nation-wide presence → **Scale?**

* Acquired smaller retailers, brick & mortar shops, online store → **Single database? Complexity?**

* Has support call center & social media accounts → **Tabular data?**

* Customers, Inventory Staff and Delivery staff expect the system to be fast & stable → **Performance**

* HR, Marketing & Sales Reports want a lot information but have not decided yet on everything they need → **Clear Requirements?**

| OLTP / OLAP | Description |
| --- | ----------- |
| | 1. Find goods and make orders (For customers) |
| Operational Process (Make it work) | 2. Stock and find goods (For inventory staff) |
| | 3. Pick-up and delivery goods (Foe delivery staff) |
| | |
| | 1. Assess the performance (For HR) |
| Analytical Process (What's going on?) | 2. See the effect of differenct sales channels (For marketing) |
| | 3. Monitor sales growth (For management) |

For a operational databases, it is good for:

* Excellent for operations
* No redundancy, high integrity

but it is bad because:

* Too slow for analytics, too many JOINs
* Too hard to understand

### Data Warehouses from a technical perspective

* A data warehouse is a copy of transaction data specifically structured for query and analysis.

* A data warehouse is a subject-oriented, integrated, non-volatile, and time-variant collection of data in support of management's decision.

1. subject-oriented: there is no one-size-fits-all
2. integrated: from many data sources
3. non-volatile: non-transient, must be persistent
4. time-variant: same question may have different answer tomorrow

* A data warehouse is a system that retrieves and consolidates data periodically, from the source system into a dimensional or normalized data source. It usually keeps years of history and is queried for business intelligence or other analytical activities. It is typically updated in batches, not every time a transaction happens in the source system.

DWH Model:

Data In → ETL → Dimensional Model → Data Out

1. ETL: Extract the data from the source systems used for operations, transform the data and load it into a dimensional model.
2. Dimensional Model: Make it easy for business users to work with the data, and improve analytical query performance.
3. Data Out: For business intelligence (BI) applications.

### Data Warehouse Arhcitectures
1. Kimball's Bus Architecture
2. Independent Data Marts
3. Inmon's Corporate Information Factory (CIF)
4. Hybrid Kimball's Bus & Inmon's CIF

### OLAP Cubes
* An OLAP Cube is an aggregation of a fact metric on a number of dimensions.
* It is easy to communicate to business users.
* Common OLAP operations include: Roll-up, Drill-down, Slice, and Dice.

For a table with Movie, Branch, and Month columns:
1. Roll-up: Sum up the sales of each city by country. (less column in Branch dimension)
2. Drill-down: Decompose the sales of each city into smaller districts. (more columns in Branch dimension)
3. Slice: Reducing N dimensions to N-1 dimensions by restricting one dimension to a single value. (Ex. Month = 'Mar')
4. Dice: Same dimensions but computing a sub-cube by restricting some of the values of the dimensions. (Ex. Month in ['FEB', 'MAR'], Movie in ['Avatar', 'Batman'], and Branch = 'NY')

### OLAP Cube Query Optimization

* Business users will typically want to slice, dice, roll-up, and drill-down all the time.
* Each such combination will potentially go through all the Facts Table, which is sub-optimal.
* The `GROUP BY CUBE (Movie, Branch, and Month)` will make one pass through the Facts Table and will aggregate **all possible combinations** of groupings of length 0, 1, 2, 3.
1. Length 0: Total Revenue.
2. Length 1: Revenue by Month, Revenue by Branch, Revenue by Movie.
3. Length 2: Revenue by Month and Branch, Revenue by Movie and Branch, Revenue by Month and Movie.
4. Length 3: Revenue by Month, Branch, and Movie.
* Saving, Materializing the output of the CUBE operation and using it is usually enough to answer all forthcoming aggregations from business users without having to process the whole Facts Table again.

### How Do We Serve these OLAP Cubes?
* Approach 1: Pre-aggregate the OLAP Cubes and saves them in a special purpose non-relational database. (MOLAP)
* Approach 2: Compute the OLAP Cubes on the fly from the existing relational databases where the dimensional model resides. (ROLAP)

### Massively Parallel Processing (MPP) Databases
* Most relational databases execute multiple queries in parallel if they have access to many cores/servers.
* However, every query is always executed on a single CPU of a single machine.
* This is acceptable for OLTP, since it has many concurrent users and each query is not very long. It mostly does updates and a few rows retrieval.
* Massively Parallel Processing (MPP) databases parallelize the execution of one query on multiple CPUs/machines.
* How? A table is partitioned and partitions are processed in parallel.
* Amazon Redshift is a cloud-managed, column-based, MPP database.
* Other examples include Teradata Aster, Oracle Exadata, and Azure SQL.


