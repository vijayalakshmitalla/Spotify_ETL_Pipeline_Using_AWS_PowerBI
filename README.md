# Spotify Data ETL Pipeline

This project demonstrates an end-to-end Extract, Transform, Load (ETL) pipeline for Spotify playlist data. It leverages the Spotify Web API, AWS S3 for data storage, AWS Lambda for data processing, Snowflake for data warehousing, and Power BI for data visualization.

  
## Project Overview

The goal of this project is to extract popular song data from Spotify, process it, store it in a data warehouse, and finally visualize key insights using a business intelligence tool. The pipeline is designed to be automated, utilizing AWS services for scheduled data ingestion and transformation.

## Architecture

The project architecture involves the following components:

1.  **Spotify Web API**: Used to extract raw playlist, track, album, and artist data.
2.  **AWS S3**: Acts as a data lake for storing raw and transformed data.
    * `spotify-etl-project-vijju`: Main bucket 
    * `raw_data/`: Stores unprocessed JSON data directly from Spotify
        * `processed/`
        * `to_processed/`
    * `transformed_data/`: Stores structured and processed data in Parquet or CSV format.
        * `album_data/`
        * `artist_data/`
        * `songs_data/`
3.  **AWS Lambda**:
    * **`spotify_api_data_extract`**: Python function responsible for extracting data from Spotify and loading it into the `raw_data/to_processed` folder in S3.
    * **Transformation Lambda Function**: A separate Python function processes the raw data from `raw_data/to_processed`, transforms it into a structured format (e.g., CSV), and then loads it into the respective subfolders within `transformed_data/` in S3.
4.  **AWS CloudWatch**: Schedules the Lambda functions to run automatically, ensuring regular data updates.
5.  **Snowflake**: Cloud data warehouse for storing and analyzing the transformed data.
    * Utilizes `STORAGE INTEGRATION` to connect to S3
    * Employs `SNOWPIPE` for continuous data loading from S3 into Snowflake tables.
    * Tables created: `TBLALBUM`, `TBLARTIST`, `TBLSONGS`
6.  **Power BI**: Business intelligence tool used to create interactive dashboards for data visualization and insights

## Features

* Automated data extraction from Spotify's "Top Songs - Global" playlist (or similar).
* Separate Lambda functions for raw data extraction and transformed data processing.
* Storage of raw and transformed data in an S3 data lake.
* Serverless data processing using AWS Lambda.
* Scheduled data pipeline execution with AWS CloudWatch.
* Robust data warehousing with Snowflake, enabling efficient querying.
* Interactive dashboard in Power BI for analyzing Spotify trends.
* Modular and scalable architecture.

## Technologies Used

* **Programming Language**: Python
* **Spotify API**: For data extraction
* **AWS Services**:
    * S3 (Simple Storage Service)
    * Lambda
    * CloudWatch
* **Data Warehouse**: Snowflake
* **Business Intelligence**: Power BI
* **Python Libraries**:
    * `spotipy` (for Spotify API interaction)
    * `boto3` (for AWS S3 interaction)
    * `datetime`

## Setup and Installation

### Spotify API Setup

