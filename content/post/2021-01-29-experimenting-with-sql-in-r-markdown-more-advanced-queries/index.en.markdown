---
title: "More experimenting with SQL in R Markdown: Pivoting data,
  outputting results to R, and creating a summary table using gt"
author: Lynley Aldridge
date: '2021-01-30'
tags:
  - Rstats
  - SQL
  - Tutorial
  - TidyTuesday
draft: yes
slug: experimenting-with-sql-in-r-markdown-more-join-queries
lastmod: '2021-01-30T20:01:24+11:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
---

Following on from my [previous post](/2021/01/13/experimenting-with-sql/), I use SELECT and JOIN statements to pivot the Taylor Swift and Beyoncé Tidy Tuesday data using RSQLite, output the results to R, and create a relatively simple summary table using gt. 

# Setup

As in the previous post, load packages, read in data, set-up a connection to the database and copy in the data, and clean release dates:


```r
#load packages
library(tidyverse)
library(odbc)
library(DBI)
library(RSQLite)
library(gt) # for tables

#load data
sales <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2020/2020-09-29/sales.csv')
charts <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2020/2020-09-29/charts.csv')

#create connection to database and copy data into database
con <- dbConnect(RSQLite::SQLite(), ":memory:")
copy_to(con, sales)
copy_to(con, charts)

#clean release dates as per previous post
dbExecute(con, "
UPDATE charts
SET released = TRIM(SUBSTR(released, 1, INSTR(released, ' (')))
WHERE INSTR(released, ' (')>0; 
                          ")
```

```
## [1] 20
```

# Previewing tables using SELECT and LIMIT queries 

Let's quickly refresh our understanding of how this data is structured. 


```r
dbGetQuery(con, '
SELECT artist, title, country, sales, released
FROM sales
LIMIT 5
                          ')
```

```
##         artist        title country    sales          released
## 1 Taylor Swift Taylor Swift      US  5720000  October 24, 2006
## 2 Taylor Swift     Fearless      WW 12000000 November 11, 2008
## 3 Taylor Swift     Fearless      US  7180000 November 11, 2008
## 4 Taylor Swift     Fearless     AUS   500000 November 11, 2008
## 5 Taylor Swift     Fearless      UK   609000 November 11, 2008
```


```r
dbGetQuery(con, '
SELECT artist, title, chart, chart_position, released
FROM charts
LIMIT 5
                          ')
```

```
##         artist        title chart chart_position         released
## 1 Taylor Swift Taylor Swift    US              5 October 24, 2006
## 2 Taylor Swift Taylor Swift   AUS             33 October 24, 2006
## 3 Taylor Swift Taylor Swift   CAN             14 October 24, 2006
## 4 Taylor Swift Taylor Swift   FRA              — October 24, 2006
## 5 Taylor Swift Taylor Swift   GER              — October 24, 2006
```

# Pivoting sales data using SELECT WHERE and JOIN queries

Please note that I am working with RSQLite in these examples in order to learn more about using SQL in R, rather than presenting this as the best way to conduct these analyses.

## Revising simple JOIN queries

First, let's review the JOIN query we used in the previous post to combine data from the charts and sales tables:


```r
dbGetQuery(con, '
SELECT charts.artist, charts.title, SUBSTR(charts.released, -4) as released, charts.chart, charts.chart_position as position, sales.sales  
FROM charts
LEFT JOIN sales
ON charts.chart = sales.country and
charts.title = sales.title
WHERE charts.chart IN ("US", "UK")
ORDER BY charts.artist DESC, released ASC
LIMIT 5
                          ')
```

```
##         artist        title released chart position   sales
## 1 Taylor Swift Taylor Swift     2006    US        5 5720000
## 2 Taylor Swift Taylor Swift     2006    UK       81      NA
## 3 Taylor Swift     Fearless     2008    US        1 7180000
## 4 Taylor Swift     Fearless     2008    UK        5  609000
## 5 Taylor Swift    Speak Now     2010    US        1 4694000
```

## Pivoting sales data using SELECT WHERE and JOIN queries

