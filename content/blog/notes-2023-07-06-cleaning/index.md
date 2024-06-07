---
title: "Project notes: Cleaning data"
output: blogdown::html_page
description: "Lecture notes for cleaning project data."
excerpt: "Lecture notes for cleaning project data."
date: 2023-07-06
lastmod: "2024-06-07"
draft: false
images: []
categories: ["Class notes"]
tags: ["2023"]
contributors: ["Monica Thieu"]
pinned: false
homepage: false
---

Again, now that we're fully in the project work stage of the course, the lecture notes after class will focus mainly on demonstrating and explaining specific techniques that people need for their particular project data.


```r
library(tidyverse)
```

```
## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
## ✔ dplyr     1.1.4     ✔ readr     2.1.5
## ✔ forcats   1.0.0     ✔ stringr   1.5.1
## ✔ ggplot2   3.5.1     ✔ tibble    3.2.1
## ✔ lubridate 1.9.3     ✔ tidyr     1.3.1
## ✔ purrr     1.0.2     
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors
```

First, I'll read in some student data that will pop up in later examples.


```r
csv_data <- read_csv(here::here("ignore",
                                "data",
                                "AI_index_db.csv"))
```

```
## Rows: 62 Columns: 13
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## chr (5): Country, Region, Cluster, Income group, Political regime
## dbl (8): Talent, Infrastructure, Operating Environment, Research, Developmen...
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

## Renaming columns

Often times, when you read in other people's data, they may have nice-looking column names that _are not legal variable names,_ usually because the column names have spaces in them.


``` r
csv_data
```

```
## # A tibble: 62 × 13
##    Country     Talent Infrastructure Operating Environmen…¹ Research Development
##    <chr>        <dbl>          <dbl>                  <dbl>    <dbl>       <dbl>
##  1 United Sta…  100             94.0                   64.6    100         100  
##  2 China         16.5          100                     91.6     71.4        80.0
##  3 United Kin…   39.6           71.4                   74.6     36.5        25.0
##  4 Canada        31.3           77.0                   93.9     30.7        25.8
##  5 Israel        35.8           67.6                   82.4     32.6        28.0
##  6 Singapore     39.4           84.3                   43.2     37.7        22.6
##  7 South Korea   14.5           85.2                   68.9     26.7        77.2
##  8 The Nether…   33.8           82.0                   88.0     25.5        30.2
##  9 Germany       27.6           77.2                   70.2     35.8        24.8
## 10 France        28.3           77.2                   80.0     25.5        21.4
## # ℹ 52 more rows
## # ℹ abbreviated name: ¹​`Operating Environment`
## # ℹ 7 more variables: `Government Strategy` <dbl>, Commercial <dbl>,
## #   `Total score` <dbl>, Region <chr>, Cluster <chr>, `Income group` <chr>,
## #   `Political regime` <chr>
```

See how some of the column names have spaces in them?

You don't _HAVE_ to rename these columns, because you can still call them using the backtick character ` as special quotation marks around the column name. Quoting out a column name with backticks in R code tells R that that column name does indeed represent ONE column name/variable, even if normally a space would mean you're ending one command name and starting another.

This can be annoying, though, so if you want to rename column names to remove spaces, here's a smooth way to do it:


``` r
csv_data <- csv_data |> 
  rename_with(.fn = \(x) str_replace_all(x, " ", "_"),
              .cols = everything())
```

You may recognize some of these code bits from other techniques like using `across()` inside `mutate()` to manipulate multiple columns at once. `rename_with()` is like `across()` but for renaming many columns at once, using a string function on the column names.

The `.fn` argument takes the function you want to pass over each column name. In this example, I'm using `str_replace_all()` to replace every space character in a column name with an underscore, to make it a legal variable name. You could even remove spaces by using `str_remove_all()` instead, but sometimes this makes column names harder to read. We always feed this function into the `.fn` argument starting with `\(x)`, which is necessary to tell R "pass this string function across every column name, one at a time".

