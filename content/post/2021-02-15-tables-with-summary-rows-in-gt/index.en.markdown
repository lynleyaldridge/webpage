---
title: Tables with summary rows in gt
author: Lynley Aldridge
date: '2021-02-16'
tags:
  - gt
  - Rstats
  - TidyTuesday
  - Tutorial
slug: tables-with-summary-rows-in-gt
lastmod: '2021-02-15T20:39:45+11:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
---

This post focuses on ways to customize summary rows in `gt` tables, to create the summary table shown here (based on the Tidy Tuesday Taylor Swift and Beyoncé data).

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

# Adding summary rows with more complex calculations

One thing I've learned in my journey with R so far is that 'finishing that off' with 'just one more thing' can involve a considerable amount of work when you're teaching yourself as you go. I was keen, however, to create a table summarizing total US and worldwide sales for each artist, and calculating what percentage of each artist's total sales was represented by US sales. I drew on general guidance for [creating summary lines](https://gt.rstudio.com/articles/creating-summary-lines.html) in creating [my first gt table](/2021/02/12/experimenting-with-sql-in-r-markdown-more-join-queries/) on this blog. 

Thanks to [josep maria porrà](https://stackoverflow.com/questions/63041655/how-can-you-add-group-percentages-to-tables-using-the-gt-package) for providing an example on stack overflow I was able to modify further to create the below code. 

First, we need to create a new data frame calculating US sales as a percentage of total sales for each artist:


```r
albums_grouped <- albums %>%
  drop_na() %>%
  group_by(artist) %>%
  summarise_at(vars(US_sales, WW_sales), sum) %>%
  rowwise() %>%
  mutate(pct_US = US_sales/WW_sales)

albums_grouped
```

```
## # A tibble: 2 x 4
## # Rowwise: 
##   artist       US_sales WW_sales pct_US
##   <chr>           <dbl>    <dbl>  <dbl>
## 1 Beyoncé          16.2     34.5  0.470
## 2 Taylor Swift     26       40.8  0.637
```

These will be the values that we pull into the summary rows. Now, let's make a table based on selected columns from the original `albums` data frame, dropping albums with missing sales data. Using the `gt()` function, we can specify desired columns to use as row names and grouping variables (`rowname_col = "title, groupname_col = "artist"`).

In order to create the summary rows, first summarize the `US_sales` and `WW_sales` columns, using the `sum` function and reporting output as `TOTAL`. Then select the `pct_US` for the `Taylor Swift` row in `albums_grouped`, and retrieve as `TOTAL` for the Taylor Swift group. Repeat for Beyoncé. The `formatter =` argument allows formatting numbers as percentages, to specified number of decimal points. Finally, we can use `tab_style()` to style our group names and summary rows: 


```r
gt1 <- albums %>%
  
  select(title, artist, year, US_chart, UK_chart, US_sales, WW_sales, US_percent) %>%
  drop_na() %>%
  gt(rowname_col = "title", groupname_col = "artist") %>%
  
    # create summary rows for each group 
    summary_rows(groups = TRUE, 
                 columns = vars(US_sales, WW_sales),
                 fns = list(TOTAL = "sum"),
                 formatter = fmt_number, decimals = 1) %>%
    summary_rows(groups = "Taylor Swift",
                 columns = vars(US_percent), 
                 fns = list(TOTAL = ~ 
                            albums_grouped %>%
                            filter(artist == "Taylor Swift") %>%
                            select(pct_US) %>%
                            pull()),
                 formatter = fmt_percent, decimals = 1) %>%
    summary_rows(groups = "Beyoncé",
                 columns = vars(US_percent), 
                 fns = list(TOTAL = ~ 
                            albums_grouped %>%
                            filter(artist == "Beyoncé") %>%
                            select(pct_US) %>%
                            pull()),
                 formatter = fmt_percent, decimals = 1) %>%

    # style summary rows and row group titles 
    tab_style(style = cell_text(weight = "bold",
                                color = "#795548"),
        locations = list(cells_summary(groups = TRUE),
                          cells_row_groups(groups = TRUE)))

gt1
```

<!--html_preserve--><style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#zokvrwlizl .gt_table {
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

#zokvrwlizl .gt_heading {
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

#zokvrwlizl .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#zokvrwlizl .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 0;
  padding-bottom: 4px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#zokvrwlizl .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#zokvrwlizl .gt_col_headings {
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

#zokvrwlizl .gt_col_heading {
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

#zokvrwlizl .gt_column_spanner_outer {
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

#zokvrwlizl .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#zokvrwlizl .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#zokvrwlizl .gt_column_spanner {
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

#zokvrwlizl .gt_group_heading {
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

#zokvrwlizl .gt_empty_group_heading {
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

#zokvrwlizl .gt_from_md > :first-child {
  margin-top: 0;
}

