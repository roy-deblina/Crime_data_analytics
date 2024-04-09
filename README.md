# Crime_data_analytics

## Introduction
The Crime Data Analysis Project aims to utilize data-driven insights to understand and address criminal activities more effectively. By analyzing two primary datasets - the Crime Data table and the Person Involvement table - we seek to identify patterns, explore demographic factors, develop predictive models, and assess law enforcement efforts. Through this project, we aim to contribute to evidence-based crime prevention and enhance public safety measures.

## Cleaning
### Column Quality Check
Checked the quality of columns in the dataset to ensure appropriate data visualization.

## Data Transformation
- Removed the Latitude column from the crime data table due to all rows containing null values.
- Changed the data type of the datetime column to only date.
- Created a copy of this column and formatted it to display in AM/PM mode.
- Added 'Time Group' Column to the Crime Data table.

## Measures
- Time Group Column Addition
```
  Time Group =
    SWITCH(
        TRUE(),
        AND(MOD(HOUR('Crime Data'[Crime Time]), 24) >= 0, MOD(HOUR('Crime Data'[Crime Time]), 24) <= 3), "12:00 AM - 2:59 AM",
        AND(MOD(HOUR('Crime Data'[Crime Time]), 24) > 3, MOD(HOUR('Crime Data'[Crime Time]), 24) <= 6), "3:00 AM - 5:59 AM",
        AND(MOD(HOUR('Crime Data'[Crime Time]), 24) > 6, MOD(HOUR('Crime Data'[Crime Time]), 24) <= 9), "6:00 AM - 8:59 AM",
        AND(MOD(HOUR('Crime Data'[Crime Time]), 24) > 9, MOD(HOUR('Crime Data'[Crime Time]), 24) <= 12), "9:00 AM - 11:59 AM",
        AND(MOD(HOUR('Crime Data'[Crime Time]), 24) > 12, MOD(HOUR('Crime Data'[Crime Time]), 24) <= 15), "12:00 PM - 2:59 PM",
        AND(MOD(HOUR('Crime Data'[Crime Time]), 24) > 15, MOD(HOUR('Crime Data'[Crime Time]), 24) <= 18), "3:00 PM - 5:59 PM",
        AND(MOD(HOUR('Crime Data'[Crime Time]), 24) > 18, MOD(HOUR('Crime Data'[Crime Time]), 24) <= 21), "6:00 PM - 8:59 PM",
        AND(MOD(HOUR('Crime Data'[Crime Time]), 24) > 21, MOD(HOUR('Crime Data'[Crime Time]), 24) <= 24), "9:00 PM - 11:59 PM"
    )
 ```
- Date Table Addition
  DAX expression creates a more comprehensive date table with additional columns for "Year", "Month", and "MonthNum", “Weekday”, and “WeekNum” based on the range of dates in the crime data. 
 ```
  DateTable =
VAR _Mindate = YEAR(MIN('Crime Data'[Crime Date]))
VAR _Maxdate = YEAR(MAX('Crime Data'[Crime Date]))
RETURN
ADDCOLUMNS(
    FILTER(
        CALENDARAUTO(),
        YEAR([Date]) >= _Mindate && YEAR([Date]) <= _Maxdate
    ),
    "Year", YEAR([Date]),
    "Month", FORMAT([Date], "mmm"),
    "MonthNum", MONTH([Date]),
    "Weekday", FORMAT([Date], "ddd"),
    "WeekNum", WEEKDAY([Date]),
)
```
Date Table Configuration
Marked the generated table as a date table using the table tool, ensuring the format of dd/mm/yyyy in visualisations.

- Label (Year)
Calculates the year-over-year change in total crime count and provides a visual indicator (🔺 for increase, 🔻 for decrease) along with the absolute change.
```
Label (Year) =
VAR _PrevYr =
    CALCULATE(
        [Total Crime],
        SAMEPERIODLASTYEAR(DateTable[Date])
    )
    VAR _YOYchange =
    IF (_PrevYr<> BLANK(),
    [Total Crime] - _PrevYr,
    BLANK()
    )
        RETURN
    SWITCH(
    TRUE(),
     _YOYchange =0,  _YOYchange & "-",
     _YOYchange >=1, "🔺"  & _YOYchange,
     _YOYchange<1, "🔻" & _YOYchange
    )
 ```
- Label (Year)
  Calculates the month-over-month change in total crime count and provides a visual indicator (🔺 for increase, 🔻 for decrease) along with the absolute change.
 ```
Label(Month) =
    VAR _Pct_MoMChange =
    IF(
        DIVIDE(
            [Total Crime] - [Crime prevMonth],
            [Crime prevMonth]
        )
        <>BLANK(),
        DIVIDE(
            [Total Crime] - [Crime prevMonth],
            [Crime prevMonth]
        ),
        BLANK()
    )
    VAR _NegativePct = -0.01
    RETURN
        SWITCH(
            TRUE(),
            _Pct_MoMChange = 0, _Pct_MoMChange& "_",
            _Pct_MoMChange >= _NegativePct, "▲" & FORMAT(_Pct_MoMChange, "0%"),
            _Pct_MoMChange < _NegativePct, "▼" & FORMAT(_Pct_MoMChange, "0%")
        )
  ```
