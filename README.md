Data Engineering Capstone Project

Project Summary

The project joins multiple datasets (I94 Immigration Data, U.S. City
Demographic Data and Airport Codes) into one data warehouse with a star
schema in order to make it easier for further analysis.

The project follows the follow steps:

-   Step 1: Scope the Project and Gather Data
-   Step 2: Explore and Assess the Data
-   Step 3: Define the Data Model
-   Step 4: Run ETL to Model the Data
-   Step 5: Complete Project Write Up

    # Do all imports and installs here
    import pandas as pd
    from pyspark.sql.types import StructType, StructField, StringType, IntegerType, FloatType, DateType
    from pyspark.sql.functions import substring, length, col, expr
    from sqlalchemy import create_engine
    import configparser
    import psycopg2

Step 1: Scope the Project and Gather Data

Scope

Explain what you plan to do in the project in more detail. What data do
you use? What is your end solution look like? What tools did you use?
etc>

The project utilizes the data coming from variuos sources -listed below-
to establish a data warehouse that can be used in analytics and
collecting insights and metrics.

Datasets:

1.  I94 Immigration Data.
2.  U.S. City Demographic Data.
3.  Airport Code Table

Tools used:

1.  Pandas Libraries, to manipulate data.
2.  PySpark, to process data in large scale efficintly.
3.  SqlAlchemy, to send data to AWS Redshift.
4.  psycopg2, to query AWS Redshift.
5.  AWS Redshift, to store the data warehouse.

Describe and Gather Data

Describe the data sets you're using. Where did it come from? What type
of information is included?

1. I94 Immigration Data:

    Contains info about immigrants; arrive date, departure date, type of visa, city code and other data, provided by US National Tourism and Trade Office, the file type is CSV.

2. U.S. City Demographic Data:

    Contains info about the demographics of all US cities and census-designated places with a population greater or equal to 65,000, provided by the US Census Bureau's 2015 American Community Survey, the file type is CSV.

