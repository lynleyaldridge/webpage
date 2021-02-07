---
title: 'More experimenting with SQL in R Markdown: Pivoting data, outputting results
  to R, and creating a summary table using gt'
author: Lynley Aldridge
date: '2021-02-08'
tags:
  - gt
  - Rstats
  - SQL
  - TidyTuesday
  - Tutorial
draft: yes
slug: experimenting-with-sql-in-r-markdown-more-join-queries
lastmod: '2021-02-08T20:01:24+11:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
---

Following on from my [previous post](/2021/01/13/experimenting-with-sql/), I use SELECT and JOIN statements to pivot the Taylor Swift and Beyoncé Tidy Tuesday data using RSQLite (after normalizing the underlying database), output the results to R, and create a relatively simple summary table using gt. 

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

Let's quickly refresh our understanding of how this data is structured (focusing only on columns relevant for this tutorial). 


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

# Database normalizing

Please note that my goal in these examples is to work with RSQLite in order to learn more about using SQL in R, rather than presenting this as the best way to conduct these analyses.

The task I set myself at the end of the last post was to join and pivot this data so that each album is represented by a single row of data, and there are separate columns for sales and chart information for each country of interest. 

This process will be easier to follow if the underlying data are normalized. This is also best practice for any database working with relational data, and involves minimizing redundancies in data storage and ensuring each table serves only a single purpose. In this case, we should create a separate albums table to record the details of each album (i.e., artist, title and release dates), so edits and additions to this information need only be made in one place. We then give each album a unique ID (a primary key) that can be used to reference this album in the charts and sales tables. 

First, let's create the new albums table, adding an `id` field to this table to store a unique id number for each distinct album.


```r
dbExecute(con, "
CREATE TABLE albums
  (id INTEGER NOT NULL PRIMARY KEY,
  artist TEXT,
  title TEXT,
  released TEXT
  );
                          ")
```

```
## [1] 0
```

We now populate this table with the unique albums found in the charts table (we could just as easily have done this from the sales table).


```r
dbExecute(con, "
INSERT INTO albums (artist, title, released)
  SELECT DISTINCT artist, title, released FROM charts; 
                            ")
```

```
## [1] 14
```

