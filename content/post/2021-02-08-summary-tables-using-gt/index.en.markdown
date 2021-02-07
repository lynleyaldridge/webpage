---
title: "Summary tables using gt"
author: Lynley Aldridge
date: '2021-02-08'
slug: summary-tables-using-gt
categories: []
draft: TRUE 
tags:
  - Rstats
  - Tutorial 
  - gt
  - TidyTuesday
subtitle: ''
summary: ''
authors: []
lastmod: '2021-01-31T20:33:29+11:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

This post shows how I experimented with more complex features of gt, to create a summary table using the Taylor Swift and Beyoncé Tidy Tuesday data, and drawing on an example 

## Setup

As in the previous post, load packages, read in data, and clean and extract data using the following syntax:


```r
#load packages
library(tidyverse)
library(odbc)
library(DBI)
library(RSQLite)
library(gt) # for summary tables

# library(reactable) # not available for this version of R

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

# Extract data as df


```sql

SELECT charts.artist, charts.title, SUBSTR(charts.released, -4) AS released, 
  US_charts.chart_position AS US_position,
  UK_charts.chart_position AS UK_position,
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
ORDER BY charts.artist DESC, released ASC;
```

## Formatting tables nicely in R Markdown

How would we go about making a nice table for R Markdown reports. Lots of packages to try, arsenal, flextable, gt ... 

Reviews and demonstrations of multiple packages available for producing tables in R can be found at:

* David Keyes' summary of [How to make beautiful tables in R](https://rfortherestofus.com/2019/11/how-to-make-beautiful-tables-in-r/) offers short reviews and demonstrations of gt, kable and kableExtra, formattable, DT, reactable, flextable, huxtable, rhandsontable, and pixiedust

* Pascal Schmidt's [How to easily create descriptive summary statistics tables in R studio - By group](https://thatdatatho.com/2018/08/20/easily-create-descriptive-summary-statistic-tables-r-studio/) reviews and demonstrates packages that are particularly useful for summary statistics tables that compare groups, including two of my favorites so far for this purpose, arsenal and tableone

and ways to create highly customized and attractive summary tables include:


GT documentation:
https://blog.rstudio.com/2020/04/08/great-looking-tables-gt-0-2/


add percent_US bars for sales (look at rule 10 in Thomas Mock blog?

https://themockup.blog/posts/2020-09-04-10-table-rules-in-r/

https://themockup.blog/posts/2020-10-31-embedding-custom-features-in-gt-tables/

Okay, if you want to analyse sales by album, here's some industry-speak:

https://chartmasters.org/2020/10/taylor-swift-albums-and-songs-sales/

Obviously global sales totals for later albums will be less?

Over 70% of sales from US - abroad improved with success of 1989 

Reputation came out after streaming was inevitable and naturally sold less but high by today's standards ... 


To do - make 0s and bars for 0/NA disappear?
Change direction of caption
Combine Title and YEAR, not artist!!!  
Fix source note
Fix color notes in function and code below 
Fix alignment - all text centred vertically in preview below (not on website)
Fix table width 

Width is proportional to column title (Sales (%))


```r
df <- mutate(df, US_percent = ifelse(is.na(US_percent), 0, US_percent))

# function to create bar chart
# source: https://themockup.blog/posts/2020-10-31-embedding-custom-features-in-gt-tables/

bar_chart <- function(value, color = "red"){
  glue::glue("<span style=\"display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: {color}; color: {color}; width: {value}%\"> &nbsp; </span>") %>% 
    as.character() %>% 
    gt::html()
}


# code for table