3. Airport Code Table:

    Contains a simple table of airport codes and corresponding cities, provided by DataHub, the file type is CSV.

    # Read in the data here
    df_img = pd.read_csv("immigration_data_sample.csv")
    df_img.head(5)

       Unnamed: 0      cicid   i94yr  i94mon  i94cit  i94res i94port  arrdate  \
    0     2027561  4084316.0  2016.0     4.0   209.0   209.0     HHW  20566.0   
    1     2171295  4422636.0  2016.0     4.0   582.0   582.0     MCA  20567.0   
    2      589494  1195600.0  2016.0     4.0   148.0   112.0     OGG  20551.0   
    3     2631158  5291768.0  2016.0     4.0   297.0   297.0     LOS  20572.0   
    4     3032257   985523.0  2016.0     4.0   111.0   111.0     CHM  20550.0   

       i94mode i94addr    ...     entdepu  matflag  biryear   dtaddto  gender  \
    0      1.0      HI    ...         NaN        M   1955.0  07202016       F   
    1      1.0      TX    ...         NaN        M   1990.0  10222016       M   
    2      1.0      FL    ...         NaN        M   1940.0  07052016       M   
    3      1.0      CA    ...         NaN        M   1991.0  10272016       M   
    4      3.0      NY    ...         NaN        M   1997.0  07042016       F   

      insnum airline        admnum  fltno  visatype  
    0    NaN      JL  5.658267e+10  00782        WT  
    1    NaN     *GA  9.436200e+10  XBLNG        B2  
    2    NaN      LH  5.578047e+10  00464        WT  
    3    NaN      QR  9.478970e+10  00739        B2  
    4    NaN     NaN  4.232257e+10   LAND        WT  

    [5 rows x 29 columns]

    df_dmg = pd.read_csv("us-cities-demographics.csv", delimiter=";")
    df_dmg.head(5)

                   City          State  Median Age  Male Population  \
    0     Silver Spring       Maryland        33.8          40601.0   
    1            Quincy  Massachusetts        41.0          44129.0   
    2            Hoover        Alabama        38.5          38040.0   
    3  Rancho Cucamonga     California        34.5          88127.0   
    4            Newark     New Jersey        34.6         138040.0   

       Female Population  Total Population  Number of Veterans  Foreign-born  \
    0            41862.0             82463              1562.0       30908.0   
    1            49500.0             93629              4147.0       32935.0   
    2            46799.0             84839              4819.0        8229.0   
    3            87105.0            175232              5821.0       33878.0   
    4           143873.0            281913              5829.0       86253.0   

       Average Household Size State Code                       Race  Count  
    0                    2.60         MD         Hispanic or Latino  25924  
    1                    2.39         MA                      White  58723  
    2                    2.58         AL                      Asian   4759  
    3                    3.18         CA  Black or African-American  24437  
    4                    2.73         NJ                      White  76402  

    df_apt = pd.read_csv("airport-codes_csv.csv")
    df_apt.head(5)

      ident           type                                name  elevation_ft  \
    0   00A       heliport                   Total Rf Heliport          11.0   
    1  00AA  small_airport                Aero B Ranch Airport        3435.0   
    2  00AK  small_airport                        Lowell Field         450.0   
    3  00AL  small_airport                        Epps Airpark         820.0   
    4  00AR         closed  Newport Hospital & Clinic Heliport         237.0   

      continent iso_country iso_region  municipality gps_code iata_code  \
    0       NaN          US      US-PA      Bensalem      00A       NaN   
    1       NaN          US      US-KS         Leoti     00AA       NaN   
    2       NaN          US      US-AK  Anchor Point     00AK       NaN   
    3       NaN          US      US-AL       Harvest     00AL       NaN   
    4       NaN          US      US-AR       Newport      NaN       NaN   

      local_code                            coordinates  
    0        00A     -74.93360137939453, 40.07080078125  
    1       00AA                 -101.473911, 38.704022  
    2       00AK            -151.695999146, 59.94919968  
    3       00AL  -86.77030181884766, 34.86479949951172  
    4        NaN                    -91.254898, 35.6087  

    from pyspark.sql import SparkSession

    spark = SparkSession.builder.\
    config("spark.jars.repositories", "https://repos.spark-packages.org/").\
    config("spark.jars.packages", "saurfang:spark-sas7bdat:2.0.0-s_2.11").\
    enableHiveSupport().getOrCreate()

    df_spark = spark.read.format('com.github.saurfang.sas.spark').load('../../data/18-83510-I94-Data-2016/i94_apr16_sub.sas7bdat')

    #write to parquet
    #df_spark.write.parquet("sas_data")
    df_spark=spark.read.parquet("sas_data")

Step 2: Explore and Assess the Data

Explore the Data

Identify data quality issues, like missing values, duplicate data, etc.

Cleaning Steps

Document steps necessary to clean the data

1.  Removing rows with Null values on all columns.
2.  Removing Duplicated rows.
3.  Transforming Dates (arrdate, depdate) from Immigration table.
4.  Parsing SAS file to get Country_Code, City_Code and State_Code.
5.  Change letters case of "City" and "State" columns in "Demographics"
    table to upper case to match the data from SAS file.
6.  Remove the first 3 charchters from "iso_region" column in
    "Airport_Codes" table that represent the country code, to have the
    state code left to match the data from SAS file.
