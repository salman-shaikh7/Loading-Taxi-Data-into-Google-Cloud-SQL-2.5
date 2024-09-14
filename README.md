# Loading Taxi Data into Google Cloud SQL 2.5 using **gcloud** command line tool

In this project we will import data from CSV text files into Cloud SQL and then carry out some basic data analysis using simple queries.

The dataset used in this lab is collected by the NYC Taxi and Limousine Commission and includes trip records from all trips completed in Yellow and Green taxis in NYC from 2009 to present, and all trips in for-hire vehicles (FHV) from 2015 to present. Records include fields capturing pick-up and drop-off dates/times, pick-up and drop-off locations, trip distances, itemized fares, rate types, payment types, and driver-reported passenger counts.

### Objectives

1.  Create Cloud SQL instance
2.  Create a Cloud SQL database
3.  Import text data into Cloud SQL
4.  Check the data for integrity


## STEP 1 : Create environment variables for the project ID and the storage bucket.
Code 
```powershell
export PROJECT_ID=$(gcloud info --format='value(config.project)')
export BUCKET=${PROJECT_ID}-ml
```

This is will save config like project id and bucket name of current project so that we can use it in our code


## STEP 2. Create a Cloud SQL instance

Code 

```powershell
gcloud sql instances create taxi \
    --tier=db-n1-standard-1 --activation-policy=ALWAYS
```
Result 

```powershell
gcloud sql instances create taxi \
    --tier=db-n1-standard-1 --activation-policy=ALWAYS
WARNING: Starting with release 233.0.0, you will need to specify either a region or a zone to create an instance.
API [sqladmin.googleapis.com] not enabled on project [myprojectid7028]. Would you like to enable and retry (this will take a few minutes)? (y/N)?  Y

Enabling service [sqladmin.googleapis.com] on project [myprojectid7028]...
Operation "operations/acat.p2-125330155311-de7c25cb-3964-459e-89c7-08bf5f0c77d4" finished successfully.
Creating Cloud SQL instance for MYSQL_8_0...done.                                                                                                                                                
Created [https://sqladmin.googleapis.com/sql/v1beta4/projects/myprojectid7028/instances/taxi].
NAME: taxi
DATABASE_VERSION: MYSQL_8_0
LOCATION: us-central1-c
TIER: db-n1-standard-1
PRIMARY_ADDRESS: 0.0.0.0 
PRIVATE_ADDRESS: -
STATUS: RUNNABLE

```

Set a root password 

```powershell
gcloud sql users set-password root --host % --instance taxi \
 --password Passw0rd
```

create an environment variable with the IP address of the Cloud Shell

```powershell
export ADDRESS=$(wget -qO - http://ipecho.net/plain)/32
```
Whitelist the Cloud Shell instance for management access SQL instance

```powershell
gcloud sql instances patch taxi --authorized-networks $ADDRESS
```

IP address of Cloud SQL instance

```powershell
MYSQLIP=$(gcloud sql instances describe \
taxi --format="value(ipAddresses.ipAddress)")
```
Return IP Address
```powershell
echo $MYSQLIP
```
logging into the mysql

```powershell
mysql --host=$MYSQLIP --user=root \
      --password --verbose
```

result 

```powershell
mysql --host=$MYSQLIP --user=root \
      --password --verbose
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 370
Server version: 8.0.31-google (Google)

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Reading history-file /home/salmanshaikh/.mysql_history
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```
We will exceute following command in sql command line 

create the schema for the trips

```sql
create database if not exists bts;
use bts;

drop table if exists trips;

create table trips (
  vendor_id VARCHAR(16),		
  pickup_datetime DATETIME,
  dropoff_datetime DATETIME,
  passenger_count INT,
  trip_distance FLOAT,
  rate_code VARCHAR(16),
  store_and_fwd_flag VARCHAR(16),
  payment_type VARCHAR(16),
  fare_amount FLOAT,
  extra FLOAT,
  mta_tax FLOAT,
  tip_amount FLOAT,
  tolls_amount FLOAT,
  imp_surcharge FLOAT,
  total_amount FLOAT,
  pickup_location_id VARCHAR(16),
  dropoff_location_id VARCHAR(16)
);
```

Check the import

```sql
describe trips;
```

```sql
select distinct(pickup_location_id) 
from trips;
```

exit mysql interactive console

```bash
exit
```



## STEP 3.  Add data to Cloud SQL 

We will copy the New York City taxi trips CSV files stored on Cloud Storage locally. To keep resource usage low, will only be working with a subset of the data (~20,000 rows).

1. Copy data to cloud storage bucket
```powershell
gcloud storage cp gs://cloud-training/OCBL013/nyc_tlc_yellow_trips_2018_subset_1.csv trips.csv-1
gcloud storage cp gs://cloud-training/OCBL013/nyc_tlc_yellow_trips_2018_subset_2.csv trips.csv-2
```

Result

```powershell
Copying gs://cloud-training/OCBL013/nyc_tlc_yellow_trips_2018_subset_1.csv to file://trips.csv-1
  Completed files 1/1 | 850.3kiB/850.3kiB                                                                                                                                                        

Average throughput: 182.9MiB/s
Copying gs://cloud-training/OCBL013/nyc_tlc_yellow_trips_2018_subset_2.csv to file://trips.csv-2
  Completed files 1/1 | 849.8kiB/849.8kiB                                                                                                                                                        

Average throughput: 173.8MiB/s
```

Connect to the mysql interactive console to load local infile data

```powershell
mysql --host=$MYSQLIP --user=root  --password  --local-infile
```

In the mysql interactive console select the database:

```sql
use bts;
```

Load the local CSV file data using local-infile:

```sql
LOAD DATA LOCAL INFILE 'trips.csv-1' INTO TABLE trips
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
IGNORE 1 LINES
(vendor_id,pickup_datetime,dropoff_datetime,passenger_count,trip_distance,rate_code,store_and_fwd_flag,payment_type,fare_amount,extra,mta_tax,tip_amount,tolls_amount,imp_surcharge,total_amount,pickup_location_id,dropoff_location_id);
```

```sql
LOAD DATA LOCAL INFILE 'trips.csv-2' INTO TABLE trips
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
IGNORE 1 LINES
(vendor_id,pickup_datetime,dropoff_datetime,passenger_count,trip_distance,rate_code,store_and_fwd_flag,payment_type,fare_amount,extra,mta_tax,tip_amount,tolls_amount,imp_surcharge,total_amount,pickup_location_id,dropoff_location_id);
```

## STEP 4 : Checking for data integrity

Query the trips table for unique pickup location regions:
This should return 159 unique ids
```sql
select distinct(pickup_location_id) from trips;

```
Result
```sql
+--------------------+
| pickup_location_id |
+--------------------+
| 68                 |
| 138                |
| 261                |
| 262                |
| 100                |
| 7                  |
| 132                |
| 264                |
| 170                |
| 237                |
| 87                 |
| 161                |
| 229                |
| 186                |
| 230                |
| 224                |
| 234                |
| 233                |
| 125                |
| 79                 |
| 162                |
| 223                |
| 236                |
| 137                |
| 225                |
| 50                 |
| 76                 |
| 13                 |
| 65                 |
| 107                |
| 231                |
| 211                |
| 48                 |
| 164                |
| 90                 |
| 24                 |
| 88                 |
| 143                |
| 246                |
| 142                |
| 215                |
| 265                |
| 238                |
| 43                 |
| 239                |
| 141                |
| 148                |
| 166                |
| 163                |
| 249                |
| 74                 |
| 158                |
| 263                |
| 144                |
| 42                 |
| 151                |
| 114                |
| 70                 |
| 75                 |
| 33                 |
| 145                |
| 140                |
| 232                |
| 113                |
| 146                |
| 45                 |
| 4                  |
| 181                |
| 152                |
| 209                |
| 260                |
| 179                |
| 93                 |
| 244                |
| 256                |
| 226                |
| 112                |
| 243                |
| 255                |
| 40                 |
| 41                 |
| 49                 |
| 189                |
| 119                |
| 66                 |
| 12                 |
| 95                 |
| 25                 |
| 37                 |
| 52                 |
| 116                |
| 17                 |
| 80                 |
| 219                |
| 130                |
| 127                |
| 194                |
| 10                 |
| 31                 |
| 197                |
| 191                |
| 171                |
| 217                |
| 129                |
| 15                 |
| 157                |
| 97                 |
| 28                 |
| 193                |
| 168                |
| 29                 |
| 216                |
| 54                 |
| 139                |
| 206                |
| 167                |
| 16                 |
| 235                |
| 102                |
| 133                |
| 185                |
| 51                 |
| 8                  |
| 198                |
| 77                 |
| 69                 |
| 61                 |
| 126                |
| 247                |
| 89                 |
| 254                |
| 106                |
| 228                |
| 169                |
| 258                |
| 173                |
| 153                |
| 196                |
| 60                 |
| 147                |
| 208                |
| 53                 |
| 35                 |
| 82                 |
| 184                |
| 47                 |
| 218                |
| 203                |
| 134                |
| 178                |
| 92                 |
| 123                |
| 36                 |
| 159                |
| 56                 |
| 83                 |
| 200                |
| 259                |
| 188                |
+--------------------+
159 rows in set (0.06 sec)

```

Let's start by digging into the trip_distance column. 

Running the following query into the console:

```sql
select
  max(trip_distance),
  min(trip_distance)
from
  trips;
```

Result

```sql
+--------------------+--------------------+
| max(trip_distance) | min(trip_distance) |
+--------------------+--------------------+
|                 85 |                  0 |
+--------------------+--------------------+
1 row in set (0.05 sec)
```

One would expect the trip distance to be greater than 0 and less than, say 1000 miles. The maximum trip distance returned of 85 miles seems reasonable but the minimum trip distance of 0 seems buggy.


*How many trips in the dataset have a trip distance of 0?*

```sql
select count(*) from trips where trip_distance = 0;
```
result

```sql
+----------+
| count(*) |
+----------+
|      155 |
+----------+
1 row in set (0.05 sec)
```

There are 155 such trips in the database. These trips warrant further exploration. You'll find that these trips have non-zero payment amounts associated with them. Perhaps these are fraudulent transactions?


```sql
select count(*) from trips where fare_amount < 0;
```

```sql
+----------+
| count(*) |
+----------+
|       14 |
+----------+
1 row in set (0.05 sec)

```

There should be 14 such trips returned. Again, these trips warrant further exploration. There may be a reasonable explanation for why the fares take on negative numbers.


Finally, let's investigate the payment_type column.
```SQL
select
  payment_type,
  count(*)
from
  trips
group by
  payment_type;
```

The results of the query indicate that there are four different payment types, with:
```SQL
+--------------+----------+
| payment_type | count(*) |
+--------------+----------+
| 1            |    13863 |
| 2            |     6016 |
| 3            |      113 |
| 4            |       32 |
+--------------+----------+
4 rows in set (0.06 sec)
```
Digging into the documentation, a payment type of 1 refers to credit card use, payment type of 2 is cash, and a payment type of 4 refers to a dispute. The figures make sense.

Exit the 'mysql' interactive console

```powershell
exit
```