df %>%
  
    filter(artist == "Taylor Swift") %>%
    
    mutate( 
    # combine values from two columns into a single column
    title_released = paste0("<span style='color:#795548'>**", title, "**</span>", 
                            "<br><span style='color:#795548'>", released, "</span>"), 
    
    # add a column containing urls to album art
    img = paste0("https://raw.githubusercontent.com/gkaramanis/tidytuesday/master/2020-week40/img/", 
                title, ".jpg"),
    
    #create bar plot column calling function defined above
    # source: https://themockup.blog/posts/2020-10-31-embedding-custom-features-in-gt-tables/
    percent_plot = map(US_percent, ~bar_chart(value = .x, 
                                              color = "#795548"))) %>% 

    # arrange by release date
    arrange(released) %>% 
  
    # select variables for inclusion in table
    select(img, title_released, US_position, UK_position, 
           WW_sales, US_sales, US_percent, percent_plot) %>%
 
  # create a table using gt
  gt() %>%
  
    # set column labels 
    cols_label(
      img = "",
      title_released = html("<div style='text-align:left;'>Album<br><span style='color:#795548'>Released</span></div>"),
      US_position = "US",
      UK_position = "UK",
      WW_sales = "World",
      US_sales = "US",
      # UK_sales = "UK",
      # other_sales = "Other",
      US_percent = "US", 
      percent_plot = "US sales (%)") %>%
      # UK_percent = "UK",
      # other_percent = "Other") %>%
  
    # format title_artist column as markdown text
    fmt_markdown(columns = c("title_released")) %>%
  
    # transform text in img column to display and appropriately size images
    text_transform(
      locations = cells_body(vars(img)),
      fn = function(x){
      web_image(url = x, height = 80)
      }
    ) %>%
  
    # transform NA values in all columns to "-"
    text_transform(
      locations = cells_body(columns = gt::everything()),
      fn = function(x) {
      str_replace(x, "NA", "–")
      }
    ) %>%
  
    
    # transform NA values in all columns to "-"
    text_transform(
      locations = cells_body(columns = gt::everything()),
      fn = function(x) {
      str_replace(x, "0.0", "–")
      }
    ) %>%

    # align cells containing numeric values right
    cols_align(align = "right", columns = c("WW_sales", "US_sales", "US_percent")) %>%
    cols_align(align = "center", columns = c("US_position", "UK_position")) %>% 
    cols_align(align = "left", columns = c("percent_plot")) %>% 
  
    # vertically align text to centre
    tab_style(
      style = cell_text(v_align = "middle"), 
        locations = cells_body()) %>%
                
    # create headings spanning multiple columns
    tab_spanner(label = "Chart position", columns = vars(US_position, UK_position)) %>%
    tab_spanner(label = "Sales ($ million)", columns = vars(WW_sales, US_sales)) %>%
    tab_spanner(label = "Sales (%)", columns = vars(US_percent)) %>%
  
    # create title for table
    tab_header(
      title = md("**Taylor Swift's Speak Now sold primarily to US audiences, while the proportion of sales made in the US was lowest for Lover**")) %>%
  
    # create source note for table
    tab_source_note("Source: Wikipedia, October 2020, Table: Modified from Georgios Karamanis") %>%
  
    # change column labels to uppercase - doesn't work in blogdown 
    tab_options(
      column_labels.text_transform = "uppercase",
      table.font.color = "#795548"
    )