To pivot data so that sales data for each country is presented in a new column (using RSQLite), we can use a series of SELECT and JOIN statements. 
The code below offers a simple illustration, using SELECT WHERE statements (in parentheses) to SELECT relevant rows (i.e., `SELECT artist, title, chart_position FROM charts WHERE chart = "US"` as a temporary table called `US_charts`) and SELECT and JOIN statements to combine extracted columns from the specified temporary tables with the sales and charts tables. (This code was created for sqlite using modified examples from [Peruz and Jeremy Field](https://stackoverflow.com/questions/37170958/how-do-i-pivot-values-into-columns-with-sqlite) on stackoverflow.) Note that specific code for pivoting data varies depending on SQL vendor, and Microsoft SQL server uses [PIVOT and UNPIVOT](https://docs.microsoft.com/en-us/sql/t-sql/queries/from-using-pivot-and-unpivot?view=sql-server-ver15) operators. 

We can also add calculated fields to this query to show what percentage of each album's  worldwide sales was comprised by sales in a specified country (e.g., `SELECT US_sales.sales/WW_sales.sales*100 AS US_percent` creates a `US_percent` column, by dividing US sales by worldwide sales, then multiplying by 100). The `round()` wrapper rounds the result of the formula defining `US_percent` to the number of decimal points specified after the comma. Note that column names included in calculations must be specified in full (e.g., `US_sales.sales`), and not by aliases that will not be assigned until the SELECT statement runs (e.g., `US_sales`).

What follows should be seen as a proof of concept, which I've used to extract just US charts and sales data:


```r
dbGetQuery(con, '
SELECT charts.artist, charts.title, SUBSTR(charts.released, -4) AS year, 
  US_charts.chart_position as US_chart,
  US_sales.sales AS US_sales, 
  WW_sales.sales AS WW_sales, 
  round(US_sales.sales/WW_sales.sales*100, 1) AS US_percent
FROM charts
LEFT JOIN (SELECT artist, title, chart_position FROM charts WHERE chart = "US") AS US_charts
ON charts.title = US_charts.title
LEFT JOIN sales
ON charts.chart = sales.country and charts.title = sales.title
LEFT JOIN (SELECT artist, title, sales FROM sales WHERE country = "US") AS US_sales
  ON sales.title = US_sales.title
LEFT JOIN (SELECT artist, title, sales FROM sales WHERE country = "WW" OR country = "World") AS WW_sales
  ON sales.title = WW_sales.title
GROUP BY charts.artist, charts.title
ORDER BY charts.artist DESC, year ASC;
                          ')
```

```
##          artist                title year US_chart US_sales WW_sales US_percent
## 1  Taylor Swift         Taylor Swift 2006        5  5720000       NA         NA
## 2  Taylor Swift             Fearless 2008        1  7180000 12000000       59.8
## 3  Taylor Swift            Speak Now 2010        1  4694000  5000000       93.9
## 4  Taylor Swift                  Red 2012        1  4465000  6000000       74.4
## 5  Taylor Swift                 1989 2014        1  6215000 10100000       61.5
## 6  Taylor Swift           Reputation 2017        1  2300000  4500000       51.1
## 7  Taylor Swift                Lover 2019        1  1085000  3200000       33.9
## 8  Taylor Swift             Folklore 2020        1       NA       NA         NA
## 9       Beyoncé  Dangerously in Love 2003        1  5100000 11000000       46.4
## 10      Beyoncé                B'Day 2006        1  3610000  8000000       45.1
## 11      Beyoncé I Am... Sasha Fierce 2008        1  3380000  8000000       42.3
## 12      Beyoncé                    4 2011        1  1500000       NA         NA
## 13      Beyoncé              Beyoncé 2013        1  2512000  5000000       50.2
## 14      Beyoncé             Lemonade 2016        1  1554000  2500000       62.2
```

## Using SQL directly in R Markdown code chunks, and outputting for manipulation in R

What if I now want to output this data to R to use in a prettily formatted table or figure in an R Markdown report?  In the following code chunk, I draw on [Andrew Couch's tutorial](https://www.youtube.com/watch?v=zAgTlZUugUE) again, to write SQL code directly into the R Markdown code chunk (by replacing the `{R}` prefix with the following `{sql, connection = con, output.var = "df"}`. The first part of this statement tells R Markdown that the chunk uses SQL code and specifies the connection to access the database. The second part of the statement (`output.var = "df"`) is used to save output as a dataframe named df.

The query below uses the same statements as the example above, but captures UK and US data, and adds a new column (`other_sales`) calculating sales in countries other than the US and UK.  


```sql

SELECT charts.artist, charts.title, SUBSTR(charts.released, -4) AS year, 
  US_charts.chart_position AS US_chart,
  UK_charts.chart_position AS UK_chart,
  round(US_sales.sales/1000000, 1) AS US_sales, 
  round(UK_sales.sales/1000000, 1) AS UK_sales, 
  round(WW_sales.sales/1000000, 1) AS WW_sales, 
  round((WW_sales.sales - US_sales.sales - UK_sales.sales)/1000000, 1) AS other_sales,
  round(US_sales.sales/WW_sales.sales*100, 1) AS US_percent,
  round(UK_sales.sales/WW_sales.sales*100, 1) AS UK_percent,
  round((WW_sales.sales - US_sales.sales - UK_sales.sales)/WW_sales.sales*100, 1) AS other_percent
FROM charts
LEFT JOIN (SELECT artist, title, chart_position FROM charts WHERE chart = "US") AS US_charts
  ON charts.title = US_charts.title
LEFT JOIN (SELECT artist, title, chart_position FROM charts WHERE chart = "UK") AS UK_charts
  ON charts.title = UK_charts.title
LEFT JOIN sales
  ON charts.chart = sales.country and charts.title = sales.title
LEFT JOIN (SELECT artist, title, sales FROM sales WHERE country = "US") AS US_sales
  ON sales.title = US_sales.title
LEFT JOIN (SELECT artist, title, sales FROM sales WHERE country = "UK") AS UK_sales
  ON sales.title = UK_sales.title
LEFT JOIN (SELECT artist, title, sales FROM sales WHERE country = "WW" OR country = "World") AS WW_sales
  ON sales.title = WW_sales.title
GROUP BY charts.artist, charts.title
ORDER BY charts.artist DESC, year ASC;
```

Let's look at the output:


```r
head(df)
```

```
##         artist        title year US_chart UK_chart US_sales UK_sales WW_sales
## 1 Taylor Swift Taylor Swift 2006        5       81      5.7       NA       NA
## 2 Taylor Swift     Fearless 2008        1        5      7.2      0.6     12.0
## 3 Taylor Swift    Speak Now 2010        1        6      4.7      0.2      5.0
## 4 Taylor Swift          Red 2012        1        1      4.5      0.7      6.0
## 5 Taylor Swift         1989 2014        1        1      6.2      1.3     10.1
## 6 Taylor Swift   Reputation 2017        1        1      2.3      0.4      4.5
##   other_sales US_percent UK_percent other_percent
## 1          NA         NA         NA            NA
## 2         4.2       59.8        5.1          35.1
## 3         0.1       93.9        3.4           2.7
## 4         0.8       74.4       11.6          14.0
## 5         2.6       61.5       12.4          26.1
## 6         1.8       51.1        8.4          40.5
```

# Creating a simple summary table using gt

There are a plethora of packages capable of creating high quality tables in R Markdown reports. Some resources for exploring these options can be found on my [resources](/../../../../resources) page. As the purpose of this particular blog post is primarily to explore the use of SQL in R Markdown, I've made a relatively simple table here. In creating this table (using gt as shown below) I drew on:

* [the package documentation for gt](https://blog.rstudio.com/2020/04/08/great-looking-tables-gt-0-2/) 

* specific instructions for [creating summary lines](https://gt.rstudio.com/articles/creating-summary-lines.html) in gt

* [10+ guidelines for better tables in R](https://themockup.blog/posts/2020-09-04-10-table-rules-in-r/), in which Thomas Mock adapts for R (using gt) tables used as examples in Jon Schwabish's [Ten guidelines for better tables](https://www.cambridge.org/core/journals/journal-of-benefit-cost-analysis/article/ten-guidelines-for-better-tables/74C6FD9FEB12038A52A95B9FBCA05A12) 

* code used by [gkaramanis](https://github.com/gkaramanis/tidytuesday/blob/master/2020-week40/beyonce-swift.R) to make a far more involved table using gt to professionally display this data 



```r
# define functions used to exclude NA values from total calculations
# source: https://gt.rstudio.com/articles/creating-summary-lines.html

fns_labels <- list(Total = ~sum(., na.rm = TRUE))


# start with the data exported from SQL database

df %>%
  
  # select variables to include in table  
  select(title, artist, year, US_chart, UK_chart, US_sales, WW_sales, 
          US_percent) %>%
  
  # create a table using gt, grouping by artist and using title for row name
  gt(rowname_col = "title", groupname_col = "artist") %>%
  
    # set column labels 
    cols_label(
      year = "Released",
      US_chart = "US",
      UK_chart = "UK",
      US_sales = "US",
      WW_sales = "WW",
      US_percent = "US sales (%)") %>%

   # transform NA values in all columns to "-"
    text_transform(
      locations = cells_body(columns = gt::everything()),
      fn = function(x) {
      str_replace(x, "NA", "–")
      }
    ) %>%
  
    # create headings spanning multiple columns
    tab_spanner(label = "Chart position", columns = vars(US_chart, 
                                                         UK_chart)) %>%
    tab_spanner(label = "Sales ($ million)", columns = vars(US_sales,
                                                            WW_sales)) %>%

    # align specified cells (containing numeric values) right
    cols_align(align = "right", columns = c("US_chart", "UK_chart",
                                            "US_sales", "WW_sales",
                                            "US_percent")) %>%

    # create title and subtitle for table, use md formatting
    tab_header(
      title = md("**Taylor Swift has sold more albums than Beyoncé, but owes a greater proportion of her success to US sales than Beyoncé**"),
      subtitle = md("*Peak chart position, sales, and US sales as a percentage of total sales by album*"))%>%
  
    # create summary rows for each group 
    summary_rows(groups = TRUE, 
                 columns = vars(US_sales, WW_sales),
                 fns = fns_labels) %>%

    # create source note for table
    tab_source_note("Source: Wikipedia, October 2020") 
```

<!--html_preserve--><style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#npfoiizhon .gt_table {
  display: table;
  border-collapse: collapse;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 16px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#npfoiizhon .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#npfoiizhon .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#npfoiizhon .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 0;
  padding-bottom: 4px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#npfoiizhon .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#npfoiizhon .gt_col_headings {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#npfoiizhon .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#npfoiizhon .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#npfoiizhon .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#npfoiizhon .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#npfoiizhon .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#npfoiizhon .gt_group_heading {
  padding: 8px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
}

#npfoiizhon .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#npfoiizhon .gt_from_md > :first-child {
  margin-top: 0;
}

#npfoiizhon .gt_from_md > :last-child {
  margin-bottom: 0;
}

#npfoiizhon .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#npfoiizhon .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 12px;
}

#npfoiizhon .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#npfoiizhon .gt_first_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
}

#npfoiizhon .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#npfoiizhon .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#npfoiizhon .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#npfoiizhon .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#npfoiizhon .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#npfoiizhon .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding: 4px;
}

#npfoiizhon .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#npfoiizhon .gt_sourcenote {
  font-size: 90%;
  padding: 4px;
}