Fourteen records have been added to this table, and viewing results with the query below shows that this has functioned as intended. Distinct albums have been added to the table, and automatically assigned a unique sequential id number. (I'm using LIMIT 5 after testing when presenting these steps to minimize display space. Omitting this statement returns the full 14 rows of data expected.)


```r
dbGetQuery(con, '
SELECT *
FROM albums
LIMIT 5
                          ')
```

```
##   id       artist        title          released
## 1  1 Taylor Swift Taylor Swift  October 24, 2006
## 2  2 Taylor Swift     Fearless November 11, 2008
## 3  3 Taylor Swift    Speak Now  October 25, 2010
## 4  4 Taylor Swift          Red  October 22, 2012
## 5  5 Taylor Swift         1989  October 27, 2014
```

Our next goal is to amend the charts and sales tables to reference albums by using their unique id number, rather than their title. The id column is created as a primary key in the albums table, and as a foreign key in the charts and sales tables (where it serves to point back to the album details stored in the albums table). (For example, we will use `FK_Sales_Albums` instead of album title in the `Sales` table to reference albums whose details are stored in the `Albums` table).

Here we create the new sales table (`sales_new`): 


```r
dbExecute(con, "
CREATE TABLE sales_new
  (id INTEGER NOT NULL PRIMARY KEY,
  FK_Sales_Albums,
  country,
  sales
  );
                          ")
```

```
## [1] 0
```

Next we need to extract records to add to this table. First, let's design and test our query and check that it works as intended. We want to join the albums table to the sales tables by matching on `Albums.title = sales.title`, and extract sales by country details for each album with the correct id attached. So we select the id field from the albums table (`Albums.id`) and label this as our foreign key (`AS FK_Sales_Albums`). To complete our query, we also select the country and sales fields from the sales table: 


```r
dbGetQuery(con, '
SELECT Albums.id AS FK_Sales_Albums, sales.country, sales.sales 
FROM sales
LEFT JOIN albums
ON Albums.title = sales.title
LIMIT 5
                          ')
```

```
##   FK_Sales_Albums country    sales
## 1               1      US  5720000
## 2               2      WW 12000000
## 3               2      US  7180000
## 4               2     AUS   500000
## 5               2      UK   609000
```

This process of drafting queries, verifying results, and then executing updates is one that works well when designing multi-stage queries. Let's now combine our JOIN query with an INSERT INTO query, to populate the `sales_new` table:


```r
dbExecute(con, "
INSERT INTO sales_new (FK_Sales_Albums, country, sales)
  SELECT Albums.id AS FK_Sales_Albums, sales.country, sales.sales 
  FROM sales
  LEFT JOIN albums
  ON Albums.title = sales.title;
                            ")
```

```
## [1] 48
```

We can repeat the same process with the charts data. First, create the structure of `charts_new`:


```r
dbExecute(con, "
CREATE TABLE charts_new
  (id INTEGER NOT NULL PRIMARY KEY,
  FK_Charts_Albums,
  chart,
  chart_position
  );
                          ")
```

```
## [1] 0
```

Next, draft and test a query extracting peak chart positions for each album by album id.


```r
dbGetQuery(con, '
SELECT Albums.id AS FK_Charts_Albums, charts.chart, charts.chart_position 
FROM charts
LEFT JOIN albums
ON Albums.title = charts.title
LIMIT 5
                          ')
```

```
##   FK_Charts_Albums chart chart_position
## 1                1    US              5
## 2                1   AUS             33
## 3                1   CAN             14
## 4                1   FRA              —
## 5                1   GER              —
```

Now, combine this JOIN query with an INSERT INTO query and execute, to insert data into our new table and check that the full 140 records have been created:


```r
dbExecute(con, "
INSERT INTO charts_new (FK_Charts_Albums, chart, chart_position)
  SELECT Albums.id AS FK_Charts_Albums, charts.chart, charts.chart_position 
  FROM charts
  LEFT JOIN albums
  ON Albums.title = charts.title;
                            ")
```

```
## [1] 140
```

## Pivoting sales data using SELECT and JOIN ON queries

To pivot data so that sales data for each country is presented in a new column (using RSQLite), we can use a series of JOIN ... ON statements.

The following code, for example, creates a temporary table called `US_charts` which joins rows from the `charts_new` table where `chart = "US"` with matching album ids. 


```r
`LEFT JOIN charts_new AS US_charts 
  ON US_charts.FK_Charts_Albums = albums.id 
  AND US_charts.chart = "US"`
```

This temporary table can then be referenced in the SELECT statement at the beginning of the query, which specifies we want to extract `US_charts.chart_position AS US_chart`.  

Note that specific code for pivoting data varies depending on SQL vendor, and Microsoft SQL server uses [PIVOT and UNPIVOT](https://docs.microsoft.com/en-us/sql/t-sql/queries/from-using-pivot-and-unpivot?view=sql-server-ver15) operators. 

We can also add calculated fields to this query to show what percentage of each album's  worldwide sales was comprised by sales in a specified country (e.g., `SELECT US_sales.sales/WW_sales.sales*100 AS US_percent` creates a `US_percent` column, by dividing US sales by worldwide sales, then multiplying by 100). The `round()` wrapper rounds the result of the formula defining `US_percent` to the number of decimal points specified after the comma. Note that column names included in calculations must be specified in full (e.g., `US_sales.sales`), and not by aliases that will not be assigned until the SELECT statement runs (e.g., `US_sales`).

What follows should be seen as a proof of concept, which I've used to extract just US charts and sales data:


```r
dbGetQuery(con, '

SELECT albums.artist, albums.title, SUBSTR(albums.released, -4) AS year,
  US_charts.chart_position AS US_chart,
  US_sales.sales AS US_sales,
  WW_sales.sales AS WW_sales,  
  round(US_sales.sales/WW_sales.sales*100, 1) AS US_percent
FROM albums
LEFT JOIN charts_new AS US_charts
  ON US_charts.FK_Charts_Albums = albums.id 
  AND US_charts.chart = "US"
LEFT JOIN sales_new AS US_sales
  ON US_sales.FK_Sales_Albums = albums.id 
  AND US_sales.country = "US"
LEFT JOIN sales_new AS WW_sales
  ON WW_sales.FK_Sales_Albums = albums.id 
  AND (WW_sales.country = "WW" OR WW_sales.country = "World")
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

The query below uses the same statements as the example above, but captures UK and US chart data, US and WW sales data, and adds a new column (`other_sales`) calculating sales in countries other than the US. For ease of presentation, sales figures are now calculated in millions.  


```sql

SELECT albums.artist, albums.title, SUBSTR(albums.released, -4) AS year,
  US_charts.chart_position AS US_chart,
  UK_charts.chart_position AS UK_chart,
  round(US_sales.sales/1000000, 1) AS US_sales, 
  round(WW_sales.sales/1000000, 1) AS WW_sales, 
  round((WW_sales.sales - US_sales.sales)/1000000, 1) AS other_sales,
  round(US_sales.sales/WW_sales.sales*100, 1) AS US_percent,
  round((WW_sales.sales - US_sales.sales)/WW_sales.sales*100, 1) AS other_percent
FROM albums
LEFT JOIN charts_new AS US_charts
  ON US_charts.FK_Charts_Albums = albums.id 
  AND US_charts.chart = "US"
LEFT JOIN charts_new AS UK_charts
  ON UK_charts.FK_Charts_Albums = albums.id 
  AND UK_charts.chart = "UK"
LEFT JOIN sales_new AS US_sales
  ON US_sales.FK_Sales_Albums = albums.id 
  AND US_sales.country = "US"
LEFT JOIN sales_new AS WW_sales
  ON WW_sales.FK_Sales_Albums = albums.id 
  AND (WW_sales.country = "WW" OR WW_sales.country = "World");
```

Let's look at the output:


```r
head(df)
```

```
##         artist        title year US_chart UK_chart US_sales WW_sales
## 1 Taylor Swift Taylor Swift 2006        5       81      5.7       NA
## 2 Taylor Swift     Fearless 2008        1        5      7.2     12.0
## 3 Taylor Swift    Speak Now 2010        1        6      4.7      5.0
## 4 Taylor Swift          Red 2012        1        1      4.5      6.0
## 5 Taylor Swift         1989 2014        1        1      6.2     10.1
## 6 Taylor Swift   Reputation 2017        1        1      2.3      4.5
##   other_sales US_percent other_percent
## 1          NA         NA            NA
## 2         4.8       59.8          40.2
## 3         0.3       93.9           6.1
## 4         1.5       74.4          25.6
## 5         3.9       61.5          38.5
## 6         2.2       51.1          48.9
```

# Creating a simple summary table using gt

There are a plethora of packages capable of creating high quality tables using R Markdown and blogdown. Some resources for exploring these options can be found on my [resources](/../../../../resources) page. As the purpose of this particular blog post is primarily to explore the use of SQL, I've made a relatively simple table here. In creating this table (using gt as shown below) I drew on:

* [the package documentation for gt](https://blog.rstudio.com/2020/04/08/great-looking-tables-gt-0-2/) 

* specific instructions for [creating summary lines](https://gt.rstudio.com/articles/creating-summary-lines.html) in gt

* [10+ guidelines for better tables in R](https://themockup.blog/posts/2020-09-04-10-table-rules-in-r/), in which Thomas Mock adapts for R (using gt) tables used as examples in Jon Schwabish's [Ten guidelines for better tables](https://www.cambridge.org/core/journals/journal-of-benefit-cost-analysis/article/ten-guidelines-for-better-tables/74C6FD9FEB12038A52A95B9FBCA05A12) 

* code used by [gkaramanis](https://github.com/gkaramanis/tidytuesday/blob/master/2020-week40/beyonce-swift.R) to make a much more visually appealing table using gt summarizing this data 



```r
# define a function to create totals for summary rows, excluding na

fns_labels <- list(Total = ~sum(., na.rm = TRUE))

# start with the data exported from SQL database

df %>%
  
  # select variables to include in table  
  select(title, artist, year, US_chart, UK_chart, US_sales, other_sales, WW_sales) %>%
  
  # create a table using gt, grouping by artist and using title for row name
  gt(rowname_col = "title", groupname_col = "artist") %>%
  
    # set column labels 
    cols_label(
      year = "Released",
      US_chart = "US",
      UK_chart = "UK",
      US_sales = "US",
      other_sales = "Other",
      WW_sales = "Total") %>%

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
                                                            other_sales,
                                                            WW_sales)) %>%

    # create title and subtitle for table, use md formatting
    tab_header(
      title = md("**Taylor Swift has higher US and Total sales than Beyoncé, but Beyoncé's international sales are higher**"),
      subtitle = md("*Peak chart position and sales by location*"))%>%
  
    # create summary rows for each group 
    summary_rows(groups = TRUE, 
                 columns = vars(US_sales, other_sales, WW_sales),
                 fns = fns_labels,
                 formatter = fmt_number, decimals = 1) %>%
  
    
    # create source note for table
    tab_source_note("Source: Billboard") 