Then, the `.cols` argument tells R which column names we want to rename. In this case, our function should only affect the names of columns that have spaces in them (otherwise, nothing gets replaced), so we can call the function on every single column using `everything()`. That way, the columns that need to get fixed get fixed, and the columns that don't need to get fixed stay the same.

Now, all column names are legal variable names, and we don't have to use the backtick quotes anymore.


``` r
csv_data
```

```
## # A tibble: 62 × 13
##    Country      Talent Infrastructure Operating_Environment Research Development
##    <chr>         <dbl>          <dbl>                 <dbl>    <dbl>       <dbl>
##  1 United Stat…  100             94.0                  64.6    100         100  
##  2 China          16.5          100                    91.6     71.4        80.0
##  3 United King…   39.6           71.4                  74.6     36.5        25.0
##  4 Canada         31.3           77.0                  93.9     30.7        25.8
##  5 Israel         35.8           67.6                  82.4     32.6        28.0
##  6 Singapore      39.4           84.3                  43.2     37.7        22.6
##  7 South Korea    14.5           85.2                  68.9     26.7        77.2
##  8 The Netherl…   33.8           82.0                  88.0     25.5        30.2
##  9 Germany        27.6           77.2                  70.2     35.8        24.8
## 10 France         28.3           77.2                  80.0     25.5        21.4
## # ℹ 52 more rows
## # ℹ 7 more variables: Government_Strategy <dbl>, Commercial <dbl>,
## #   Total_score <dbl>, Region <chr>, Cluster <chr>, Income_group <chr>,
## #   Political_regime <chr>
```


## Dropping columns with `select()`