7.  Fill NaN values with 0.

    # Performing cleaning tasks here
    # Step 1 and 2
    # Immigration Table
    df_img = df_img.dropna(how='all').drop_duplicates()

    # Demographics Table
    df_dmg = df_dmg.dropna(how='all').drop_duplicates()

    # Airport Codes Table
    df_apt = df_apt.dropna(how='all').drop_duplicates()

    # 3- Transform Date
    df_img['arrdate'] = pd.to_timedelta(df_img['arrdate'], unit='D') + pd.Timestamp('1960-1-1')
    df_img['depdate'] = pd.to_timedelta(df_img['depdate'], unit='D') + pd.Timestamp('1960-1-1')
    df_img.head(5)

       Unnamed: 0      cicid   i94yr  i94mon  i94cit  i94res i94port    arrdate  \
    0     2027561  4084316.0  2016.0     4.0   209.0   209.0     HHW 2016-04-22   
    1     2171295  4422636.0  2016.0     4.0   582.0   582.0     MCA 2016-04-23   
    2      589494  1195600.0  2016.0     4.0   148.0   112.0     OGG 2016-04-07   
    3     2631158  5291768.0  2016.0     4.0   297.0   297.0     LOS 2016-04-28   
    4     3032257   985523.0  2016.0     4.0   111.0   111.0     CHM 2016-04-06   

       i94mode i94addr    ...    entdepu  matflag  biryear   dtaddto  gender  \
    0      1.0      HI    ...        NaN        M   1955.0  07202016       F   
    1      1.0      TX    ...        NaN        M   1990.0  10222016       M   
    2      1.0      FL    ...        NaN        M   1940.0  07052016       M   
    3      1.0      CA    ...        NaN        M   1991.0  10272016       M   
    4      3.0      NY    ...        NaN        M   1997.0  07042016       F   

      insnum airline        admnum  fltno  visatype  
    0    NaN      JL  5.658267e+10  00782        WT  
    1    NaN     *GA  9.436200e+10  XBLNG        B2  
    2    NaN      LH  5.578047e+10  00464        WT  
    3    NaN      QR  9.478970e+10  00739        B2  
    4    NaN     NaN  4.232257e+10   LAND        WT  

    [5 rows x 29 columns]

    # 4- Parse SAS
    with open("I94_SAS_Labels_Descriptions.SAS") as file:
        SAS = file.readlines()
        
    country_code = {}
    for countries in SAS[10:298]:
        pair = countries.split('=')
        code, country = pair[0].strip(), pair[1].strip().strip("'")
        country_code[code] = country
        
    city_code = {}
    for cities in SAS[303:962]:
        pair = cities.split('=')
        code, city = pair[0].strip("\t").strip().strip("'"), pair[1].strip('\t').strip().strip("''")
        city_code[code] = city
        
    state_code = {}
    for states in SAS[982:1036]:
        pair = states.split('=')
        code, state = pair[0].strip('\t').strip("'"), pair[1].strip().strip("'")
        state_code[code] = state

    df_country_code = pd.DataFrame(list(country_code.items()), columns=['code', 'country'])
    df_country_code.head(5)

      code      country
    0  236  AFGHANISTAN
    1  101      ALBANIA
    2  316      ALGERIA
    3  102      ANDORRA
    4  324       ANGOLA

    df_city_code = pd.DataFrame(list(city_code.items()), columns=['code', 'city'])
    df_city_code.head(5)

      code                          city
    0  ANC        ANCHORAGE, AK         
    1  BAR  BAKER AAF - BAKER ISLAND, AK
    2  DAC        DALTONS CACHE, AK     
    3  PIZ    DEW STATION PT LAY DEW, AK
    4  DTH        DUTCH HARBOR, AK      

    df_state_code = pd.DataFrame(list(state_code.items()), columns=['code', 'state'])
    df_state_code.head(5)

      code       state
    0   AK      ALASKA
    1   AZ     ARIZONA
    2   AR    ARKANSAS
    3   CA  CALIFORNIA
    4   CO    COLORADO

    # 5- Change letters case in Demographics table
    df_dmg['City'] = df_dmg['City'].str.upper()
    df_dmg['State'] = df_dmg['State'].str.upper()
    df_dmg.head(5)

                   City          State  Median Age  Male Population  \
    0     SILVER SPRING       MARYLAND        33.8          40601.0   
    1            QUINCY  MASSACHUSETTS        41.0          44129.0   
    2            HOOVER        ALABAMA        38.5          38040.0   
    3  RANCHO CUCAMONGA     CALIFORNIA        34.5          88127.0   
    4            NEWARK     NEW JERSEY        34.6         138040.0   

       Female Population  Total Population  Number of Veterans  Foreign-born  \
    0            41862.0             82463              1562.0       30908.0   
    1            49500.0             93629              4147.0       32935.0   
    2            46799.0             84839              4819.0        8229.0   
    3            87105.0            175232              5821.0       33878.0   
    4           143873.0            281913              5829.0       86253.0   

       Average Household Size State Code                       Race  Count  
    0                    2.60         MD         Hispanic or Latino  25924  
    1                    2.39         MA                      White  58723  
    2                    2.58         AL                      Asian   4759  
    3                    3.18         CA  Black or African-American  24437  
    4                    2.73         NJ                      White  76402  

    # 6- Remove the country code from "iso_region" col. in Airport_Code table
    df_apt['iso_region'] = df_apt['iso_region'].str.slice(start = 3)
    df_apt.rename(columns = {'iso_region' : 'state_code'}, inplace = True)
    df_apt.head(5)

      ident           type                                name  elevation_ft  \
    0   00A       heliport                   Total Rf Heliport          11.0   
    1  00AA  small_airport                Aero B Ranch Airport        3435.0   
    2  00AK  small_airport                        Lowell Field         450.0   
    3  00AL  small_airport                        Epps Airpark         820.0   
    4  00AR         closed  Newport Hospital & Clinic Heliport         237.0   

      continent iso_country state_code  municipality gps_code iata_code  \
    0       NaN          US         PA      Bensalem      00A       NaN   
    1       NaN          US         KS         Leoti     00AA       NaN   
    2       NaN          US         AK  Anchor Point     00AK       NaN   
    3       NaN          US         AL       Harvest     00AL       NaN   
    4       NaN          US         AR       Newport      NaN       NaN   

      local_code                            coordinates  
    0        00A     -74.93360137939453, 40.07080078125  
    1       00AA                 -101.473911, 38.704022  
    2       00AK            -151.695999146, 59.94919968  
    3       00AL  -86.77030181884766, 34.86479949951172  
    4        NaN                    -91.254898, 35.6087  

    # 7-Fill NaN values with zero
    df_img['insnum'] = df_img['insnum'].fillna(0)
    df_img.head(5)

       Unnamed: 0      cicid   i94yr  i94mon  i94cit  i94res i94port    arrdate  \
    0     2027561  4084316.0  2016.0     4.0   209.0   209.0     HHW 2016-04-22   
    1     2171295  4422636.0  2016.0     4.0   582.0   582.0     MCA 2016-04-23   
    2      589494  1195600.0  2016.0     4.0   148.0   112.0     OGG 2016-04-07   
    3     2631158  5291768.0  2016.0     4.0   297.0   297.0     LOS 2016-04-28   
    4     3032257   985523.0  2016.0     4.0   111.0   111.0     CHM 2016-04-06   

       i94mode i94addr    ...    entdepu  matflag  biryear   dtaddto  gender  \
    0      1.0      HI    ...        NaN        M   1955.0  07202016       F   
    1      1.0      TX    ...        NaN        M   1990.0  10222016       M   
    2      1.0      FL    ...        NaN        M   1940.0  07052016       M   
    3      1.0      CA    ...        NaN        M   1991.0  10272016       M   
    4      3.0      NY    ...        NaN        M   1997.0  07042016       F   

      insnum airline        admnum  fltno  visatype  
    0    0.0      JL  5.658267e+10  00782        WT  
    1    0.0     *GA  9.436200e+10  XBLNG        B2  
    2    0.0      LH  5.578047e+10  00464        WT  
    3    0.0      QR  9.478970e+10  00739        B2  
    4    0.0     NaN  4.232257e+10   LAND        WT  

    [5 rows x 29 columns]

    df_dmg['Male Population'] = df_dmg['Male Population'].fillna(0)
    df_dmg['Female Population'] = df_dmg['Female Population'].fillna(0)
    df_dmg['Number of Veterans'] = df_dmg['Number of Veterans'].fillna(0)
    df_dmg['Foreign-born'] = df_dmg['Foreign-born'].fillna(0)
    df_dmg.head(5)

                   City          State  Median Age  Male Population  \
    0     SILVER SPRING       MARYLAND        33.8          40601.0   
    1            QUINCY  MASSACHUSETTS        41.0          44129.0   
    2            HOOVER        ALABAMA        38.5          38040.0   
    3  RANCHO CUCAMONGA     CALIFORNIA        34.5          88127.0   
    4            NEWARK     NEW JERSEY        34.6         138040.0   

       Female Population  Total Population  Number of Veterans  Foreign-born  \
    0            41862.0             82463              1562.0       30908.0   
    1            49500.0             93629              4147.0       32935.0   
    2            46799.0             84839              4819.0        8229.0   
    3            87105.0            175232              5821.0       33878.0   
    4           143873.0            281913              5829.0       86253.0   

       Average Household Size State Code                       Race  Count  
    0                    2.60         MD         Hispanic or Latino  25924  
    1                    2.39         MA                      White  58723  
    2                    2.58         AL                      Asian   4759  
    3                    3.18         CA  Black or African-American  24437  
    4                    2.73         NJ                      White  76402  

