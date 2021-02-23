---
title: Summary tables using gt
author: Lynley Aldridge
date: '2021-02-14'
tags:
  - gt
  - Rstats
  - TidyTuesday
  - Tutorial
slug: summary-tables-using-gt
lastmod: '2021-02-15T20:33:29+11:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
---

In this post, I walk through the various steps involved in creating the summary table shown here (based on the Tidy Tuesday Taylor Swift data), showcasing various capabilities of the `gt` package. 

<!--more-->

# Setup

First, let's load packages and read in the data (as manipulated in a [previous post](/2021/02/12/experimenting-with-sql-in-r-markdown-more-join-queries/)):


```r
#load packages
library(tidyverse)
library(gt) # for summary tables

#load data
albums <- read.csv('https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/data/albums.csv')
```

# Tables in R Markdown and blogdown

There are a variety of packages available for producing tables in R, and it's my impression that choosing the correct option depends partly on the functionality you desire and partly on the output formats you'll be using. I've included some guides to different packages available for generating tables with R on my [resources](/../../../../resources) page. In making the below tables in gt, I drew on the [package documentation for gt](https://gt.rstudio.com/articles/intro-creating-gt-tables.html), [a worked Dancing With the Stars example from JLaw](https://jlaw.netlify.app/2020/11/24/what-s-the-most-successful-dancing-with-the-stars-profession-visualizing-with-gt/), and resources from Thomas Mock and Georgios Karamanis referred to below.

# A simple table with an inline bar chart

Let's start by creating a simple table in gt, using gt's capacity to process custom html functions. 

First, we need to define the function we're wanting to use to create our inline bar chart. I adapted this from Thomas Mock's [Embedding custom HTML in gt tables](https://themockup.blog/posts/2020-10-31-embedding-custom-features-in-gt-tables/), which has great examples of many more things custom HTML enables you to do in gt tables:


```r
bar_chart <- function(value, color = "#795548"){
  glue::glue("<span style=\"display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: {color}; color: {color}; width: {value}%\"> &nbsp; </span>") %>% 
    as.character() %>% 
    gt::html()
}
```

Next, let's refresh our memory of the data we have available:


```r
head(albums)
```

```
##   X       artist        title year US_chart UK_chart US_sales WW_sales
## 1 1 Taylor Swift Taylor Swift 2006        5       81      5.7       NA
## 2 2 Taylor Swift     Fearless 2008        1        5      7.2     12.0
## 3 3 Taylor Swift    Speak Now 2010        1        6      4.7      5.0
## 4 4 Taylor Swift          Red 2012        1        1      4.5      6.0
## 5 5 Taylor Swift         1989 2014        1        1      6.2     10.1
## 6 6 Taylor Swift   Reputation 2017        1        1      2.3      4.5
##   other_sales US_percent other_percent
## 1          NA         NA            NA
## 2         4.8       59.8          40.2
## 3         0.3       93.9           6.1
## 4         1.5       74.4          25.6
## 5         3.9       61.5          38.5
## 6         2.2       51.1          48.9
```

To create a table with an inline bar chart using gt, let's first use the code below to create a new data frame (`albums with plots`) to use as the basis of this table. This filters `albums` to include only Taylor Swift albums, dropping albums for which full sales data are unavailable (Taylor Swift and Folklore). We then use `mutate()` to create a new column (`percent_plot`) which contains the results of applying the `bar_chart` function to the values in the `other_percent` column. Finally, we select columns we want to use in our table:


```r
albums_with_plots <- albums %>%
    filter(artist == "Taylor Swift") %>%
    drop_na() %>%
    mutate(percent_plot = map(other_percent, ~bar_chart(value = .x))) %>% 
    select(title, year, US_chart, UK_chart, WW_sales, US_sales, other_sales, other_percent, percent_plot) 
```

Let's explore what we've done by viewing relevant parts of our new data frame (here I select the first two rows and the first, second, eighth and ninth columns):


```r
view(albums_with_plots)[1:2, c(1, 2, 8, 9)]
```

```
##       title year other_percent
## 1  Fearless 2008          40.2
## 2 Speak Now 2010           6.1
##                                                                                                                                                           percent_plot
## 1 <span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 40.2%"> &nbsp; </span>
## 2  <span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 6.1%"> &nbsp; </span>
```

We can see that the html code needed to draw a bar of appropriate width (40.2 for the first row, and 6.1 for the second) has been mapped to the percent_plot column.

Creating a basic table using gt is as simple as applying the `gt()` function to the data frame:


```r
gt(albums_with_plots)
```

<!--html_preserve--><style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#tsqbwjbfuc .gt_table {
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

#tsqbwjbfuc .gt_heading {
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

#tsqbwjbfuc .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#tsqbwjbfuc .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 0;
  padding-bottom: 4px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#tsqbwjbfuc .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#tsqbwjbfuc .gt_col_headings {
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

#tsqbwjbfuc .gt_col_heading {
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

#tsqbwjbfuc .gt_column_spanner_outer {
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

#tsqbwjbfuc .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#tsqbwjbfuc .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#tsqbwjbfuc .gt_column_spanner {
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

#tsqbwjbfuc .gt_group_heading {
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

#tsqbwjbfuc .gt_empty_group_heading {
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

#tsqbwjbfuc .gt_from_md > :first-child {
  margin-top: 0;
}