#zokvrwlizl .gt_from_md > :last-child {
  margin-bottom: 0;
}

#zokvrwlizl .gt_row {
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

#zokvrwlizl .gt_stub {
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

#zokvrwlizl .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#zokvrwlizl .gt_first_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
}

#zokvrwlizl .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#zokvrwlizl .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#zokvrwlizl .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#zokvrwlizl .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#zokvrwlizl .gt_footnotes {
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

#zokvrwlizl .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding: 4px;
}

#zokvrwlizl .gt_sourcenotes {
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

#zokvrwlizl .gt_sourcenote {
  font-size: 90%;
  padding: 4px;
}

#zokvrwlizl .gt_left {
  text-align: left;
}

#zokvrwlizl .gt_center {
  text-align: center;
}

#zokvrwlizl .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#zokvrwlizl .gt_font_normal {
  font-weight: normal;
}

#zokvrwlizl .gt_font_bold {
  font-weight: bold;
}

#zokvrwlizl .gt_font_italic {
  font-style: italic;
}

#zokvrwlizl .gt_super {
  font-size: 65%;
}

#zokvrwlizl .gt_footnote_marks {
  font-style: italic;
  font-size: 65%;
}
</style>
<div id="zokvrwlizl" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;"><table class="gt_table">
  
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1"></th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">year</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">US_chart</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">UK_chart</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1">US_sales</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1">WW_sales</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1">US_percent</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr class="gt_group_heading_row">
      <td colspan="7" class="gt_group_heading" style="color: #795548; font-weight: bold;">Taylor Swift</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Fearless</td>
      <td class="gt_row gt_center">2008</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">5</td>
      <td class="gt_row gt_right">7.2</td>
      <td class="gt_row gt_right">12.0</td>
      <td class="gt_row gt_right">59.8</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Speak Now</td>
      <td class="gt_row gt_center">2010</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">6</td>
      <td class="gt_row gt_right">4.7</td>
      <td class="gt_row gt_right">5.0</td>
      <td class="gt_row gt_right">93.9</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Red</td>
      <td class="gt_row gt_center">2012</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_right">4.5</td>
      <td class="gt_row gt_right">6.0</td>
      <td class="gt_row gt_right">74.4</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">1989</td>
      <td class="gt_row gt_center">2014</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_right">6.2</td>
      <td class="gt_row gt_right">10.1</td>
      <td class="gt_row gt_right">61.5</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Reputation</td>
      <td class="gt_row gt_center">2017</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_right">2.3</td>
      <td class="gt_row gt_right">4.5</td>
      <td class="gt_row gt_right">51.1</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Lover</td>
      <td class="gt_row gt_center">2019</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_right">1.1</td>
      <td class="gt_row gt_right">3.2</td>
      <td class="gt_row gt_right">33.9</td>
    </tr>
    <tr>
      <td class="gt_row gt_stub gt_right gt_summary_row gt_first_summary_row">TOTAL</td>
      <td class="gt_row gt_center gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">&mdash;</td>
      <td class="gt_row gt_center gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">&mdash;</td>
      <td class="gt_row gt_center gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">&mdash;</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">26.0</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">40.8</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">63.7&percnt;</td>
    </tr>
    <tr class="gt_group_heading_row">
      <td colspan="7" class="gt_group_heading" style="color: #795548; font-weight: bold;">Beyoncé</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Dangerously in Love</td>
      <td class="gt_row gt_center">2003</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_right">5.1</td>
      <td class="gt_row gt_right">11.0</td>
      <td class="gt_row gt_right">46.4</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">B'Day</td>
      <td class="gt_row gt_center">2006</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">3</td>
      <td class="gt_row gt_right">3.6</td>
      <td class="gt_row gt_right">8.0</td>
      <td class="gt_row gt_right">45.1</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">I Am... Sasha Fierce</td>
      <td class="gt_row gt_center">2008</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">2</td>
      <td class="gt_row gt_right">3.4</td>
      <td class="gt_row gt_right">8.0</td>
      <td class="gt_row gt_right">42.3</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Beyoncé</td>
      <td class="gt_row gt_center">2013</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">2</td>
      <td class="gt_row gt_right">2.5</td>
      <td class="gt_row gt_right">5.0</td>
      <td class="gt_row gt_right">50.2</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Lemonade</td>
      <td class="gt_row gt_center">2016</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_center">1</td>
      <td class="gt_row gt_right">1.6</td>
      <td class="gt_row gt_right">2.5</td>
      <td class="gt_row gt_right">62.2</td>
    </tr>
    <tr>
      <td class="gt_row gt_stub gt_right gt_summary_row gt_first_summary_row">TOTAL</td>
      <td class="gt_row gt_center gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">&mdash;</td>
      <td class="gt_row gt_center gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">&mdash;</td>
      <td class="gt_row gt_center gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">&mdash;</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">16.2</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">34.5</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">47.0&percnt;</td>
    </tr>
  </tbody>
  
  
