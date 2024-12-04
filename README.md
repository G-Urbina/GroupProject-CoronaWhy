
CoronaWhy Tutorial



//DOWNLOAD FILES
//login
ssh username@your_server_ip
 
//download files, code needs to be one line
wget -O Public_Covid-19_Canada_Cases.csv https://github.com/G-Urbina/GroupProject-CoronaWhy/releases/download/CSV-files/Public_COVID-19_Canada_Cases.csv
 
//make sure file was downloaded
ls
 
 
//MAKE DIRECTORIES
hdfs dfs -mkdir CoronaWhy
 
hdfs dfs -mkdir CoronaWhy/cases
 
//this directory will be used later to download csv file
hdfs dfs -mkdir CoronaWhy/cases_dateformat
 
//move downloaded file
hdfs dfs -put Public_Covid-19_Canada_Cases.csv CoronaWhy/cases
 
//make sure file was put in directory
hdfs dfs -ls CoronaWhy/cases
 
 
//USE HIVE
//launch git-bash or any program you use
 
//login
ssh username@your_server_ip
 
//use Hive2
beeline
 
//use your directory
use username;
 
 
//CREATE TABLES
DROP TABLE IF EXISTS cases;
 
!!!!replace username with yours using a text editor!!!!
 
CREATE EXTERNAL TABLE IF NOT EXISTS cases(case_id INT, provincial_case INT,
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
 
//make sure table was created
show tables;
 
//query should return 10 rows
select * from cases limit 10;


 
 
DROP TABLE IF EXISTS cases_dateformat;
 
!!!!replace username with yours using a text editor!!!!
 
CREATE TABLE IF NOT EXISTS cases_dateformat
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE LOCATION '/user/username/CoronaWhy/cases_dateformat/'
AS
SELECT
    case_id,
    provincial_case,
    age,
    sex,
    health_region,
    province,
    country,
    from_unixtime(unix_timestamp(date_report ,'dd-MM-yyyy'), 'MM-dd-yyyy') date_report,
    from_unixtime(unix_timestamp(report_week ,'dd-MM-yyyy'), 'MM-dd-yyyy') report_week,
    travel_yn,
    travel_history_country,
    locally_acquired
FROM cases;
 
//query should return 10 rows
select * from cases_dateformat limit 10;
 


 
//DOWNLOAD CSV FILE TO LOCAL PC
 
//get file and rename it
hdfs dfs -get CoronaWhy/cases_dateformat/000000_0
 
//open another terminal and download file to your pc, replace username with yours
scp username@your_server_ip:/home/username/000000_0 CanadaCases.csv
 
 
//OPEN EXCEL
 
1. Open a new blank workbook
2. Click on the data tab
3. Click on Get Data > From File > From Text/CSV
4. Select the CanadaCases.csv file you downloaded, click import
5. When you see a preview of the data, click on Transform Data
6. Rename the columns from Column1 to Column12 with case_id, provincial_case, age, sex, health_region, province, country, date_report, report_week, travel_yn, locally_acquired
7. Click on Close & Load
8. Save the workbook as Canada_Covid_Report.xlsx
 
 



//CREATE 3D MAP IN EXCEL - Cities
1. Click on Insert and then 3D Map tab
2. With the 3D Map window open, click on the Map Labels tab
3. For Location click on Add Field and select health_region, change it to City
4. For Height select case_id and change it to (Count - Not Blank)
5. For Category select health_region
6. For Time select date_reported
7. For filters add health_region, select all and then uncheck Not Reported
8. Hover mouse over the bars to view total cases per city (case_id - Count)
9. (Optional) To get a better view of the data click on the 3D Map down arrow to get a better view of the bars. If you don’t see the legends click on the Legends tab

//CREATE 3D MAP IN EXCEL - Provinces
1. Follow the directions 1-7 in the previous step
2. Change Location to province 
3. Change Category to province
4. Hover mouse over the bars to view total cases per province (case_id - Count)