#tsqbwjbfuc .gt_from_md > :last-child {
  margin-bottom: 0;
}

#tsqbwjbfuc .gt_row {
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

#tsqbwjbfuc .gt_stub {
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

#tsqbwjbfuc .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#tsqbwjbfuc .gt_first_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
}

#tsqbwjbfuc .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#tsqbwjbfuc .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#tsqbwjbfuc .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#tsqbwjbfuc .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#tsqbwjbfuc .gt_footnotes {
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

#tsqbwjbfuc .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding: 4px;
}

#tsqbwjbfuc .gt_sourcenotes {
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

#tsqbwjbfuc .gt_sourcenote {
  font-size: 90%;
  padding: 4px;
}

#tsqbwjbfuc .gt_left {
  text-align: left;
}

#tsqbwjbfuc .gt_center {
  text-align: center;
}

#tsqbwjbfuc .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#tsqbwjbfuc .gt_font_normal {
  font-weight: normal;
}

#tsqbwjbfuc .gt_font_bold {
  font-weight: bold;
}

#tsqbwjbfuc .gt_font_italic {
  font-style: italic;
}

#tsqbwjbfuc .gt_super {
  font-size: 65%;
}

#tsqbwjbfuc .gt_footnote_marks {
  font-style: italic;
  font-size: 65%;
}
</style>
<div id="tsqbwjbfuc" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;"><table class="gt_table">
  
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1">title</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">year</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">US_chart</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">UK_chart</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1">WW_sales</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1">US_sales</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1">other_sales</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1">other_percent</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">percent_plot</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr>
      <td class="gt_row gt_left">Fearless</td>
      <td class="gt_row gt_center">2008</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">5</td>
      <td class="gt_row gt_right">12.0</td>
      <td class="gt_row gt_right">7.2</td>
      <td class="gt_row gt_right">4.8</td>
      <td class="gt_row gt_right">40.2</td>
      <td class="gt_row gt_center"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 40.2%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left">Speak Now</td>
      <td class="gt_row gt_center">2010</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">6</td>
      <td class="gt_row gt_right">5.0</td>
      <td class="gt_row gt_right">4.7</td>
      <td class="gt_row gt_right">0.3</td>
      <td class="gt_row gt_right">6.1</td>
      <td class="gt_row gt_center"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 6.1%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left">Red</td>
      <td class="gt_row gt_center">2012</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_right">6.0</td>
      <td class="gt_row gt_right">4.5</td>
      <td class="gt_row gt_right">1.5</td>
      <td class="gt_row gt_right">25.6</td>
      <td class="gt_row gt_center"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 25.6%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left">1989</td>
      <td class="gt_row gt_center">2014</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_right">10.1</td>
      <td class="gt_row gt_right">6.2</td>
      <td class="gt_row gt_right">3.9</td>
      <td class="gt_row gt_right">38.5</td>
      <td class="gt_row gt_center"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 38.5%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left">Reputation</td>
      <td class="gt_row gt_center">2017</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_right">4.5</td>
      <td class="gt_row gt_right">2.3</td>
      <td class="gt_row gt_right">2.2</td>
      <td class="gt_row gt_right">48.9</td>
      <td class="gt_row gt_center"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 48.9%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left">Lover</td>
      <td class="gt_row gt_center">2019</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_right">3.2</td>
      <td class="gt_row gt_right">1.1</td>
      <td class="gt_row gt_right">2.1</td>
      <td class="gt_row gt_right">66.1</td>
      <td class="gt_row gt_center"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 66.1%"> &nbsp; </span></td>
    </tr>
  </tbody>
  
  
</table></div><!--/html_preserve-->

# Adding images and combining columns

Before looking at the functionality available within gt to format this table, let's look at some further manipulations we can make to our underlying data frame.  