Refer to the [previous class notes on select()](../class-notes-june-15-2023/#select-choose-a-subset-of-columns) to see a variety of ways you can use `select()` to choose just some columns of your data.

Another way you can use `select()` that I didn't show in the earlier class is to _drop just some unwanted columns, keeping all the rest._

You can use the minus `-` in front of the column name to designate columns to _drop._ All columns not mentioned will remain in the data.

For example:


``` r
csv_data <- csv_data |> 
  select(-Cluster, -Political_regime)
```

Those columns should now be gone:


``` r
csv_data
```

```
## # A tibble: 62 × 11
##    Country      Talent Infrastructure Operating_Environment Research Development
##    <chr>         <dbl>          <dbl>                 <dbl>    <dbl>       <dbl>
##  1 United Stat…  100             94.0                  64.6    100         100  
##  2 China          16.5          100                    91.6     71.4        80.0
##  3 United King…   39.6           71.4                  74.6     36.5        25.0
##  4 Canada         31.3           77.0                  93.9     30.7        25.8
##  5 Israel         35.8           67.6                  82.4     32.6        28.0
##  6 Singapore      39.4           84.3                  43.2     37.7        22.6
##  7 South Korea    14.5           85.2                  68.9     26.7        77.2
##  8 The Netherl…   33.8           82.0                  88.0     25.5        30.2
##  9 Germany        27.6           77.2                  70.2     35.8        24.8
## 10 France         28.3           77.2                  80.0     25.5        21.4
## # ℹ 52 more rows
## # ℹ 5 more variables: Government_Strategy <dbl>, Commercial <dbl>,
## #   Total_score <dbl>, Region <chr>, Income_group <chr>
```

## Dropping rows with `filter()`

Refer to the [previous class notes on `filter()`](../class-notes-june-15-2023/#filter-choose-a-subset-of-rows) to see different logical & relational statements you can use to choose only the rows in your data you want to keep.

In particular, you might want to use `%in%` to keep only the rows of your data where the values in a particular column are in a list of interest. For example:


``` r
csv_data
```

```
## # A tibble: 62 × 11
##    Country      Talent Infrastructure Operating_Environment Research Development
##    <chr>         <dbl>          <dbl>                 <dbl>    <dbl>       <dbl>
##  1 United Stat…  100             94.0                  64.6    100         100  
##  2 China          16.5          100                    91.6     71.4        80.0
##  3 United King…   39.6           71.4                  74.6     36.5        25.0
##  4 Canada         31.3           77.0                  93.9     30.7        25.8
##  5 Israel         35.8           67.6                  82.4     32.6        28.0
##  6 Singapore      39.4           84.3                  43.2     37.7        22.6
##  7 South Korea    14.5           85.2                  68.9     26.7        77.2
##  8 The Netherl…   33.8           82.0                  88.0     25.5        30.2
##  9 Germany        27.6           77.2                  70.2     35.8        24.8
## 10 France         28.3           77.2                  80.0     25.5        21.4
## # ℹ 52 more rows
## # ℹ 5 more variables: Government_Strategy <dbl>, Commercial <dbl>,
## #   Total_score <dbl>, Region <chr>, Income_group <chr>
```

Right now, we see this data has row for 62 countries. Maybe you don't care about all those countries. In that case, you can use %in%, with a vector of countries you want to keep, specified with `c("Country 1", "Country 2", etc)`. Remember to spell the country names (or whatever other level names you want to keep) exactly how they're spelled in the data! 


``` r
csv_data_filtered <- csv_data |> 
  filter(Country %in% c("United States of America", "Canada", "Mexico"))

csv_data_filtered
```

```
## # A tibble: 3 × 11
##   Country       Talent Infrastructure Operating_Environment Research Development
##   <chr>          <dbl>          <dbl>                 <dbl>    <dbl>       <dbl>
## 1 United State… 100              94.0                  64.6   100         100   
## 2 Canada         31.3            77.0                  93.9    30.7        25.8 
## 3 Mexico          1.72           41.8                  97.0     8.11        4.46
## # ℹ 5 more variables: Government_Strategy <dbl>, Commercial <dbl>,
## #   Total_score <dbl>, Region <chr>, Income_group <chr>
```

## Fixing number columns encoded as character for some reason

This is one of the more common reasons you might need to clean data. R is normally pretty good at recognizing columns of numbers and storing them as numeric data. However, here are a few instances in which you might need to repair data when a number column is being stored as character.

### The missing value placeholder is being read in as text

R is totally fine with having `NA` values in otherwise numeric columns, but if R reads those missing values in from your dataset file as _text,_ the entire column will be read in as text to be safe. Check out the [project notes on reading in data](../project-notes-reading-in-data/#other-arguments-you-may-need) to see how to use the `na` argument in your file-reading function to render those placeholder values as `NA` and allow the rest of the column to read in as numeric.

### The column names are getting read in as observations

Again, if R detects even _one_ text-like value in a column of a dataset file, it will read the entire column in as text data to be safe. This means that sometimes columns will read in as text when _the column name is not read in as the column name, but instead as the first row of data._ This might happen if you have extra rows of non-data text on the top of your CSV or other plain-text data file. Refer to the [project notes on reading in data](../project-notes-reading-in-data/#other-arguments-you-may-need) to see how to use the `skip` argument in your file-reading function to skip those rows and read in the column names as the top row of the file, no matter where it might actually be.

### The numbers are spelled in an unexpected way

By default, R expects numbers in data files to be formatted according to American English defaults. So, for example, if you read in a column of numbers where each number value contains _only_ digits, R unambiguously knows how to read these in.


``` r
# Don't worry about the syntax in these first two lines
# I am using these as an example to show you
# how read_csv() will interpret numbers spelled this way in your data file
c("value", "1000", "2000", "3000") |> 
  I() |> 
  # I am piping the data into read_csv()
  # so I don't have to manually specify the first argument
  read_csv()
```

```
## Rows: 3 Columns: 1
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## dbl (1): value
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

```
## # A tibble: 3 × 1
##   value
##   <dbl>
## 1  1000
## 2  2000
## 3  3000
```

Specific to American English, R also knows that the comma is often used to break up every third place value of digits, so data containing numbers with commas will still get read in as numeric data.


``` r
c("value", "1,000", "2,000", "3,000") |> 
  I() |> 
  read_csv()
```

```
## Warning: One or more parsing issues, call `problems()` on your data frame for details,
## e.g.:
##   dat <- vroom(...)
##   problems(dat)
```

```
## Rows: 3 Columns: 1
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## num (1): value
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

```
## # A tibble: 3 × 1
##   value
##   <dbl>
## 1  1000
## 2  2000
## 3  3000
```
R also knows that the decimal is specified with a point in American English, so numbers with periods get read in as decimal values.


``` r
c("value", "1.01", "2.01", "3.01") |> 
  I() |> 
  read_csv()
```

```
## Rows: 3 Columns: 1
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## dbl (1): value
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

```
## # A tibble: 3 × 1
##   value
##   <dbl>
## 1  1.01
## 2  2.01
## 3  3.01
```
However, sometimes you might need to read in data with different thousands-grouping or decimal markers. For example, in Europe, they have the decimal and the grouping markers switched. One thousand is spelled like 1.000, and one-point-five is spelled like 1,5 (if you have seen a gas station outside the US, this might be familiar to you). Or, for example, you might have data where the grouping marker for the thousands is a space instead of a comma OR a period. The space is considered an ["international" thousands-grouping marker,](https://www.bipm.org/en/committees/cg/cgpm/22-2003/resolution-10) because it can't be confused for the other two.

These number-spelling differences are considered part of the **locale** of your data, because they are regional. R's default locale is the US, so we normally don't notice anything odd when we use R, but you can always manually specify a locale setting when you read in data spelled according to the conventions of a different locale. Use the `locale` argument of `read_csv()` (or `read_excel()`, or any functions in the family) to manually specify what the decimal mark is and what the thousands-grouping mark is.

For example, these should get read in as the numbers 1, 2, and 3 if we tell R that the decimal is a comma and the thousands-grouping is a period. (If you need to tell R _either_ that the decimal is a comma _or_ that the thousands-grouping is a period, I recommend setting both at the same time so R doesn't get confused with the American English defaults.)


``` r
c("value", "1,000", "2,000", "3,000") |> 
  I() |> 
  read_csv(locale = locale(decimal_mark = ",",
                           grouping_mark = "."))
```

```
## Warning: One or more parsing issues, call `problems()` on your data frame for details,
## e.g.:
##   dat <- vroom(...)
##   problems(dat)
```

```
## Rows: 3 Columns: 1
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## dbl (1): value
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

```
## # A tibble: 3 × 1
##   value
##   <dbl>
## 1     1
## 2     2
## 3     3
```

And these should get read in as regular thousands if we tell R that the thousands-grouping mark is a space, per international weights and measures standards. (We only need to specify the thousands-grouping mark because it should not affect the existing default decimal mark this time.)


``` r
c("value", "1 000", "2 000", "3 000") |> 
  I() |> 
  read_csv(locale = locale(grouping_mark = " "))
```

```
## Rows: 3 Columns: 1
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## num (1): value
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

```
## # A tibble: 3 × 1
##   value
##   <dbl>
## 1  1000
## 2  2000
## 3  3000
```

You can use the `locale` argument along with all the other typical arguments you might set in your file-reading function.

## Recoding categorical variables with `mutate()` and `fct_recode()`

Check out [the previous class notes](../class-notes-june-12-2023/#recoding-categorical-columns) for the detailed description of using `fct_recode()` inside `mutate()` to recode categorical variables.

A few reminders:

The levels always go `"new_level" = "old_level"`. In general, programming syntax order reads right to left, so it's old on the right turning into new on the left!

You only need to do `fct_recode(as.character(YOUR_CATEGORICAL_COLUMN))` if you're starting with a _numeric_ column where _the numbers represent categorical levels._ If your starting column is ALREADY categorical text, you do not need to coerce it to character again.

## Reshaping data with `pivot_longer()`

(To learn more about the technique presented in this section, refer to [section 6.3 of the textbook.](https://r4ds.hadley.nz/data-tidy.html#sec-pivoting))

If you are dealing with data collected at multiple points in time, you may sometimes read in data where each data column contains data from the _same variable,_ but measured at _different points in time._

For example, in these data we see that the maternal mortality rate has been measured in a number of countries every 5 years from 2000 to 2020. Each year's column contains data from the _same variable._


``` r
# Need readxl to read in excel files
library(readxl)

excel_data <- read_excel(here::here("ignore",
                                    "data",
                                    "MMR-maternal-deaths-and-LTR_MMEIG-trends_2000-2020_released-Feb_2023.xlsx"),
                         sheet = 2,
                         range = "A5:H190")
```


``` r
excel_data
```

```
## # A tibble: 185 × 8
##    `UNICEF Region` `ISO Code` Country         `2000` `2005` `2010` `2015` `2020`
##    <chr>           <chr>      <chr>            <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
##  1 ROSA            AFG        Afghanistan       1346   1103    899    776    620
##  2 ECARO           ALB        Albania             14     11      9      7      8
##  3 MENA            DZA        Algeria            159    144    112     89     78
##  4 ESARO           AGO        Angola             860    550    367    274    222
##  5 LACRO           ATG        Antigua and Ba…     51     34     31     27     21
##  6 LACRO           ARG        Argentina           72     63     55     39     45
##  7 ECARO           ARM        Armenia             50     38     33     25     27
##  8 Industrialized  AUS        Australia            7      5      5      5      3
##  9 Industrialized  AUT        Austria              6      6      6      6      5
## 10 ECARO           AZE        Azerbaijan          56     44     33     29     41
## # ℹ 175 more rows
```

To work with these data in R, we want to _reshape_ them so that there is _only one column containing the maternal mortality rate data._ Right now, each row contains all the observations for a particular country, but we need to make the data _longer in rows and narrower in columns_ (or, more hot-dog shaped if you like) so that each row contains the observations for a particular country _in a particular year._ This will let us analyze data _either_ by country _or_ by year (or both).

To reshape the data and make it longer, we will use the function `pivot_longer()`.

Once you pipe your data into `pivot_longer()`, you _always_ need to set the following three arguments:

1. `cols`: Which columns contain the repeated-measure data that you need to hot-dog-ify? Here, because all the year columns are adjacent to one another, we can use the colon `:` to go _from_ the 2000 column `through` the 2020 column. (You can select columns using any valid syntax you might use to choose columns inside `select()`.) Notice that the year column names are quoted with backtick quotes, otherwise R would treat them as numbers and not column names.
1. `names_to`: What do you want the new name of the column containing the old column names to be? In quotes.
1. `values_to`: What do you want the new name of the column containing the actual observations to be? In quotes.

Here, I've also set the `names_transform` argument so that the new `Year` column will get turned into numbers, because that column would be text by default. R expects column names to be text, so the `names_to` column comes out as text by default unless you tell R otherwise.


``` r
excel_data_long <- excel_data |> 
  pivot_longer(cols = `2000`:`2020`,
               names_to = "Year",
               values_to = "MMR",
               names_transform = list(Year = as.numeric))
```

And now the data have one long column for maternal mortality rate, with identifying columns _both_ for country _and_ for year the data were collected!


``` r
excel_data_long
```

```
## # A tibble: 925 × 5
##    `UNICEF Region` `ISO Code` Country      Year   MMR
##    <chr>           <chr>      <chr>       <dbl> <dbl>
##  1 ROSA            AFG        Afghanistan  2000  1346
##  2 ROSA            AFG        Afghanistan  2005  1103
##  3 ROSA            AFG        Afghanistan  2010   899
##  4 ROSA            AFG        Afghanistan  2015   776
##  5 ROSA            AFG        Afghanistan  2020   620
##  6 ECARO           ALB        Albania      2000    14
##  7 ECARO           ALB        Albania      2005    11
##  8 ECARO           ALB        Albania      2010     9
##  9 ECARO           ALB        Albania      2015     7
## 10 ECARO           ALB        Albania      2020     8
## # ℹ 915 more rows
```