1.  **Create a Spotify Developer Account**: Go to [Spotify for Developers](https://developer.spotify.com/) and create an account.
2.  **Create an Application**: Create a new application on the dashboard.
3.  **Obtain Credentials**: Note down your `Client ID` and `Client Secret`.These will be used for authentication.
4.  **Find a Playlist Link**: Identify the Spotify playlist you want to extract data from, e.g., "Top Songs - Global".

### AWS Setup

1.  **S3 Bucket Creation**:
    * Create an S3 bucket named `spotify-etl-project-vijju` (or your preferred name).
    * Create the following folder structure within the bucket:
        * `raw_data/`
            * `processed/`
            * `to_processed/`
        * `transformed_data/`
            * `album_data/`
            * `artist_data/`
            * `songs_data/`
2.  **IAM Role for Lambda**:
    * Create an IAM role with permissions to access S3 (read/write) and CloudWatch logs.
3.  **Lambda Functions**:
    * **`spotify_api_data_extract`**:
        * Upload `lambda_function.py` (your Python code for extraction) to this Lambda function.
        * Configure environment variables for `CLIENT_ID` and `CLIENT_SECRET`.
        * Ensure the `spotipy` and `requests` libraries are included as a Lambda layer or within your deployment package.
    * **Transformation Lambda Function**:
        * Create a separate Lambda function for processing the raw data from `raw_data/to_processed`, performing the necessary transformations, and then uploading the structured data to the `transformed_data/` folders (e.g., `album_data/`, `artist_data/`, `songs_data/`).
        * This function will also require appropriate S3 read/write permissions.
4.  **CloudWatch Trigger**:
    * Set up CloudWatch Event Rules to trigger the `spotify_api_data_extract` Lambda function on a scheduled basis (e.g., daily).
    * Set up another CloudWatch Event Rule to trigger the Transformation Lambda Function, typically after the raw data extraction completes, or on a separate schedule.

### Snowflake Setup

1.  **Create Snowflake Account**:Sign up for a Snowflake account.
2.  **Database and Schema**: Create a database (e.g., `SPOTIFY_ETL_PROJECT`) and a schema within it.
3.  **Storage Integration**:
    * Execute the `CREATE OR REPLACE STORAGE INTEGRATION` command to allow Snowflake to access your S3 bucket
    * Grant the necessary S3 permissions to the IAM role specified in the storage integration.
4.  **File Format**:
    * Define a file format for CSV data, e.g., `CREATE OR REPLACE FILE FORMAT csv_file_format`.
5.  **Tables**:
    * Create the following tables to store your transformed data:
        * `TBLALBUM`
        * `TBLARTIST`
        * `TBLSONGS`
6.  **Snowpipes**:
    * Set up Snowpipes for continuous data loading from the `transformed_data/` folders in S3 into the respective Snowflake tables.

### Power BI Setup

1.  **Install Power BI Desktop**: Download and install Power BI Desktop.
2.  **Connect to Snowflake**:
    * Open Power BI and select "Get Data" -> "Snowflake".
    * Enter your Snowflake account details and connect to the `SPOTIFY_ETL_PROJECT` database.
3.  **Load Data**: Select the `TBLALBUM`, `TBLARTIST`, and `TBLSONGS` tables.
4.  **Build Dashboard**: Create visualizations and build your interactive dashboard based on the loaded data.

## Usage

Once the pipeline is set up, the CloudWatch schedules will automatically extract raw data, process and transform it using separate Lambda functions, and then load it into Snowflake. You can then refresh your Power BI dashboard to view the latest insights.

## Data Model (Snowflake)

The Snowflake database contains the following tables, representing the transformed Spotify data:

* **`TBLALBUM`**: Contains information about albums.
    * `album_id`
    * `album_name`
    * `album_release_date`
    * `album_total_tracks`
    * `album_url`
* **`TBLARTIST`**: Contains information about artists.
    * `artist_id`
    * `artist_name`
    * `external_url`
* **`TBLSONGS`**: Contains information about individual songs.
    * `song_id`
    * `song_name`
    * `song_duration`
    * `song_url`
    * `song_popularity`
    * `album_id` (Foreign Key to `TBLALBUM`)
    * `artist_id` (Foreign Key to `TBLARTIST`)
    * `song_added` (Timestamp when the song was added to the playlist)

## Dashboard (Power BI)

The Power BI dashboard provides an interactive view of the Spotify data. Key visualizations include:

* **Total Unique Artists** and **Total Number of Songs**.
* **Songs by Day of Month**: Trends of songs added over the month.
* **All Songs with Popularity Score**: Bar chart showing songs with their popularity scores.
* **Top 10 Songs by Popularity**: A detailed list of the most popular songs.
* Filters for `SONG_NAME`, `SONG_DURATION`, and `POPULARITY` to allow dynamic analysis.

## Future Enhancements

* **Error Handling and Logging**: Implement more robust error handling and detailed logging for both Lambda functions.
* **Data Validation**: Add data validation steps to ensure the data quality throughout the pipeline.
* **Scalability**: Optimize Lambda functions for larger datasets and consider AWS Step Functions for workflow orchestration between the extraction and transformation steps.
* **Advanced Analytics**: Explore more sophisticated analytics in Snowflake,such as clustering or trend analysis.
* **More Data Sources**: Integrate data from other Spotify API endpoints.
* **Dashboard Enhancements**: Add more visualizations and filters to the Power BI dashboard for deeper insights.