One of the tables that sparked my interest in experimenting with gt was [the table Georgios Karamanis made summarizing this data](https://github.com/gkaramanis/tidytuesday/blob/master/2020-week40/). I adapted his code below to use album images and combine columns.

This code uses the `mutate()` and `paste0()` functions to paste data in the title and year columns into the new `title_released` column (to make a string consisting of the asterisks to signify bold in markdown formatting around the album title, a line break, and then the year the album was released). The stem required for image urls is pasted into the `ìmg` column before the album name, with `.jpg` following, to create the path to each image:


```r
albums_with_urls <- albums_with_plots %>% 

    # create new column combining values from title and released columns
    mutate(title_released = paste0("**", title, "**", 
                            "<br>", year)) %>% 
    
    # create new column containing urls to album art
    mutate(img = paste0("https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/", title, ".jpg")) %>%
  
    # select columns for inclusion in table in desired order  
    select(img, title_released, US_chart, UK_chart, WW_sales, US_sales, other_sales, other_percent, percent_plot)
```

Let's look at the changes we've made to our data frame:


```r
view(albums_with_urls)[1:2, c("title_released", "img")]
```

```
##          title_released
## 1  **Fearless**<br>2008
## 2 **Speak Now**<br>2010
##                                                                                                    img
## 1  https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/Fearless.jpg
## 2 https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/Speak Now.jpg
```

Now let's see what happens when we use these new columns of data in a table. Here we create and view a table (`gt1`) using the updated data, using the `fmt_markdown()` function to format the `title_released` column as markdown text, and the `text_transform()` function to apply a function to the text in the `img` column, retrieving the image at the specified url, with specified height:


```r
# create table
gt1 <- gt(albums_with_urls) %>%
  
    fmt_markdown(columns = c("title_released")) %>%
  
    text_transform(
      locations = cells_body(vars(img)),
      fn = function(x){
      web_image(url = x, height = 80)
      }
    ) 

# view table
gt1
```

<!--html_preserve--><style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#nvclgijmyz .gt_table {
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

#nvclgijmyz .gt_heading {
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

#nvclgijmyz .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#nvclgijmyz .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 0;
  padding-bottom: 4px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#nvclgijmyz .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#nvclgijmyz .gt_col_headings {
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

#nvclgijmyz .gt_col_heading {
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

#nvclgijmyz .gt_column_spanner_outer {
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

#nvclgijmyz .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#nvclgijmyz .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#nvclgijmyz .gt_column_spanner {
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

#nvclgijmyz .gt_group_heading {
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

#nvclgijmyz .gt_empty_group_heading {
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

#nvclgijmyz .gt_from_md > :first-child {
  margin-top: 0;
}

#nvclgijmyz .gt_from_md > :last-child {
  margin-bottom: 0;
}

#nvclgijmyz .gt_row {
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

#nvclgijmyz .gt_stub {
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

#nvclgijmyz .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#nvclgijmyz .gt_first_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
}

#nvclgijmyz .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#nvclgijmyz .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#nvclgijmyz .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#nvclgijmyz .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#nvclgijmyz .gt_footnotes {
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

#nvclgijmyz .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding: 4px;
}

#nvclgijmyz .gt_sourcenotes {
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

#nvclgijmyz .gt_sourcenote {
  font-size: 90%;
  padding: 4px;
}

#nvclgijmyz .gt_left {
  text-align: left;
}

#nvclgijmyz .gt_center {
  text-align: center;
}

#nvclgijmyz .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#nvclgijmyz .gt_font_normal {
  font-weight: normal;
}

#nvclgijmyz .gt_font_bold {
  font-weight: bold;
}

#nvclgijmyz .gt_font_italic {
  font-style: italic;
}

#nvclgijmyz .gt_super {
  font-size: 65%;
}

#nvclgijmyz .gt_footnote_marks {
  font-style: italic;
  font-size: 65%;
}
</style>
<div id="nvclgijmyz" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;"><table class="gt_table">
  
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1">img</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1">title_released</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">US_chart</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">UK_chart</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1">WW_sales</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1">US_sales</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1">other_sales</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1">other_percent</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">percent_plot</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr>
      <td class="gt_row gt_left"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/Fearless.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left"><div class='gt_from_md'><p><strong>Fearless</strong><br>2008</p>
</div></td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">5</td>
      <td class="gt_row gt_right">12.0</td>
      <td class="gt_row gt_right">7.2</td>
      <td class="gt_row gt_right">4.8</td>
      <td class="gt_row gt_right">40.2</td>
      <td class="gt_row gt_center"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 40.2%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/Speak Now.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left"><div class='gt_from_md'><p><strong>Speak Now</strong><br>2010</p>