</table></div><!--/html_preserve-->

To finalize this table, we follow the steps outlined in a [previous post on gt tables](/2021/02/14/summary-tables-using-gt/) to assign column labels and column spanner headings, set column alignment, cell color and formatting, and give the table a title, subtitle and source note:


```r
gt2 <- gt1 %>%
  
    cols_label(
      year = "Released",
      US_chart = "US",
      UK_chart = "UK",
      US_sales = "US",
      WW_sales = "Total",
      US_percent = "US sales (%)") %>%

    tab_spanner(label = "Chart position", columns = vars(US_chart, 
                                                         UK_chart)) %>%
    tab_spanner(label = "Sales (millions)", columns = vars(US_sales,
                                                            WW_sales)) %>%

    cols_align(align = "right", columns = c("US_chart", "UK_chart",
                                            "US_sales", "WW_sales",
                                            "US_percent")) %>%
  
    tab_style(style = cell_text(color = "#795548"),
          locations = list(
            cells_column_labels(everything()),
            cells_stub(rows = TRUE),
            cells_body()))%>%
  
    tab_style(style = cell_text(
      color = "#795548",
      weight = "bold"),
      locations = cells_column_spanners(spanners = vars("Chart position", 
                                                      "Sales (millions)")))%>%
  
    tab_header(
      title = md(
        "**Taylor Swift has sold more albums than Beyoncé, but owes a greater proportion of her success to US sales than Beyoncé**"),
      subtitle = md(
        "*Peak chart position, copies sold, and US sales as a percentage of total sales by album*")) %>%

    tab_source_note(
      source_note = md("<span style = 'color:#795548'>Source: Billboard via Wikipedia, October 2020<br>Excludes 3 albums for which worldwide sales data was not available - Taylor Swift, Folklore, and 4</span>")) %>%
  
    tab_style(style = cell_text(color = "#795548", size = "large"),
          locations = cells_title(groups = "title")) %>%
  
    tab_style(style = cell_text(color = "#795548", size = "medium"),
          locations = cells_title(groups = "subtitle")) 

gt2
```

<!--html_preserve--><style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#ignikggiga .gt_table {
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

#ignikggiga .gt_heading {
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

#ignikggiga .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#ignikggiga .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 0;
  padding-bottom: 4px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#ignikggiga .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#ignikggiga .gt_col_headings {
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

#ignikggiga .gt_col_heading {
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

#ignikggiga .gt_column_spanner_outer {
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

#ignikggiga .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#ignikggiga .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#ignikggiga .gt_column_spanner {
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

#ignikggiga .gt_group_heading {
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

#ignikggiga .gt_empty_group_heading {
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

#ignikggiga .gt_from_md > :first-child {
  margin-top: 0;
}

#ignikggiga .gt_from_md > :last-child {
  margin-bottom: 0;
}

#ignikggiga .gt_row {
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

#ignikggiga .gt_stub {
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

#ignikggiga .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#ignikggiga .gt_first_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
}

#ignikggiga .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#ignikggiga .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#ignikggiga .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#ignikggiga .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#ignikggiga .gt_footnotes {
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

#ignikggiga .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding: 4px;
}

#ignikggiga .gt_sourcenotes {
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

#ignikggiga .gt_sourcenote {
  font-size: 90%;
  padding: 4px;
}

#ignikggiga .gt_left {
  text-align: left;
}

#ignikggiga .gt_center {
  text-align: center;
}

#ignikggiga .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#ignikggiga .gt_font_normal {
  font-weight: normal;
}

