# group4_project2
## Kevin Behlke: https://github.com/kwb95124
## Guy Mayer: https://github.com/guymayer1/urbancrimetimeseries/blob/main/README.md
## David Moreno: ​​https://github.com/dmon67854/urbancrimetimeseries/tree/main
## Nadia Nazeem: https://github.com/nadianaz07/group4_project2/blob/main/README.md
## Mariana Munoz: https://github.com/Mariana-Munoz-70445/PROJECT-2-GROUP-4/blob/main/README.md?plain=1

## DATASET DESCRIPTIONS 📝
###   We chose the City Police Departments dataset from the Snowflake Marketplace, provided by Snowflake Public Data Free, and is listed as the Urban Crime Timeseries dataset. We chose this dataset because it is easy to understand and relevant to real-world situations, while still being complex enough to support meaningful analysis. Compared to datasets like economic indicators or housing finance reports, crime data feels more straightforward and relevant to everyday life.
###   The dataset is structured as a single table and contains 11,876,843 rows, with each row representing a specific combination of date, location, and type of crime. It tracks crime across several major U.S. cities, including San Francisco, Chicago, Seattle, Los Angeles, Houston, and New York City. Each record includes the date, ZIP code, city, type of crime (such as theft, battery/assault, or deceptive practice), and the number of times that crime occurred. Because the dataset spans different locations and time periods, it allows for comparisons across both geography and time.
###   One thing that makes this dataset more interesting is that each city updates its data at a different rate. For example, San Francisco, Chicago, and Seattle update daily, Los Angeles updates weekly, Houston updates monthly, and New York City updates quarterly. This adds a layer of complexity when comparing trends across cities, since the data isn’t reported in the same way everywhere.

### Key columns in the dataset include: 
| Column Name   | Data Type | Description                                                         |
| ------------- | --------- | ------------------------------------------------------------------- |
| DATE          | DATE      | The date associated with the reported data                          |
| GEO_ID        | VARCHAR   | A unique identifier for a geographic area (such as a ZIP code)      |
| CITY          | VARCHAR   | The city where the crime occurred                                   |
| VARIABLE      | VARCHAR   | An internal identifier for the type of crime                        |
| VARIABLE_NAME | VARCHAR   | A human-readable name for the crime category (e.g., theft, assault) |
| VALUE         | NUMBER    | The number of reported incidents for that crime on that date        |

#### While the dataset does not define a formal primary key, each record can be uniquely identified by the combination of DATE, GEO_ID, and VARIABLE, which together act as a composite key.
#### Overall, this dataset can be useful in a few different ways. Police departments could use it to better understand crime patterns and make decisions about resource allocation, while individuals could use it to stay informed about safety in different areas and make more informed decisions about where they spend their time.



## QUESTIONS & JUSTIFICATIONS

### 1. Which types of crime are most common in different neighborhoods?
#### This question is relevant to both individuals and police departments. For example, people who are planning to travel to or move into a city would likely want to understand what types of crimes are most common in certain areas. Someone might feel more comfortable living in an area with higher non-violent crime rates, like theft, rather than areas with more frequent violent crimes such as assault or battery, depending on their personal situation.

#### From a policing perspective, this information is also important because different types of crime require different strategies. A city or neighborhood with higher theft rates may require a different patrol approach compared to one with higher rates of violent crime. Understanding these patterns can help departments better allocate resources and respond more effectively.

### 2. What kinds of patterns do we see in the occurrence of crime over time? 
#### This question focuses on identifying trends over time and seeing whether similar patterns exist across different cities. Instead of looking at day-to-day variation, it examines how crime levels increase or decrease over longer periods and whether those changes follow consistent patterns. For example, there may be noticeable rises or drops in crime during certain years or in response to major events, such as the decline in crime observed around 2020.
#### These patterns are especially useful for police departments when planning long-term strategies and allocating resources. If certain trends are consistent across cities or over time, departments can better anticipate periods of higher or lower activity and adjust their approach accordingly. Overall, analyzing crime over time helps provide a broader understanding of how external factors influence crime and how resources can be used more effectively to prevent and respond to it.

## DATA MANIPULATIONS