</div></td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">6</td>
      <td class="gt_row gt_right">5.0</td>
      <td class="gt_row gt_right">4.7</td>
      <td class="gt_row gt_right">0.3</td>
      <td class="gt_row gt_right">6.1</td>
      <td class="gt_row gt_center"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 6.1%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/Red.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left"><div class='gt_from_md'><p><strong>Red</strong><br>2012</p>
</div></td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_right">6.0</td>
      <td class="gt_row gt_right">4.5</td>
      <td class="gt_row gt_right">1.5</td>
      <td class="gt_row gt_right">25.6</td>
      <td class="gt_row gt_center"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 25.6%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/1989.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left"><div class='gt_from_md'><p><strong>1989</strong><br>2014</p>
</div></td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_right">10.1</td>
      <td class="gt_row gt_right">6.2</td>
      <td class="gt_row gt_right">3.9</td>
      <td class="gt_row gt_right">38.5</td>
      <td class="gt_row gt_center"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 38.5%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/Reputation.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left"><div class='gt_from_md'><p><strong>Reputation</strong><br>2017</p>
</div></td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_right">4.5</td>
      <td class="gt_row gt_right">2.3</td>
      <td class="gt_row gt_right">2.2</td>
      <td class="gt_row gt_right">48.9</td>
      <td class="gt_row gt_center"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 48.9%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/Lover.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left"><div class='gt_from_md'><p><strong>Lover</strong><br>2019</p>
</div></td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_right">3.2</td>
      <td class="gt_row gt_right">1.1</td>
      <td class="gt_row gt_right">2.1</td>
      <td class="gt_row gt_right">66.1</td>
      <td class="gt_row gt_center"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 66.1%"> &nbsp; </span></td>
    </tr>
  </tbody>
  
  
</table></div><!--/html_preserve-->

# Column labels and spanner headings

Next, we want to add column labels and spanner headings across specified groups of columns, using the `cols_label()` and `tab_spanner()` functions. The code below also applies html formatting to the text in the label we've given to the `title_released` column:


```r
# create table

gt2 <- gt1 %>%
  
    # set column labels 
    cols_label(
      img = "",
      title_released = html(
        "<div style = 'text-align:left;'>
        <span style='font-weight:bold'>Album</span><br> 
        <span style='font-weight:normal'>Released</span>
        </div>"),
      US_chart = "US",
      UK_chart = "UK",
      WW_sales = "Total",
      US_sales = "US",
      other_sales = "Intl",
      other_percent = "%",
      percent_plot = "Plot") %>%
  
    # create headings spanning multiple columns
    tab_spanner(label = "Chart position", columns = vars(US_chart, UK_chart)) %>%
    tab_spanner(label = "Sales (millions)", columns = vars(WW_sales, US_sales, other_sales)) %>%
    tab_spanner(label = "International sales", columns = vars(other_percent, percent_plot)) 
  
# view table
gt2
```

<!--html_preserve--><style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#tmolsksbcs .gt_table {
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

#tmolsksbcs .gt_heading {
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

#tmolsksbcs .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#tmolsksbcs .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 0;
  padding-bottom: 4px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#tmolsksbcs .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#tmolsksbcs .gt_col_headings {
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

#tmolsksbcs .gt_col_heading {
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

#tmolsksbcs .gt_column_spanner_outer {
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

#tmolsksbcs .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#tmolsksbcs .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#tmolsksbcs .gt_column_spanner {
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

#tmolsksbcs .gt_group_heading {
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

#tmolsksbcs .gt_empty_group_heading {
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

#tmolsksbcs .gt_from_md > :first-child {
  margin-top: 0;
}

#tmolsksbcs .gt_from_md > :last-child {
  margin-bottom: 0;
}

#tmolsksbcs .gt_row {
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

#tmolsksbcs .gt_stub {
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

#tmolsksbcs .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#tmolsksbcs .gt_first_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
}

#tmolsksbcs .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#tmolsksbcs .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#tmolsksbcs .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#tmolsksbcs .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#tmolsksbcs .gt_footnotes {
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

#tmolsksbcs .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding: 4px;
}

#tmolsksbcs .gt_sourcenotes {
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

#tmolsksbcs .gt_sourcenote {
  font-size: 90%;
  padding: 4px;
}

#tmolsksbcs .gt_left {
  text-align: left;
}

#tmolsksbcs .gt_center {
  text-align: center;
}

#tmolsksbcs .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#tmolsksbcs .gt_font_normal {
  font-weight: normal;
}

#tmolsksbcs .gt_font_bold {
  font-weight: bold;
}

#tmolsksbcs .gt_font_italic {
  font-style: italic;
}

#tmolsksbcs .gt_super {
  font-size: 65%;
}

