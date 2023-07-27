# Restaurant Monitoring System

The Restaurant Monitoring System is a backend API that helps restaurant
owners track the online and offline status of their stores during
business hours. The system polls each store roughly every hour and
records whether the store was active or not in a CSV file. The system
also has data on the business hours of all the stores and the timezone
for each store.

The system provides two APIs:

1.  /trigger_report endpoint that triggers the generation of a report
    from the data provided (stored in the database). The API has no
    input and returns a report ID (a random string). The report ID is
    used to poll the status of report completion.

2.  /get_report endpoint that returns the status of the report or the
    CSV. The API takes a report ID as input and returns the following:

    -   If report generation is not complete, return "Running" as the
        output
    -   If report generation is complete, return "Complete" along with
        the CSV file with the following schema: store_id,
        uptime_last_hour(in minutes), uptime_last_day(in hours),
        update_last_week(in hours), downtime_last_hour(in minutes),
        downtime_last_day(in hours), downtime_last_week(in hours) The
        uptime and downtime reported in the CSV only include
        observations within business hours. The system extrapolates
        uptime and downtime based on the periodic polls we have ingested
        to the entire time interval.

## Data Sources 

We will have 3 sources of data 

1. We poll every store roughly every hour and have data about whether the store was active or not in a CSV.  The CSV has 3 columns (`store_id, timestamp_utc, status`) where status is active or inactive.  All timestamps are in **UTC**
    1. Data can be found in CSV format [here](https://drive.google.com/file/d/1UIx1hVJ7qt_6oQoGZgb8B3P2vd1FD025/view?usp=sharing)
2. We have the business hours of all the stores - schema of this data is `store_id, dayOfWeek(0=Monday, 6=Sunday), start_time_local, end_time_local`
    1. These times are in the **local time zone**
    2. If data is missing for a store, assume it is open 24*7
    3. Data can be found in CSV format [here](https://drive.google.com/file/d/1va1X3ydSh-0Rt1hsy2QSnHRA4w57PcXg/view?usp=sharing)
3. Timezone for the stores - schema is `store_id, timezone_str`
    1. If data is missing for a store, assume it is America/Chicago
    2. This is used so that data sources 1 and 2 can be compared against each other. 
    3. Data can be found in CSV format [here](https://drive.google.com/file/d/101P9quxHoMZMZCVWQ5o-shonk2lgK1-o/view?usp=sharing)
    
**_NOTE:_**  Data files cannot be pushed due to lfs issue 
     
         #### Files structure 
         
         ```
            data/
            
              ├── Menu hours.csv
              
              ├── store status.csv
              
              └── bq-results-20230125-202210-1674678181880.csv
            ```

## System Requirements 

* The data sources are not static, and the system
should not precompute the answers. The system should keep updating the
data every hour. 
* The system should store the CSVs into a relevant
database and make API calls to get the data.

## Installation 

To install the required packages, run the following command in the project directory:
```
    pip install -r requirements.txt
```

## Usage

To start the server, run the following command in the project directory:
```
    python run.py

```

The server will start running on http://localhost:5000

Note: Remember to add url prefix in the api like http://localhost:5000/api

## API Documentation

- ### /trigger_report

    This endpoint triggers the generation of a report from the data provided (stored in the database).

    * Request
    ```
    POST /api/trigger_report HTTP/1.1

    ```
    * Response
    ```
        HTTP/1.1 200 OK
    Content-Type: application/json

    {
        'message': 'Success', 
        'error_code':200,
        "report_id": "random_string"
    }

    ```

- ### /get_report

    This endpoint returns the status of the report or the CSV.
    
    * Request
    ```
    GET /api/get_report?report_id=random_string HTTP/1.1

    ```
    * Response
    ```
        HTTP/1.1 200 OK
        Content-Type: text/csv

        store_id, uptime_last_hour(in minutes), uptime_last_day(in hours), update_last_week(in hours), downtime_last_hour(in minutes), downtime_last_day(in hours), downtime_last_week(in hours)
        1, 60, 


    ```

## Functionalities 

Used advance python features like -
 - Multithreading
 - Caching

