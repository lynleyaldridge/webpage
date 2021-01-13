---
title: "Experimenting with SQL in RMarkdown: SELECT, UPDATE and JOIN queries using RSQLite"
author: Lynley Aldridge
date: '2021-01-13'
slug: experimenting-with-sql
categories: []
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2021-01-13T17:20:19+11:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

In this post I talk through, step-by-step, a process I've been using to document the SQL learning I've been doing using R Markdown. I draw on a tutorial by Andrew Couch to create a database I can manipulate using RSQLite, using a Tidy Tuesday dataset featuring Taylor Swift and Beyoncé album data.  I then run some example SQL queries using this data.

# Introduction

I've been wanting to document some SQL learning I've been doing lately, and was grateful to find a you tube tutorial from Andrew Couch. Here he talks through how to connect to a database using R Studio and query this database using SQL (from within an R Markdown document). In this tutorial I load csv data into a simple sql database and run some queries, to document my SQL learning journey.

{{< youtube zAgTlZUugUE >}}

# Setup

## Load packages

Let's start by loading packages. First, install any packages not previously used by typing the following command directly into the console (to install odbc, for example): `install.packages("odbc")`. 

Then run the following.


```r
library(tidyverse)
library(odbc)
library(DBI)
library(RSQLite)
```

## Load data

Next, we need to load in our data. I've been wanting to experiment with the [Tidy Tuesday dataset containing Taylor Swift and Beyoncé song lyrics and album details](https://github.com/rfordatascience/tidytuesday/blob/master/data/2020/2020-09-29/readme.md) for a while. This is easy to download directly from the Internet, and looks like it would have some promise for practicing SQL. So let's try using the album sales and chart datasets for this exercise.


```r
sales <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2020/2020-09-29/sales.csv')

charts <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2020/2020-09-29/charts.csv')
```

