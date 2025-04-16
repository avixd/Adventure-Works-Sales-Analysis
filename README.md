# Problem Statements
1.	Analyze historical KPIs (Sales, Profit,  Average order value etc.) over multiple dimensions ( Product, Location, Time etc.). 
2.	Identify bestselling products and the primary locations driving sales growth. 
3.	What were the demographics of the top-selling customers?. 

[Click here to download the analysis](https://github.com/avixd/dudaniavinash.github.io/blob/main/AdventureWorks_SalesAnalysis_v1.pbix)

# Adventure Works Sales Analysis
Using the Adventure Works Database, Analysis of Sales from 2015-2017 were performed using the "[AdventureWorks_Sales_Dataset](https://github.com/microsoft/powerbi-desktop-samples/blob/main/AdventureWorks%20Sales%20Sample/AdventureWorks%20Sales.xlsx)". 


# Sales Insights

 - Overall KPI Highlights and KPI Timeseries

![](https://github.com/avixd/dudaniavinash.github.io/blob/main/images/Overall%20KPI%20Highlights%20and%20KPI%20Timeseries.PNG)

```
- 4 of the 6 KPIs indicate YTD improvements.
- Cost and Average order value declined TY YTD vs LY YTD
- The time series indicates 3 months of consecutive revenue decline in FY 2017 April - June, versus the same period last year.
```

- YoY Metric Comparison
![](https://github.com/avixd/dudaniavinash.github.io/blob/main/images/YoY%20Metric%20Comparison.PNG)
```
- Central Region indicates a YoY decline in sales in FY 2018, whereas all sub categories had revenue growth YoY
```

 - Pareto Analysis
![](https://github.com/avixd/dudaniavinash.github.io/blob/main/images/Pareto%20Analysis.PNG)
```
- Top 3 Subcategories namely Road Bikes (10M or 43%), Mountain Bikes (8M or 35%), and Touring bikes (4M or 17%) account for 22M or 95% of Total Revenue
```
## Following functionalities available:

1. Dynamic Parameters enabling KPI selection.
2. This Year YTD vs Last Year YTD KPI Comparisons 
3. Timeseries (Drilldown from Fiscal Quarter -> Month -> Week) indicating YoY KPI declines.
4. Two interactive tables with dynamic dimension axis enabling drilling down into the root causes of trends.
5. A dynamic pareto chart which uses the 80-20 principle to identify best sellers.


## Steps to recreate the Sales Analysis:

- Step 1 : Create a KPI Field Parameter:

Either using the code below
```
Financial Metric = {
    ("Cost", NAMEOF('Key Measures'[Total Cost]), 1),
    ("Revenue", NAMEOF([Total Revenue]), 0),
    ("Profit", NAMEOF([Total Profit]), 2),
    ("Order", NAMEOF([Orders]), 3),
    ("AOV", NAMEOF([AOV]), 4),
    ("Units Sold", NAMEOF([Units Sold]), 5)
}
```

OR,  Using the "Modeling" Ribbon -> "New Parameter" -> "Fields" -> Select the fields i.e. Cost, Revenue, Profit etc.

- Step 2 : Create a slicer with the Financial Metric and enable "Single select" from the "Format" Pane -> "Slicer settings" -> "Options" -> "Selection"
![](https://github.com/avixd/dudaniavinash.github.io/blob/main/images/Single%20Select.PNG)


2. This Year YTD vs Last Year YTD KPI Comparisons 

[Link to the Date table](//https://www.daxpatterns.com/week-related-calculations/)

Date Table Configuration to ensure:

1. Fiscal Year:  1 April - 30 March
2. Fiscal Week: Monday - Sunday
 
```
VAR FirstFiscalMonth = 4       -- First month of the fiscal year
VAR FirstDayOfWeek = 1         -- 0 = Sunday, 1 = Monday, ...
VAR FirstSalesDate = MIN ( Sales[Order Date] )
VAR LastSalesDate = MAX ( Sales[Order Date] )
VAR TypeStartFiscalYear = 1     -- Fiscal year as Calendar Year of : 
                                -- 0 - First day of fiscal year
                                -- 1 - Last day of fiscal year
VAR QuarterWeekType = "445" -- Supports only "445", "454", and "544"
VAR WeeklyType = "Nearest" -- Use: "Nearest" or "Last" 
```

Create the measures below for the KPI Card
```
Sales YTD = IF (
    [ShowValueForDates],
    VAR LastDayAvailable = MAX ( 'Date'[Day of Fiscal Year Number] )
    VAR LastFiscalYearAvailable = MAX ( 'Date'[Fiscal Year Number] )
    VAR Result =
        CALCULATE (
            [Total Revenue],
            ALLEXCEPT ( 'Date', 'Date'[Working Day], 'Date'[Day of Week] ),
            'Date'[Day of Fiscal Year Number] <= LastDayAvailable,
            'Date'[Fiscal Year Number] = LastFiscalYearAvailable
        )
    RETURN
        Result
)
```

```
Sales PYTD = IF (
    [ShowValueForDates],
    VAR PreviousFiscalYear = MAX ( 'Date'[Fiscal Year Number] ) - 1
    VAR LastDayOfFiscalYearAvailable =
        CALCULATE (
            MAX ( 'Date'[Day of Fiscal Year Number] ),
            REMOVEFILTERS (               -- Remove filters from
                'Date'[Working Day],      -- filter-safe columns
                'Date'[Day of Week],      -- to get the last day with data
                'Date'[Day of Week Number] -- selected in the report
            ),
            'Date'[DateWithSales] = TRUE
        )
    VAR Result =
        CALCULATE (
            [Total Revenue],
            ALLEXCEPT ( 'Date', 'Date'[Working Day], 'Date'[Day of Week] ),
            'Date'[Fiscal Year Number] = PreviousFiscalYear,
            'Date'[Day of Fiscal Year Number] <= LastDayOfFiscalYearAvailable,
            'Date'[DateWithSales] = TRUE
        )
    RETURN
        Result
)
```

```
Sales YOY = 
VAR ValueCurrentPeriod = [Total Revenue]
VAR ValuePreviousPeriod = [Sales PY]
VAR Result =
    IF (
        NOT ISBLANK ( ValueCurrentPeriod )
            && NOT ISBLANK ( ValuePreviousPeriod ),
        ValueCurrentPeriod - ValuePreviousPeriod
    )
RETURN
    Result
```

```
Sales YOY % = DIVIDE (
    [Sales YOY],
    [Sales PY]
)
```

3. Timeseries (Drilldown from Fiscal Quarter -> Month -> Week) indicating YoY KPI declines.
4. Two interactive tables with dynamic dimension axis enabling drilling down into the root causes of trends.
5. A dynamic pareto chart which uses the 80-20 principle to identify best sellers.