### QUESTION ONE: Which types of crime are most common in different neighborhoods?

#### For this analysis, we used SQL to group and simplify crime categories before creating the stacked bar chart. Because the dataset contains many detailed crime labels, we applied a CASE statement to combine similar crimes into broader categories such as Theft, Burglary, and Battery or Assault, making the results easier to interpret. We also filtered the data using LOWER(VARIABLE_NAME) with LIKE conditions to focus on the most relevant crime types. Finally, we used SUM(VALUE) to calculate the total number of incidents for each crime category within each city, allowing the chart to clearly show how different types of crime contribute to overall crime levels.
```sql
SELECT CITY,
       CASE
           WHEN LOWER(VARIABLE_NAME) LIKE '%theft%' 
                AND LOWER(VARIABLE_NAME) LIKE '%vehicle%' 
           THEN 'Motor Vehicle Theft'

           WHEN LOWER(VARIABLE_NAME) LIKE '%theft%' 
           THEN 'Theft'

           WHEN LOWER(VARIABLE_NAME) LIKE '%burglary%' THEN 'Burglary'

           WHEN LOWER(VARIABLE_NAME) LIKE '%battery%' 
             OR LOWER(VARIABLE_NAME) LIKE '%assault%' 
           THEN 'Battery or Assault'

           WHEN LOWER(VARIABLE_NAME) LIKE '%criminal damage%' 
           THEN 'Criminal Damage'

           ELSE VARIABLE_NAME
       END AS CRIME_TYPE,
       SUM(VALUE) AS TOTAL_INCIDENTS
FROM SNOWFLAKE_PUBLIC_DATA_FREE.PUBLIC_DATA_FREE.URBAN_CRIME_TIMESERIES
WHERE (
       (LOWER(VARIABLE_NAME) LIKE '%theft%' AND LOWER(VARIABLE_NAME) LIKE '%vehicle%')
    OR LOWER(VARIABLE_NAME) LIKE '%theft%'
    OR LOWER(VARIABLE_NAME) LIKE '%burglary%'
    OR LOWER(VARIABLE_NAME) LIKE '%battery%'
    OR LOWER(VARIABLE_NAME) LIKE '%assault%'
    OR LOWER(VARIABLE_NAME) LIKE '%criminal damage%'
)
GROUP BY CITY,
         CASE
           WHEN LOWER(VARIABLE_NAME) LIKE '%theft%' 
                AND LOWER(VARIABLE_NAME) LIKE '%vehicle%' 
           THEN 'Motor Vehicle Theft'

           WHEN LOWER(VARIABLE_NAME) LIKE '%theft%' 
           THEN 'Theft'

           WHEN LOWER(VARIABLE_NAME) LIKE '%burglary%' THEN 'Burglary'

           WHEN LOWER(VARIABLE_NAME) LIKE '%battery%' 
             OR LOWER(VARIABLE_NAME) LIKE '%assault%' 
           THEN 'Battery or Assault'

           WHEN LOWER(VARIABLE_NAME) LIKE '%criminal damage%' 
           THEN 'Criminal Damage'

           ELSE VARIABLE_NAME
       END
ORDER BY CITY, TOTAL_INCIDENTS DESC
LIMIT 100;
```
#### Data manipulation:The first query filters raw crime event data to relevant categories and pretty much helps rename crime labels using a CASE WHEN statement, as well as aggregating the total incidents by city and crime type. The result is a structured dataset that compares the distribution of crime across cities.


### QUESTION TWO: What kinds of patterns do we see in the occurrence of crime over time? 


#### For the over-time crime analysis, we used SQL to clean and summarize the data before visualization. We filtered out the “Daily count of incidents, all incidents” category to avoid double-counting when aggregating results. We then used DATE_TRUNC('month', DATE) to group data by month, making trends easier to interpret over time, and applied SUM(VALUE) to calculate total incidents for each city within each month.

```sql
SELECT CITY, DATE_TRUNC('month', DATE) AS DAY_NAME, SUM(VALUE) AS TOTAL_INCIDENTS
FROM SNOWFLAKE_PUBLIC_DATA_FREE.PUBLIC_DATA_FREE.URBAN_CRIME_TIMESERIES
WHERE VARIABLE_NAME != 'Daily count of incidents, all incidents'
GROUP BY CITY, DAY_NAME, 
ORDER BY CITY, DAY_NAME
LIMIT 1000;
```