```

<!--html_preserve--><style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#nkybvopvnn .gt_table {
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

#nkybvopvnn .gt_heading {
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

#nkybvopvnn .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#nkybvopvnn .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 0;
  padding-bottom: 4px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#nkybvopvnn .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#nkybvopvnn .gt_col_headings {
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

#nkybvopvnn .gt_col_heading {
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

#nkybvopvnn .gt_column_spanner_outer {
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

#nkybvopvnn .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#nkybvopvnn .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#nkybvopvnn .gt_column_spanner {
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

#nkybvopvnn .gt_group_heading {
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

#nkybvopvnn .gt_empty_group_heading {
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

#nkybvopvnn .gt_from_md > :first-child {
  margin-top: 0;
}

#nkybvopvnn .gt_from_md > :last-child {
  margin-bottom: 0;
}

#nkybvopvnn .gt_row {
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

#nkybvopvnn .gt_stub {
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

#nkybvopvnn .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#nkybvopvnn .gt_first_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
}

#nkybvopvnn .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#nkybvopvnn .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#nkybvopvnn .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#nkybvopvnn .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#nkybvopvnn .gt_footnotes {
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

#nkybvopvnn .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding: 4px;
}

#nkybvopvnn .gt_sourcenotes {
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

#nkybvopvnn .gt_sourcenote {
  font-size: 90%;
  padding: 4px;
}

#nkybvopvnn .gt_left {
  text-align: left;
}

#nkybvopvnn .gt_center {
  text-align: center;
}

#nkybvopvnn .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#nkybvopvnn .gt_font_normal {
  font-weight: normal;
}

#nkybvopvnn .gt_font_bold {
  font-weight: bold;
}

#nkybvopvnn .gt_font_italic {
  font-style: italic;
}

#nkybvopvnn .gt_super {
  font-size: 65%;
}

#nkybvopvnn .gt_footnote_marks {
  font-style: italic;
  font-size: 65%;
}
</style>
<div id="nkybvopvnn" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;"><table class="gt_table">
  <thead class="gt_header">
    <tr>
      <th colspan="7" class="gt_heading gt_title gt_font_normal" style><strong>Taylor Swift has higher US and Total sales than Beyoncé, but Beyoncé's international sales are higher</strong></th>
    </tr>
    <tr>
      <th colspan="7" class="gt_heading gt_subtitle gt_font_normal gt_bottom_border" style><em>Peak chart position and sales by location</em></th>
    </tr>
  </thead>
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="2" colspan="1"></th>
      <th class="gt_col_heading gt_center gt_columns_bottom_border" rowspan="2" colspan="1">Released</th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="2">
        <span class="gt_column_spanner">Chart position</span>
      </th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="3">
        <span class="gt_column_spanner">Sales ($ million)</span>
      </th>
    </tr>
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">US</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">UK</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">US</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">Other</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">Total</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr class="gt_group_heading_row">
      <td colspan="7" class="gt_group_heading">Taylor Swift</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Taylor Swift</td>
      <td class="gt_row gt_left">2006</td>
      <td class="gt_row gt_left">5</td>
      <td class="gt_row gt_left">81</td>
      <td class="gt_row gt_right">5.7</td>
      <td class="gt_row gt_right">–</td>
      <td class="gt_row gt_right">–</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Fearless</td>
      <td class="gt_row gt_left">2008</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">5</td>
      <td class="gt_row gt_right">7.2</td>
      <td class="gt_row gt_right">4.8</td>
      <td class="gt_row gt_right">12.0</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Speak Now</td>
      <td class="gt_row gt_left">2010</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">6</td>
      <td class="gt_row gt_right">4.7</td>
      <td class="gt_row gt_right">0.3</td>
      <td class="gt_row gt_right">5.0</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Red</td>
      <td class="gt_row gt_left">2012</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_right">4.5</td>
      <td class="gt_row gt_right">1.5</td>
      <td class="gt_row gt_right">6.0</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">1989</td>
      <td class="gt_row gt_left">2014</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_right">6.2</td>
      <td class="gt_row gt_right">3.9</td>
      <td class="gt_row gt_right">10.1</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Reputation</td>
      <td class="gt_row gt_left">2017</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_right">2.3</td>
      <td class="gt_row gt_right">2.2</td>
      <td class="gt_row gt_right">4.5</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Lover</td>
      <td class="gt_row gt_left">2019</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_right">1.1</td>
      <td class="gt_row gt_right">2.1</td>
      <td class="gt_row gt_right">3.2</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Folklore</td>
      <td class="gt_row gt_left">2020</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_right">–</td>
      <td class="gt_row gt_right">–</td>
      <td class="gt_row gt_right">–</td>
    </tr>
    <tr>
      <td class="gt_row gt_stub gt_right gt_summary_row gt_first_summary_row">Total</td>
      <td class="gt_row gt_left gt_summary_row gt_first_summary_row">&mdash;</td>
      <td class="gt_row gt_left gt_summary_row gt_first_summary_row">&mdash;</td>
      <td class="gt_row gt_left gt_summary_row gt_first_summary_row">&mdash;</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row">31.7</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row">14.8</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row">40.8</td>
    </tr>
    <tr class="gt_group_heading_row">
      <td colspan="7" class="gt_group_heading">Beyoncé</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Dangerously in Love</td>
      <td class="gt_row gt_left">2003</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_right">5.1</td>
      <td class="gt_row gt_right">5.9</td>
      <td class="gt_row gt_right">11.0</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">B'Day</td>
      <td class="gt_row gt_left">2006</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">3</td>
      <td class="gt_row gt_right">3.6</td>
      <td class="gt_row gt_right">4.4</td>
      <td class="gt_row gt_right">8.0</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">I Am... Sasha Fierce</td>
      <td class="gt_row gt_left">2008</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">2</td>
      <td class="gt_row gt_right">3.4</td>
      <td class="gt_row gt_right">4.6</td>
      <td class="gt_row gt_right">8.0</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">4</td>
      <td class="gt_row gt_left">2011</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_right">1.5</td>
      <td class="gt_row gt_right">–</td>
      <td class="gt_row gt_right">–</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Beyoncé</td>
      <td class="gt_row gt_left">2013</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">2</td>
      <td class="gt_row gt_right">2.5</td>
      <td class="gt_row gt_right">2.5</td>
      <td class="gt_row gt_right">5.0</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Lemonade</td>
      <td class="gt_row gt_left">2016</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_right">1.6</td>
      <td class="gt_row gt_right">0.9</td>
      <td class="gt_row gt_right">2.5</td>
    </tr>
    <tr>
      <td class="gt_row gt_stub gt_right gt_summary_row gt_first_summary_row">Total</td>
      <td class="gt_row gt_left gt_summary_row gt_first_summary_row">&mdash;</td>
      <td class="gt_row gt_left gt_summary_row gt_first_summary_row">&mdash;</td>
      <td class="gt_row gt_left gt_summary_row gt_first_summary_row">&mdash;</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row">17.7</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row">18.3</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row">34.5</td>
    </tr>
  </tbody>
  <tfoot class="gt_sourcenotes">
    <tr>
      <td class="gt_sourcenote" colspan="7">Source: Billboard</td>
    </tr>
  </tfoot>
  
</table></div><!--/html_preserve-->

## One more SQL trick: UPDATE query using CASE WHEN statements

In upcoming blog posts, I plan to further refine this summary table using the `gt` package, and to use `ggplot2` and `ggflags` to graph data by country. But first, we need to update country codes in our data to be consistent with those used in `ggflags`.

Last time we worked with this data, we updated the country field to use 'WW' consistently to reflect worldwide sales. Let's update multiple country codes from `sales_new` at once, using an UPDATE query and a series of CASE WHEN statements:


```r
dbExecute(con, "
UPDATE sales_new
SET country = CASE WHEN country = 'World' THEN 'WW'
                    WHEN country = 'WW' THEN 'WW'
                    WHEN country = 'AUS' THEN 'AU'
                    WHEN country = 'JPN' THEN 'JP'
                    WHEN country = 'UK' THEN 'GB'
                    WHEN country = 'CAN' THEN 'CA'
                    WHEN country = 'FRA' THEN 'FR'
                    WHEN country = 'FR' THEN 'FR'
                    WHEN country = 'US' THEN 'US'
                    WHEN country = 'NA' THEN 'NA'
                    END;
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
