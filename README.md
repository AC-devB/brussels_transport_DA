# Brussels Public Transportation Data Analysis & Dashboarding
This project demonstrates a complete data analysis process, from raw data collection and cleaning to creating an interactive, insight-driven dashboard for the public transportation network of Brussels.

## Technologies
* **Data Collection & Cleaning:** Python (Pandas)
* **Database:** SQLite
* **Data Analysis:** SQL
* **Visualization:** Power BI

## [Collecting and cleaning data with Python and Pandas]

The project began by collecting raw data, which was split across multiple text files. Python and Pandas was used to:
* Download and extract the compressed data.
* Clean and transform the raw data into a structured format.
* Load the cleaned data into an SQLite database for efficient querying.
<img width="1460" height="820" alt="bxl_transport_BI_namur" src="https://github.com/user-attachments/assets/0457c2fd-3dec-4991-bb5e-05dff5df8006" />


### *Python script for cleaning and sorting data*
```Python
import pandas as pd

gtfs_path = r"C:\Users\.../"

try:
    routes = pd.read_csv(f"{gtfs_path}routes.txt")
    trips = pd.read_csv(f"{gtfs_path}trips.txt")
    stops = pd.read_csv(f"{gtfs_path}stops.txt")
    stop_times = pd.read_csv(f"{gtfs_path}stop_times.txt")

    print("Files loaded successfully!")

except FileNotFoundError as e:
    print(f"Error: {e}.")

# Database Loading
from sqlalchemy import create_engine

db_name = 'brussels_transport.db'
engine = create_engine(f'sqlite:///{db_name}')

# Renaming columns
stops.rename(columns={'stop_id': 'stop_pk', 'stop_lat': 'latitude', 'stop_lon': 'longitude'}, inplace=True)
routes.rename(columns={'route_id': 'route_pk'}, inplace=True)
trips.rename(columns={'trip_id': 'trip_pk', 'route_id': 'route_fk'}, inplace=True)
stop_times.rename(columns={'trip_id': 'trip_fk', 'stop_id': 'stop_fk'}, inplace=True)

# Loading into SQL database
try:
    print("\nLoading data into the database...")
    routes.to_sql('dim_routes', engine, if_exists='replace', index=False)
    stops.to_sql('dim_stops', engine, if_exists='replace', index=False)
    trips.to_sql('dim_trips', engine, if_exists='replace', index=False)
    stop_times.to_sql('fact_stop_times', engine, if_exists='replace', index=False)

    print(f"Success! Database '{db_name}' has been created.")
    print("Tables created: dim_routes, dim_stops, dim_trips, fact_stop_times")

except Exception as e:
    print(f"An error occurred: {e}")
```


## [Database Design & SQL Analysis]
The data model was optimized to make analysis easier and SQL queries were created to extract key insights. The main fact table keeps detailed records of each stop made and the dimension tables show extra details about the stations, trips and routes. As an example, the query below shows the busiest hubs in the network, which helps with managing resources and city planning.

### *Which are the top 10 busiest stops in the network?*
```sql

SELECT
    s.stop_name,
    COUNT(st.stop_fk) AS total_visits
FROM
    fact_stop_times AS st
JOIN
    dim_stops AS s ON st.stop_fk = s.stop_pk
GROUP BY
    s.stop_name
ORDER BY
    total_visits DESC
LIMIT 10;

```
## [Power BI Dashboard & Visualizations]

Connected Power BI to a SQLite database via ODBC. Built an interactive dashboard to analyze network performance, including a new column in Power Query in order to convert a table into a time format to make daily patterns easier to see. Visualizations include:

* **Network Overview (Cards)**: Total Stops Made, Number of Stations
* **Busiest Stops (Bar Chart)**: Top 10 stops by activity
* **Daily Usage (Line Chart)**: Total stops made by time of day, highlighting peak hours
* **Transport Type Distribution (Donut Chart)**: Total scheduled trips per transport type
* **Network Locations (Map Chart)**: Interactive map of Brussels' stations showing usage, top stops and transport modes with area and route details.

## [Insights]
Through this project, messy data has been turned into a simple tool to understand network performance. Below are some important findings:

* **Rush hour commuting is important.** Daily usage analysis showed clear peaks in the morning and evening, indicating that the network is mainly used for work-related travel.

* **Top stations are network hubs**. The analysis pinpointed the busiest stations, showing where the majority of network activity is concentrated. This information is important for managing passenger flow.

* **Buses are the most used**. Data shows that buses make up the highest percentage of all scheduled trips.
