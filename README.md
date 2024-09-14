# Loading Taxi Data into Google Cloud SQL 2.5 using **gcloud** command line tool

In this project we will import data from CSV text files into Cloud SQL and then carry out some basic data analysis using simple queries.

The dataset used in this lab is collected by the NYC Taxi and Limousine Commission and includes trip records from all trips completed in Yellow and Green taxis in NYC from 2009 to present, and all trips in for-hire vehicles (FHV) from 2015 to present. Records include fields capturing pick-up and drop-off dates/times, pick-up and drop-off locations, trip distances, itemized fares, rate types, payment types, and driver-reported passenger counts.

### Objectives

1.  Create Cloud SQL instance
2.  Create a Cloud SQL database
3.  Import text data into Cloud SQL
4.  Check the data for integrity


## STEP 1 : Create environment variables for the project ID and the storage bucket.

```bash
export PROJECT_ID=$(gcloud info --format='value(config.project)')
export BUCKET=${PROJECT_ID}-ml
```




