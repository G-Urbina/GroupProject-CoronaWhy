1.
//launch git-bash or any terminal you use and login to your server

ssh username@your_server_ip
 
//download the zip file containing the dataset from GitHub, code should be one line
```
wget -O CoronaWhyDataset.zip https://github.com/G-Urbina/GroupProject-CoronaWhy/releases/download/CSV-files/CoronaWhy_Dataset.zip
```

//unzip the file

unzip CoronaWhyDataset.zip

//make sure you have the csv files

ls Public*

----------------------------------------------------------------------

2.
//create CoronaWhy directory

hdfs dfs -mkdir CoronaWhy
 
//create four directories inside CoronaWhy for our tables, code should be one line

hdfs dfs -mkdir CoronaWhy/cases CoronaWhy/mortality CoronaWhy/recovered CoronaWhy/testing
 
//create five directories for our cleaned tables, code should be one line

hdfs dfs -mkdir CoronaWhy/cases_clean CoronaWhy/mortality_clean CoronaWhy/recovered_clean CoronaWhy/combined_death_recovery CoronaWhy/testing_clean

//make sure directories were created

hdfs dfs -ls CoronaWhy

//move unzipped csv files to the first four directories you created

hdfs dfs -put Public_COVID-19_Canada_Cases.csv CoronaWhy/cases

hdfs dfs -put Public_COVID-19_Canada_Mortality.csv CoronaWhy/mortality

hdfs dfs -put Public_COVID-19_Canada_Recovered.csv CoronaWhy/recovered

hdfs dfs -put Public_COVID-19_Canada_Testing.csv CoronaWhy/testing
 
//make sure files were put in the directories

hdfs dfs -ls CoronaWhy/cases

hdfs dfs -ls CoronaWhy/mortality

hdfs dfs -ls CoronaWhy/recovered

hdfs dfs -ls CoronaWhy/testing

----------------------------------------------------------------------

3.
//launch another git-bash or any terminal you use
 
//login to your server

ssh username@your_server_ip
 
//use HiveServer2

beeline
 
//use your directory

use username;


3.1 Cases Table

//drop “cases” table if it already exists and create table

!!!!replace username with yours using a text editor for all tables!!!!
```
DROP TABLE IF EXISTS cases;

CREATE EXTERNAL TABLE IF NOT EXISTS cases(case_id INT, 
    provincial_case INT,
    age STRING,
    sex STRING,
    health_region STRING,
    province STRING,
    country STRING,
    date_report STRING,
    report_week STRING,
    travel_yn INT,
    travel_history_country STRING,
    locally_acquired STRING,
    case_source STRING,
    additional_info STRING,
    additional_source STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE LOCATION '/user/username/CoronaWhy/cases'
TBLPROPERTIES ('skip.header.line.count'='1');
```
//make sure table was created

show tables;
 
//query should return 10 rows

select * from cases limit 10;

 3.2 Cleaned Cases Table
 
//create clean cases table, remove redundant fields, rename fields, and format date
```
DROP TABLE IF EXISTS cases_clean;

CREATE TABLE IF NOT EXISTS cases_clean
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE LOCATION '/user/username/CoronaWhy/cases_clean/'
AS
SELECT
    case_id,
    provincial_case,
    age AS age_range,
    sex,
    health_region AS city,
    province,
    country,
    from_unixtime(unix_timestamp(date_report ,'dd-MM-yyyy'), 'MM-dd-yyyy') date_report,
    from_unixtime(unix_timestamp(report_week ,'dd-MM-yyyy'), 'MM-dd-yyyy') report_week,
    travel_yn,
    travel_history_country,
    locally_acquired
FROM cases;
```
//make sure table was created

show tables;
 
//query should return 10 rows

select * from cases_clean limit 10;

3.3 Mortality Table

//create mortality table
```
DROP TABLE IF EXISTS mortality;

CREATE EXTERNAL TABLE IF NOT EXISTS mortality (
    death_id INT,
    province_death INT,
    case_id INT,
    age STRING,
    sex STRING,
    health_region STRING,
    province STRING,
    country STRING,
    date_death_report STRING, 
    death_source STRING,
    additional_info STRING,
    additional_source STRING
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
STORED AS TEXTFILE LOCATION '/user/username/CoronaWhy/mortality'
TBLPROPERTIES ('skip.header.line.count'='1');
```
//make sure table was created

show tables;

//query should return 10 rows

select * from mortality limit 10;

3.4 Cleaned Mortality Table

//create clean mortality table, remove redundant fields, and format date
```
DROP TABLE IF EXISTS mortality_clean;

CREATE TABLE IF NOT EXISTS mortality_clean
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',' 
STORED AS TEXTFILE 
LOCATION '/user/username/CoronaWhy/mortality_clean'
AS
SELECT 
    death_id,
    province,
    country,
    from_unixtime(unix_timestamp(date_death_report, 'dd-MM-yyyy'), 'MM-dd-yyyy') AS date_death_report
FROM mortality;
```
//make sure table was created

show tables;

//query should return 10 rows

