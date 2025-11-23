
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
# view(pop_raw)

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
# view(income_raw)

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
# view(rev_raw)

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
# view(budget_raw)

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
# view(spend_raw)
```

The next step is to begin cleaning each data set on various thing: 1.
Selecting the correct attributes as the proper variable type 2. Making a
cohesive date and county name attribute to make joining tables easier 3.
Filter out data with a year not in a shared range. In this case, there
is sufficient data in each data set to use the years 2010 to 2022. 4.
Editing any variables to make them look nice and creating any calculated
fields that might be important later.

*Population Data*

In cleaning the Iowa County Population data, the only requirement was
reformatting the county name as just the name of the county in all
uppercase (for example, “Story County” becomes “STORY”) as well as
extracting the year from the date, filtering by years between 2010 and
2024 and only keeping County, Year, and Population columns.

``` r
pop_raw$County <- replace(pop_raw$County, pop_raw$County == "O'Brien County", "Obrien County")

pop <- pop_raw |>
  mutate(
    County = toupper(substring(County, 1, nchar(County) - 7)),
    Year = year(mdy(Year))
  ) |>
  filter(Year >= 2010, Year <= 2022) |>
  select(County, Year, Population)

str(pop)
```

    ## tibble [1,287 × 3] (S3: tbl_df/tbl/data.frame)
    ##  $ County    : chr [1:1287] "STORY" "CLAYTON" "JOHNSON" "JEFFERSON" ...
    ##  $ Year      : num [1:1287] 2011 2012 2021 2018 2016 ...
    ##  $ Population: num [1:1287] 91136 17946 155047 18256 222188 ...

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
income_raw$Name <- replace(income_raw$Name, income_raw$Name == "O'Brien", "Obrien")

income_1 <- income_raw |>
  mutate(
    County = toupper(Name),
    Year = year(mdy(Date))
  ) |>
  filter(Year >= 2010, Year <= 2022) |>
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
  )

str(income)
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
  filter(Year >= 2010, Year <= 2022) |>
  select(County, Year, Total_Revenue, Revenue_Percent_Property_Tax, Revenue_Percent_Grant, Revenue_Percent_Activities, Revenue_Percent_Other)

str(rev)
```

    ## tibble [1,287 × 7] (S3: tbl_df/tbl/data.frame)
    ##  $ County                      : chr [1:1287] "MONROE" "JOHNSON" "ADAMS" "MUSCATINE" ...
    ##  $ Year                        : num [1:1287] 2019 2016 2019 2010 2020 ...
    ##  $ Total_Revenue               : num [1:1287] 3.13e+06 1.13e+08 9.73e+06 4.02e+07 9.94e+06 ...
    ##  $ Revenue_Percent_Property_Tax: num [1:1287] 1 0.435 0.351 0.32 0.264 ...
    ##  $ Revenue_Percent_Grant       : num [1:1287] 0 0.181 0.36 0.235 0.34 ...
    ##  $ Revenue_Percent_Activities  : num [1:1287] 0 0.0479 0.0605 0.0328 0.0559 0.0441 0.0364 0.0284 0.0357 0.0663 ...
    ##  $ Revenue_Percent_Other       : num [1:1287] 0 0.336 0.228 0.412 0.34 ...

*Budgeted Expenditure Data*

In cleaning the Iowa County Budgeted Expenditures Data, the County Name
was already in the desired format. The year was extracted from the date
and used to filter to ensure the data was between 2010 and 2024. “Total
Budget” was kept and the various types of spending were transformed as a
percentage of “Total Budget”. The parse_number function was utilized as
the numbers used the dollar sign and commas.