Step 3: Define the Data Model

3.1 Conceptual Data Model

Map out the conceptual data model and explain why you chose that model

The data model uses star schema (Kimball model) to use the output in
Analytics.

Fact Table:

  fact_immigration
  ------------------
  cic_id
  year
  month
  city_code
  state_code
  arrive_date
  departure_date
  mode
  visa

Dimension Tables:

  dim_immigration_citizen
  -------------------------
  cic_id
  citizen_country
  residence_country
  birth_year
  gender
  ins_num

  dim_dmg_population
  --------------------
  city_code
  male_pop
  female_pop
  num_vetarans
  foreign_born
  race

  dim_dmg_statistics
  --------------------
  city_code
  median_age
  avg_household_size

  dim_airport_code
  ------------------
  state_code
  type
  name
  coordinates

  dim_country_code
  ------------------
  country_code
  country

  dim_state_code
  ----------------
  state_code
  state

  dim_city_code
  ---------------
  city_code
  city
  state_code

3.2 Mapping Out Data Pipelines

List the steps necessary to pipeline the data into the chosen data model

1.  Reading the data from the sources.
2.  Cleaning the raw data.
3.  Tranforming the data (Removing Nulls and Duplicates, transforming
    Dates, Parsing SAS file, etc.).
4.  Creating the tables according to the star schema data model.
    1.  Creating the fact and dimension dataframes out of the original
        dataframes.
    2.  Changing the column names to a more understandable names.
    3.  Join "Demographics" table with "City_Code" table to add
        "city_code" column to "Demographics".