#tmolsksbcs .gt_footnote_marks {
  font-style: italic;
  font-size: 65%;
}
</style>
<div id="tmolsksbcs" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;"><table class="gt_table">
  
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_center gt_columns_bottom_border" rowspan="2" colspan="1"></th>
      <th class="gt_col_heading gt_center gt_columns_bottom_border" rowspan="2" colspan="1"><div style = 'text-align:left;'>
        <span style='font-weight:bold'>Album</span><br> 
        <span style='font-weight:normal'>Released</span>
        </div></th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="2">
        <span class="gt_column_spanner">Chart position</span>
      </th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="3">
        <span class="gt_column_spanner">Sales (millions)</span>
      </th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="2">
        <span class="gt_column_spanner">International sales</span>
      </th>
    </tr>
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">US</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">UK</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">Total</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">US</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">Intl</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">%</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">Plot</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr>
      <td class="gt_row gt_left"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/Fearless.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left"><div class='gt_from_md'><p><strong>Fearless</strong><br>2008</p>
</div></td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">5</td>
      <td class="gt_row gt_right">12.0</td>
      <td class="gt_row gt_right">7.2</td>
      <td class="gt_row gt_right">4.8</td>
      <td class="gt_row gt_right">40.2</td>
      <td class="gt_row gt_center"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 40.2%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/Speak Now.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left"><div class='gt_from_md'><p><strong>Speak Now</strong><br>2010</p>
</div></td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">6</td>
      <td class="gt_row gt_right">5.0</td>
      <td class="gt_row gt_right">4.7</td>
      <td class="gt_row gt_right">0.3</td>
      <td class="gt_row gt_right">6.1</td>
      <td class="gt_row gt_center"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 6.1%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/Red.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left"><div class='gt_from_md'><p><strong>Red</strong><br>2012</p>
</div></td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_right">6.0</td>
      <td class="gt_row gt_right">4.5</td>
      <td class="gt_row gt_right">1.5</td>
      <td class="gt_row gt_right">25.6</td>
      <td class="gt_row gt_center"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 25.6%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/1989.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left"><div class='gt_from_md'><p><strong>1989</strong><br>2014</p>
</div></td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_right">10.1</td>
      <td class="gt_row gt_right">6.2</td>
      <td class="gt_row gt_right">3.9</td>
      <td class="gt_row gt_right">38.5</td>
      <td class="gt_row gt_center"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 38.5%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/Reputation.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left"><div class='gt_from_md'><p><strong>Reputation</strong><br>2017</p>
</div></td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_right">4.5</td>
      <td class="gt_row gt_right">2.3</td>
      <td class="gt_row gt_right">2.2</td>
      <td class="gt_row gt_right">48.9</td>
      <td class="gt_row gt_center"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 48.9%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/Lover.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left"><div class='gt_from_md'><p><strong>Lover</strong><br>2019</p>
</div></td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_right">3.2</td>
      <td class="gt_row gt_right">1.1</td>
      <td class="gt_row gt_right">2.1</td>
      <td class="gt_row gt_right">66.1</td>
      <td class="gt_row gt_center"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 66.1%"> &nbsp; </span></td>
    </tr>
  </tbody>
  
  
</table></div><!--/html_preserve-->

# Cell styles (color and alignment)

The `tab_style()` function allows us to apply various styles to cells targeted with the `locations = ` argument. Customizations possible using this function include changes to cell background color; cell text color, font and size; cell alignment; and so on. The code below applies color, bold formatting, and appropriate alignments to column labels, cells in the body of the table, column spanners, and specified columns in the table body:


```r
gt3 <- gt2 %>%
  
    # color column labels and cells in table body 
    tab_style(style = cell_text(color = "#795548"),
        locations = list(
          cells_column_labels(everything()),
          cells_body()
        ) 
    )%>%
  
    # color column spanners and make bold
    tab_style(style = cell_text(
          color = "#795548", 
          weight = "bold"
        ),
        locations = cells_column_spanners(spanners = vars("Chart position", 
                                    "Sales (millions)", 
                                    "International sales")
        ) 
    )%>%
  
    # horizontal alignment of cells [using column position] 
    tab_style(style = cell_text(align = 'center'),
      locations = cells_body(columns = 3:4)) %>%
  
    # horizontal alignment of cells [using column names]
    tab_style(style = cell_text(align = 'right'),
      locations = cells_body(columns = c("WW_sales", 
                                         "US_sales", 
                                         "other_sales",
                                         "other_percent"))) %>%
  
      tab_style(style = cell_text(align = 'left'),
      locations = cells_body(columns = c("percent_plot"))) %>%

    # vertical alignment of cells
    tab_style(style = cell_text(v_align = "middle"), 
        locations = cells_body()) 

gt3
```

