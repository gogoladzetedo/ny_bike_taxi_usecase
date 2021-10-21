# NY bike rental and taxi usecase

### Business Need
This is a scenario where the data engineering solution is provided to the bike rental company. Having the analytical database with the information about the bike trips, taxi trips and the weather can help the decision makers analyze
* When is their service more/less popular
* In which areas are their service more/less popular
* When and how often do the customer take longer or shorter rides
* Comparison of their service vs the alternative transport service (Taxi vs Bike) demand
* Do the weather conditions impact on the demand and how
* Do the weather conditions impact choosing the alternative transport (Taxi over Bike)
* , etc.

### Datasets
Datasets used in this project are publicly available. NYC bike and trips data is available on [BigQuery Public Datasets](https://cloud.google.com/bigquery/public-data) *`bigquery-public-data.new_york_citibike.citibike_trips`, `bigquery-public-data.new_york_taxi_trips.tlc_green_trips_2018`*, and the weather history data is accessible from world [weather online website](https://www.worldweatheronline.com/developer/api/historical-weather-api.aspx).
Here is the figure of the available datasett attributes:

![datasets](/images/model2.png)

### Data Pipeline Architecture
Architecture of the data pipeline is given on the figure below. 

![data pipeline architecture diagram](/images/model.png)

### Data Pipeline Steps
* For the completion of the task, Python environment is used. Analysis of the data is done in Jupyter Notebook for the better presentation, file [data_pipeline.ipynb](https://github.com/gogoladzetedo/ny_data_pipeline/blob/main/data_pipeline.ipynb) is available in this repo.
* Data extraction from the BigQuery datasets are done using Python BigQuery and BigQuey Storage client libraries. Here, Google Cloud account with the IAM user is needed that has BigQuery Read Access. For the world weather API the trial account is sufficient that gives acess on 500 requests during 60 days. 
* Extracted data is moved to Pandas dataframes and then stored into the Amazon S3 storage bucket for the landing zone purpose. Project uses AWS account, awswrangler library for storing the data to S3, and connecting to AWS with boto library using the AWS access key and secret.
* The database solution here is AWS RDS for PostgreSQL. The solution includes database objects creation scripts. There are two schemas - staging with three tables respective to the data sources, and dw with the proposed dimension and facts tables. DW model here is Star schema, although it contains just a few tables to properly implement this particular model. Naturally, all the SQL script are run on PostgreSQL from Python environment via psycopg2 library.
* After the database objects are on place, landing zone files data are moved into the database staging area. As a typical staging area, this solution also takes all the files with the same structure, giving the varchar data types to all of the fields. Additionally, the ingestion time and unique identifier of the row is added into the staging area. Data loading from AWS S3 to AWS RDS PostgreSQL is done with AWS-s native COPY command implementation [aws_s3.table_import_from_s3](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/PostgreSQL.Procedural.Importing.html). 
* Next step is calculating the dimensions and facts. Here, the following data model is used: ![data model](/images/model3.png) . Idea of the Date dimension is that it can be easily analyzed in the context of the different date context. Bike station dimensions are slowly changing, therefore, it has the respective model and the data calculation SQL script also takes that into account. Facts tables refer to those dimension ids and calculate aggregates of the trips data on an hourly basis, while the weather data is already present in an hourly basis, so the aggregations are not needed there.


### What's to be done next
* **Data Model Extension:** In the task the solution includes the data pipeline from sourcing till the dimensional data model. For a more complete architecture, it could have a presentation layer / data mart on top of the data warehouse. Presentation Layer would calculate the metrics defined by the business and it would be visualized in the BI software. Alternatively, BI software can query the needed metric calculation logic from the existing dimensional data.
* **Solution Code**: The solution is done in Jupyter Notebook for a better presentability. Proper solution would be creating the Python solution, splitting the solution to the modules, creating the SQL scripts as the PostgreSQL procedures and calling them from python.
* **Automation** - creating the proper data flow with the correct dependency - ideally using airflow, modifying the data ingestion and calculation scripts to be date-specific executable as now they are static. Creating the resource manager templates for the automatic creation of the cloud solutions used here - RDS database, S3 bucket.
* **Data Quality Checks** - at this point, only basic data quality checks are performed, Need more robust checks on them to get proper filters.