5.  Filling the AWS Redshift cluster DBtables with the transformed data.

Step 4: Run Pipelines to Model the Data

4.1 Create the data model

Build the data pipelines to create the data model.

    # Configuring AWS Redshift
    config = configparser.ConfigParser()
    config.read('dwh.cfg')

    ['dwh.cfg']

    conn = create_engine('postgresql://{}:{}@{}'\
    .format(config['CLUSTER']['DB_USER'], config['CLUSTER']['DB_PASSWORD'], config['CLUSTER']['DB_URL']))

    #Steps 4.A, 4.B
    #fact_immigration Table
    fact_img = df_img[['cicid', 'i94yr', 'i94mon', 'i94port', 'i94addr', 'arrdate', 'depdate', 'i94mode', 'visatype']]
    fact_img = fact_img.astype({'cicid':'int', 'i94yr':'int', 'i94mon':'int', 'i94mode':'int'})
    fact_img.columns = ['cicid', 'year', 'month', 'city_code', 'state_code', 'arrive_date', 'departure_date', 'mode', 'visa']
    fact_img.to_sql('fact_immigration', conn, index=False, if_exists='replace')

    #dim_img_citizen
    dim_img_citizen = df_img[['cicid', 'i94cit', 'i94res', 'biryear', 'gender', 'insnum']]
    dim_img_citizen = dim_img_citizen.astype({'cicid':'int', 'i94cit':'int', 'i94res':'int', 'biryear':'int', 'insnum':'int'})
    dim_img_citizen.columns = ['cicid', 'citizen_country', 'residence_country', 'birth_year', 'gender', 'ins_num']
    dim_img_citizen.to_sql('dim_immigration_citizen', conn, index=False, if_exists='replace')

    #dim_dmg_popultion
    dim_dmg_pop = df_dmg[['City', 'State', 'Male Population', 'Female Population', 'Number of Veterans', 'Foreign-born', 'Race']]
    dim_dmg_pop = dim_dmg_pop.astype({'Male Population':'int', 'Female Population':'int', 'Number of Veterans':'int', 'Foreign-born':'int'})
    dim_dmg_pop.columns = ['city', 'state', 'male_pop', 'female_pop', 'num_vetarans', 'foreign_born', 'race']
    dim_dmg_pop = dim_dmg_pop.merge(df_city_code, on='city', how='outer')
    dim_dmg_pop.to_sql('dim_demographics_population', conn, index=False, if_exists='replace')

    #dim_dmg_statistics
    dim_dmg_stat = df_dmg[['City', 'State', 'Median Age', 'Average Household Size']]
    dim_dmg_stat.columns = ['city', 'state', 'median_age', 'avg_household_size']
    dim_dmg_stat = dim_dmg_stat.merge(df_city_code, on='city', how='outer')
    dim_dmg_stat.to_sql('dim_demographics_statistics', conn, index=False, if_exists='replace')

    #dim_airport_code
    dim_airport = df_apt[['state_code', 'type', 'name', 'coordinates']]
    dim_airport.to_sql('dim_airport_code', conn, index=False, if_exists='replace')

    #dim_country_code
    dim_country = df_country_code[['code', 'country']]
    dim_country = dim_country.astype({'code':'int'})
    dim_country.to_sql('dim_country_code', conn, index=False, if_exists='replace')

    #dim_state_code
    dim_state = df_state_code[['code', 'state']]
    dim_state.to_sql('dim_state_code', conn, index=False, if_exists='replace')

    #dim_city_code
    dim_city = df_city_code[['code', 'city']]
    dim_city['state_code'] = dim_city['city'].str.split(',').str[1]
    dim_city['city'] = dim_city['city'].str.split(',').str[0]
    dim_city.head()
    dim_city.to_sql('dim_city_code', conn, index=False, if_exists='replace')