``` r
budget_raw$`County Name` <- replace(budget_raw$`County Name`, budget_raw$`County Name` == "O'BRIEN", "OBRIEN")

budget <- budget_raw |>
  rename(
    County_Id = County,
    County = `County Name`,
    Year = `Fiscal Year Ending`,
    Total_Budget = `Total Expenditures`
  ) |>
  mutate(
    Year = year(mdy(Year)),
    Total_Budget = parse_number(Total_Budget),
    Budgeted_Public_Safety_Percent = parse_number(`Public Safety & Legal Services`) / Total_Budget,
    Budgeted_Mental_Health_Percent = parse_number(`Mental Health MR, DD`) / Total_Budget,
    Budgeted_Roads_Transportation_Percent = parse_number(`Roads & Transportation`) / Total_Budget,
    Budgeted_Administration_Percent = parse_number(Administration) / Total_Budget,
    Budgeted_Debt_Percent = parse_number(`Debt Service`) / Total_Budget,
    Budgeted_Physical_Health_Percent = parse_number(`Physical Health & Social Services`) / Total_Budget,
    Budgeted_Environment_Education_Percent = parse_number(`County Environment & Education`) / Total_Budget,
    Budgeted_Services_to_Residents_Percent = parse_number(`Government Services to Residents`) / Total_Budget,
    Budgeted_Refunded_Programs_Percent = parse_number(`Nonprogram Current`) / Total_Budget,
    Budgeted_Projects_Percent = parse_number(`Capital Projects`) / Total_Budget,
    Budgeted_Transfers_Out_Percent = parse_number(`Operating Transfers Out`) / Total_Budget,
    Budgeted_Refunded_Debt_Percent = parse_number(`Refunded Debt/Escrow`) / Total_Budget
  ) |>
  select(County, Year, Total_Budget, Budgeted_Public_Safety_Percent, Budgeted_Mental_Health_Percent, Budgeted_Roads_Transportation_Percent, Budgeted_Administration_Percent, Budgeted_Debt_Percent, Budgeted_Physical_Health_Percent, Budgeted_Environment_Education_Percent, Budgeted_Services_to_Residents_Percent, Budgeted_Refunded_Programs_Percent, Budgeted_Projects_Percent, Budgeted_Transfers_Out_Percent, Budgeted_Refunded_Debt_Percent) |>
  filter(Year >= 2010, Year <= 2022)

str(budget)
```

    ## tibble [1,287 × 15] (S3: tbl_df/tbl/data.frame)
    ##  $ County                                : chr [1:1287] "WEBSTER" "IDA" "IDA" "MILLS" ...
    ##  $ Year                                  : num [1:1287] 2011 2015 2016 2020 2021 ...
    ##  $ Total_Budget                          : num [1:1287] 28919272 8345191 8148853 24736697 17165027 ...
    ##  $ Budgeted_Public_Safety_Percent        : num [1:1287] 0.17 0.0958 0.0973 0.1588 0.18 ...
    ##  $ Budgeted_Mental_Health_Percent        : num [1:1287] 0.1744 0.0474 0.0307 0.0112 0.0182 ...
    ##  $ Budgeted_Roads_Transportation_Percent : num [1:1287] 0.218 0.364 0.433 0.259 0.32 ...
    ##  $ Budgeted_Administration_Percent       : num [1:1287] 0.0876 0.0906 0.0942 0.1811 0.0875 ...
    ##  $ Budgeted_Debt_Percent                 : num [1:1287] 0.0198 0 0 0.0542 0 ...
    ##  $ Budgeted_Physical_Health_Percent      : num [1:1287] 0.0817 0.0368 0.0405 0.0926 0.047 ...
    ##  $ Budgeted_Environment_Education_Percent: num [1:1287] 0.0404 0.0465 0.0493 0.0688 0.0674 ...
    ##  $ Budgeted_Services_to_Residents_Percent: num [1:1287] 0.0274 0.0315 0.0341 0.0257 0.0316 ...
    ##  $ Budgeted_Refunded_Programs_Percent    : num [1:1287] 0.00692 0 0 0 0 ...
    ##  $ Budgeted_Projects_Percent             : num [1:1287] 0.102 0.1552 0.0841 0.0437 0.1503 ...
    ##  $ Budgeted_Transfers_Out_Percent        : num [1:1287] 0.0719 0.1325 0.1365 0.1052 0.0979 ...
    ##  $ Budgeted_Refunded_Debt_Percent        : num [1:1287] 0 0 0 0 0 0 0 0 0 0 ...

*Actual Expenditure Data*

In cleaning the Iowa County Actual Expenditures Data, the County Name
and Year were already in the desired format. The year was used to filter
to ensure the data was between 2010 and 2024. “Total Expenditures” was
kept and the various types of spending were transformed as a percentage
of “Total Budget”.

