---
title: "Class notes: June 20, 2023"
output: blogdown::html_page
description: "Lecture notes and helpful info after class."
excerpt: "Lecture notes and helpful info after class."
date: 2023-06-20
lastmod: "2023-06-21"
draft: false
images: []
categories: ["Class notes"]
tags: []
contributors: ["Monica Thieu"]
pinned: false
homepage: false
---



## Lecture notes

This isn't the exact _order_ that we practiced these concepts in class, but I've reordered these into an order that reads/flows more sensibly after the fact.

As before, you can read more about these in [Chapter 4 of R for Data Science](https://r4ds.hadley.nz/data-transform.html).

First, we need to read in the data used for the demonstrations below.


```r
big5_raw <- read_tsv(here::here("static", "datasets", "BIG5_20140518", "data.csv"))
```

```
## Rows: 19719 Columns: 57
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: "\t"
## chr  (1): country
## dbl (56): race, age, engnat, gender, hand, source, E1, E2, E3, E4, E5, E6, E...
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

Then, we'll do some baseline data cleaning.


```r
big5_modified <- big5_raw |> 
  mutate(gender_char = fct_recode(as.character(gender),
                                  "male" = "1",
                                  "female" = "2",
                                  "other" = "3",
                                  # I'm using this to set the missing values as NA
                                  NULL = "0"),
         hand_char = fct_recode(as.character(hand),
                                "right" = "1",
                                "left" = "2",
                                "both" = "3",
                                NULL = "0"),
         engnat_char = fct_recode(as.character(engnat),
                                  "yes" = "1",
                                  "no" = "2",
                                  NULL = "0"),
         across(.cols = c(E2, E4, E6, E8, E10, N2, N4, A1, A3, A5, A7, C2, C4, C6, C8, O2, O4, O6),
         .fns = \(x) 6-x)
         ) |> 
  select(ends_with("char"), E1:O10) |> 
  rename(gender = gender_char,
         hand = hand_char,
         engnat = engnat_char)
```

### Calculating sums etc. across columns using `rowwise()`, `mutate()`, `c_across()`

In our `mutate()` section above, where we manipulated our columns, we only did one-to-one changes where _one_ input column was transformed into _one_ output column. Sometimes, we may want to create columns that are combinations of many other columns.

For example, we can do that with arithmetic operators because they can operate basically independently on the values in each row.


```r
# we are OVERWRITING the old content of big5_modified
big5_modified <- big5_modified |> 
  mutate(C_sum = C1+C2+C3+C4+C5+C6+C7+C8+C9+C10,
         # This is to show that you can do whatever math stuff you want
         # with numeric columns inside mutate()
         C_avg = (C1+C2+C3+C4+C5+C6+C7+C8+C9+C10) / 10)
```

However, specifically for sums and means, there are more elegant ways of calculating sums and means across the different column variables within each row.

In general, R is optimized for dataframe operations _down columns_ (like mutating and summarizing), so to do this dataframe operation _across rows,_ we need to use some special functions.

1. First, use `rowwise()` to prepare the dataframe for calculating the combo columns
2. Use `c_across()` _inside_ your summarizing function (probably `sum()` or `mean()` most of the time) when creating new columns with `mutate()`
3. Use `ungroup()` to un-rowwise your dataframe and go back to normal


```r
# we are OVERWRITING the old content of big5_modified
big5_modified <- big5_modified |> 
  rowwise() |> 
  mutate(A_sum = sum(c_across(A1:A10)),
         N_sum = sum(c_across(N1:N10)),
         O_sum = sum(c_across(O1:O10)),
         E_sum = sum(c_across(E1:E10)),
         # to show you can do it with mean()
         # instead of sum()
         A_mean = mean(c_across(A1:A10)),
         N_mean = mean(c_across(N1:N10)),
         O_mean = mean(c_across(O1:O10)),
         E_mean = mean(c_across(E1:E10))) |> 
  ungroup()
```

To learn more about the nuts and bolts of why these functions are designed the way they are, you can read this [dplyr article](https://dplyr.tidyverse.org/articles/rowwise.html) about row-wise operations.

Finally, for organization, let's use `select()` to move those new sum columns to the left of the data, because they are the personality score columns we really care about. 

New bonus: When we mainly want to use `select()` to move columns around, without actually dropping any columns, we can use the helper function `everything()` inside `select()` to make sure any columns not explicitly mentioned still go into the result dataframe. They'll all end up on the right side (but in the same relative order they were in before you reordered).


```r
big5_modified <- big5_modified |> 
  select(gender, hand, engnat, ends_with("sum"), everything())
```

### Grouped summaries with `group_by()` and `summarize()`

Remember that we can calculate summary values _down all the rows in a column_ using `summarize()`.


```r
big5_modified |> 
  summarize(conscientiousness_mean = mean(C_sum),
            agreeableness_mean = mean(A_sum),
            neuroticism_mean = mean(N_sum),
            openness_mean = mean(O_sum),
            extraversion_mean = mean(E_sum))
```

```
## # A tibble: 1 × 5
##   conscientiousness_mean agreeableness_mean neuroticism_mean openness_mean
##                    <dbl>              <dbl>            <dbl>         <dbl>
## 1                   33.5               38.4             31.0          39.1
## # ℹ 1 more variable: extraversion_mean <dbl>
```

If we want to calculate these summary values, but _separately_ for each of the levels of a categorical variable, we can **group** the data before summarizing using `group_by()`, with the columns we want to group by.


```r
big5_modified |> 
  group_by(gender) |> 
  summarize(conscientiousness_mean = mean(C_sum),
            agreeableness_mean = mean(A_sum),
            neuroticism_mean = mean(N_sum),
            openness_mean = mean(O_sum),
            extraversion_mean = mean(E_sum))
```

```
## # A tibble: 4 × 6
##   gender conscientiousness_mean agreeableness_mean neuroticism_mean
##   <fct>                   <dbl>              <dbl>            <dbl>
## 1 male                     33.2               36.5             29.1
## 2 female                   33.6               39.7             32.1
## 3 other                    31.4               35.3             34.1
## 4 <NA>                     33.3               38.4             27.5
## # ℹ 2 more variables: openness_mean <dbl>, extraversion_mean <dbl>
```

Similar to `count()`-ing multiple categorical variables, if you `group_by()` multiple categorical variables, you will get a summary value for every combo level of those variables crossed together.


```r
big5_modified |> 
  group_by(gender, hand) |> 
  summarize(conscientiousness_mean = mean(C_sum),
            agreeableness_mean = mean(A_sum),
            neuroticism_mean = mean(N_sum),
            openness_mean = mean(O_sum),
            extraversion_mean = mean(E_sum))
```

```
## `summarise()` has grouped output by 'gender'. You can override using the
## `.groups` argument.
```

```
## # A tibble: 15 × 7
## # Groups:   gender [4]
##    gender hand  conscientiousness_mean agreeableness_mean neuroticism_mean
##    <fct>  <fct>                  <dbl>              <dbl>            <dbl>
##  1 male   right                   33.3               36.6             29.1
##  2 male   left                    32.8               36.1             29.4
##  3 male   both                    33.3               35.1             28.6
##  4 male   <NA>                    33.4               35.1             28.3
##  5 female right                   33.6               39.7             32.1
##  6 female left                    33.7               40.0             32.4
##  7 female both                    34.8               38.0             31.0
##  8 female <NA>                    35.4               39.2             32.6
##  9 other  right                   30.8               35.6             34.6
## 10 other  left                    33                 37.1             32  
## 11 other  both                    33.1               34.2             32.9
## 12 other  <NA>                    33                 24               32.5
## 13 <NA>   right                   32.7               39.7             25.9
## 14 <NA>   both                    36.5               24               31.5
## 15 <NA>   <NA>                    34.2               39.8             32.5
## # ℹ 2 more variables: openness_mean <dbl>, extraversion_mean <dbl>
```
