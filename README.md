### Data Engineering Capstone Project

#### Project Summary
The project joins multiple datasets (I94 Immigration Data, U.S. City Demographic Data and Airport Codes) into one data warehouse with a star schema in order to make it easier for further analysis.

The project follows the follow steps:
* Step 1: Scope the Project and Gather Data
* Step 2: Explore and Assess the Data
* Step 3: Define the Data Model
* Step 4: Run ETL to Model the Data
* Step 5: Complete Project Write Up


### Step 1: Scope the Project and Gather Data

#### Scope 
Explain what you plan to do in the project in more detail. What data do you use? What is your end solution look like? What tools did you use? etc>

The project utilizes the data coming from variuos sources -listed below- to establish a data warehouse that can be used in analytics and collecting insights and metrics.

Datasets:
1. I94 Immigration Data.
2. U.S. City Demographic Data.
3. Airport Code Table

Tools used:
1. Pandas Libraries, to manipulate data.
2. PySpark, to process data in large scale efficintly. 
3. SqlAlchemy, to send data to AWS Redshift.
4. psycopg2, to query AWS Redshift.
5. AWS Redshift, to store the data warehouse.

#### Describe and Gather Data 
Describe the data sets you're using. Where did it come from? What type of information is included?

##### 1. I94 Immigration Data:
    
    Contains info about immigrants; arrive date, departure date, type of visa, city code and other data, provided by US National Tourism and Trade Office, the file type is CSV.


##### 2. U.S. City Demographic Data:
    
    Contains info about the demographics of all US cities and census-designated places with a population greater or equal to 65,000, provided by the US Census Bureau's 2015 American Community Survey, the file type is CSV.

##### 3. Airport Code Table:
    
    Contains a simple table of airport codes and corresponding cities, provided by DataHub, the file type is CSV.

### Step 2: Explore and Assess the Data
#### Explore the Data 
Identify data quality issues, like missing values, duplicate data, etc.

#### Cleaning Steps
Document steps necessary to clean the data

1. Removing rows with Null values on all columns.
2. Removing Duplicated rows.
3. Transforming Dates (arrdate, depdate) from Immigration table.
4. Parsing SAS file to get Country_Code, City_Code and State_Code.
5. Change letters case of "City" and "State" columns in "Demographics" table to upper case to match the data from SAS file.
6. Remove the first 3 charchters from "iso_region" column in "Airport_Codes" table that represent the country code, to have the state code left to match the data from SAS file.
7. Fill NaN values with 0.


### Step 3: Define the Data Model
#### 3.1 Conceptual Data Model
Map out the conceptual data model and explain why you chose that model

The data model uses star schema (Kimball model) to use the output in Analytics.

##### Fact Table:
| fact_immigration |
| ---      |
|cic_id|
|year|
|month|
|city_code|
|state_code|
|arrive_date|
|departure_date|
|mode|
|visa|

##### Dimension Tables:
| dim_immigration_citizen |
| ---      |
|cic_id|
|citizen_country|
|residence_country|
|birth_year|
|gender|
|ins_num|

| dim_dmg_population |
| ---      |
|city_code|
|male_pop|
|female_pop|
|num_vetarans|
|foreign_born|
|race|

| dim_dmg_statistics |
| ---      |
|city_code|
|median_age|
|avg_household_size|

| dim_airport_code |
| ---      |
|state_code|
|type|
|name|
|coordinates|

| dim_country_code |
|--------------|
| country_code  |
| country      |

| dim_state_code |
|------------|
| state_code |
| state      |

| dim_city_code |
|-----------|
| city_code |
| city      |
| state_code |

#### 3.2 Mapping Out Data Pipelines
List the steps necessary to pipeline the data into the chosen data model

1. Reading the data from the sources.
2. Cleaning the raw data.
3. Tranforming the data (Removing Nulls and Duplicates, transforming Dates, Parsing SAS file, etc.).
4. Creating the tables according to the star schema data model.
    1. Creating the fact and dimension dataframes out of the original dataframes.
    2. Changing the column names to a more understandable names.
    3. Join "Demographics" table with "City_Code" table to add "city_code" column to "Demographics".
6. Filling the AWS Redshift cluster DBtables with the transformed data.

#### 4.2 Data Quality Checks
Explain the data quality checks you'll perform to ensure the pipeline ran as expected. These could include:
 * Integrity constraints on the relational database (e.g., unique key, data type, etc.)
 * Unit tests for the scripts to ensure they are doing the right thing
 * Source/Count checks to ensure completeness
 
Run Quality Checks

#### 4.3 Data dictionary 
Create a data dictionary for your data model. For each field, provide a brief description of what the data is and where it came from. You can include the data dictionary in the notebook or in a separate file.

| fact_immigration | Type      |
|------------------|-----------|
| cic_id           | BIGINT    |
| year             | INT       |
| month            | INT       |
| city_code        | CHAR(3)   |
| state_code       | CHAR(2)   |
| mode             | INT       |
| visa             | INT       |
| arrive_date      | TIMESTAMP |
| departure_date   | TIMESTAMP |

| dim_immigrationg_citizen  | Type    |
|---------------------------|---------|
| cic_id                    | BIGINT  |
| citizen_country           | INT     |
| residence_country         | INT     |
| birth_year                | INT     |
| gender                    | CHAR(1) |
| ins_num                   | INT     |


| dim_demographics_population | Type    |
|-----------------------------|---------|
| city_code                   | CHAR(3) |
| male_population             | INT     |
| famale_population           | INT     |
| num_veterans                | INT     |
| foreign_born                | INT     |
| race                        | VARCHAR |

| dim_demographics_statistics | Type     |
|-----------------------------|----------|
| city_code                   | CHAR(3)  |
| median_age                  | INT      |
| avg_household_size          | FLOAT    |

| dim_airport_code       | Type     |
|------------------------|----------|
| state_code             | CHAR(2)  |
| type                   | VARCHAR  |
| name                   | VARCHAR  |
| coordinates            | VARCHAR  |

| dim_country_code | Type    |
|--------------|-------------|
| country_id   | INT         |
| country      | VARCHAR     |

| dim_city_code  | Type        |
|----------------|-------------|
| city_code      | CHAR(2)     |
| city           | VARCHAR     |
| state_code     | CHAR(2)     |

| dim_state_code | Type        |
|----------------|-------------|
| state_code     | CHAR(3)     |
| state          | VARCHAR     |

#### Step 5: Complete Project Write Up
#### * Clearly state the rationale for the choice of tools and technologies for the project.
    1. Pandas is used to manipulate the data in its easy-to-use dataframe
    2. AWS Redshift is used to hold the data in a data warehose that is distributed and widly accessable.
#### * Propose how often the data should be updated and why.
    1. Data that comes from Immigration dataset should be updated monthly.
    2. Data that comes from Demographics dataset should be updated annually.
    3. Data about Citys, States, and Countries should be updated on demand.
    4. Data about Airports codes should be updated on demand.
#### * Write a description of how you would approach the problem differently under the following scenarios:
#### * The data was increased by 100x.
     1. Apache Spark will be used instead of Pandas libiraries to leverage the advantages of distributed proccessing.
     2. AWS EMR will be used to easily manage the Apache Spark cluster.
#### * The data populates a dashboard that must be updated on a daily basis by 7am every day.
     1. Apache Airflow will be used to schedule the run of the pipline.
#### * The database needed to be accessed by 100+ people.
     1. AWS Redshift database can handle up to 500 connections simultaneously, to handle more than 500 connections, using another Airflow pipline to duplicate the database periodically to work as a load balancer would be a suggested solution.