<!--html_preserve--><style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#ipncwxwjzx .gt_table {
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

#ipncwxwjzx .gt_heading {
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

#ipncwxwjzx .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#ipncwxwjzx .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 0;
  padding-bottom: 4px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#ipncwxwjzx .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#ipncwxwjzx .gt_col_headings {
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

#ipncwxwjzx .gt_col_heading {
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

#ipncwxwjzx .gt_column_spanner_outer {
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

#ipncwxwjzx .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#ipncwxwjzx .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#ipncwxwjzx .gt_column_spanner {
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

#ipncwxwjzx .gt_group_heading {
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

#ipncwxwjzx .gt_empty_group_heading {
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

#ipncwxwjzx .gt_from_md > :first-child {
  margin-top: 0;
}

#ipncwxwjzx .gt_from_md > :last-child {
  margin-bottom: 0;
}

#ipncwxwjzx .gt_row {
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

#ipncwxwjzx .gt_stub {
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

#ipncwxwjzx .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#ipncwxwjzx .gt_first_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
}

#ipncwxwjzx .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#ipncwxwjzx .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#ipncwxwjzx .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#ipncwxwjzx .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#ipncwxwjzx .gt_footnotes {
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

#ipncwxwjzx .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding: 4px;
}

#ipncwxwjzx .gt_sourcenotes {
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

#ipncwxwjzx .gt_sourcenote {
  font-size: 90%;
  padding: 4px;
}

#ipncwxwjzx .gt_left {
  text-align: left;
}

#ipncwxwjzx .gt_center {
  text-align: center;
}

#ipncwxwjzx .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#ipncwxwjzx .gt_font_normal {
  font-weight: normal;
}

#ipncwxwjzx .gt_font_bold {
  font-weight: bold;
}

#ipncwxwjzx .gt_font_italic {
  font-style: italic;
}

#ipncwxwjzx .gt_super {
  font-size: 65%;
}

#ipncwxwjzx .gt_footnote_marks {
  font-style: italic;
  font-size: 65%;
}
</style>
<div id="ipncwxwjzx" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;"><table class="gt_table">
  
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_center gt_columns_bottom_border" rowspan="2" colspan="1" style="color: #795548;"></th>
      <th class="gt_col_heading gt_center gt_columns_bottom_border" rowspan="2" colspan="1" style="color: #795548;"><div style = 'text-align:left;'>
        <span style='font-weight:bold'>Album</span><br> 
        <span style='font-weight:normal'>Released</span>
        </div></th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="2" style="color: #795548; font-weight: bold;">
        <span class="gt_column_spanner">Chart position</span>
      </th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="3" style="color: #795548; font-weight: bold;">
        <span class="gt_column_spanner">Sales (millions)</span>
      </th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="2" style="color: #795548; font-weight: bold;">
        <span class="gt_column_spanner">International sales</span>
      </th>
    </tr>
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="color: #795548;">US</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="color: #795548;">UK</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="color: #795548;">Total</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="color: #795548;">US</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="color: #795548;">Intl</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="color: #795548;">%</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="color: #795548;">Plot</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/Fearless.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><div class='gt_from_md'><p><strong>Fearless</strong><br>2008</p>
</div></td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">1</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">5</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">12.0</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">7.2</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">4.8</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">40.2</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: left; vertical-align: middle;"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 40.2%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/Speak Now.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><div class='gt_from_md'><p><strong>Speak Now</strong><br>2010</p>
</div></td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">1</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">6</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">5.0</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">4.7</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">0.3</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">6.1</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: left; vertical-align: middle;"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 6.1%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/Red.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><div class='gt_from_md'><p><strong>Red</strong><br>2012</p>
</div></td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">1</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">1</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">6.0</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">4.5</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">1.5</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">25.6</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: left; vertical-align: middle;"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 25.6%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/1989.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><div class='gt_from_md'><p><strong>1989</strong><br>2014</p>
</div></td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">1</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">1</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">10.1</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">6.2</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">3.9</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">38.5</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: left; vertical-align: middle;"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 38.5%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/Reputation.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><div class='gt_from_md'><p><strong>Reputation</strong><br>2017</p>
</div></td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">1</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">1</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">4.5</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">2.3</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">2.2</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">48.9</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: left; vertical-align: middle;"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 48.9%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/Lover.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><div class='gt_from_md'><p><strong>Lover</strong><br>2019</p>
</div></td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">1</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">1</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">3.2</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">1.1</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">2.1</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">66.1</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: left; vertical-align: middle;"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 66.1%"> &nbsp; </span></td>
    </tr>
  </tbody>
  
  