4.2 Data Quality Checks

Explain the data quality checks you'll perform to ensure the pipeline
ran as expected. These could include:

-   Integrity constraints on the relational database (e.g., unique key,
    data type, etc.)
-   Unit tests for the scripts to ensure they are doing the right thing
-   Source/Count checks to ensure completeness

Run Quality Checks

1.  Check if tables match the data model.
2.  Redshift tables are not empty.

    connec = psycopg2.connect("host={} dbname={} user={} password={} port={}".format(*config['CLUSTER'].values()))
    cur = connec.cursor()

    #1. Check if tables match the data model.
    # fact_immigration
    cur.execute("SELECT table_name, column_name, data_type FROM information_schema.columns WHERE table_name = 'fact_immigration' ")
    cur.fetchall()

    [('fact_immigration', 'departure_date', 'timestamp without time zone'),
     ('fact_immigration', 'arrive_date', 'timestamp without time zone'),
     ('fact_immigration', 'visa', 'character varying'),
     ('fact_immigration', 'state_code', 'character varying'),
     ('fact_immigration', 'city_code', 'character varying'),
     ('fact_immigration', 'mode', 'bigint'),
     ('fact_immigration', 'month', 'bigint'),
     ('fact_immigration', 'year', 'bigint'),
     ('fact_immigration', 'cic_id', 'bigint')]

    # dim_immigration_citizen
    cur.execute("SELECT table_name, column_name, data_type FROM information_schema.columns WHERE table_name = 'dim_immigration_citizen' ")
    cur.fetchall()

    # dim_demographics_population
    cur.execute("SELECT table_name, column_name, data_type FROM information_schema.columns WHERE table_name = 'dim_demographics_population' ")
    cur.fetchall()

    # dim_demographics_statistics
    cur.execute("SELECT table_name, column_name, data_type FROM information_schema.columns WHERE table_name = 'dim_demographics_statistics' ")
    cur.fetchall()

    # dim_airport_code
    cur.execute("SELECT table_name, column_name, data_type FROM information_schema.columns WHERE table_name = 'dim_airport_code' ")
    cur.fetchall()

    # dim_country_code
    cur.execute("SELECT table_name, column_name, data_type FROM information_schema.columns WHERE table_name = 'dim_country_code' ")
    cur.fetchall()

    # dim_state_code
    cur.execute("SELECT table_name, column_name, data_type FROM information_schema.columns WHERE table_name = 'dim_state_code' ")
    cur.fetchall()

    # dim_city_code
    cur.execute("SELECT table_name, column_name, data_type FROM information_schema.columns WHERE table_name = 'dim_city_code' ")
    cur.fetchall()

    # 2- Redshift tables are not empty
    empty_flag = False

    # fact_immigration
    list=[]
    cur.execute('select * from fact_immigration limit 1')
    list = cur.fetchall()
    if list:
        print('check')
    else:
        empty_flag = True
        print("fact_immigration is empty")

    1000

    # dim_immigration_citizen
    list=[]
    cur.execute('select * from dim_immigration_citizen limit 1')
    list = cur.fetchall()
    if list:
        print('check')
    else:
        empty_flag = True
        print("dim_immigration_citizen is empty")

    Check

    # dim_demographics_population
    list=[]
    cur.execute('select * from dim_demographics_population limit 1')
    list = cur.fetchall()
    if list:
        print('check')
    else:
        empty_flag = True
        print("dim_demographics_population is empty")

    check

    # dim_demographics_statistics
    list=[]
    cur.execute('select * from dim_demographics_statistics limit 1')
    list = cur.fetchall()
    if list:
        print('check')
    else:
        empty_flag = True
        print("dim_demographics_statistics is empty")

    check

    # dim_airport_code
    list=[]
    cur.execute('select * from dim_airport_code limit 1')
    list = cur.fetchall()
    if list:
        print('check')
    else:
        empty_flag = True
        print("dim_airport_code is empty")

    # dim_country_code
    list=[]
    cur.execute('select * from dim_country_code limit 1')
    list = cur.fetchall()
    if list:
        print('check')
    else:
        empty_flag = True
        print("dim_country_code is empty")

    # dim_state_code
    list=[]
    cur.execute('select * from dim_state_code limit 1')
    list = cur.fetchall()
    if list:
        print('check')
    else:
        empty_flag = True
        print("dim_state_code is empty")

    # dim_city_code
    list=[]
    cur.execute('select * from dim_city_code limit 1')
    list = cur.fetchall()
    if list:
        print('check')
    else:
        empty_flag = True
        print("dim_city_code is empty")