```

<!--html_preserve--><style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#kylyyvjkez .gt_table {
  display: table;
  border-collapse: collapse;
  margin-left: auto;
  margin-right: auto;
  color: #795548;
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

#kylyyvjkez .gt_heading {
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

#kylyyvjkez .gt_title {
  color: #795548;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#kylyyvjkez .gt_subtitle {
  color: #795548;
  font-size: 85%;
  font-weight: initial;
  padding-top: 0;
  padding-bottom: 4px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#kylyyvjkez .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#kylyyvjkez .gt_col_headings {
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

#kylyyvjkez .gt_col_heading {
  color: #795548;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: uppercase;
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

#kylyyvjkez .gt_column_spanner_outer {
  color: #795548;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: uppercase;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#kylyyvjkez .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#kylyyvjkez .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#kylyyvjkez .gt_column_spanner {
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

#kylyyvjkez .gt_group_heading {
  padding: 8px;
  color: #795548;
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

#kylyyvjkez .gt_empty_group_heading {
  padding: 0.5px;
  color: #795548;
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

#kylyyvjkez .gt_from_md > :first-child {
  margin-top: 0;
}

#kylyyvjkez .gt_from_md > :last-child {
  margin-bottom: 0;
}

#kylyyvjkez .gt_row {
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

#kylyyvjkez .gt_stub {
  color: #795548;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 12px;
}

#kylyyvjkez .gt_summary_row {
  color: #795548;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#kylyyvjkez .gt_first_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
}

#kylyyvjkez .gt_grand_summary_row {
  color: #795548;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#kylyyvjkez .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#kylyyvjkez .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#kylyyvjkez .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#kylyyvjkez .gt_footnotes {
  color: #795548;
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

#kylyyvjkez .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding: 4px;
}

#kylyyvjkez .gt_sourcenotes {
  color: #795548;
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

#kylyyvjkez .gt_sourcenote {
  font-size: 90%;
  padding: 4px;
}

#kylyyvjkez .gt_left {
  text-align: left;
}

#kylyyvjkez .gt_center {
  text-align: center;
}

#kylyyvjkez .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#kylyyvjkez .gt_font_normal {
  font-weight: normal;
}

#kylyyvjkez .gt_font_bold {
  font-weight: bold;
}

#kylyyvjkez .gt_font_italic {
  font-style: italic;
}

#kylyyvjkez .gt_super {
  font-size: 65%;
}

#kylyyvjkez .gt_footnote_marks {
  font-style: italic;
  font-size: 65%;
}
</style>
<div id="kylyyvjkez" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;"><table class="gt_table">
  <thead class="gt_header">
    <tr>
      <th colspan="8" class="gt_heading gt_title gt_font_normal" style><strong>Taylor Swift's Speak Now sold primarily to US audiences, while the proportion of sales made in the US was lowest for Lover</strong></th>
    </tr>
    <tr>
      <th colspan="8" class="gt_heading gt_subtitle gt_font_normal gt_bottom_border" style></th>
    </tr>
  </thead>
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_center gt_columns_bottom_border" rowspan="2" colspan="1"></th>
      <th class="gt_col_heading gt_center gt_columns_bottom_border" rowspan="2" colspan="1"><div style='text-align:left;'>Album<br><span style='color:#795548'>Released</span></div></th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="2">
        <span class="gt_column_spanner">Chart position</span>
      </th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="2">
        <span class="gt_column_spanner">Sales ($ million)</span>
      </th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="1">
        <span class="gt_column_spanner">Sales (%)</span>
      </th>
      <th class="gt_col_heading gt_center gt_columns_bottom_border" rowspan="2" colspan="1">US sales (%)</th>
    </tr>
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">US</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">UK</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">World</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">US</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">US</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr>
      <td class="gt_row gt_left" style="vertical-align: middle;"><img src="https://raw.githubusercontent.com/gkaramanis/tidytuesday/master/2–-week40/img/Taylor Swift.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left" style="vertical-align: middle;"><div class='gt_from_md'><p><span style='color:#795548'><strong>Taylor Swift</strong></span><br><span style='color:#795548'>2006</span></p>
</div></td>
      <td class="gt_row gt_center" style="vertical-align: middle;">5</td>
      <td class="gt_row gt_center" style="vertical-align: middle;">81</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">–</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">5.7</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">–</td>
      <td class="gt_row gt_left" style="vertical-align: middle;"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 0%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left" style="vertical-align: middle;"><img src="https://raw.githubusercontent.com/gkaramanis/tidytuesday/master/2–-week40/img/Fearless.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left" style="vertical-align: middle;"><div class='gt_from_md'><p><span style='color:#795548'><strong>Fearless</strong></span><br><span style='color:#795548'>2008</span></p>
</div></td>
      <td class="gt_row gt_center" style="vertical-align: middle;">1</td>
      <td class="gt_row gt_center" style="vertical-align: middle;">5</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">12.0</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">7.2</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">59.8</td>
      <td class="gt_row gt_left" style="vertical-align: middle;"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 59.8%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left" style="vertical-align: middle;"><img src="https://raw.githubusercontent.com/gkaramanis/tidytuesday/master/2–-week40/img/Speak Now.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left" style="vertical-align: middle;"><div class='gt_from_md'><p><span style='color:#795548'><strong>Speak Now</strong></span><br><span style='color:#795548'>2–</span></p>
</div></td>
      <td class="gt_row gt_center" style="vertical-align: middle;">1</td>
      <td class="gt_row gt_center" style="vertical-align: middle;">6</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">5.0</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">4.7</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">93.9</td>
      <td class="gt_row gt_left" style="vertical-align: middle;"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 93.9%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left" style="vertical-align: middle;"><img src="https://raw.githubusercontent.com/gkaramanis/tidytuesday/master/2–-week40/img/Red.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left" style="vertical-align: middle;"><div class='gt_from_md'><p><span style='color:#795548'><strong>Red</strong></span><br><span style='color:#795548'>2012</span></p>
</div></td>
      <td class="gt_row gt_center" style="vertical-align: middle;">1</td>
      <td class="gt_row gt_center" style="vertical-align: middle;">1</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">6.0</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">4.5</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">74.4</td>
      <td class="gt_row gt_left" style="vertical-align: middle;"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 74.4%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left" style="vertical-align: middle;"><img src="https://raw.githubusercontent.com/gkaramanis/tidytuesday/master/2–-week40/img/1989.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left" style="vertical-align: middle;"><div class='gt_from_md'><p><span style='color:#795548'><strong>1989</strong></span><br><span style='color:#795548'>2014</span></p>
</div></td>
      <td class="gt_row gt_center" style="vertical-align: middle;">1</td>
      <td class="gt_row gt_center" style="vertical-align: middle;">1</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">10.1</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">6.2</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">61.5</td>
      <td class="gt_row gt_left" style="vertical-align: middle;"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 61.5%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left" style="vertical-align: middle;"><img src="https://raw.githubusercontent.com/gkaramanis/tidytuesday/master/2–-week40/img/Reputation.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left" style="vertical-align: middle;"><div class='gt_from_md'><p><span style='color:#795548'><strong>Reputation</strong></span><br><span style='color:#795548'>2017</span></p>
</div></td>
      <td class="gt_row gt_center" style="vertical-align: middle;">1</td>
      <td class="gt_row gt_center" style="vertical-align: middle;">1</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">4.5</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">2.3</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">51.1</td>
      <td class="gt_row gt_left" style="vertical-align: middle;"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 51.1%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left" style="vertical-align: middle;"><img src="https://raw.githubusercontent.com/gkaramanis/tidytuesday/master/2–-week40/img/Lover.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left" style="vertical-align: middle;"><div class='gt_from_md'><p><span style='color:#795548'><strong>Lover</strong></span><br><span style='color:#795548'>2019</span></p>
</div></td>
      <td class="gt_row gt_center" style="vertical-align: middle;">1</td>
      <td class="gt_row gt_center" style="vertical-align: middle;">1</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">3.2</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">1.1</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">33.9</td>
      <td class="gt_row gt_left" style="vertical-align: middle;"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 33.9%"> &nbsp; </span></td>
    </tr>
    <tr>
      <td class="gt_row gt_left" style="vertical-align: middle;"><img src="https://raw.githubusercontent.com/gkaramanis/tidytuesday/master/2–-week40/img/Folklore.jpg" style="height:80px;"></td>
      <td class="gt_row gt_left" style="vertical-align: middle;"><div class='gt_from_md'><p><span style='color:#795548'><strong>Folklore</strong></span><br><span style='color:#795548'>2–</span></p>
</div></td>
      <td class="gt_row gt_center" style="vertical-align: middle;">1</td>
      <td class="gt_row gt_center" style="vertical-align: middle;">1</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">–</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">–</td>
      <td class="gt_row gt_right" style="vertical-align: middle;">–</td>
      <td class="gt_row gt_left" style="vertical-align: middle;"><span style="display: inline-block; direction: ltr; border-radius: 4px; padding-right: 2px; background-color: #795548; color: #795548; width: 0%"> &nbsp; </span></td>
    </tr>
  </tbody>
  <tfoot class="gt_sourcenotes">
    <tr>
      <td class="gt_sourcenote" colspan="8">Source: Wikipedia, October 2020, Table: Modified from Georgios Karamanis</td>
    </tr>
  </tfoot>
  
</table></div><!--/html_preserve-->


    # size columns - not used
    cols_width(
    vars(title_artist) ~ px(150),
    vars(released) ~ px(80),
    vars(US_position) ~ px(35),
    vars(UK_position) ~ px(35),
    vars(WW_sales) ~ px(50),
    vars(US_sales) ~ px(50),
    vars(US_percent) ~ px(50)
    ) %>%


# A simpler table example

Add average rows for each artist, 



```r
df %>%
  
  # create a table using gt, grouping by artist and using title_artist for column names
  gt(rowname_col = "title", groupname_col = "artist") %>%
  
    # set column labels 
    cols_label(
      US_position = "US",
      UK_position = "UK",
      US_sales = "US",
      # UK_sales = "UK",
      WW_sales = "Worldwide",
      # other_sales = "Other",
      US_percent = "US") %>%
      # UK_percent = "UK",
      # other_percent = "Other") %>%
  
   # transform NA values in all columns to "-"
    text_transform(
      locations = cells_body(columns = gt::everything()),
      fn = function(x) {
      str_replace(x, "NA", "–")
      }
    ) %>%
  
    # create headings spanning multiple columns
    tab_spanner(label = "Chart position", columns = vars(US_position, UK_position)) %>%
    tab_spanner(label = "Sales (millions)", columns = vars(WW_sales, US_sales)) %>%
    tab_spanner(label = "Sales (percent)", columns = vars(US_percent)) %>%
  
    # create title for table
    tab_header(
      title = md("Taylor Swift and Beyoncé"))