Taylor Swift data comes from [Taylor Swift albums discography](https://en.wikipedia.org/wiki/Taylor_Swift_albums_discography).

Beyoncé data comes from [Beyoncé discography](https://en.wikipedia.org/wiki/Beyonc%C3%A9_discography).

## Create database

Let's create a connection to a SQLite database, store it in memory, and copy the sales and charts data into this database.


```r
con <- dbConnect(RSQLite::SQLite(), ":memory:")
copy_to(con, sales)
copy_to(con, charts)
```

# SQL queries in R Markdown (an introduction using sales data)

## SELECT query to retrieve all records 

The following code opens the connection to our database, as specified previously, and runs a simple SQL query to SELECT the first five (specified) columns of the sales table. 


```r
dbGetQuery(con, '
SELECT artist, title, country, sales, released
FROM sales
                          ')
```

```
##          artist                title country    sales
## 1  Taylor Swift         Taylor Swift      US  5720000
## 2  Taylor Swift             Fearless      WW 12000000
## 3  Taylor Swift             Fearless      US  7180000
## 4  Taylor Swift             Fearless     AUS   500000
## 5  Taylor Swift             Fearless      UK   609000
## 6  Taylor Swift            Speak Now      WW  5000000
## 7  Taylor Swift            Speak Now      US  4694000
## 8  Taylor Swift            Speak Now      UK   169281
## 9  Taylor Swift                  Red      WW  6000000
## 10 Taylor Swift                  Red      US  4465000
## 11 Taylor Swift                  Red     JPN   256000
## 12 Taylor Swift                  Red      UK   693000
## 13 Taylor Swift                 1989      WW 10100000
## 14 Taylor Swift                 1989      US  6215000
## 15 Taylor Swift                 1989     CAN   542000
## 16 Taylor Swift                 1989     FRA    70000
## 17 Taylor Swift                 1989     JPN   268200
## 18 Taylor Swift                 1989      UK  1250000
## 19 Taylor Swift           Reputation      WW  4500000
## 20 Taylor Swift           Reputation      US  2300000
## 21 Taylor Swift           Reputation     CAN    80000
## 22 Taylor Swift           Reputation     FRA    13000
## 23 Taylor Swift           Reputation      UK   378000
## 24 Taylor Swift                Lover      WW  3200000
## 25 Taylor Swift                Lover      US  1085000
## 26 Taylor Swift                Lover     CAN    61000
## 27 Taylor Swift                Lover     FRA     5700
## 28 Taylor Swift                Lover     JPN    32601
## 29 Taylor Swift                Lover      UK   221654
## 30 Taylor Swift             Folklore    <NA>       NA
## 31      Beyoncé  Dangerously in Love   World 11000000
## 32      Beyoncé  Dangerously in Love      US  5100000
## 33      Beyoncé  Dangerously in Love      UK  1260000
## 34      Beyoncé                B'Day   World  8000000
## 35      Beyoncé                B'Day      US  3610000
## 36      Beyoncé                B'Day      UK   700000
## 37      Beyoncé I Am... Sasha Fierce   World  8000000
## 38      Beyoncé I Am... Sasha Fierce      US  3380000
## 39      Beyoncé I Am... Sasha Fierce      UK  1740000
## 40      Beyoncé                    4      US  1500000
## 41      Beyoncé                    4      UK   791000
## 42      Beyoncé              Beyoncé   World  5000000
## 43      Beyoncé              Beyoncé      US  2512000
## 44      Beyoncé              Beyoncé      UK   418000
## 45      Beyoncé             Lemonade   World  2500000
## 46      Beyoncé             Lemonade      US  1554000
## 47      Beyoncé             Lemonade      UK   328000
## 48      Beyoncé             Lemonade      FR    30000
##                      released
## 1            October 24, 2006
## 2           November 11, 2008
## 3           November 11, 2008
## 4           November 11, 2008
## 5           November 11, 2008
## 6            October 25, 2010
## 7            October 25, 2010
## 8            October 25, 2010
## 9            October 22, 2012
## 10           October 22, 2012
## 11           October 22, 2012
## 12           October 22, 2012
## 13           October 27, 2014
## 14           October 27, 2014
## 15           October 27, 2014
## 16           October 27, 2014
## 17           October 27, 2014
## 18           October 27, 2014
## 19          November 10, 2017
## 20          November 10, 2017
## 21          November 10, 2017
## 22          November 10, 2017
## 23          November 10, 2017
## 24            August 23, 2019
## 25            August 23, 2019
## 26            August 23, 2019
## 27            August 23, 2019
## 28            August 23, 2019
## 29            August 23, 2019
## 30              July 24, 2020
## 31     June 23, 2003 (UK)[39]
## 32     June 23, 2003 (UK)[39]
## 33     June 23, 2003 (UK)[39]
## 34 September 1, 2006 (US)[51]
## 35 September 1, 2006 (US)[51]
## 36 September 1, 2006 (US)[51]
## 37          November 14, 2008
## 38          November 14, 2008
## 39          November 14, 2008
## 40              June 24, 2011
## 41              June 24, 2011
## 42          December 13, 2013
## 43          December 13, 2013
## 44          December 13, 2013
## 45             April 23, 2016
## 46             April 23, 2016
## 47             April 23, 2016
## 48             April 23, 2016
```

As shown above, this table contains sales data across multiple countries for albums by Taylor Swift and Beyoncé (a total of 48 observations), along with a character field specifying the release date of each album.  

## UPDATE query using SUBSTR to fix release dates

We can see from the above that the release dates for Beyoncé's *Dangerously in Love* and *B'Day* have both been extracted with superfluous details following the date.  The following UPDATE query changes the released date to only a specified substring (all text preceding a space preceding an opening bracket) WHERE strings contained this character combination.  This took close inspection of the data and a process of trial and error to figure out, drawing on stack overflow contributions by [scaisEdge](https://stackoverflow.com/questions/40708459/update-column-to-remove-everything-before-and-including-a-space-in-sqlite) and [CodeBird](https://stackoverflow.com/questions/22558411/how-to-remove-everything-after-certain-character-in-sql). 


```r
dbExecute(con, "
UPDATE sales
SET released = TRIM(SUBSTR(released, 1, INSTR(released, ' (')))
WHERE INSTR(released, ' (')>0; 
                          ")
```

```
## [1] 6
```

This query has updated 6 records. Let's check to see if the relevant dates have now been repaired.


```r
dbGetQuery(con, '
SELECT artist, title, country, sales, released
FROM sales
WHERE title LIKE "%Day" OR TITLE = "Dangerously in Love"
                          ')
```

```
##    artist               title country    sales          released
## 1 Beyoncé Dangerously in Love   World 11000000     June 23, 2003
## 2 Beyoncé Dangerously in Love      US  5100000     June 23, 2003
## 3 Beyoncé Dangerously in Love      UK  1260000     June 23, 2003
## 4 Beyoncé               B'Day   World  8000000 September 1, 2006
## 5 Beyoncé               B'Day      US  3610000 September 1, 2006
## 6 Beyoncé               B'Day      UK   700000 September 1, 2006
```

Now that we have formatted these dates consistently, let's try to extract the last four characters of each string to create a `year_released` column. 

The query below uses the `SUBSTR` function to select only the last four characters of the `released` character field, that will allow us to sort data by artist and year of album release as follows.


```r
dbGetQuery(con, '
SELECT artist, title, SUBSTR(released, -4) as year_released
FROM sales
GROUP BY title
ORDER BY artist DESC, year_released ASC
                          ')
```

```
##          artist                title year_released
## 1  Taylor Swift         Taylor Swift          2006
## 2  Taylor Swift             Fearless          2008
## 3  Taylor Swift            Speak Now          2010
## 4  Taylor Swift                  Red          2012
## 5  Taylor Swift                 1989          2014
## 6  Taylor Swift           Reputation          2017
## 7  Taylor Swift                Lover          2019
## 8  Taylor Swift             Folklore          2020
## 9       Beyoncé  Dangerously in Love          2003
## 10      Beyoncé                B'Day          2006
## 11      Beyoncé I Am... Sasha Fierce          2008
## 12      Beyoncé                    4          2011
## 13      Beyoncé              Beyoncé          2013
## 14      Beyoncé             Lemonade          2016
```

Note from this section that we use [`dbGetQuery`](https://www.rdocumentation.org/packages/DBI/versions/0.5-1/topics/dbGetQuery) from the DBI package to *query* the database, and [`dbExecute`](https://www.rdocumentation.org/packages/DBI/versions/0.5-1/topics/dbExecute) to *update* records.

## UPDATE query using REPLACE to edit sales country field

While we're updating the data, let's also update the country field to use 'WW' consistently to reflect worldwide sales.


```r
dbExecute(con, "
UPDATE sales
SET country = REPLACE(country, 'World', 'WW')
WHERE country = 'World'; 
                          ")
```

```
## [1] 5
```

The `WHERE country = 'World'` filter in the above is not strictly necessary, but it does mean we get output confirming the number of records that have been modified.

We can use the following query to check worldwide sales data is now being reported consistently:


```r
dbGetQuery(con, '
SELECT artist, title, country, sales, released
FROM sales
WHERE country = "World" or country = "WW"
                          ')
```

```
##          artist                title country    sales          released
## 1  Taylor Swift             Fearless      WW 12000000 November 11, 2008
## 2  Taylor Swift            Speak Now      WW  5000000  October 25, 2010
## 3  Taylor Swift                  Red      WW  6000000  October 22, 2012
## 4  Taylor Swift                 1989      WW 10100000  October 27, 2014
## 5  Taylor Swift           Reputation      WW  4500000 November 10, 2017
## 6  Taylor Swift                Lover      WW  3200000   August 23, 2019
## 7       Beyoncé  Dangerously in Love      WW 11000000     June 23, 2003
## 8       Beyoncé                B'Day      WW  8000000 September 1, 2006
## 9       Beyoncé I Am... Sasha Fierce      WW  8000000 November 14, 2008
## 10      Beyoncé              Beyoncé      WW  5000000 December 13, 2013
## 11      Beyoncé             Lemonade      WW  2500000    April 23, 2016
```

## SELECT DISTINCT and GROUP By query to identify unique values present

Let's use a SELECT DISTINCT query to find out more about the composition of our data. Grouping by artist and title, we learn we have data for 6 unique albums by Beyoncé (comprising 18 entries), and 8 unique albums by Taylor Swift (comprising 30 entries.)  Each album has between 1-6 rows of associated sales data. 


```r
dbGetQuery(con, '
SELECT DISTINCT artist, title, SUBSTR(released, -4) as year_released, count(title) as count
FROM sales
GROUP BY artist, title
ORDER BY artist DESC, year_released ASC
                          ')
```

```
##          artist                title year_released count
## 1  Taylor Swift         Taylor Swift          2006     1
## 2  Taylor Swift             Fearless          2008     4
## 3  Taylor Swift            Speak Now          2010     3
## 4  Taylor Swift                  Red          2012     4
## 5  Taylor Swift                 1989          2014     6
## 6  Taylor Swift           Reputation          2017     5
## 7  Taylor Swift                Lover          2019     6
## 8  Taylor Swift             Folklore          2020     1
## 9       Beyoncé  Dangerously in Love          2003     3
## 10      Beyoncé                B'Day          2006     3
## 11      Beyoncé I Am... Sasha Fierce          2008     3
## 12      Beyoncé                    4          2011     2
## 13      Beyoncé              Beyoncé          2013     3
## 14      Beyoncé             Lemonade          2016     4
```

## SELECT and GROUP_CONCAT to summarise available data

Our first glimpse of the sales data showed that this dataset contained data for Worldwide sales as well as for a varying number of individual countries. In the following query, I used `GROUP_CONCAT` to show the countries I had sales data for, alongside a count of these countries, for each album. 


```r
dbGetQuery(con, '
SELECT title, SUBSTR(released, -4) as year_released, GROUP_CONCAT(Country) as countries, count(Country) AS country_count 
FROM sales
GROUP BY artist, title
ORDER BY artist DESC, year_released ASC
                          ')
```

```
##                   title year_released            countries country_count
## 1          Taylor Swift          2006                   US             1
## 2              Fearless          2008         WW,US,AUS,UK             4
## 3             Speak Now          2010             WW,US,UK             3
## 4                   Red          2012         WW,US,JPN,UK             4
## 5                  1989          2014 WW,US,CAN,FRA,JPN,UK             6
## 6            Reputation          2017     WW,US,CAN,FRA,UK             5
## 7                 Lover          2019 WW,US,CAN,FRA,JPN,UK             6
## 8              Folklore          2020                 <NA>             0
## 9   Dangerously in Love          2003             WW,US,UK             3
## 10                B'Day          2006             WW,US,UK             3
## 11 I Am... Sasha Fierce          2008             WW,US,UK             3
## 12                    4          2011                US,UK             2
## 13              Beyoncé          2013             WW,US,UK             3
## 14             Lemonade          2016          WW,US,UK,FR             4
```

This confirms that for almost all albums in our dataset, we have US, UK and worldwide sales data. 

# Working with the charts data and using JOIN to merge tables 

## SELECT query using WHERE ... IN to filter records  

Now, let's view data available from the charts table as a precursor to merging this data with the sales data. This table lists artist, title, chart, and (peak) chart_position for each album, across a number of charts. Based on the review of sales data above, I'm going to extract only results for the US and UK to join to the data in the sales table.  


```r
dbGetQuery(con, '
SELECT artist, title, chart, chart_position, released
FROM charts
WHERE chart IN ("US", "UK")
                          ')
```

```
##          artist                title chart chart_position
## 1  Taylor Swift         Taylor Swift    US              5
## 2  Taylor Swift         Taylor Swift    UK             81
## 3  Taylor Swift             Fearless    US              1
## 4  Taylor Swift             Fearless    UK              5
## 5  Taylor Swift            Speak Now    US              1
## 6  Taylor Swift            Speak Now    UK              6
## 7  Taylor Swift                  Red    US              1
## 8  Taylor Swift                  Red    UK              1
## 9  Taylor Swift                 1989    US              1
## 10 Taylor Swift                 1989    UK              1
## 11 Taylor Swift           Reputation    US              1
## 12 Taylor Swift           Reputation    UK              1
## 13 Taylor Swift                Lover    US              1
## 14 Taylor Swift                Lover    UK              1
## 15 Taylor Swift             Folklore    US              1
## 16 Taylor Swift             Folklore    UK              1
## 17      Beyoncé  Dangerously in Love    US              1
## 18      Beyoncé  Dangerously in Love    UK              1
## 19      Beyoncé                B'Day    US              1
## 20      Beyoncé                B'Day    UK              3
## 21      Beyoncé I Am... Sasha Fierce    US              1
## 22      Beyoncé I Am... Sasha Fierce    UK              2
## 23      Beyoncé                    4    US              1
## 24      Beyoncé                    4    UK              1
## 25      Beyoncé              Beyoncé    US              1
## 26      Beyoncé              Beyoncé    UK              2
## 27      Beyoncé             Lemonade    US              1
## 28      Beyoncé             Lemonade    UK              1
##                      released
## 1            October 24, 2006
## 2            October 24, 2006
## 3           November 11, 2008
## 4           November 11, 2008
## 5            October 25, 2010
## 6            October 25, 2010
## 7            October 22, 2012
## 8            October 22, 2012
## 9            October 27, 2014
## 10           October 27, 2014
## 11          November 10, 2017
## 12          November 10, 2017
## 13            August 23, 2019
## 14            August 23, 2019
## 15              July 24, 2020
## 16              July 24, 2020
## 17     June 23, 2003 (UK)[39]
## 18     June 23, 2003 (UK)[39]
## 19 September 1, 2006 (US)[51]
## 20 September 1, 2006 (US)[51]
## 21          November 14, 2008
## 22          November 14, 2008
## 23              June 24, 2011
## 24              June 24, 2011
## 25          December 13, 2013
## 26          December 13, 2013
## 27             April 23, 2016
## 28             April 23, 2016
```

This gives us 28 rows of chart_position data, for 14 albums, across two countries.

I'm going to manipulate released dates to exclude extraneous characters, as before:


```r
dbExecute(con, "
UPDATE charts
SET released = TRIM(SUBSTR(released, 1, INSTR(released, ' (')))
WHERE INSTR(released, ' (')>0; 
                          ")
```

```
## [1] 20
```

## JOIN query to join tables

Now let's try a simple left join to combine sales and chart position data for each album. This gives all rows in the sales table, and matching rows in the charts table.

Note that an inner join would give us 25 rows of data. We would lose the UK row for Taylor Swift's self-titled album released in 2006, and the US and UK rows for Folklore, due to lack of rows with matching country codes in the sales table for these records.  


```r
dbGetQuery(con, '
SELECT charts.artist, charts.title, SUBSTR(charts.released, -4) as year_released, charts.chart, charts.chart_position, sales.sales  
FROM charts
LEFT JOIN sales
ON charts.chart = sales.country and
charts.title = sales.title
WHERE charts.chart IN ("US", "UK")
ORDER BY charts.artist DESC, year_released ASC
                          ')
```

```
##          artist                title year_released chart chart_position   sales
## 1  Taylor Swift         Taylor Swift          2006    US              5 5720000
## 2  Taylor Swift         Taylor Swift          2006    UK             81      NA
## 3  Taylor Swift             Fearless          2008    US              1 7180000
## 4  Taylor Swift             Fearless          2008    UK              5  609000
## 5  Taylor Swift            Speak Now          2010    US              1 4694000
## 6  Taylor Swift            Speak Now          2010    UK              6  169281
## 7  Taylor Swift                  Red          2012    US              1 4465000
## 8  Taylor Swift                  Red          2012    UK              1  693000
## 9  Taylor Swift                 1989          2014    US              1 6215000
## 10 Taylor Swift                 1989          2014    UK              1 1250000
## 11 Taylor Swift           Reputation          2017    US              1 2300000
## 12 Taylor Swift           Reputation          2017    UK              1  378000
## 13 Taylor Swift                Lover          2019    US              1 1085000
## 14 Taylor Swift                Lover          2019    UK              1  221654
## 15 Taylor Swift             Folklore          2020    US              1      NA
## 16 Taylor Swift             Folklore          2020    UK              1      NA
## 17      Beyoncé  Dangerously in Love          2003    US              1 5100000
## 18      Beyoncé  Dangerously in Love          2003    UK              1 1260000
## 19      Beyoncé                B'Day          2006    US              1 3610000
## 20      Beyoncé                B'Day          2006    UK              3  700000
## 21      Beyoncé I Am... Sasha Fierce          2008    US              1 3380000
## 22      Beyoncé I Am... Sasha Fierce          2008    UK              2 1740000
## 23      Beyoncé                    4          2011    US              1 1500000
## 24      Beyoncé                    4          2011    UK              1  791000
## 25      Beyoncé              Beyoncé          2013    US              1 2512000
## 26      Beyoncé              Beyoncé          2013    UK              2  418000
## 27      Beyoncé             Lemonade          2016    US              1 1554000
## 28      Beyoncé             Lemonade          2016    UK              1  328000
```

# Next steps

There are a lot of things I'd like to explore further in my SQL learning journey. How would I use the data in the sales table to calculate the percentage of worldwide sales represented by UK and US sales respectively, for each album?  Can I pivot this data easily, to create a table with a column for US chart position/sales and a column for UK chart position/sales? How do I export this data in a way that makes it easy to generate nice tables and graphs in R Markdown?  

Let these, however, be questions for a future post.

# Finally, remember to disconnect

At the end of this process, best practice is always to disconnect from the database.


```r
dbDisconnect(con)
```

# References and additional resources:

Here are some references I drew on when preparing this tutorial, and pointers to useful resources for future experimentation with SQL.

https://db.rstudio.com/getting-started/
https://www.rdocumentation.org/packages/DBI/versions/0.5-1/topics/dbGetQuery
https://www.rdocumentation.org/packages/DBI/versions/0.5-1/topics/dbExecute

https://sciencificity-blog.netlify.app/posts/2020-12-31-using-tidyverse-with-dbs-partiii/
https://www.r-bloggers.com/2012/11/r-and-sqlite-part-1/

https://rpubs.com/JaneDoe/698719

https://programminghistorian.org/en/lessons/getting-started-with-mysql-using-r

https://jagg19.github.io/2019/05/mysql-r/

https://irene.rbind.io/post/using-sql-in-rstudio/

https://tbradley1013.github.io/2017/08/26/sql-management-in-r/