#npfoiizhon .gt_left {
  text-align: left;
}

#npfoiizhon .gt_center {
  text-align: center;
}

#npfoiizhon .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#npfoiizhon .gt_font_normal {
  font-weight: normal;
}

#npfoiizhon .gt_font_bold {
  font-weight: bold;
}

#npfoiizhon .gt_font_italic {
  font-style: italic;
}

#npfoiizhon .gt_super {
  font-size: 65%;
}

#npfoiizhon .gt_footnote_marks {
  font-style: italic;
  font-size: 65%;
}
</style>
<div id="npfoiizhon" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;"><table class="gt_table">
  <thead class="gt_header">
    <tr>
      <th colspan="7" class="gt_heading gt_title gt_font_normal" style><strong>Taylor Swift has sold more albums than Beyoncé, but owes a greater proportion of her success to US sales than Beyoncé</strong></th>
    </tr>
    <tr>
      <th colspan="7" class="gt_heading gt_subtitle gt_font_normal gt_bottom_border" style><em>Peak chart position, sales, and US sales as a percentage of total sales by album</em></th>
    </tr>
  </thead>
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="2" colspan="1"></th>
      <th class="gt_col_heading gt_center gt_columns_bottom_border" rowspan="2" colspan="1">Released</th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="2">
        <span class="gt_column_spanner">Chart position</span>
      </th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="2">
        <span class="gt_column_spanner">Sales ($ million)</span>
      </th>
      <th class="gt_col_heading gt_center gt_columns_bottom_border" rowspan="2" colspan="1">US sales (%)</th>
    </tr>
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">US</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">UK</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">US</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">WW</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr class="gt_group_heading_row">
      <td colspan="7" class="gt_group_heading">Taylor Swift</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Taylor Swift</td>
      <td class="gt_row gt_left">2006</td>
      <td class="gt_row gt_right">5</td>
      <td class="gt_row gt_right">81</td>
      <td class="gt_row gt_right">5.7</td>
      <td class="gt_row gt_right">–</td>
      <td class="gt_row gt_right">–</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Fearless</td>
      <td class="gt_row gt_left">2008</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">5</td>
      <td class="gt_row gt_right">7.2</td>
      <td class="gt_row gt_right">12.0</td>
      <td class="gt_row gt_right">59.8</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Speak Now</td>
      <td class="gt_row gt_left">2010</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">6</td>
      <td class="gt_row gt_right">4.7</td>
      <td class="gt_row gt_right">5.0</td>
      <td class="gt_row gt_right">93.9</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Red</td>
      <td class="gt_row gt_left">2012</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">4.5</td>
      <td class="gt_row gt_right">6.0</td>
      <td class="gt_row gt_right">74.4</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">1989</td>
      <td class="gt_row gt_left">2014</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">6.2</td>
      <td class="gt_row gt_right">10.1</td>
      <td class="gt_row gt_right">61.5</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Reputation</td>
      <td class="gt_row gt_left">2017</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">2.3</td>
      <td class="gt_row gt_right">4.5</td>
      <td class="gt_row gt_right">51.1</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Lover</td>
      <td class="gt_row gt_left">2019</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">1.1</td>
      <td class="gt_row gt_right">3.2</td>
      <td class="gt_row gt_right">33.9</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Folklore</td>
      <td class="gt_row gt_left">2020</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">–</td>
      <td class="gt_row gt_right">–</td>
      <td class="gt_row gt_right">–</td>
    </tr>
    <tr>
      <td class="gt_row gt_stub gt_right gt_summary_row gt_first_summary_row">Total</td>
      <td class="gt_row gt_left gt_summary_row gt_first_summary_row">&mdash;</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row">&mdash;</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row">&mdash;</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row">31.70</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row">40.80</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row">&mdash;</td>
    </tr>
    <tr class="gt_group_heading_row">
      <td colspan="7" class="gt_group_heading">Beyoncé</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Dangerously in Love</td>
      <td class="gt_row gt_left">2003</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">5.1</td>
      <td class="gt_row gt_right">11.0</td>
      <td class="gt_row gt_right">46.4</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">B'Day</td>
      <td class="gt_row gt_left">2006</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">3</td>
      <td class="gt_row gt_right">3.6</td>
      <td class="gt_row gt_right">8.0</td>
      <td class="gt_row gt_right">45.1</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">I Am... Sasha Fierce</td>
      <td class="gt_row gt_left">2008</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">2</td>
      <td class="gt_row gt_right">3.4</td>
      <td class="gt_row gt_right">8.0</td>
      <td class="gt_row gt_right">42.3</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">4</td>
      <td class="gt_row gt_left">2011</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">1.5</td>
      <td class="gt_row gt_right">–</td>
      <td class="gt_row gt_right">–</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Beyoncé</td>
      <td class="gt_row gt_left">2013</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">2</td>
      <td class="gt_row gt_right">2.5</td>
      <td class="gt_row gt_right">5.0</td>
      <td class="gt_row gt_right">50.2</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Lemonade</td>
      <td class="gt_row gt_left">2016</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">1.6</td>
      <td class="gt_row gt_right">2.5</td>
      <td class="gt_row gt_right">62.2</td>
    </tr>
    <tr>
      <td class="gt_row gt_stub gt_right gt_summary_row gt_first_summary_row">Total</td>
      <td class="gt_row gt_left gt_summary_row gt_first_summary_row">&mdash;</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row">&mdash;</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row">&mdash;</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row">17.70</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row">34.50</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row">&mdash;</td>
    </tr>
  </tbody>
  <tfoot class="gt_sourcenotes">
    <tr>
      <td class="gt_sourcenote" colspan="7">Source: Wikipedia, October 2020</td>
    </tr>
  </tfoot>
  