4.3 Data dictionary

Create a data dictionary for your data model. For each field, provide a
brief description of what the data is and where it came from. You can
include the data dictionary in the notebook or in a separate file.

  fact_immigration   Type
  ------------------ -----------
  cic_id             BIGINT
  year               INT
  month              INT
  city_code          CHAR(3)
  state_code         CHAR(2)
  mode               INT
  visa               INT
  arrive_date        TIMESTAMP
  departure_date     TIMESTAMP

  dim_immigrationg_citizen   Type
  -------------------------- ---------
  cic_id                     BIGINT
  citizen_country            INT
  residence_country          INT
  birth_year                 INT
  gender                     CHAR(1)
  ins_num                    INT

  dim_demographics_population   Type
  ----------------------------- ---------
  city_code                     CHAR(3)
  male_population               INT
  famale_population             INT
  num_veterans                  INT
  foreign_born                  INT
  race                          VARCHAR

  dim_demographics_statistics   Type
  ----------------------------- ---------
  city_code                     CHAR(3)
  median_age                    INT
  avg_household_size            FLOAT

  dim_airport_code   Type
  ------------------ ---------
  state_code         CHAR(2)
  type               VARCHAR
  name               VARCHAR
  coordinates        VARCHAR

  dim_country_code   Type
  ------------------ ---------
  country_id         INT
  country            VARCHAR

  dim_city_code   Type
  --------------- ---------
  city_code       CHAR(2)
  city            VARCHAR
  state_code      CHAR(2)

  dim_state_code   Type
  ---------------- ---------
  state_code       CHAR(3)
  state            VARCHAR

Step 5: Complete Project Write Up

* Clearly state the rationale for the choice of tools and technologies for the project.

    1. Pandas is used to manipulate the data in its easy-to-use dataframe
    2. AWS Redshift is used to hold the data in a data warehose that is distributed and widly accessable.

* Propose how often the data should be updated and why.

    1. Data that comes from Immigration dataset should be updated monthly.
    2. Data that comes from Demographics dataset should be updated annually.
    3. Data about Citys, States, and Countries should be updated on demand.
    4. Data about Airports codes should be updated on demand.

* Write a description of how you would approach the problem differently under the following scenarios:

* The data was increased by 100x.

     1. Apache Spark will be used instead of Pandas libiraries to leverage the advantages of distributed proccessing.
     2. AWS EMR will be used to easily manage the Apache Spark cluster.

* The data populates a dashboard that must be updated on a daily basis by 7am every day.

     1. Apache Airflow will be used to schedule the run of the pipline.

* The database needed to be accessed by 100+ people.

     1. AWS Redshift database can handle up to 500 connections simultaneously, to handle more than 500 connections, using another Airflow pipline to duplicate the database periodically to work as a load balancer would be a suggested solution.