- Blank Measure (Year)
  Provides a default value of 200 for cases where no data is available.
 ```
 Blank Measure(Year) = 200
 ```
- Total Crime
  Counts the total number of rows in the crime data table, giving the total crime count.
 ```
Total Crime = COUNTROWS('Crime Data')
 ```
- Conditional Formatting for Year
  Applies conditional formatting to visualise the year-over-year change in total crime count. Grey indicates no change, green indicates an increase, and red indicates a decrease.
  ```
CF(Year) =
VAR _PrevYr =
    CALCULATE(
        [Total Crime],
        SAMEPERIODLASTYEAR(DateTable[Date])
    )
    VAR _YOYchange =
    IF (_PrevYr<> BLANK(),
    [Total Crime] - _PrevYr,
    BLANK()
    )
        RETURN
    SWITCH(
    TRUE(),
     _YOYchange =0,  "Grey",
     _YOYchange >=1, "Green",
     _YOYchange<1, "Red"
    )
  ```
- Conditional Formatting for Month
  Applies conditional formatting to visualise the month-over-month change in total crime count. Grey indicates no change, green indicates an increase, and red indicates a decrease.
  ```
 CF(Month) =
    VAR _Pct_MoMChange =
    IF(
        DIVIDE(
            [Total Crime] - [Crime prevMonth],
            [Crime prevMonth]
        )
        <>BLANK(),
        DIVIDE(
            [Total Crime] - [Crime prevMonth],
            [Crime prevMonth]
        ),
        BLANK()
    )
    VAR _NegativePct = -0.01
    RETURN
        SWITCH(
            TRUE(),
            _Pct_MoMChange = 0, "Grey",
            _Pct_MoMChange >= _NegativePct, "Green",
            _Pct_MoMChange < _NegativePct, "Red"
        )
  ```
- Crime Unresolved
  Calculates the percentage of unsolved crimes by dividing the count of unresolved crimes by the total crime count.
  ```
  Crime Unresolved =
    DIVIDE(
            CALCULATE(
                [Total Crime],
                'Crime Data'[Resolved] = 0
            ),
                [Total Crime]
    )
  ```
- Crime Resolved
  Calculates the percentage of resolved crimes by dividing the count of resolved crimes by the total crime count.
  ```
  Crime Resolved =
    DIVIDE(
            CALCULATE(
                [Total Crime],
                'Crime Data'[Resolved] = 1
            ),
                [Total Crime]
    )
  ```
- Crime Previous Year
  Calculates the total crime count for the previous year to facilitate year-over-year comparison.
  ```
  Crime prev. Year =
    CALCULATE(
        [Total Crime],
        SAMEPERIODLASTYEAR(DateTable[Date])
    )
   ```
## Data Modeling
Established a connection between the crime date column of the crime data table and the date part of the date table and checked if all the tables are connected together.

## Visualisation
### Cards
Added a card to display the total crime count.

### Text Box
Created a separate text box to display the percentages of unresolved and resolved crimes. Also added titles for each visual.

### Earliest Crime Time
Utilized a table to display the earliest crime time by filtering based on the "Total Crime" column.

### Crime Rate Map
Generated a map visualization to comprehend the crime rate on a country level.

### Month Table
Added a table to display the month part of the date table.

### Clustered Column Chart
Added a clustered column chart to exhibit the year-wise total crime and another one in other page to show crime type wise total crime (where Time-group and weekday added in drill through).

### Clustered Bar Chart
Added a clustered bar chart to exhibit the Time Group-wise total crime.

### Line Chart
Added line charts to exhibit different Time-wise total crime.

### Area Chart
Added line charts to exhibit different Time-wise and Month-wise total crime, and in drill through time-group and weekday added.

### Matrix (HeatMap)
Visualized Month, Weekday, and Total Crime fields in the matrix. Adjusted cell formatting to represent values with different background and font colors.

### Column Chart
Created a column chart to depict total crime versus each month.

### Slicer
Added a slicer to enable viewing all visuals for every crime from crime types.

### Crime Data Variation GIF
![Crime Data Variation](![crime-visual gif](https://github.com/roy-deblina/Crime_data_analytics/assets/164593876/8da44321-ea46-4c31-8386-8c7b5932ec2a)
)

This GIF demonstrates how different crime types vary over time, providing insights into the dynamics of criminal activities.

### Total Crime Variation Over Time
![Total Crime Variation](![drill_page](https://github.com/roy-deblina/Crime_data_analytics/assets/164593876/e8c99ee1-ed5a-45d4-b0a4-15245021a18e)
)

These illustrates the change in total crime occurrences over different times of the day, enabling a deeper understanding of crime patterns and trends.

## Conclusion

This README provides an overview of the Crime Project, including its objectives and the visualizations it encompasses. For further details, refer to the project documentation and codebase.
Based on general visual,
- Priority on Austria: Focus efforts on Austria, where the highest number of crimes occur.
- Seasonal Vigilance: Be prepared for crime spikes in October and September with targeted interventions.
- Nighttime Attention: Increase patrols and surveillance from 12:00 AM to 2:59 AM, the peak crime hours.
- Weekday Resources: Allocate resources strategically on weekdays when crime rates are consistently high.
- Targeted Approaches: Prioritize prevention and intervention efforts for violence and sexual harassment, the most prevalent crime types.