</table></div><!--/html_preserve-->

## One more SQL trick: UPDATE query using nested REPLACE statements

In an upcoming blog post, I plan to further refine this summary table using the `gt` package, and to use `ggplot2` and `ggflags` to graph data by country. But first, we need to update country codes in our data to be consistent with those used in `ggflags`.

Last time we worked with this data, we updated the country field to use 'WW' consistently to reflect worldwide sales. Let's update multiple country codes at once, using nested replace statements based on a suggestion from [Turophile](https://stackoverflow.com/questions/28493238/how-to-replace-multiple-words-in-sqlite-database-using-update-query) on stackoverflow.


```r
dbExecute(con, "
UPDATE sales
SET country = REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(country, 'World', 'WW'), 'AUS', 'AU'), 'JPN', 'JP'), 'UK', 'GB'), 'CAN', 'CA'), 'FRA', 'FR');
                          ")
```

```
## [1] 48
```

## Next steps

As outlined above, the next steps for this particular project for me are to further refine my output table using `gt` (I'm wanting to include a barplot visualizing US sales as a percentage of total sales), and using `ggplot2` and `ggflags` to create bar plots showing sales data by country, assigning a flag to each bar.  Once again, however, these are questions for a future post. 

## Finally, remember to disconnect

At the end of this process, best practice is always to disconnect from the database.


```r
dbDisconnect(con)
```