</table></div><!--/html_preserve-->

# Table titles, source notes, footnotes, borders and column width

Next, we can give our table titles and source notes using the `tab_header()` and `tab_source_note()` functions. Using `md()` around text means this text will be formatted as markdown, and we can use `tab_style()` as above to format the color and size of these headings. Finally, the default width for tables with the gt/blogdown/Hugo/Wowchemy combination I'm using appears to vary depending on the contents of the columns (and to expand to fit the full page, once a heading is added to the table). We can use the `col_width()` function below to manually alter column widths as necessary (e.g., to ensure the plot is large enough):


```r
gt4 <- gt3 %>%
  
    # create title for table, formatting as markdown
    tab_header(
      title = md("**Taylor Swift's Speak Now sold primarily to US audiences, but international sales comprised an increasing proportion of her sales for each subsequent album**"),
      subtitle = md("*Peak chart position and number of copies sold by album and location*")) %>%
  
    # create source note for table, formatting as md and applying color
    tab_source_note(source_note = md("<span style = 'color:#795548'>Source: Billboard via Wikipedia, October 2020; excludes albums for which worldwide sales were unavailable<br>Table: Modified from Georgios Karamanis</span>")) %>%

    # color and size title and subtitle 
    tab_style(style = cell_text(
      color = "#795548",
      size = "large"),
      locations = cells_title(groups = "title")) %>%

    tab_style(style = cell_text(
      color = "#795548",
      size = "medium"),
      locations = cells_title(groups = "subtitle")) %>%
  
    # set width of columns
    cols_width(
      vars("title_released", "percent_plot") ~ px(150)) %>%

    # remove label for plot column
    cols_label(percent_plot = "")

gt4
```

<!--html_preserve--><style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#xetlojmnfw .gt_table {
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

#xetlojmnfw .gt_heading {
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

#xetlojmnfw .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#xetlojmnfw .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 0;
  padding-bottom: 4px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#xetlojmnfw .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#xetlojmnfw .gt_col_headings {
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

#xetlojmnfw .gt_col_heading {
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

#xetlojmnfw .gt_column_spanner_outer {
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

#xetlojmnfw .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#xetlojmnfw .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#xetlojmnfw .gt_column_spanner {
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

#xetlojmnfw .gt_group_heading {
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

#xetlojmnfw .gt_empty_group_heading {
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

#xetlojmnfw .gt_from_md > :first-child {
  margin-top: 0;
}

#xetlojmnfw .gt_from_md > :last-child {
  margin-bottom: 0;
}

#xetlojmnfw .gt_row {
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

#xetlojmnfw .gt_stub {
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

#xetlojmnfw .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#xetlojmnfw .gt_first_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
}

#xetlojmnfw .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#xetlojmnfw .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#xetlojmnfw .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#xetlojmnfw .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#xetlojmnfw .gt_footnotes {
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

#xetlojmnfw .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding: 4px;
}

#xetlojmnfw .gt_sourcenotes {
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

#xetlojmnfw .gt_sourcenote {
  font-size: 90%;
  padding: 4px;
}

#xetlojmnfw .gt_left {
  text-align: left;
}

#xetlojmnfw .gt_center {
  text-align: center;
}

#xetlojmnfw .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#xetlojmnfw .gt_font_normal {
  font-weight: normal;
}

#xetlojmnfw .gt_font_bold {
  font-weight: bold;
}

#xetlojmnfw .gt_font_italic {
  font-style: italic;
}

#xetlojmnfw .gt_super {
  font-size: 65%;
}