#ignikggiga .gt_font_bold {
  font-weight: bold;
}

#ignikggiga .gt_font_italic {
  font-style: italic;
}

#ignikggiga .gt_super {
  font-size: 65%;
}

#ignikggiga .gt_footnote_marks {
  font-style: italic;
  font-size: 65%;
}
</style>
<div id="ignikggiga" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;"><table class="gt_table">
  <thead class="gt_header">
    <tr>
      <th colspan="7" class="gt_heading gt_title gt_font_normal" style="color: #795548; font-size: large;"><strong>Taylor Swift has sold more albums than Beyoncé, but owes a greater proportion of her success to US sales than Beyoncé</strong></th>
    </tr>
    <tr>
      <th colspan="7" class="gt_heading gt_subtitle gt_font_normal gt_bottom_border" style="color: #795548; font-size: medium;"><em>Peak chart position, copies sold, and US sales as a percentage of total sales by album</em></th>
    </tr>
  </thead>
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="2" colspan="1"></th>
      <th class="gt_col_heading gt_center gt_columns_bottom_border" rowspan="2" colspan="1" style="color: #795548;">Released</th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="2" style="color: #795548; font-weight: bold;">
        <span class="gt_column_spanner">Chart position</span>
      </th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="2" style="color: #795548; font-weight: bold;">
        <span class="gt_column_spanner">Sales (millions)</span>
      </th>
      <th class="gt_col_heading gt_center gt_columns_bottom_border" rowspan="2" colspan="1" style="color: #795548;">US sales (%)</th>
    </tr>
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="color: #795548;">US</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="color: #795548;">UK</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="color: #795548;">US</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="color: #795548;">Total</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr class="gt_group_heading_row">
      <td colspan="7" class="gt_group_heading" style="color: #795548; font-weight: bold;">Taylor Swift</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub" style="color: #795548;">Fearless</td>
      <td class="gt_row gt_center" style="color: #795548;">2008</td>
      <td class="gt_row gt_right" style="color: #795548;">1</td>
      <td class="gt_row gt_right" style="color: #795548;">5</td>
      <td class="gt_row gt_right" style="color: #795548;">7.2</td>
      <td class="gt_row gt_right" style="color: #795548;">12.0</td>
      <td class="gt_row gt_right" style="color: #795548;">59.8</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub" style="color: #795548;">Speak Now</td>
      <td class="gt_row gt_center" style="color: #795548;">2010</td>
      <td class="gt_row gt_right" style="color: #795548;">1</td>
      <td class="gt_row gt_right" style="color: #795548;">6</td>
      <td class="gt_row gt_right" style="color: #795548;">4.7</td>
      <td class="gt_row gt_right" style="color: #795548;">5.0</td>
      <td class="gt_row gt_right" style="color: #795548;">93.9</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub" style="color: #795548;">Red</td>
      <td class="gt_row gt_center" style="color: #795548;">2012</td>
      <td class="gt_row gt_right" style="color: #795548;">1</td>
      <td class="gt_row gt_right" style="color: #795548;">1</td>
      <td class="gt_row gt_right" style="color: #795548;">4.5</td>
      <td class="gt_row gt_right" style="color: #795548;">6.0</td>
      <td class="gt_row gt_right" style="color: #795548;">74.4</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub" style="color: #795548;">1989</td>
      <td class="gt_row gt_center" style="color: #795548;">2014</td>
      <td class="gt_row gt_right" style="color: #795548;">1</td>
      <td class="gt_row gt_right" style="color: #795548;">1</td>
      <td class="gt_row gt_right" style="color: #795548;">6.2</td>
      <td class="gt_row gt_right" style="color: #795548;">10.1</td>
      <td class="gt_row gt_right" style="color: #795548;">61.5</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub" style="color: #795548;">Reputation</td>
      <td class="gt_row gt_center" style="color: #795548;">2017</td>
      <td class="gt_row gt_right" style="color: #795548;">1</td>
      <td class="gt_row gt_right" style="color: #795548;">1</td>
      <td class="gt_row gt_right" style="color: #795548;">2.3</td>
      <td class="gt_row gt_right" style="color: #795548;">4.5</td>
      <td class="gt_row gt_right" style="color: #795548;">51.1</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub" style="color: #795548;">Lover</td>
      <td class="gt_row gt_center" style="color: #795548;">2019</td>
      <td class="gt_row gt_right" style="color: #795548;">1</td>
      <td class="gt_row gt_right" style="color: #795548;">1</td>
      <td class="gt_row gt_right" style="color: #795548;">1.1</td>
      <td class="gt_row gt_right" style="color: #795548;">3.2</td>
      <td class="gt_row gt_right" style="color: #795548;">33.9</td>
    </tr>
    <tr>
      <td class="gt_row gt_stub gt_right gt_summary_row gt_first_summary_row">TOTAL</td>
      <td class="gt_row gt_center gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">&mdash;</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">&mdash;</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">&mdash;</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">26.0</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">40.8</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">63.7&percnt;</td>
    </tr>
    <tr class="gt_group_heading_row">
      <td colspan="7" class="gt_group_heading" style="color: #795548; font-weight: bold;">Beyoncé</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub" style="color: #795548;">Dangerously in Love</td>
      <td class="gt_row gt_center" style="color: #795548;">2003</td>
      <td class="gt_row gt_right" style="color: #795548;">1</td>
      <td class="gt_row gt_right" style="color: #795548;">1</td>
      <td class="gt_row gt_right" style="color: #795548;">5.1</td>
      <td class="gt_row gt_right" style="color: #795548;">11.0</td>
      <td class="gt_row gt_right" style="color: #795548;">46.4</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub" style="color: #795548;">B'Day</td>
      <td class="gt_row gt_center" style="color: #795548;">2006</td>
      <td class="gt_row gt_right" style="color: #795548;">1</td>
      <td class="gt_row gt_right" style="color: #795548;">3</td>
      <td class="gt_row gt_right" style="color: #795548;">3.6</td>
      <td class="gt_row gt_right" style="color: #795548;">8.0</td>
      <td class="gt_row gt_right" style="color: #795548;">45.1</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub" style="color: #795548;">I Am... Sasha Fierce</td>
      <td class="gt_row gt_center" style="color: #795548;">2008</td>
      <td class="gt_row gt_right" style="color: #795548;">1</td>
      <td class="gt_row gt_right" style="color: #795548;">2</td>
      <td class="gt_row gt_right" style="color: #795548;">3.4</td>
      <td class="gt_row gt_right" style="color: #795548;">8.0</td>
      <td class="gt_row gt_right" style="color: #795548;">42.3</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub" style="color: #795548;">Beyoncé</td>
      <td class="gt_row gt_center" style="color: #795548;">2013</td>
      <td class="gt_row gt_right" style="color: #795548;">1</td>
      <td class="gt_row gt_right" style="color: #795548;">2</td>
      <td class="gt_row gt_right" style="color: #795548;">2.5</td>
      <td class="gt_row gt_right" style="color: #795548;">5.0</td>
      <td class="gt_row gt_right" style="color: #795548;">50.2</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub" style="color: #795548;">Lemonade</td>
      <td class="gt_row gt_center" style="color: #795548;">2016</td>
      <td class="gt_row gt_right" style="color: #795548;">1</td>
      <td class="gt_row gt_right" style="color: #795548;">1</td>
      <td class="gt_row gt_right" style="color: #795548;">1.6</td>
      <td class="gt_row gt_right" style="color: #795548;">2.5</td>
      <td class="gt_row gt_right" style="color: #795548;">62.2</td>
    </tr>
    <tr>
      <td class="gt_row gt_stub gt_right gt_summary_row gt_first_summary_row">TOTAL</td>
      <td class="gt_row gt_center gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">&mdash;</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">&mdash;</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">&mdash;</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">16.2</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">34.5</td>
      <td class="gt_row gt_right gt_summary_row gt_first_summary_row" style="color: #795548; font-weight: bold;">47.0&percnt;</td>
    </tr>
  </tbody>
  <tfoot class="gt_sourcenotes">
    <tr>
      <td class="gt_sourcenote" colspan="7"><span style = 'color:#795548'>Source: Billboard via Wikipedia, October 2020<br>Excludes 3 albums for which worldwide sales data was not available - Taylor Swift, Folklore, and 4</span></td>
    </tr>
  </tfoot>
  
</table></div><!--/html_preserve-->

To save this table as a .png image, use the following code (amending path as desired):


```r
library(here)
gtsave(gt2, here::here("compare-sales.png"))
```

See [my tidytuesday repository](https://github.com/lynleyaldridge/tidytuesday/tree/main/2020/2020-week40) for the full code used to generate the image at the top of this post, which includes additional code for footnotes and borders.