``` r
spend <- spend_raw |>
  rename(
    Year = `FISCAL YEAR`,
    Total_Expenditure = `TOTAL EXPENDITURES & OTHER USES`
  ) |>
  mutate(
    Actual_Public_Safety_Percent = `PUBLIC SAFETY AND LEGAL SERVICES` / Total_Expenditure,
    Actual_Mental_Health_Percent = `MENTAL HEALTH, ID & DD` / Total_Expenditure,
    Actual_Roads_Transportation_Percent = `ROADS & TRANSPORTATION` / Total_Expenditure,
    Actual_Administration_Percent = ADMINISTRATION / Total_Expenditure,
    Actual_Debt_Percent = `DEBT SERVICE` / Total_Expenditure,
    Actual_Physical_Health_Percent = `PHYSICAL HEALTH SOCIAL SERVICES` / Total_Expenditure,
    Actual_Environment_Education_Percent = `COUNTY ENVIRONMENT AND EDUCATION` / Total_Expenditure,
    Actual_Services_to_Residents_Percent = `GOVERNMENT SERVICES TO RESIDENTS` / Total_Expenditure,
    Actual_Refunded_Programs_Percent = `NONPROGRAM CURRENT` / Total_Expenditure,
    Actual_Projects_Percent = `CAPITAL PROJECTS` / Total_Expenditure,
    Actual_Transfers_Out_Percent = `OPERATING TRANSFERS OUT` / Total_Expenditure,
    Actual_Refunded_Debt_Percent = `REFUNDED DEBT/PAYMENTS TO ESCROW` / Total_Expenditure
  ) |>
  select(County, Year, Total_Expenditure, Actual_Public_Safety_Percent, Actual_Mental_Health_Percent, Actual_Roads_Transportation_Percent, Actual_Administration_Percent, Actual_Debt_Percent, Actual_Physical_Health_Percent, Actual_Environment_Education_Percent, Actual_Services_to_Residents_Percent, Actual_Refunded_Programs_Percent, Actual_Projects_Percent, Actual_Transfers_Out_Percent, Actual_Refunded_Debt_Percent) |>
  filter(Year >= 2010, Year <= 2022)

str(spend)
```

    ## tibble [1,287 × 15] (S3: tbl_df/tbl/data.frame)
    ##  $ County                              : chr [1:1287] "JOHNSON" "ADAMS" "MUSCATINE" "IDA" ...
    ##  $ Year                                : num [1:1287] 2016 2019 2010 2020 2018 ...
    ##  $ Total_Expenditure                   : num [1:1287] 1.07e+08 9.93e+06 3.57e+07 2.15e+07 1.34e+07 ...
    ##  $ Actual_Public_Safety_Percent        : num [1:1287] 0.1961 0.1475 0.2031 0.0449 0.1742 ...
    ##  $ Actual_Mental_Health_Percent        : num [1:1287] 0.07158 0.018 0.13283 0.00982 0.03449 ...
    ##  $ Actual_Roads_Transportation_Percent : num [1:1287] 0.0843 0.3555 0.1821 0.1447 0.322 ...
    ##  $ Actual_Administration_Percent       : num [1:1287] 0.0715 0.0781 0.0719 0.0422 0.1021 ...
    ##  $ Actual_Debt_Percent                 : num [1:1287] 0.1347 0.0878 0.1073 0.0249 0 ...
    ##  $ Actual_Physical_Health_Percent      : num [1:1287] 0.08376 0.01998 0.03935 0.00995 0.02598 ...
    ##  $ Actual_Environment_Education_Percent: num [1:1287] 0.0372 0.0872 0.0514 0.0378 0.0748 ...
    ##  $ Actual_Services_to_Residents_Percent: num [1:1287] 0.0201 0.0332 0.022 0.0143 0.038 ...
    ##  $ Actual_Refunded_Programs_Percent    : num [1:1287] 2.77e-05 0.00 2.35e-03 0.00 7.87e-03 ...
    ##  $ Actual_Projects_Percent             : num [1:1287] 0.1051 0.0621 0.1134 0.5879 0.0708 ...
    ##  $ Actual_Transfers_Out_Percent        : num [1:1287] 0.1956 0.1106 0.0742 0.0836 0.1497 ...
    ##  $ Actual_Refunded_Debt_Percent        : num [1:1287] 0 0 0 0 0 0 0 0 0 0 ...

Now that each data set has been cleaned individually, they can now be
joined on “County” and “Year”.

``` r
join1 <- inner_join(pop, income, by = c('County', 'Year'))
join2 <- inner_join(join1, rev, by = c('County', 'Year'))
join3 <- inner_join(join2, budget, by = c('County', 'Year'))
data <- inner_join(join3, spend, by = c('County', 'Year'))

# data |>
#  group_by(Year) |>
#  summarize(
#    count = n_distinct(County)
#  )

# sort(unique(data$County))
```

After checking that the join worked, it seemed that the use of an
apostrophe in some datasets for O’Brien county did not work with the
ones that did not so it was necessary to ensure this was fixed.

### Data Structure

## Results

## Conclusion