#xetlojmnfw .gt_footnote_marks {
  font-style: italic;
  font-size: 65%;
}
</style>
<div id="xetlojmnfw" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;"><table class="gt_table" style="table-layout: fixed;">
  <colgroup>
    <col/>
    <col style="width:150px;"/>
    <col/>
    <col/>
    <col/>
    <col/>
    <col/>
    <col/>
    <col style="width:150px;"/>
  </colgroup>
  <thead class="gt_header">
    <tr>
      <th colspan="9" class="gt_heading gt_title gt_font_normal" style="color: #795548; font-size: large;"><strong>Taylor Swift's Speak Now sold primarily to US audiences, but international sales comprised an increasing proportion of her sales for each subsequent album</strong></th>
    </tr>
    <tr>
      <th colspan="9" class="gt_heading gt_subtitle gt_font_normal gt_bottom_border" style="color: #795548; font-size: medium;"><em>Peak chart position and number of copies sold by album and location</em></th>
    </tr>
  </thead>
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_center gt_columns_bottom_border" rowspan="2" colspan="1" style="color: #795548;"></th>
      <th class="gt_col_heading gt_center gt_columns_bottom_border" rowspan="2" colspan="1" style="color: #795548;"><div style = 'text-align:left;'>
        <span style='font-weight:bold'>Album</span><br> 
        <span style='font-weight:normal'>Released</span>
        </div></th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="2" style="color: #795548; font-weight: bold;">
        <span class="gt_column_spanner">Chart position</span>
      </th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="3" style="color: #795548; font-weight: bold;">
        <span class="gt_column_spanner">Sales (millions)</span>
      </th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="2" style="color: #795548; font-weight: bold;">
        <span class="gt_column_spanner">International sales</span>
      </th>
    </tr>
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="color: #795548;">US</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="color: #795548;">UK</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="color: #795548;">Total</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="color: #795548;">US</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="color: #795548;">Intl</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="color: #795548;">%</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="color: #795548;"></th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/Fearless.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><div class='gt_from_md'><p><strong>Fearless</strong><br>2008</p>
</div></td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">1</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">5</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">12.0</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">7.2</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">4.8</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">40.2</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: left; vertical-align: middle;"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 40.2%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/Speak Now.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><div class='gt_from_md'><p><strong>Speak Now</strong><br>2010</p>
</div></td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">1</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">6</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">5.0</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">4.7</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">0.3</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">6.1</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: left; vertical-align: middle;"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 6.1%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/Red.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><div class='gt_from_md'><p><strong>Red</strong><br>2012</p>
</div></td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">1</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">1</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">6.0</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">4.5</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">1.5</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">25.6</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: left; vertical-align: middle;"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 25.6%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/1989.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><div class='gt_from_md'><p><strong>1989</strong><br>2014</p>
</div></td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">1</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">1</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">10.1</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">6.2</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">3.9</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">38.5</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: left; vertical-align: middle;"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 38.5%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/Reputation.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><div class='gt_from_md'><p><strong>Reputation</strong><br>2017</p>
</div></td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">1</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">1</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">4.5</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">2.3</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">2.2</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">48.9</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: left; vertical-align: middle;"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 48.9%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><img src="https://raw.githubusercontent.com/lynleyaldridge/tidytuesday/main/2020/2020-week40/img/Lover.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left" style="color: #795548; vertical-align: middle;"><div class='gt_from_md'><p><strong>Lover</strong><br>2019</p>
</div></td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">1</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: center; vertical-align: middle;">1</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">3.2</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">1.1</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">2.1</td>
      <td class="gt_row gt_right" style="color: #795548; text-align: right; vertical-align: middle;">66.1</td>
      <td class="gt_row gt_center" style="color: #795548; text-align: left; vertical-align: middle;"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 66.1%"> &nbsp; </span></td>
    </tr>
  </tbody>
  <tfoot class="gt_sourcenotes">
    <tr>
      <td class="gt_sourcenote" colspan="9"><span style = 'color:#795548'>Source: Billboard via Wikipedia, October 2020; excludes albums for which worldwide sales were unavailable<br>Table: Modified from Georgios Karamanis</span></td>
    </tr>
  </tfoot>
  
</table></div><!--/html_preserve-->

Note that there is also a `table_footnote()` function but I couldn't get this formatting correctly using gt, blogdown, Hugo and the Wowchemy theme. There are also options for formatting table borders, using `table_options()`, but these options changed the table I was previewing in R Studio, but not the output on my blog. Thus the code above is abbreviated to include only elements that work well in my environment. See [my tidytuesday repository](https://github.com/lynleyaldridge/tidytuesday/tree/main/2020/2020-week40) for the full code used to generate the image at the top of this post, which includes additional code for footnotes and borders.

# Next steps

It appears that blogdown and gt don't always play nicely together, and some of the functionality of gt won't be available unless I dig deeper into their interplay. It's possible to generate images in R and then save as a .png file, however, which is how I created the preview image used for this blogpost. 

I'm sure these tables could be further beautified with additional experimentation. I'd love to print percentage labels directly onto the inline bar chart, instead of having them as separate columns, for example. And I think inline bar charts showing the total number of copies sold, with shading highlighting the proportion of this represented by international sales, would be a more accurate visualization of this data.

Overall, I'm excited by what I've learned about the capabilities of gt, and I'm pleased with the additional knowledge of GitHub I've developed setting up my Tidy Tuesday repository and storing files online to draw on in these code examples. I've also been experimenting with ways of customizing summary rows at the bottom of tables in gt. But this post is long enough already, so let's leave that as a topic for a future post. 