```

<!--html_preserve--><style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#tzvhjzwuau .gt_table {
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

#tzvhjzwuau .gt_heading {
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

#tzvhjzwuau .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#tzvhjzwuau .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 0;
  padding-bottom: 4px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#tzvhjzwuau .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#tzvhjzwuau .gt_col_headings {
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

#tzvhjzwuau .gt_col_heading {
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

#tzvhjzwuau .gt_column_spanner_outer {
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

#tzvhjzwuau .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#tzvhjzwuau .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#tzvhjzwuau .gt_column_spanner {
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

#tzvhjzwuau .gt_group_heading {
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

#tzvhjzwuau .gt_empty_group_heading {
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

#tzvhjzwuau .gt_from_md > :first-child {
  margin-top: 0;
}

#tzvhjzwuau .gt_from_md > :last-child {
  margin-bottom: 0;
}

#tzvhjzwuau .gt_row {
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

#tzvhjzwuau .gt_stub {
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

#tzvhjzwuau .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#tzvhjzwuau .gt_first_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
}

#tzvhjzwuau .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#tzvhjzwuau .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#tzvhjzwuau .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#tzvhjzwuau .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#tzvhjzwuau .gt_footnotes {
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

#tzvhjzwuau .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding: 4px;
}

#tzvhjzwuau .gt_sourcenotes {
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

#tzvhjzwuau .gt_sourcenote {
  font-size: 90%;
  padding: 4px;
}

#tzvhjzwuau .gt_left {
  text-align: left;
}

#tzvhjzwuau .gt_center {
  text-align: center;
}

#tzvhjzwuau .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#tzvhjzwuau .gt_font_normal {
  font-weight: normal;
}

#tzvhjzwuau .gt_font_bold {
  font-weight: bold;
}

#tzvhjzwuau .gt_font_italic {
  font-style: italic;
}

#tzvhjzwuau .gt_super {
  font-size: 65%;
}

#tzvhjzwuau .gt_footnote_marks {
  font-style: italic;
  font-size: 65%;
}
</style>
<div id="tzvhjzwuau" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;"><table class="gt_table">
  <thead class="gt_header">
    <tr>
      <th colspan="11" class="gt_heading gt_title gt_font_normal" style>Taylor Swift and Beyoncé</th>
    </tr>
    <tr>
      <th colspan="11" class="gt_heading gt_subtitle gt_font_normal gt_bottom_border" style></th>
    </tr>
  </thead>
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="2" colspan="1"></th>
      <th class="gt_col_heading gt_center gt_columns_bottom_border" rowspan="2" colspan="1">released</th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="2">
        <span class="gt_column_spanner">Chart position</span>
      </th>
      <th class="gt_col_heading gt_center gt_columns_bottom_border" rowspan="2" colspan="1">UK_sales</th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="2">
        <span class="gt_column_spanner">Sales (millions)</span>
      </th>
      <th class="gt_col_heading gt_center gt_columns_bottom_border" rowspan="2" colspan="1">other_sales</th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="1">
        <span class="gt_column_spanner">Sales (percent)</span>
      </th>
      <th class="gt_col_heading gt_center gt_columns_bottom_border" rowspan="2" colspan="1">UK_percent</th>
      <th class="gt_col_heading gt_center gt_columns_bottom_border" rowspan="2" colspan="1">other_percent</th>
    </tr>
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">US</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">UK</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">Worldwide</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">US</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">US</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr class="gt_group_heading_row">
      <td colspan="11" class="gt_group_heading">Taylor Swift</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Taylor Swift</td>
      <td class="gt_row gt_left">2006</td>
      <td class="gt_row gt_left">5</td>
      <td class="gt_row gt_left">81</td>
      <td class="gt_row gt_right">–</td>
      <td class="gt_row gt_right">–</td>
      <td class="gt_row gt_right">5.7</td>
      <td class="gt_row gt_right">–</td>
      <td class="gt_row gt_right">0.0</td>
      <td class="gt_row gt_right">–</td>
      <td class="gt_row gt_right">–</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Fearless</td>
      <td class="gt_row gt_left">2008</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">5</td>
      <td class="gt_row gt_right">0.6</td>
      <td class="gt_row gt_right">12.0</td>
      <td class="gt_row gt_right">7.2</td>
      <td class="gt_row gt_right">4.2</td>
      <td class="gt_row gt_right">59.8</td>
      <td class="gt_row gt_right">5.1</td>
      <td class="gt_row gt_right">35.1</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Speak Now</td>
      <td class="gt_row gt_left">2010</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">6</td>
      <td class="gt_row gt_right">0.2</td>
      <td class="gt_row gt_right">5.0</td>
      <td class="gt_row gt_right">4.7</td>
      <td class="gt_row gt_right">0.1</td>
      <td class="gt_row gt_right">93.9</td>
      <td class="gt_row gt_right">3.4</td>
      <td class="gt_row gt_right">2.7</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Red</td>
      <td class="gt_row gt_left">2012</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_right">0.7</td>
      <td class="gt_row gt_right">6.0</td>
      <td class="gt_row gt_right">4.5</td>
      <td class="gt_row gt_right">0.8</td>
      <td class="gt_row gt_right">74.4</td>
      <td class="gt_row gt_right">11.6</td>
      <td class="gt_row gt_right">14.0</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">1989</td>
      <td class="gt_row gt_left">2014</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_right">1.3</td>
      <td class="gt_row gt_right">10.1</td>
      <td class="gt_row gt_right">6.2</td>
      <td class="gt_row gt_right">2.6</td>
      <td class="gt_row gt_right">61.5</td>
      <td class="gt_row gt_right">12.4</td>
      <td class="gt_row gt_right">26.1</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Reputation</td>
      <td class="gt_row gt_left">2017</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_right">0.4</td>
      <td class="gt_row gt_right">4.5</td>
      <td class="gt_row gt_right">2.3</td>
      <td class="gt_row gt_right">1.8</td>
      <td class="gt_row gt_right">51.1</td>
      <td class="gt_row gt_right">8.4</td>
      <td class="gt_row gt_right">40.5</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Lover</td>
      <td class="gt_row gt_left">2019</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_right">0.2</td>
      <td class="gt_row gt_right">3.2</td>
      <td class="gt_row gt_right">1.1</td>
      <td class="gt_row gt_right">1.9</td>
      <td class="gt_row gt_right">33.9</td>
      <td class="gt_row gt_right">6.9</td>
      <td class="gt_row gt_right">59.2</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Folklore</td>
      <td class="gt_row gt_left">2020</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_right">–</td>
      <td class="gt_row gt_right">–</td>
      <td class="gt_row gt_right">–</td>
      <td class="gt_row gt_right">–</td>
      <td class="gt_row gt_right">0.0</td>
      <td class="gt_row gt_right">–</td>
      <td class="gt_row gt_right">–</td>
    </tr>
    <tr class="gt_group_heading_row">
      <td colspan="11" class="gt_group_heading">Beyoncé</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Dangerously in Love</td>
      <td class="gt_row gt_left">2003</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_right">1.3</td>
      <td class="gt_row gt_right">11.0</td>
      <td class="gt_row gt_right">5.1</td>
      <td class="gt_row gt_right">4.6</td>
      <td class="gt_row gt_right">46.4</td>
      <td class="gt_row gt_right">11.5</td>
      <td class="gt_row gt_right">42.2</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">B'Day</td>
      <td class="gt_row gt_left">2006</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">3</td>
      <td class="gt_row gt_right">0.7</td>
      <td class="gt_row gt_right">8.0</td>
      <td class="gt_row gt_right">3.6</td>
      <td class="gt_row gt_right">3.7</td>
      <td class="gt_row gt_right">45.1</td>
      <td class="gt_row gt_right">8.8</td>
      <td class="gt_row gt_right">46.1</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">I Am... Sasha Fierce</td>
      <td class="gt_row gt_left">2008</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">2</td>
      <td class="gt_row gt_right">1.7</td>
      <td class="gt_row gt_right">8.0</td>
      <td class="gt_row gt_right">3.4</td>
      <td class="gt_row gt_right">2.9</td>
      <td class="gt_row gt_right">42.3</td>
      <td class="gt_row gt_right">21.8</td>
      <td class="gt_row gt_right">36.0</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">4</td>
      <td class="gt_row gt_left">2011</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_right">0.8</td>
      <td class="gt_row gt_right">–</td>
      <td class="gt_row gt_right">1.5</td>
      <td class="gt_row gt_right">–</td>
      <td class="gt_row gt_right">0.0</td>
      <td class="gt_row gt_right">–</td>
      <td class="gt_row gt_right">–</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Beyoncé</td>
      <td class="gt_row gt_left">2013</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">2</td>
      <td class="gt_row gt_right">0.4</td>
      <td class="gt_row gt_right">5.0</td>
      <td class="gt_row gt_right">2.5</td>
      <td class="gt_row gt_right">2.1</td>
      <td class="gt_row gt_right">50.2</td>
      <td class="gt_row gt_right">8.4</td>
      <td class="gt_row gt_right">41.4</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Lemonade</td>
      <td class="gt_row gt_left">2016</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_left">1</td>
      <td class="gt_row gt_right">0.3</td>
      <td class="gt_row gt_right">2.5</td>
      <td class="gt_row gt_right">1.6</td>
      <td class="gt_row gt_right">0.6</td>
      <td class="gt_row gt_right">62.2</td>
      <td class="gt_row gt_right">13.1</td>
      <td class="gt_row gt_right">24.7</td>
    </tr>
  </tbody>
  
  
</table></div><!--/html_preserve-->

# Finally, remember to disconnect

Usual PSA about disconnecting from the database at the end of each session.


```r
dbDisconnect(con)
```

