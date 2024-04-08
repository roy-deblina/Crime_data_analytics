# Crime_data_analytics

Crime Project

Introduction
The Crime Data Analysis Project aims to utilise data-driven insights to understand and address criminal activities more effectively. By analysing two primary datasets - the Crime Data table and the Person Involvement table - we seek to identify patterns, explore demographic factors, develop predictive models, and assess law enforcement efforts. Through this project, we aim to contribute to evidence-based crime prevention and enhance public safety measures.
   #Cleaning
Column Quality Check
Checked the quality of columns in the dataset to ensure the appropriate data visualisation.
#Data Transformation
Removed the Latitude column from the crime data table due to all rows containing null values.
Changed the data type of the datetime column to only date. Also, created a copy of this column and formatted it to display in AM/PM mode.
‘Time Group’ Column added to the Crime Data table 
Following measures added accordingly for better insight of the problem.

Time Group Column Addition
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

Date Table Addition
DAX expression generates a basic date table with a "Year" column.
DateTable =
ADDCOLUMNS(
    CALENDARAUTO(),
    "Year", YEAR([Date])
    )


DAX expression creates a more comprehensive date table with additional columns for "Year", "Month", and "MonthNum", “Weekday”, and “WeekNum” based on the range of dates in the crime data. 
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


Date Table Configuration
Marked the generated table as a date table using the table tool, ensuring the format of dd/mm/yyyy in visualisations.
Measures
Label (Year)
Calculates the year-over-year change in total crime count and provides a visual indicator (🔺 for increase, 🔻 for decrease) along with the absolute change.

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


Label (Year)
Calculates the month-over-month change in total crime count and provides a visual indicator (🔺 for increase, 🔻 for decrease) along with the absolute change.


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



Blank Measure (Year)
Provides a default value of 200 for cases where no data is available.
Blank Measure(Year) = 200


Total Crime
Counts the total number of rows in the crime data table, giving the total crime count.
Total Crime = COUNTROWS('Crime Data')

Conditional Formatting for Year
Applies conditional formatting to visualise the year-over-year change in total crime count. Grey indicates no change, green indicates an increase, and red indicates a decrease.
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


Conditional Formatting for Month
Applies conditional formatting to visualise the month-over-month change in total crime count. Grey indicates no change, green indicates an increase, and red indicates a decrease.


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




Crime Unresolved
Calculates the percentage of unsolved crimes by dividing the count of unresolved crimes by the total crime count.

Crime Unresolved =
    DIVIDE(
            CALCULATE(
                [Total Crime],
                'Crime Data'[Resolved] = 0
            ),
                [Total Crime]
    )


Crime Resolved
Calculates the percentage of resolved crimes by dividing the count of resolved crimes by the total crime count.

Crime Resolved =
    DIVIDE(
            CALCULATE(
                [Total Crime],
                'Crime Data'[Resolved] = 1
            ),
                [Total Crime]
    )


Crime Previous Year
Calculates the total crime count for the previous year to facilitate year-over-year comparison.
Crime prev. Year =
    CALCULATE(
        [Total Crime],
        SAMEPERIODLASTYEAR(DateTable[Date])
    )

Crime Previous Month
Calculates the total crime count for the previous month to facilitate month-over-month comparison.
Crime prevMonth =
    CALCULATE(
        [Total Crime],
        DATEADD(DateTable[Date], -1, Month)
    )


Data Modeling
Established a connection between the crime date column of the crime data table and the date part of the date table and checked if all the tables are connected together.

Visualisation Part
#Cards
Added a card to display the total crime count.
Text Box
Created a separate text box to display the percentages of unresolved and resolved crimes.
Text box to generate titles for each visual and the main titles has been added.


Emojis were added using the windows + ; button.
Earliest Crime Time
Utilised a table to display the earliest crime time by filtering based on the "Total Crime" column. Applied a basic filter with "Top N" selected and set to 1 to display the earliest time. Converted the visual to a card. And add a card to enter it as dangerous crime time.
Crime Rate Map
Generated a map visualisation to comprehend the crime rate on a country level.
Total crime added in Bubble size and in bubble colour settings fx, format style was set to gradient, with field name ‘Total Crime’, and lowest value and maximum value is set as blue and red respectively.


Month Table
Added a table to display the month part of the date table.
Clustered Column Chart
Added a clustered column chart to exhibit the year-wise total crime.
Configure 'blank' measure and 'total crime' on the y-axis and 'year' on the x-axis.
Enabled data labels and the orientation was set to horizontal, set the y-axis value to none for better visualisation.
Added the 'label(year)' measure in the series.


In the details sheet for more information, added a clustered column chart to exhibit the crime type-wise total crime to understand what type of crimes are happening at what rate and Time Group added in drill-through.



Clustered Bar Chart
Added a clustered bar chart to exhibit the Time Group-wise total crime.
Added field name ‘Total Crime’ inside the field value and the bar colour fx to maximum and minimum values to red and green respectively similar to matrix colour.

Line Chart
#1
Added a line chart to exhibit the different Time-wise total crime.
#2
A visual to represent the month-wise total crime percentage is generated.
A line chart is selected, with the field value format set to "Label (Month)".
Data labels are turned on, and "Blank measure (Month)" is chosen in the series.
In the values part, "Label (Month)" is selected, and conditional formatting (CF) is applied only in the colour fx section.
The line associated with the blank measure is removed from the visual.
In the format options, specifically the lines section, "Blank measure (Month)" is selected in the series part.
The stroke width is reduced to 0 to eliminate the line.
The blank measure is modified to effectively highlight differences in the visual.




Matrix visualisation (HeatMap)

"Month," "Weekday," and "Total Crime" fields are added to the matrix.
Both column and row subtotals and totals are disabled in the formatting options.
Cell formatting settings are adjusted to change the background and font colour, setting the highest and lowest values to green and red respectively.
#Column Chart

Another column chart is created to depict total crime versus each month.
This chart is positioned atop the heatmap for flexible visual adjustments.


A horizontal column chart is created to illustrate crime occurrences across weekdays.
This chart is positioned beside the heatmap, aligning with the weekday data for a cohesive visualisation (look at the visual for better understanding).

For both cases, add the field name ‘Total Crime’ inside the field value and the bar colour fx to maximum and minimum values to red and green respectively similar to matrix colour.


Slicer
A slicer is added to enable viewing all visuals for every crime from crime types and the slicer setting was changed to dropdown.



Conclusion

Priority on Austria: Focus efforts on Austria, where the highest number of crimes occur.

Seasonal Vigilance: Be prepared for crime spikes in October and September with targeted interventions.

Nighttime Attention: Increase patrols and surveillance from 12:00 AM to 2:59 AM, the peak crime hours.

Weekday Resources: Allocate resources strategically on weekdays when crime rates are consistently high.

Targeted Approaches: Prioritize prevention and intervention efforts for violence and sexual harassment, the most prevalent crime types.










