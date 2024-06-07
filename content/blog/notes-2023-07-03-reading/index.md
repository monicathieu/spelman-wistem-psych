---
title: "Project notes: Reading in data"
output: blogdown::html_page
description: "Lecture notes for reading project data into R."
excerpt: "Lecture notes for reading project data into R."
date: 2023-07-03
lastmod: "2024-06-07"
draft: false
images: []
categories: ["Class notes"]
tags: ["2023"]
contributors: ["Monica Thieu"]
pinned: false
homepage: false
---

Now that we're fully in the project work stage of the course, the lecture notes after class will focus mainly on demonstrating and explaining specific techniques that people need for their particular project data.

## Reading in data


``` r
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

``` r
library(readxl)
```


### Plain-text files (CSVs, normally)

Plain-text files are by far the easiest to read in to R. You can tell a file is plain text because it will be human-readable (although perhaps ugly) when you open it in Notepad (Windows) or TextEdit (Mac). Your plain-text file is probably a .csv, but it's possible for it to be saved as .txt or another text file ending.

CSV stands for **c**omma-**s**eparated **v**alues. This means that the individual pieces of data are separated from one another in the file, or **delimited,** with commas. You can get other plain-text data files that are delimited with other characters, like semicolons or tabs (which are indeed a single character!). We need to know what delimiter our file uses so that R knows what to expect when reading in the data.

The vast majority of the time, a file that ends in .csv is delimited with commas. However, just to be sure, you might want to open your data in TextEdit (or the raw file viewer in RStudio Cloud after you've uploaded it to your project) and look to see what character is separating each observation.

If it is delimited with commas, you can use `read_csv()` to read in your data, and it _should_ just work with the file path as the only argument:


``` r
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

(You can use `read_tsv()` if the data are delimited with tabs. For any other delimiter character, you can use the flexible `read_delim()` but you will need to set the `delim` argument to tell R what delimiter to use. For example, for a semicolon, it would need to be something like `read_delim("path/to/data.txt", delim = ";")`.)

Now we see that the data are read in as a dataframe we can interact with in R! Woohoo.


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

#### Other arguments you may need

Depending on how your data are coded, you may need to tell R that some values specifically represent missing data. For example, missing data could be represented with "NA", an empty cell "", "N/A", "None", 0, or basically anything else depending on the preference of the people who made the data.

By default, `read_csv()` will read in both empty cells "" and values of "NA" (spelled _exactly_ like that) as proper missing `NA` values. If you need to specify any other values to be read in as missing, set the `na` argument.

For example, if "None" is the value for missing data:

```r
# This code is fake! It's not actually running btw
example_data <- read_csv("path/to/data.csv",
                         na = "None")
```

Or if it's sometimes spelled "N/A" in caps and sometimes "n/a" in lowercase, and _both_ represent missing data, you can specify multiple missing values using `c()`:

```r
# This code is also fake!
example_data <- read_csv("path/to/data.csv",
                         na = c("N/A", "n/a"))
```

You might also sometimes need to skip one or more rows when reading in a CSV file. Sometimes people put the title of the dataset, or some other comments, in the first couple rows of the text file. This information is useful to read, but it disrupts the shape of the dataframe when R reads it in. Instead, we always want to start reading the CSV at the row containing the column names.

If you want to toss the first row, set `skip = 1`:

```r
# This code is also fake!
example_data <- read_csv("path/to/data.csv",
                         skip = 1)
```

In general, you can set the `skip` argument to whatever number of rows you want to skip. The data will start reading in on the _next_ row. For example, if you set `skip = 5`, the _6th_ row of the CSV will be read in as the column names.

### Excel files

Excel files are _not_ plain text, because they require Excel or another specialized program to open them. If you open an Excel file in Notepad or TextEdit, you will see gibberish because the data in an Excel file is stored in a way _only_ computers can read, but not people.

Thankfully, the `read_excel()` function from the `readxl` package will take care of most of the Excel-specific weirdness when reading into R. You can only read in data from one block of one sheet (tab) in a workbook (the whole Excel file) at a time in R. Accordingly, you will need to specify a few more _required_ arguments:

- `sheet`: Which sheet or tab of data do you want to read from? You can specify using number index (1 for the first sheet, etc) or using the name of the sheet you want as text.
- `range`: Which cells do you want to read from in that sheet? You will need to specify this as a string, with quotation marks, going from the cell on the top left (should be the first column name) to the cell on the bottom right (the bottom-most observation in the right-most column).

You will need to visually inspect the data in Excel before reading it into R in order to figure out which sheet and range to read from. After that, though, you shouldn't need to interact with the data in Excel anymore.


``` r
excel_data <- read_excel(here::here("ignore",
                                    "data",
                                    "MMR-maternal-deaths-and-LTR_MMEIG-trends_2000-2020_released-Feb_2023.xlsx"),
                         sheet = 2,
                         range = "A5:H190")
```

(This also works--notice that the `sheet` argument is now set to the name of the sheet in the workbook in quotes:)


``` r
excel_data_from_sheet_name <- read_excel(here::here("ignore",
                                    "data",
                                    "MMR-maternal-deaths-and-LTR_MMEIG-trends_2000-2020_released-Feb_2023.xlsx"),
                         sheet = "MMR_country_level",
                         range = "A5:H190")
```

And we can confirm just to be sure that the data read in these two ways are exactly the same.


``` r
# the compare() function from the waldo package is a nice way to check for object equality in R
waldo::compare(excel_data, excel_data_from_sheet_name)
```

```
## ✔ No differences
```

When we print the dataframe contents out, we can see that it looks like a normal R dataframe. Yay!


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

#### Other arguments you may need

If you have certain values you want to read in as `NA`, `read_excel()` also takes an `na` argument that behaves the same as the one in `read_csv()`, so you can follow those instructions.

You shouldn't need to skip any rows when reading in an Excel spreadsheet because you are already using the `range` argument to tell R exactly which cells of data you want.