#### The calculated field TOTAL_INCIDENTS represents the total number of reported crimes for each city during each month. This transformation allows us to compare crime trends across cities over time in a clearer and more meaningful way.

## 📊ANALYSIS & RESULTS📊

### QUESTION 1 CHART: Which types of crime are most common in different neighborhoods?
<img width="817" height="586" alt="image" src="https://github.com/user-attachments/assets/d3b48816-15b3-4b64-b228-1ac46e279648" />

#### INTERPRETATION:

#### This stacked bar chart shows how the top five most reported crimes in the United States vary between cities. The categories include theft, battery/assault, motor vehicle theft, criminal damage, and burglary. We chose a bar chart because we are comparing total crime counts rather than changes over time, and the stacked format allows us to see how different cities contribute to each crime category in one view. One clear pattern is that theft is the most reported crime across nearly all cities, with New York showing significantly higher incident counts than any other city. Chicago also reports high levels of theft and battery/assault, while cities like Seattle and San Francisco consistently show lower totals.

#### In addition, violent crimes such as battery/assault tend to be higher in larger cities, while non-violent crimes like burglary and motor vehicle theft are more evenly distributed. For example, Los Angeles shows relatively high levels of burglary compared to other cities, even though it does not lead in more severe categories. These differences highlight that crime is not uniform across cities and that each location may require a different approach to prevention and enforcement. Cities with higher theft rates may benefit from stronger property crime prevention strategies, while those with higher violent crime may require more targeted policing, helping support more effective resource allocation.

### Question 2 CHART: What kinds of patterns do we see in the occurrence of crime over time? 
<img width="811" height="565" alt="image" src="https://github.com/user-attachments/assets/abec034f-f7bd-47ad-8ae2-8a8596e072e7" />

#### INTERPRETATION: 
#### This visualization shows how total crime incidents change over time across different cities, revealing several clear patterns. Although each city has different overall crime levels, many follow similar trends, suggesting that crime is influenced by broader external factors rather than being random. One of the most noticeable trends is a sharp drop in crime around 2020 across nearly all cities, which aligns with the COVID-19 pandemic. Lockdowns and reduced public activity likely limited opportunities for certain types of crime.

#### After this decline, crime levels rise again in most cities as activity returns to normal, with places like New York and Chicago approaching or exceeding pre-pandemic levels. These patterns are important because they highlight predictable shifts over time rather than small daily fluctuations. Understanding these trends allows police departments to better plan ahead, adjust staffing, and allocate resources more effectively during periods of higher or lower crime activity.


## 🎯STREAMLIT APP🎯

### Which types of crime are most common in different neighborhoods? 
<img width="971" height="605" alt="image" src="https://github.com/user-attachments/assets/cab2d6f0-26e8-43b0-8a59-f459ca721985" />

### What kinds of patterns do we see in the occurrence of crime over time?

<img width="978" height="522" alt="image" src="https://github.com/user-attachments/assets/1f3fe26b-4048-4e22-910c-15866f0741b1" />

#### This dashboard allows users to explore crime patterns by selecting a specific city using the dropdown filter. In this example we selected Chicago. Once a city is selected, both the bar chart and time series visualization update dynamically, allowing users to analyze the most common types of crime as well as how crime trends change over time within that city.

#### This interaction adds analytical value by letting users focus on one city at a time rather than viewing aggregated data across all cities. As a result, users can more easily identify city-specific patterns, compare crime types, and observe trends such as increases or decreases in crime over time. This makes the analysis more targeted and helps highlight insights that may not be visible in a combined view.

##### 🤖AI USAGE🤖
###### AI (ChatGPT) was used to enhance the snowsight dashboards by generating code to filter the streamlit dashboards by City. It was also used to refine and improve the clarity of some of our descriptions. 
<img width="977" height="155" alt="image" src="https://github.com/user-attachments/assets/0eace0f0-8c70-460b-8621-442671887515" />

