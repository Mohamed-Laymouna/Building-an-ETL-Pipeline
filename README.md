# Energy Sector ETL Pipeline

## Project Overview

This project demonstrates a foundational ETL (Extract, Transform, Load) pipeline built to process energy sector data for an energy company. The primary goal is to automate the previously manual and time-consuming process of retrieving and cleaning quarterly data, transitioning it to an efficient monthly pipeline. This automation provides analysts with more rapid, timely insights into electricity sales and generation capabilities.

The pipeline ingests data from two distinct sources: a CSV file containing electricity sales information and a nested JSON file with electricity capability data. It then cleans, reshapes, and filters the data according to business requirements, and finally loads the processed data into clean, analysis-ready formats (CSV and Parquet).

This project showcases proficiency in core data engineering concepts, including data ingestion from multiple file types, data transformation and cleaning with Pandas, and data loading into different storage formats.

## Data Sources

The pipeline processes data from two raw files:

1.  **`electricity_sales.csv`**: A tabular dataset containing electricity sales records with the following schema:
    | Field | Data Type | Description |
    | :--- | :---: | :--- |
    | `period` | `str` | The time period of the record (e.g., `202201`). |
    | `stateid` | `str` | The two-letter state abbreviation (e.g., `CA`). |
    | `stateDescription` | `str` | The full name of the state. |
    | `sectorid` | `str` | An identifier for the energy sector. |
    | `sectorName` | `str` | The name of the energy sector (e.g., `residential`). |
    | `price` | `float` | The price of electricity. |
    | `price-units` | `str` | The units for the price (e.g., `cents per kilowatthour`). |

2.  **`electricity_capability_nested.json`**: A nested JSON file containing data about electricity generation capabilities.

## ETL Pipeline Architecture

The pipeline is built with modular Python functions, each responsible for a specific stage of the ETL process.

!ETL Pipeline Diagram <!-- You can replace image.png with a diagram you create -->

### 1. Extract

-   **Tabular Data (`.csv`)**: A function reads the `electricity_sales.csv` file into a Pandas DataFrame. It is designed to be extensible to other tabular formats like Parquet.
-   **JSON Data (`.json`)**: A separate function ingests the `electricity_capability_nested.json` file. It leverages `pandas.json_normalize()` to effectively flatten the nested structure into a tabular DataFrame, making it immediately usable for analysis.

### 2. Transform

The transformation logic is applied specifically to the electricity sales data to meet the requirements of the downstream analysts:

-   **Data Cleaning**: Records with missing `price` values are dropped to ensure data quality.
-   **Filtering**: The dataset is filtered to retain only records relevant to the **"residential"** and **"transportation"** sectors.
-   **Feature Engineering**: The `period` column (e.g., `202201`) is split into two new columns, `year` and `month`, for easier time-series analysis and aggregation.
-   **Schema Selection**: The final DataFrame is trimmed to include only the essential columns: `year`, `month`, `stateid`, `price`, and `price-units`.

### 3. Load

The processed DataFrames are loaded into new files, ready for consumption by data analysts or other downstream processes:

-   The transformed electricity sales data is saved as **`loaded__electricity_sales.csv`**.
-   The flattened electricity capability data is saved as **`loaded__electricity_capability.parquet`**. Parquet is chosen here for its efficient compression and columnar storage benefits, which is ideal for larger datasets.

## Technologies Used

-   **Language**: Python 3
-   **Library**: Pandas for data manipulation and analysis.
-   **File Formats**: CSV, JSON, Parquet

## How to Run the Project

1.  **Prerequisites**:
    -   Ensure you have Python 3.8+ installed.
    -   Install the required libraries:
        ```bash
        pip install pandas pyarrow
        ```
        *(Note: `pyarrow` is required for writing to Parquet files).*

2.  **Project Structure**:
    Ensure your project directory is structured as follows:
    ```
    .
    ├── data/
    │   ├── electricity_sales.csv
    │   └── electricity_capability_nested.json
    ├── output/
    ├── notebook.ipynb
    └── README.md
    ```
    *(You may need to adjust file paths in the notebook to point to `data/` and `output/` directories).*

3.  **Execution**:
    Run the cells in the `notebook.ipynb` Jupyter Notebook sequentially. Upon successful execution, the processed files will be generated in the specified output location (e.g., `loaded__electricity_sales.csv` and `loaded__electricity_capability.parquet`).

## Future Improvements

-   **Orchestration**: Integrate an orchestrator like Apache Airflow or Mage to schedule and monitor the pipeline runs automatically.
-   **Scalability**: For larger datasets, refactor the pipeline to use a distributed computing framework like Apache Spark.
-   **Data Quality**: Implement a data quality framework like Great Expectations to add automated data validation checks at each step of the pipeline.
-   **Configuration Management**: Externalize configurations (e.g., file paths, filter conditions) into a separate `config.yaml` file to avoid hardcoding values.
-   **Containerization**: Dockerize the application to ensure a consistent and reproducible runtime environment.
