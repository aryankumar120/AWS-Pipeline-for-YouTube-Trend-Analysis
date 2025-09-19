# A Serverless AWS Pipeline for YouTube Trend Analysis - Data Engineering Project 😳🤖

# 📌 Project Overview
This project showcases a secure, scalable, and fully automated end-to-end data engineering pipeline built on AWS. It ingests raw YouTube trending statistics from multiple regions, processes the structured and semi-structured data using a serverless ETL workflow, and makes it available for analysis and visualization on an interactive business intelligence dashboard.

The core objective is to transform raw, multi-format data from a Kaggle dataset into clean, queryable insights that answer questions about video trends, category performance, and engagement metrics across different countries.

# 🏛️ Solution Architecture
The architecture is designed around a serverless data lake, promoting scalability and cost-efficiency. The data flows through a multi-stage process, ensuring a clear separation between raw, processed, and analytics-ready data.

<img width="1424" height="763" alt="Screenshot 2025-09-19 at 4 10 38 PM" src="https://github.com/user-attachments/assets/bfeaf05a-1530-4234-9b18-828a14244e7c" />


# Data Flow Explained:
📥 Data Ingestion (S3 Landing Zone): Raw CSV and JSON data are ingested in bulk to the S3 "Landing Area." This serves as the initial entry point into our data lake, keeping the original source data unmodified for auditing and reprocessing.

⚙️ ETL & Data Processing (AWS Glue):

- An AWS Glue Crawler scans the landing zone to automatically infer schemas and populate the AWS Glue Data Catalog, making the raw data visible to other AWS services.
- A serverless AWS Glue ETL job (using PySpark) is triggered. This job is the workhorse of the pipeline, performing key transformations:
- Cleaning: Handles data quality issues in the source files.
- Enriching: Joins the video statistics with the JSON category data to add meaningful context.
- Optimizing: Converts the data to the columnar Apache Parquet format and partitions it by region for dramatically faster and cheaper queries.

🗄️ Data Storage (S3 Cleansed & Analytics Zone):

- The processed, partitioned Parquet data is written back to the "Cleansed / Enriched" zone in S3.
- A final aggregation job creates the final_analytics table in the "Analytics / Reporting" zone, ready for BI consumption.

🔎 Data Querying (Amazon Athena): 

- Amazon Athena is used as a serverless query engine to run standard SQL queries directly on the processed data in S3.
- Because the data is partitioned, a query for a specific region (e.g., WHERE region = 'us') only scans that region's data, saving time and money.

📈 Data Visualization (Amazon QuickSight): 

- The final step connects Amazon QuickSight to Athena. This allows for the creation of interactive and insightful dashboards that visualize key metrics like top video categories, trends in likes/views, and channel performance.

🔐 Security & Orchestration:

- AWS IAM provides granular permissions, ensuring each service only has access to the resources it needs (Principle of Least Privilege).
- AWS Step Functions or Lambda can be used to orchestrate and automate the entire workflow from ingestion to ETL.

# 🛠️ Tech Stack & Services
- 🪣 Amazon S3: Scalable object storage for our multi-layered data lake (Raw, Cleansed, Analytics).

- ⚙️ AWS Glue: The core of our serverless ETL pipeline for data cataloging, transformation, and processing.

- 🔎 Amazon Athena: Serverless SQL query engine for interactive analysis of data in S3.

- 📊 Amazon QuickSight: Business intelligence service to build and share interactive dashboards.

- 🔐 AWS IAM: Securely manages access and permissions for all AWS services.

# 🚀 Challenges & Learnings (For Recruiters)
   This project provided valuable hands-on experience in overcoming common data engineering challenges. Here are a few key problems I diagnosed and solved:

   Problem: Handling Real-World, Imperfect Data

 - Challenge: The initial Glue jobs were failing with Unable to parse file errors on several CSV files (MXvideos.csv, JPvideos.csv). This is a classic real-world scenario where source data is not perfectly formatted.

 - Action: I diagnosed the issue as malformed rows within the CSV files. Instead of manually cleaning each file (which is not scalable), I engineered a robust solution. I configured the Glue Data Catalog table properties to force the correct csv classification and set the reader mode to DROPMALFORMED.

 - Result: This made the pipeline resilient. The job now automatically skips bad rows while successfully processing all the valid data, preventing data quality issues from halting the entire ETL process.

   Problem: Schema & Partition Synchronization

 - Challenge: After partitioning the data in S3 (e.g., region=ca/), Athena queries were not showing any performance improvement and the partitions weren't visible.

 - Action: I identified that creating partition folders in S3 is not enough; the Glue Data Catalog must also be made aware of them. I used the MSCK REPAIR TABLE command in Athena to sync the table's metadata with the partition structure in S3.

 - Result: This correctly registered the partitions, enabling Athena to perform partition pruning. Queries with a WHERE region = '...' clause became significantly faster and cheaper because they only scanned the relevant data.

   Problem: Cross-Database Joins and UI Discrepancies

 - Challenge: In my organized setup (separate databases for raw and cleaned data), I couldn't find the raw_statistics table in the db_youtube_cleaned database, which was different from the tutorial I was following.

 - Action: I realized that my setup was valid but required a different approach. I adapted my Athena queries to perform a cross-database join, explicitly specifying the database for each table (e.g., "de_youtube_raw"."raw_statistics").

 - Result: This demonstrated the flexibility to adapt solutions to different architectural patterns and a deeper understanding of how Athena resolves table locations using the Glue Data Catalog.

# 📊 Dataset
The project utilizes the Trending YouTube Video Statistics dataset from Kaggle.

🔗 Link to the Kaggle Dataset: https://www.kaggle.com/datasets/datasnaek/youtube-new

# 📈 Final Dashboard Preview
<img width="901" height="717" alt="image" src="https://github.com/user-attachments/assets/67fde460-b600-4cc3-aac3-b0c3f5b5b853" />


This dashboard answers key business questions, such as:

- Which video categories have the most likes and views across all regions?

- What is the correlation between views, likes, and comments?

- Which channels are trending most often in each region?
