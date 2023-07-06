---
title: 'Class notes: June 22, 2023'
output: blogdown::html_page
description: "Lecture notes on developing data cleaning plans."
excerpt: "Lecture notes  on developing data cleaning plans."
date: 2023-06-22
lastmod: "2023-07-06"
draft: false
images: []
categories: ["Class notes"]
tags: []
contributors: ["Monica Thieu"]
pinned: false
homepage: false
---



## Lecture notes

Through the rest of the course, we will start touching on topics across the R for Data Science textbook. A lot of today's topics are still mentioned in chapter 4, but you will also benefit from reading more about specific techniques from chapters 13-20.

### Read in the data

First, we need to read in the _newer, larger_ Big 5 personality dataset from [openpsychometrics.org](https://openpsychometrics.org/_rawdata/).

This dataset file also has its values separated with tabs, so we can use `read_tsv()` like we did before (that **t** stands for tab).

Note that the path I am using to read in this data is the path on my local computer, and it won't work for you if you directly copy and paste! You will need to change this to the path to where the data are saved on _your_ RStudio Cloud project, relative to _your_ RStudio Cloud home folder.


```r
big5_2018_raw <- read_tsv(here::here("ignore", "data", "IPIP-FFM-data-8Nov2018", "data-random-half.csv"))
```

```
## Rows: 203068 Columns: 110
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: "\t"
## chr  (107): EXT1, EXT2, EXT3, EXT4, EXT5, EXT6, EXT7, EXT8, EXT9, EXT10, EST...
## dbl    (2): endelapse, IPC
## dttm   (1): dateload
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```
The next sections will cover our **data cleaning to-do list**, and one example solution to each point on the to-do list.

### Why are all the columns reading in as character data?

When we first attempt to read in the data using all the default settings of `read_tsv()`, almost all of the columns are getting read in as character (or text) data. That's weird! The personality columns should all be numbers...


```r
big5_2018_raw
```

```
## # A tibble: 203,068 × 110
##    EXT1  EXT2  EXT3  EXT4  EXT5  EXT6  EXT7  EXT8  EXT9  EXT10 EST1  EST2  EST3 
##    <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr>
##  1 3     4     3     3     3     2     3     3     3     3     4     4     4    
##  2 3     2     4     2     5     1     5     3     4     3     3     3     3    
##  3 2     2     4     2     3     2     2     3     3     5     2     3     1    
##  4 3     1     3     1     5     1     2     4     2     4     4     3     4    
##  5 2     2     4     4     2     1     2     2     5     2     4     2     4    
##  6 4     2     5     3     4     2     5     3     2     3     4     4     4    
##  7 4     3     2     5     2     2     1     5     2     4     2     5     1    
##  8 5     2     5     1     5     1     4     2     4     2     4     2     5    
##  9 4     3     4     2     4     2     4     2     4     3     2     5     2    
## 10 1     3     4     4     4     2     2     4     2     4     2     3     3    
## # ℹ 203,058 more rows
## # ℹ 97 more variables: EST4 <chr>, EST5 <chr>, EST6 <chr>, EST7 <chr>,
## #   EST8 <chr>, EST9 <chr>, EST10 <chr>, AGR1 <chr>, AGR2 <chr>, AGR3 <chr>,
## #   AGR4 <chr>, AGR5 <chr>, AGR6 <chr>, AGR7 <chr>, AGR8 <chr>, AGR9 <chr>,
## #   AGR10 <chr>, CSN1 <chr>, CSN2 <chr>, CSN3 <chr>, CSN4 <chr>, CSN5 <chr>,
## #   CSN6 <chr>, CSN7 <chr>, CSN8 <chr>, CSN9 <chr>, CSN10 <chr>, OPN1 <chr>,
## #   OPN2 <chr>, OPN3 <chr>, OPN4 <chr>, OPN5 <chr>, OPN6 <chr>, OPN7 <chr>, …
```

We can use `count()` to check what levels are in one of the personality question columns that's supposed to be numeric, to check for any sneaky text values in there that are causing the whole column to read in as text.


```r
big5_2018_raw |> 
  count(EXT1)
```

```
## # A tibble: 7 × 2
##   EXT1      n
##   <chr> <int>
## 1 0       743
## 2 1     50122
## 3 2     39875
## 4 3     57449
## 5 4     38345
## 6 5     16167
## 7 NULL    367
```

Aha! This "NULL" value must be the problem. If we use `filter()` to get only the data that has "NULL" in the first column, we can see it clearly represents missing data across all the columns.


```r
big5_2018_raw |> 
  filter(EXT1 == "NULL")
```

```
## # A tibble: 367 × 110
##    EXT1  EXT2  EXT3  EXT4  EXT5  EXT6  EXT7  EXT8  EXT9  EXT10 EST1  EST2  EST3 
##    <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr>
##  1 NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL 
##  2 NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL 
##  3 NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL 
##  4 NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL 
##  5 NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL 
##  6 NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL 
##  7 NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL 
##  8 NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL 
##  9 NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL 
## 10 NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL  NULL 
## # ℹ 357 more rows
## # ℹ 97 more variables: EST4 <chr>, EST5 <chr>, EST6 <chr>, EST7 <chr>,
## #   EST8 <chr>, EST9 <chr>, EST10 <chr>, AGR1 <chr>, AGR2 <chr>, AGR3 <chr>,
## #   AGR4 <chr>, AGR5 <chr>, AGR6 <chr>, AGR7 <chr>, AGR8 <chr>, AGR9 <chr>,
## #   AGR10 <chr>, CSN1 <chr>, CSN2 <chr>, CSN3 <chr>, CSN4 <chr>, CSN5 <chr>,
## #   CSN6 <chr>, CSN7 <chr>, CSN8 <chr>, CSN9 <chr>, CSN10 <chr>, OPN1 <chr>,
## #   OPN2 <chr>, OPN3 <chr>, OPN4 <chr>, OPN5 <chr>, OPN6 <chr>, OPN7 <chr>, …
```

Let's fix this by adding an argument to `read_tsv()`. The `na` argument allows us to specify any values that we _know_ represent missing data, so that R will read them in as `NA` and not as text.


```r
big5_2018_raw <- read_tsv(here::here("ignore", "data", "IPIP-FFM-data-8Nov2018", "data-random-half.csv"),
                          na = "NULL")
```

```
## Rows: 203068 Columns: 110
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: "\t"
## chr    (3): country, lat_appx_lots_of_err, long_appx_lots_of_err
## dbl  (106): EXT1, EXT2, EXT3, EXT4, EXT5, EXT6, EXT7, EXT8, EXT9, EXT10, EST...
## dttm   (1): dateload
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

Let's also use `filter()` to remove the rows of every participant who has `NA` in the personality columns, because they actually have no real data.

The logical statement `!is.na(EXT1)` allows us to keep only the rows where the EXT1 column is _not_ missing (the not comes from the !).


```r
big5_2018 <- big5_2018_raw |> 
  filter(!is.na(EXT1))
```

### Can we drop some columns to save memory?

By reading the codebook, we can see that the only data columns are the personality question columns, and the reaction time columns for each question. I don't have plans to analyze the reaction time data, so we can use `select()` to keep only the personality question columns.

I can select from `EXT1:OPN10` because I know all those columns are adjacent to one another.


```r
big5_2018 <- big5_2018 |> 
  select(EXT1:OPN10)
```

### What do the 0 personality responses represent?

The codebook file tells us that responses on each personality question go from 1 to 5, just like in the previous dataset. But we saw earlier that some values are 0... what do we think this means?

If we look at some rows where participants have a response of 0 for `EXT1`, we can see that it looks like 0 _also_ represents **missing** data. However, some of these people do have responses for some questions and not others, not like "NULL" when every single response was missing.


```r
big5_2018 |> 
  filter(EXT1 == 0)
```

```
## # A tibble: 743 × 50
##     EXT1  EXT2  EXT3  EXT4  EXT5  EXT6  EXT7  EXT8  EXT9 EXT10  EST1  EST2  EST3
##    <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
##  1     0     0     0     0     0     0     0     0     0     0     0     0     0
##  2     0     0     0     0     0     0     0     0     0     0     0     0     0
##  3     0     2     4     5     4     2     3     1     5     2     3     5     4
##  4     0     0     0     0     2     0     0     0     0     0     0     0     0
##  5     0     3     5     1     0     5     5     1     3     2     5     3     5
##  6     0     3     3     3     3     2     2     4     2     4     2     3     4
##  7     0     2     3     4     3     2     2     4     4     4     2     4     4
##  8     0     0     0     0     0     0     0     0     0     0     0     0     0
##  9     0     0     0     0     0     0     0     0     0     0     0     0     0
## 10     0     0     0     0     0     0     0     0     0     0     0     0     0
## # ℹ 733 more rows
## # ℹ 37 more variables: EST4 <dbl>, EST5 <dbl>, EST6 <dbl>, EST7 <dbl>,
## #   EST8 <dbl>, EST9 <dbl>, EST10 <dbl>, AGR1 <dbl>, AGR2 <dbl>, AGR3 <dbl>,
## #   AGR4 <dbl>, AGR5 <dbl>, AGR6 <dbl>, AGR7 <dbl>, AGR8 <dbl>, AGR9 <dbl>,
## #   AGR10 <dbl>, CSN1 <dbl>, CSN2 <dbl>, CSN3 <dbl>, CSN4 <dbl>, CSN5 <dbl>,
## #   CSN6 <dbl>, CSN7 <dbl>, CSN8 <dbl>, CSN9 <dbl>, CSN10 <dbl>, OPN1 <dbl>,
## #   OPN2 <dbl>, OPN3 <dbl>, OPN4 <dbl>, OPN5 <dbl>, OPN6 <dbl>, OPN7 <dbl>, …
```

### Counting up how many actual responses each person has

We can apply some of the techniques we used before, with some new functions as well, to turn each of the 50 personality question columns into 0/1 _binary_ data, with 1 whenever the participant DID say something (anything from 1-5). Then, we can count how many 1 values each participant has across all 50 columns, to count up the number of questions people actually responded to.

We will do this by creating a _new_ dataframe JUST to hold the question count column.


```r
big5_missing_counts <- big5_2018 |> 
  # First, use across() in mutate() to turn each personality col to binary
  # Use 1 if they DID respond, and 0 if they did NOT
  mutate(
    across(.cols = everything(),
           .fns = \(x) if_else(x > 0, 1, 0))
  ) |> 
  # Second, use special mutate() to COUNT the 1 values across the binary cols
  rowwise() |> 
  mutate(n_responded = sum(c_across(everything()))) |> 
  ungroup() |> 
  # Finally, drop everything except the count of responded questions
  select(n_responded)
```


```r
big5_missing_counts
```

```
## # A tibble: 202,701 × 1
##    n_responded
##          <dbl>
##  1          50
##  2          49
##  3          50
##  4          49
##  5          50
##  6          50
##  7          50
##  8          50
##  9          50
## 10          50
## # ℹ 202,691 more rows
```

This concludes what we got to at the end of class. Everything below is how I would actually finish the data cleaning, but we didn't get to it as a group. I'm including it here so you can follow my train of thought all the way to the final clean data.

## The rest of the data cleaning (that we didn't finish in class)

### Binding the response count column back onto the main data

Now that we have the response counts in a separate dataframe, we need to stick it horizontally onto the end of our main dataframe, so that we can use it to actually filter our main data later.

We can stick these dataframes together using `bind_cols()`, which will combine two dataframes _horizontally._ **They need to have the same number of rows for this to work!** We know here that they should have the same number of rows because we made `big5_missing_counts` from `big5_2018_raw` above, without dropping any rows.

`bind_cols()` takes the input dataframes as separate arguments, divided with commas.

After that, I'm piping into `select()` so I can move our new column all the way to the left. That way, it'll pop up when we print out the dataframe so I can see what's in that column.


```r
# I'm overwriting the previous big5_2018 with the bind_cols version
big5_2018 <- bind_cols(big5_2018, big5_missing_counts) |> 
  select(n_responded, everything())
```


```r
big5_2018
```

```
## # A tibble: 202,701 × 51
##    n_responded  EXT1  EXT2  EXT3  EXT4  EXT5  EXT6  EXT7  EXT8  EXT9 EXT10  EST1
##          <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
##  1          50     3     4     3     3     3     2     3     3     3     3     4
##  2          49     3     2     4     2     5     1     5     3     4     3     3
##  3          50     2     2     4     2     3     2     2     3     3     5     2
##  4          49     3     1     3     1     5     1     2     4     2     4     4
##  5          50     2     2     4     4     2     1     2     2     5     2     4
##  6          50     4     2     5     3     4     2     5     3     2     3     4
##  7          50     4     3     2     5     2     2     1     5     2     4     2
##  8          50     5     2     5     1     5     1     4     2     4     2     4
##  9          50     4     3     4     2     4     2     4     2     4     3     2
## 10          50     1     3     4     4     4     2     2     4     2     4     2
## # ℹ 202,691 more rows
## # ℹ 39 more variables: EST2 <dbl>, EST3 <dbl>, EST4 <dbl>, EST5 <dbl>,
## #   EST6 <dbl>, EST7 <dbl>, EST8 <dbl>, EST9 <dbl>, EST10 <dbl>, AGR1 <dbl>,
## #   AGR2 <dbl>, AGR3 <dbl>, AGR4 <dbl>, AGR5 <dbl>, AGR6 <dbl>, AGR7 <dbl>,
## #   AGR8 <dbl>, AGR9 <dbl>, AGR10 <dbl>, CSN1 <dbl>, CSN2 <dbl>, CSN3 <dbl>,
## #   CSN4 <dbl>, CSN5 <dbl>, CSN6 <dbl>, CSN7 <dbl>, CSN8 <dbl>, CSN9 <dbl>,
## #   CSN10 <dbl>, OPN1 <dbl>, OPN2 <dbl>, OPN3 <dbl>, OPN4 <dbl>, OPN5 <dbl>, …
```

### Filtering out people with too many missing responses

Now, we can use `filter()` to filter out participants whose numbers are too low. Let's see what happens when we keep only people who responded to all 50 questions.


```r
big5_2018 <- big5_2018 |> 
  filter(n_responded == 50)
```


```r
big5_2018
```

```
## # A tibble: 174,974 × 51
##    n_responded  EXT1  EXT2  EXT3  EXT4  EXT5  EXT6  EXT7  EXT8  EXT9 EXT10  EST1
##          <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
##  1          50     3     4     3     3     3     2     3     3     3     3     4
##  2          50     2     2     4     2     3     2     2     3     3     5     2
##  3          50     2     2     4     4     2     1     2     2     5     2     4
##  4          50     4     2     5     3     4     2     5     3     2     3     4
##  5          50     4     3     2     5     2     2     1     5     2     4     2
##  6          50     5     2     5     1     5     1     4     2     4     2     4
##  7          50     4     3     4     2     4     2     4     2     4     3     2
##  8          50     1     3     4     4     4     2     2     4     2     4     2
##  9          50     3     3     4     5     4     5     3     5     1     5     2
## 10          50     3     3     4     4     4     4     2     4     3     4     3
## # ℹ 174,964 more rows
## # ℹ 39 more variables: EST2 <dbl>, EST3 <dbl>, EST4 <dbl>, EST5 <dbl>,
## #   EST6 <dbl>, EST7 <dbl>, EST8 <dbl>, EST9 <dbl>, EST10 <dbl>, AGR1 <dbl>,
## #   AGR2 <dbl>, AGR3 <dbl>, AGR4 <dbl>, AGR5 <dbl>, AGR6 <dbl>, AGR7 <dbl>,
## #   AGR8 <dbl>, AGR9 <dbl>, AGR10 <dbl>, CSN1 <dbl>, CSN2 <dbl>, CSN3 <dbl>,
## #   CSN4 <dbl>, CSN5 <dbl>, CSN6 <dbl>, CSN7 <dbl>, CSN8 <dbl>, CSN9 <dbl>,
## #   CSN10 <dbl>, OPN1 <dbl>, OPN2 <dbl>, OPN3 <dbl>, OPN4 <dbl>, OPN5 <dbl>, …
```

That gives us 174974 rows remaining, when we started with 203068 rows. I think we'll be okay with the remaining data.

### Calculating sum scores for each of the 5 personality factors

This we already know how to do, from the previous dataset!


```r
big5_2018 <- big5_2018 |> 
  rowwise() |> 
  mutate(CSN_sum = sum(c_across(CSN1:CSN10)),
         AGR_sum = sum(c_across(AGR1:AGR10)),
         EST_sum = sum(c_across(EST1:EST10)),
         OPN_sum = sum(c_across(OPN1:OPN10)),
         EXT_sum = sum(c_across(EXT1:EXT10))) |> 
  ungroup() |> 
  select(ends_with("sum"))
```


```r
big5_2018
```

```
## # A tibble: 174,974 × 5
##    CSN_sum AGR_sum EST_sum OPN_sum EXT_sum
##      <dbl>   <dbl>   <dbl>   <dbl>   <dbl>
##  1      31      31      31      34      30
##  2      30      32      27      32      28
##  3      36      33      34      40      26
##  4      28      34      26      25      33
##  5      42      26      34      34      30
##  6      35      27      32      35      31
##  7      32      31      25      36      32
##  8      29      29      28      30      30
##  9      25      24      28      31      38
## 10      25      29      25      35      35
## # ℹ 174,964 more rows
```

Now this dataframe is actually ready for analysis. 🎉