select * from mortality_clean limit 10;

3.5 Recovered Table

//create recovered table
```
DROP TABLE IF EXISTS recovered;

CREATE EXTERNAL TABLE IF NOT EXISTS recovered (
    date_recovered STRING,
    province STRING,
    cumulative_recovered STRING,
    province_source STRING,
    source STRING,
    additional_source STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE LOCATION '/user/username/CoronaWhy/recovered'
TBLPROPERTIES ('skip.header.line.count'='1');
```
//make sure table was created

show tables;

//query should return 10 rows

select * from recovered limit 10;

3.6 Cleaned Recovered Table

//create clean recovered table, remove redundant fields, format dates, and replace “NA”
```
DROP TABLE IF EXISTS recovered_clean;

CREATE TABLE IF NOT EXISTS recovered_clean
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',' 
STORED AS TEXTFILE 
LOCATION '/user/username/CoronaWhy/recovered_clean/'
AS 
SELECT 
    FROM_UNIXTIME(UNIX_TIMESTAMP(date_recovered, 'dd-MM-yyyy'), 'MM-dd-yyyy') AS date_recovered,
    province,
    CASE 
        WHEN cumulative_recovered = 'NA' THEN 0
        ELSE cumulative_recovered
    END AS cumulative_recovered
FROM recovered;
```
//make sure table was created

show tables;

//query should return 10 rows

select * from recovered_clean limit 10;

3.7 Combine Mortality and Recovered Table

//create combined_death_recovery table
```
DROP TABLE IF EXISTS combined_death_recovery;

CREATE TABLE IF NOT EXISTS combined_death_recovery (
    event_date STRING,
    event_id INT,
    event_type STRING,
    event_province STRING,
    event_country STRING,
    cumulative_recovered INT
)
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',' 
STORED AS TEXTFILE 
LOCATION '/user/username/CoronaWhy/combined_death_recovery/';
```
//make sure table was created

show tables;


//insert fields from the mortality_clean and recovered_clean tables into combined_death_recovered using UNION ALL
```
INSERT INTO TABLE combined_death_recovery
SELECT
  
    from_unixtime(unix_timestamp(date_death_report, 'dd-MM-yyyy'), 'MM-dd-yyyy') AS event_date,
    death_id AS event_id,
    'Death' AS event_type,
    province AS event_province,
    country AS event_country,
    1 AS cumulative_recovered 

FROM mortality_clean

UNION ALL

SELECT
    
    from_unixtime(unix_timestamp(date_recovered, 'dd-MM-yyyy'), 'MM-dd-yyyy') AS event_date,      
    ROW_NUMBER() OVER (ORDER BY date_recovered) + 35 AS event_id, 
    'Recovery' AS event_type,
    province AS event_province,
    'Canada' AS event_country,
    CASE 
        WHEN cumulative_recovered = 'NA' THEN 0
        ELSE CAST(cumulative_recovered AS INT)  
    END AS cumulative_recovered
FROM recovered_clean;
```
//query should return 10 rows

select * from combined_death_recovery limit 10;

3.8 Testing Table

//create testing table
```
DROP TABLE IF EXISTS testing;

CREATE EXTERNAL TABLE IF NOT EXISTS testing (
    date_testing STRING,
    province STRING,
    cumulative_testing INT,
    province_source STRING,
    source STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE LOCATION '/user/username/CoronaWhy/testing'
TBLPROPERTIES ('skip.header.line.count'='1');
```
//make sure table was created

show tables;

//query should return 3 rows

 select * from testing limit 3;

3.9 Cleaned Testing Table

//create clean testing table, format date, correct province names, replace null values
```
DROP TABLE IF EXISTS testing_clean;

CREATE TABLE testing_clean
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',' 
STORED AS TEXTFILE 
LOCATION '/user/username/CoronaWhy/testing_clean/' 

AS 
SELECT 
    CASE
        WHEN date_testing IS NULL OR TRIM(date_testing) = '' THEN ' ' 
        ELSE 
            CASE 
                WHEN unix_timestamp(date_testing, 'dd-MM-yyyy') IS NOT NULL 
                THEN from_unixtime(unix_timestamp(date_testing, 'dd-MM-yyyy'), 'MM-dd-yyyy') 
                ELSE '' 
            END
    END AS date_testing,

    CASE
        WHEN province = 'BC' THEN 'British Columbia'
        WHEN province = 'NL' THEN 'Newfoundland and Labrador'
        WHEN province = 'PEI' THEN 'Prince Edward Island'
        WHEN province = 'NWT' THEN 'Northwest Territories'
        ELSE province
    END AS province,

    CASE 
        WHEN cumulative_testing IS NULL AND (date_testing IS NULL OR TRIM(date_testing) = '') THEN ' ' 
        WHEN cumulative_testing IS NULL AND date_testing IS NOT NULL THEN '0' 
        ELSE cumulative_testing 
    END AS cumulative_testing
FROM testing;
```
//make sure table was created

show tables;

//query should return 3 rows

 select * from testing_clean limit 3;
