
# Analyzing Differences in Iowa Counties’ Spending

#### Alexander M. Todd - November 23rd, 2025

## Introduction

Government’s spend money on various different services. With many ways
to collect and spend money, there is significant variety in how a
Government allocated funds. The goal of this report is to understand the
differences in how the various counties in Iowa collect, allocate, and
spend its wealth.

The questions explored in this report are:

1.  
2.  
3.  
4.  

## Data

### Original Data

The data used in this report comes from the [State of Iowa Open Data
Website](https://data.iowa.gov/). There were 5 sets of data utilized in
this report:

1.  [Annual Personal Income by
    County](https://data.iowa.gov/Economic-Statistics/Annual-Personal-Income-for-State-of-Iowa-by-County/st2k-2ti2/about_data)

Included is the ‘Personal Income’ and ‘Personal Income Per Capita’ for
each Iowa County for each year between 1997 and 2022.

2.  [County Actual
    Revenues](https://data.iowa.gov/Local-Government-Finance/County-Actual-Revenues-By-Type-By-Fiscal-Year/fp2u-f5hd/about_data)

Included is the Total Revenue gained by each Iowa County for each year
between 2010 and 2024. There are many variables that split the Revenue
by type, most notably the revenue levied by Property Tax and that
provided by county issued goods and services.

3.  [County Budgeted
    Expenditures](https://data.iowa.gov/Local-Government-Finance/County-Budgeted-Expenditures-By-Service-Area-By-Fi/gk9s-gz9c/about_data)

Included is the budgeted spending for each Iowa County for each year
between 2005 and 2026. The budget is split based on type of spending for
which the money is allocated in the budget.

4.  [County Actual
    Expenditures](https://data.iowa.gov/Local-Government-Finance/County-Actual-Expenditures-by-Service-Area-by-Fisc/fxbr-vb9c/about_data)

Included is actual spending for each Iowa County for each year between
2010 and 2024. The spending is split based on type of spending for which
the money was used.

5.  [County
    Population](https://data.iowa.gov/Community-Demographics/County-Population-in-Iowa-by-Year/qtnr-zsrc/about_data)

Included is the number of people in each Iowa County for each year
between 1990 and 2024.

### Data Preparation and Cleaning

The first step in this process is to read the data sets using readr’s
read_csv function.

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ## ✔ ggplot2   3.5.2     ✔ tibble    3.3.0
    ## ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ## ✔ purrr     1.1.0     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(lubridate)

pop_raw <- read_csv("data/County_Population_in_Iowa_by_Year_20251123.csv")
```

    ## Rows: 3465 Columns: 5
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (3): County, Year, Primary Point
    ## dbl (1): FIPS
    ## num (1): Population
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
# view(pop)

income_raw <- read_csv("data/Annual_Personal_Income_for_State_of_Iowa_by_County_20251123.csv")
```

    ## Rows: 5148 Columns: 9
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (8): Row ID, Name, Variable Code, Variable, Value, Variable Unit, Date, ...
    ## dbl (1): Geography ID
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
# view(income)

rev_raw <- read_csv("data/County_Actual_Revenues_By_Type_By_Fiscal_Year_20251123.csv")
```

    ## Rows: 1485 Columns: 21
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (1): COUNTY NAME
    ## dbl  (2): FISCAL YEAR, COUNTY NUMBER
    ## num (17): TAXES LEVIED ON PROPERTY, LESS: UNCOLLECTED DELINQUENT TAXES - LEV...
    ## lgl  (1): LOCATION
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
# view(rev)

budget_raw <- read_csv("data/County_Budgeted_Expenditures_By_Service_Area_By_Fiscal_Year_20251123.csv")
```

    ## Rows: 2178 Columns: 22
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (20): Unique Line ID, Fiscal Year Ending, County Name, Public Safety & L...
    ## dbl  (1): County
    ## lgl  (1): Primary County Coordinates
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
# view(budget)

spend_raw <- read_csv("data/County_Actual_Expenditures_by_Service_Area_by_Fiscal_Year_20251123.csv")
```

    ## Rows: 1485 Columns: 18
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (1): County
    ## dbl  (2): FISCAL YEAR, COUNTY NUMBER
    ## num (14): PUBLIC SAFETY AND LEGAL SERVICES, PHYSICAL HEALTH SOCIAL SERVICES,...
    ## lgl  (1): Primary County Coordinates
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
# view(spend)
```

The next step is to begin cleaning each data set on various thing: 1.
Selecting the correct attributes as the proper variable type 2. Making a
cohesive date and county name attribute to make joining tables easier 3.
Filter out data with a year not in a shared range. In this case, there
is sufficient data in each data set to use the years 2010 to 2024. 4.
Editing any variables to make them look nice and creating any calculated
fields that might be important later.

*Population Data*

In cleaning the Iowa County Population data, the only requirement was
reformatting the county name as just the name of the county in all
uppercase (for example, “Story County” becomes “STORY”) as well as
extracting the year from the date, filtering by years between 2010 and
2024 and only keeping County, Year, and Population columns.

``` r
pop <- pop_raw |>
  mutate(
    County = toupper(substring(County, 1, nchar(County) - 7)),
    Year = year(mdy(Year))
  ) |>
  filter(Year >= 2010, Year <= 2024) |>
  select(County, Year, Population) |>
  str()
```

    ## tibble [1,485 × 3] (S3: tbl_df/tbl/data.frame)
    ##  $ County    : chr [1:1485] "STORY" "CLAYTON" "JOHNSON" "JEFFERSON" ...
    ##  $ Year      : num [1:1485] 2011 2012 2021 2018 2016 ...
    ##  $ Population: num [1:1485] 91136 17946 155047 18256 222188 ...

*Income Data*

In cleaning the Iowa County Annual Personal Income Data, the County
names had to be reformatted in uppercase and the year needed to be
extracted from the date. The data had to be filtered between the years
2010 and 2024. Finally, the pivot_wider function was utilized to make
distinct columns for “Personal Income” and “Personal Income Per Capita”
and because the values utilized a dollar sign, the parse_number function
was utilized to make it numerical and remove the dollar sign. Finally,
as the personal income was in thousands of dollars, it was multiplied by
1000 to make it consistent and easily used.

``` r
income_1 <- income_raw |>
  mutate(
    County = toupper(Name),
    Year = year(mdy(Date))
  ) |>
  filter(Year >= 2010, Year <= 2024) |>
  select(County, Year, Variable, Value)

income <- income_1 |>
  pivot_wider(names_from = Variable,
              values_from = Value
              ) |>
  rename('Personal_Income' = `Personal income`,
         'Personal_Income_Per_Capita' = `Per capita personal income`
         ) |>
  mutate(
    Personal_Income = parse_number(Personal_Income),
    Personal_Income_Per_Capita = parse_number(Personal_Income_Per_Capita),
    Personal_Income = Personal_Income * 1000
  ) |>
  str()
```

    ## tibble [1,287 × 4] (S3: tbl_df/tbl/data.frame)
    ##  $ County                    : chr [1:1287] "ADAIR" "ADAIR" "ADAIR" "ADAIR" ...
    ##  $ Year                      : num [1:1287] 2010 2011 2012 2013 2014 ...
    ##  $ Personal_Income           : num [1:1287] 2.69e+08 3.06e+08 3.09e+08 3.24e+08 3.27e+08 ...
    ##  $ Personal_Income_Per_Capita: num [1:1287] 35015 40249 40842 43033 43300 ...

*Revenue Data*

In cleaning the Iowa County Actual Revenue Data, the County name was
standardized to fit the other data sets to be all uppercase letters. The
important variable here to keep was “Total Revenue” and simple
percentages of how much of the County’s total revenue came from what.

“Revenue_Percent_Property_Tax” is the percent of Total Revenue that
comes from Property Taxes (in addition to the “Delinquent” or late
property taxes either paid or failed to be paid during the year).

“Revenue_Percent_Grant” is the percent of Total Revenue that comes from
“Intergovernmental” sources, most commonly as grants from the State of
Iowa or the Federal Government.

“Revenue_Percent_Activities” is the percent of Total Revenue that comes
from people choosing to spend money for Government goods and services
such as rent, interest from borrowed money, licenses, and permits.

“Revenue_Percent_Other” is simply the percent of Total Revenue that
comes from any other source.

``` r
rev <- rev_raw |>
  rename(
    Year = `FISCAL YEAR`,
    County = `COUNTY NAME`,
    Total_Revenue = `TOTAL REVENUES & OTHER SOURCES`
  ) |>
  mutate(
    County = toupper(County),
    Revenue_Percent_Property_Tax = round((`NET CURRENT PROPERTY TAXES` + `DELINQUENT PROPERTY TAX REVENUE`) / Total_Revenue, 4),
    Revenue_Percent_Grant = round(INTERGOVERNMENTAL / Total_Revenue, 4),
    Revenue_Percent_Activities = round((`LICENSES & PERMITS` + `CHARGES FOR SERVICE` + `USE OF MONEY & PROPERTY`) / Total_Revenue, 4),
    Revenue_Percent_Other = round(1 - Revenue_Percent_Property_Tax - Revenue_Percent_Grant - Revenue_Percent_Activities, 4)
  ) |>
  filter(Year >= 2010, Year <= 2024) |>
  select(County, Year, Total_Revenue, Revenue_Percent_Property_Tax, Revenue_Percent_Grant, Revenue_Percent_Activities, Revenue_Percent_Other) |>
  str()
```

    ## tibble [1,485 × 7] (S3: tbl_df/tbl/data.frame)
    ##  $ County                      : chr [1:1485] "MONROE" "JOHNSON" "ADAMS" "MUSCATINE" ...
    ##  $ Year                        : num [1:1485] 2019 2016 2019 2010 2023 ...
    ##  $ Total_Revenue               : num [1:1485] 3.13e+06 1.13e+08 9.73e+06 4.02e+07 1.54e+07 ...
    ##  $ Revenue_Percent_Property_Tax: num [1:1485] 1 0.435 0.351 0.32 0.365 ...
    ##  $ Revenue_Percent_Grant       : num [1:1485] 0 0.181 0.36 0.235 0.397 ...
    ##  $ Revenue_Percent_Activities  : num [1:1485] 0 0.0479 0.0605 0.0328 0.0499 0.0559 0.0441 0.0337 0.0404 0.0364 ...
    ##  $ Revenue_Percent_Other       : num [1:1485] 0 0.336 0.228 0.412 0.188 ...

### Data Structure

## Results

## Conclusion
