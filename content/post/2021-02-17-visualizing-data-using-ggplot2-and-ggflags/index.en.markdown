---
title: Visualizing Data using ggplot2 and ggflags
author: Lynley Aldridge
date: '2021-02-17'
tags:
  - Rstats
  - Tutorial
  - Visualization
slug: visualizing-data-using-ggplot2-and-ggflags
draft: yes
lastmod: '2021-02-16T20:44:06+11:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
---

In this post I explore ways of using ggplot2 and ggflags to plot album sales and chart data from the Taylor Swift and Beyoncé Tidy Tuesday dataset (as shown here).

<!--more-->

# References and setup

Useful posts introducing the [ggflags](https://github.com/rensa/ggflags) package drawn on in the examples posted here (alongside other options for using maps and flags in R) include:

* [All around the world": Maps and flags in R](https://quantixed.org/2019/03/20/all-around-the-world-maps-and-flags-in-r/) by quantixed

* [Examining speeches from the UN Security Council Part I](https://rforpoliticalscience.com/2021/02/17/examining-speeches-from-the-un-security-council-part-1/) and [Add circular flags to maps and graphs with ggflags package in R](https://rforpoliticalscience.com/2021/01/13/add-circular-flags-to-maps-and-graphs-with-ggflag-package-in-r/) from R for Political Science 

* An advanced example of using a modified version of ggflags and gganimate for [animating top 20 national football teams by goals scored](https://github.com/thomasp85/gganimate/issues/317) by basarabam 
First, let's load packages and read in the data:


```r
#load packages
library(tidyverse)
library(ggflags) 
library(countrycode) # to convert country names to country codes
library(tidytext) # for reorder_within
library(scales) # for application of common formats to scale labels (e.g., comma, percent, dollar)

#load data
sales <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2020/2020-09-29/sales.csv')
```

ggflags is the first R package I've used that is not available on CRAN. This meant I needed to install the devtools package first, and use `devtools::install_github('rensa/ggflags')` to install the ggflags package from github.  

# Fix country codes

First, let's see what country codes are included in the sales data:


```r
sales %>%
  distinct(country) 
```

```
## # A tibble: 10 x 1
##    country
##    <chr>  
##  1 US     
##  2 WW     
##  3 AUS    
##  4 UK     
##  5 JPN    
##  6 CAN    
##  7 FRA    
##  8 <NA>   
##  9 World  
## 10 FR
```

Reviewing the ggflags [documentation](https://github.com/rensa/ggflags) shows the [iso2c](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#AA) country codes associated with each flag. Let's use `mutate()` and `ifelse()` to change non-standard codes:


```r
sales <- sales %>%
  mutate(country = ifelse(country == "AUS", "AU", country)) %>%
  mutate(country = ifelse(country == "CAN", "CA", country)) %>%
  mutate(country = ifelse(country == "FRA", "FR", country)) %>%
  mutate(country = ifelse(country == "JPN", "JP", country)) %>%
  mutate(country = ifelse(country == "UK", "GB", country)) %>%
  mutate(country = ifelse(country == "World", "WW", country))
```

And check that we have recoded successfully:


```r
sales %>%
  distinct(country) 
```

```
## # A tibble: 8 x 1
##   country
##   <chr>  
## 1 US     
## 2 WW     
## 3 AU     
## 4 GB     
## 5 JP     
## 6 CA     
## 7 FR     
## 8 <NA>
```

Alternatively, if the data were in a standardized format, we could use the `countrycode()` function from [`countrycode`](https://cran.r-project.org/web/packages/countrycode/countrycode.pdf) to map country names or country codes from one format to another. The example shown below (not run), for example, would create a new variable (`code`) in the sales data frame which translates the `country.name` in the `country` column to `iso2c` format:  


```r
sales$code <- countrycode(sales$country, "country.name", "iso2c")
```

If you are given a database of country codes and aren't sure what format they are in, using `guess_field()` shows what percentage of values match which coding system.


```r
countrycode::guess_field(sales$country, min_similarity= 80)
```

```
##                                code percent_of_unique_matched
## ecb                             ecb                      87.5
## genc2c                       genc2c                      87.5
## iso2c                         iso2c                      87.5
## wb_api2c                   wb_api2c                      87.5
## cldr.name.bem         cldr.name.bem                      87.5
## cldr.name.cu           cldr.name.cu                      87.5
## cldr.name.dua         cldr.name.dua                      87.5
## cldr.name.kkj         cldr.name.kkj                      87.5
## cldr.name.kl           cldr.name.kl                      87.5
## cldr.name.mgo         cldr.name.mgo                      87.5
## cldr.name.nnh         cldr.name.nnh                      87.5
## cldr.name.pa_arab cldr.name.pa_arab                      87.5
## cldr.name.rw           cldr.name.rw                      87.5
## cldr.name.uz_arab cldr.name.uz_arab                      87.5
## cldr.name.xh           cldr.name.xh                      87.5
## cldr.short.gv         cldr.short.gv                      87.5
## cldr.variant.rw     cldr.variant.rw                      87.5
## cldr.variant.xh     cldr.variant.xh                      87.5
```

This tells us that after recoding, 87.5% of our rows are coded with a valid `iso2c` country code (note that both WW and NA are invalid codes we will need to deal with appropriately when graphing data). 

# Compare sales by artist

First, let's graph total sales by country for Beyoncé and Taylor Swift, and use the `geom_flag` function to plot circular flags on each bar in our graph representing a country. 

In order to do this, we filter to include only rows where the country field is not equal to `WW`. Note that by default this also excludes rows where country is coded as NA. If we don't exclude these rows and apply the `geomflag()` function to an object containing invalid codes, we receive an error message: "unable to find an inherited method for function 'grobify' for signature "NULL".

The `geom_flag` function needs iso2c country codes in lower case format, so the next part of this code converts country codes to lower case (as `code`).

The next line of code sets country as a factor and reorders countries manually by descending order of copies sold. We should also be able to use `mutate(country = fct_reorder(country, sales))` but this did not work for me. See the R Graph Gallery's [Reorder a variable with ggplot2](https://www.r-graph-gallery.com/267-reorder-a-variable-in-ggplot2.html) for help with reordering variables.

We then mutate the sales data by to calculate copies sold in millions, and take the millions and country data, to plot a column graph. 

We use the `geom_flag` function to plot the flags at a specified location of the x axis, using the `code` variable to assign flags, and the `size` argument to vary the size of the flags.

We then give our axes appropriate labels, set our x axis to use an appropriate scale, create separate graphs for each artist using `facet_wrap(~artist)`, and choose our desired theme for the graph.

Here is the full code used to generate the graph:


```r
sales %>%
  filter((country != "WW")) %>%
  mutate(code = tolower(country)) %>% 
  mutate(country = fct_relevel(country, "FR", "AU", "JP", "CA", "GB", "US")) %>%
  mutate(millions = sales/1000000) %>%
  ggplot(aes(millions, country)) +
  geom_col() +
  geom_flag(x = -0, aes(country = code), size = 7) +
  labs(y = "Country", x = "Copies sold (Millions)", 
       title = "Total album sales by country for Beyoncé and Taylor Swift", 
       caption = "Data source: Billboard via Wikipedia, October 2020") +  
  facet_wrap(~artist) +
  theme_minimal()
```

<img src="/post/2021-02-17-visualizing-data-using-ggplot2-and-ggflags/index.en_files/figure-html/comparison-1.png" width="672" />

This chart confirms what we already know about this data, that we have more country-specific sales data for Taylor Swift than Beyoncé (and the worldwide sales figures in our extract are greater than the sum of the country-specific data provided). We need to specify only relevant countries to compare if we're looking comparatively at sales by the two artists. First, let's try one more graph exploring the Taylor Swift data using ggflags. 

# Plotting sales data for a single album by country ... 

Here we graph sales data by country specifically for Taylor Swift's 2019 album, *Lover*:


```r
# chart with flags

sales %>%
  filter((title %in% c("Lover") & country != "WW")) %>%
  mutate(country = fct_reorder(country, sales)) %>%
  mutate(code = tolower(country)) %>% 
  ggplot(aes(sales, country)) +
  geom_col() +
  geom_flag(x = -10, aes(country = code), size = 10) +
  labs(y = "Country", x = "Copies sold", title = "Sales of Taylor Swift's 'Lover' (2019) by country", 
       caption = "Data source: Billboard via Wikipedia, October 2020") +
  scale_x_continuous(labels = comma) +
  theme_minimal()
```

<img src="/post/2021-02-17-visualizing-data-using-ggplot2-and-ggflags/index.en_files/figure-html/flags-one-album-1.png" width="672" />

# Stacked bar graph comparing sales data for the US and Great Britain

Because our data is incomplete, we need to be careful in the comparisons we make. It does seem reasonable to compare the number of copies of each album sold by Beyoncé and Taylor Swift in the US compared to the UK.  

Here we split the data into facets using the artist variable, and use `reorder_within()` and `scale_x_reordered()` to sort titles within each group in descending order of sales. I am grateful to Julia Silge's [Reordering and facetting for ggplot2](https://juliasilge.com/blog/reorder-within/) for a worked example of these functions. Note that Taylor Swift's self-titled album lacks UK sales data, and has been presented first by default, breaking this pattern. The use of the `scales = free_y` argument in the `facet_wrap()` function is necessary so that albums aren't repeated within each artist group. 

Here is the full code:


```r
# chart without flags

sales %>%
  filter(country %in% c("US", "GB")) %>%
  mutate(artist = as.factor(artist),
         title = reorder_within(title, sales, artist)) %>%
  mutate(millions = sales/1000000) %>%
  ggplot(aes(x = title, y = millions, fill = country)) +
  geom_col() +
  coord_flip() +
  scale_x_reordered() +
  facet_wrap(~artist, scales = "free_y") +
  labs(y = "Copies sold (Millions)", x = "Album", 
       title = "Stacked bar chart comparing US and GB sales by album", 
       caption = "Data source: Billboard via Wikipedia, October 2020") +
  theme_minimal()
```

<img src="/post/2021-02-17-visualizing-data-using-ggplot2-and-ggflags/index.en_files/figure-html/all-albums-1.png" width="672" />

# Grouped bar graph comparing sales data for the US and Great Britain

This data could similarly be charted in a grouped bar graph, which makes it easier to compare across individual bars, rather than totals. In this code, we need to use the `position = position_dodge2` argument in the `geom_col()` function, and `preserve = "single"` to ensure that groups with only one row (e.g., Taylor Swift's self-titled album) don't take up double the space in the graph:


```r
sales %>%
  filter(country %in% c("US", "GB")) %>%
  mutate(code = tolower(country)) %>% 
  mutate(artist = as.factor(artist),
         title = reorder_within(title, sales, artist)) %>%
  mutate(millions = sales/1000000) %>%
  ggplot(aes(millions, title, fill = country)) +
  geom_col(position = position_dodge2(preserve = "single")) +
  scale_y_reordered() +
  facet_wrap(~ artist, scales = "free_y") +
  labs(x = "Copies sold (Millions)", y = "Album",
       title = "Grouped bar chart comparing US and GB sales by album", 
       caption = "Data source: Billboard via Wikipedia, October 2020") +
  theme_minimal()
```

<img src="/post/2021-02-17-visualizing-data-using-ggplot2-and-ggflags/index.en_files/figure-html/unnamed-chunk-6-1.png" width="672" />


## Plotting sales data for multiple albums ... 


```r
# chart with flags

options(scipen=10000)

sales %>%
  filter(country != "WW") %>%
  mutate(millions = sales/1000000) %>%
  mutate(code = tolower(country)) %>% 
  ggplot(aes(millions, country)) +
  geom_col() +
  geom_flag(x = -10, aes(country = code), size = 2) +
  facet_wrap(artist ~title, scales = "free_y") +
  labs(y = "", x = "Sales in millions")
```

<img src="/post/2021-02-17-visualizing-data-using-ggplot2-and-ggflags/index.en_files/figure-html/flags-all-albums-1.png" width="672" />

## Comparative sales data - US and UK (missing UK flags)

The following graph shows sales data for the US and UK, but only assigns flags to US rows.

(Current version won't assign codes to either row and has Taylor Swift row double width)


```r
# chart with flags

sales %>%
  filter(country %in% c("US", "GB")) %>%
  mutate(code = tolower(country)) %>% 
  ggplot(aes(title, sales, fill = country)) +
  geom_col(stat = identity, position = position_dodge2()) +
  coord_flip() +
  geom_flag(x = -5, aes(country = code), size = 4) +
  labs(y = "Sales", x = "Album", fill = "Country") +
  # geom_text(aes(label=round(Proportion, digits = 0)), hjust = 1.6, color="white", size=3.5) +
  theme_minimal()
```

```
## Warning: Ignoring unknown parameters: stat
```

<img src="/post/2021-02-17-visualizing-data-using-ggplot2-and-ggflags/index.en_files/figure-html/flags-1.png" width="672" />

# Comparative sales by artist



```r
sales %>%
  filter(country %in% c("US", "GB")) %>%
  mutate(code = tolower(country)) %>% 
  ggplot(aes(sales, title, fill = country)) +
  geom_col(position = position_dodge2(preserve = "single")) +
  scale_x_continuous() +
  labs(x = "Sales in dollars", y = "Album") +
  theme_minimal()
```

<img src="/post/2021-02-17-visualizing-data-using-ggplot2-and-ggflags/index.en_files/figure-html/unnamed-chunk-7-1.png" width="672" />



# Comparative sales by artist - facet wrap

Might be better excluding Taylor Swift album 







```r
sales %>%
  filter(artist == "Taylor Swift") %>%
  drop_na() %>%
  ggplot(aes(title, percent, group = 1)) +
  geom_line() +
  theme_minimal()
```



```r
# doesn't work for some reason

df %>%
  filter(artist == "Beyonce") %>%
  drop_na() %>%
  ggplot(aes(title, percent, group = 1)) +
  geom_line() +
  theme_minimal()
```


# Graphing worldwide sales data using ggplot2

Here's how we can plot worldwide sales data from this data frame by descending order of sales in a column graph, coloring columns by artist.


```r
sales %>%
  filter(country %in% c("WW", "World")) %>%
  mutate(title = fct_reorder(title, sales)) %>%
  ggplot(aes(sales, title, fill = artist)) +
  geom_col() +
  scale_x_continuous(labels = scales::dollar) +
  labs(x = "Sales (Worldwide)", y = "Album") +
  theme_minimal()
```

<img src="/post/2021-02-17-visualizing-data-using-ggplot2-and-ggflags/index.en_files/figure-html/wwsales-1.png" width="672" />

# Graphing US sales data using ggplot2


```r
sales %>%
  filter(country %in% c("US")) %>%
  mutate(title = fct_reorder(title, sales)) %>%
  ggplot(aes(sales, title, fill = artist)) +
  geom_col() +
  scale_x_continuous(labels = scales::dollar)
```

<img src="/post/2021-02-17-visualizing-data-using-ggplot2-and-ggflags/index.en_files/figure-html/graph-1.png" width="672" />
# Total sales by country for each artist


```r
options(scipen=10000)

sales %>%
  filter(country != "WW") %>%
  mutate(code = tolower(country)) %>% 
  ggplot(aes(x = artist, y = sales, fill = country)) +
  geom_col() +
  coord_flip() +
  geom_flag(x = -10, aes(country = code), size = 10) +
  labs(x = "Album", y = "Sales") +
  # geom_text(aes(label=round(Proportion, digits = 0)), hjust = 1.6, color="white", size=3.5) +
  theme_minimal()
```

<img src="/post/2021-02-17-visualizing-data-using-ggplot2-and-ggflags/index.en_files/figure-html/flags-two-artists-1.png" width="672" />

# Plotting sales data for two albums - stacked bar chart that needs to be shown side by side instead ... 

Can we do this for more than one album?  A work in progress ...


```r
# chart with flags

options(scipen=10000)

sales$country <- factor(sales$country,
          levels = unique(sales$country[order(sales$sales)]))

sales %>%
  filter((title %in% c("Lover", "Fearless") & country != "WW")) %>%
  mutate(code = tolower(country)) %>% 
  ggplot(aes(x = title, y = sales, fill = country)) +
  geom_col() +
  coord_flip() +
  geom_flag(x = -10, aes(country = code), size = 10) +
  labs(x = "Album", y = "Sales") +
  # geom_text(aes(label=round(Proportion, digits = 0)), hjust = 1.6, color="white", size=3.5) +
  theme_minimal()
```

<img src="/post/2021-02-17-visualizing-data-using-ggplot2-and-ggflags/index.en_files/figure-html/flags-two-albums-1.png" width="672" />


```r
sales %>%
  filter((artist == "Taylor Swift" & title %in% c("Fearless", "Red", "1989") & country != "WW")) %>%
  mutate(code = tolower(country)) %>% 
  mutate(title = fct_relevel(title, "Fearless", "Red", "1989")) %>%
  mutate(country = fct_relevel(country, "FR", "AU", "JP", "CA", "GB", "US")) %>%
  mutate(millions = sales/1000000) %>%
  ggplot(aes(millions, country)) +
  geom_col() +
  geom_flag(x = 0, aes(country = code), size = 4) +
  labs(y = "Country", x = "Sales (Millions)") +
  facet_wrap(~title) +
  scale_x_continuous(labels = scales::dollar) +
  theme_minimal()
```

<img src="/post/2021-02-17-visualizing-data-using-ggplot2-and-ggflags/index.en_files/figure-html/flags-more-albums-1.png" width="672" />


# Summarize at 


```r
sales %>%
  group_by(country) %>%
  summarise_at(vars(sales), sum)
```

```
## # A tibble: 8 x 2
##   country    sales
##   <fct>      <dbl>
## 1 FR        118700
## 2 JP        556801
## 3 CA        683000
## 4 GB       8557935
## 5 AU        500000
## 6 US      49315000
## 7 WW      75300000
## 8 <NA>          NA
```

# Mutate and case when


```r
sales <- sales %>%
  mutate(country = case_when(country == "AUS" ~ "AU",
                             country == "CAN" ~ "CA",
                             country == "FR" ~ "FR",
                             country == "FRA" ~ "FR",
                             country == "JPN" ~ "JP",
                             country == "UK" ~ "GB",
                             country == "US" ~ "US",
                             country == "World" ~ "WW",
                             country == "WW" ~ "WW"))
```

# Summarize at and sort


```r
sales %>%
  filter((country != "WW")) %>%
  group_by(artist, country) %>%
  summarise_at(vars(sales), sum) %>%
  arrange(artist, sales)
```

```
## # A tibble: 4 x 3
## # Groups:   artist [2]
##   artist       country    sales
##   <chr>        <chr>      <dbl>
## 1 Beyoncé      FR         30000
## 2 Beyoncé      US      17656000
## 3 Taylor Swift FR         88700
## 4 Taylor Swift US      31659000
```

# Worldwide sales totals


```r
sales %>%
  filter((country == "WW")) %>%
  group_by(artist) %>%
  summarise_at(vars(sales), sum)
```

```
## # A tibble: 2 x 2
##   artist          sales
##   <chr>           <dbl>
## 1 Beyoncé      34500000
## 2 Taylor Swift 40800000
```

May need to set scientific notation off:

options(scipen=10000)
