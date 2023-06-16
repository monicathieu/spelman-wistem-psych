---
title: "Class notes: June 15, 2023"
output: blogdown::html_page
description: "Lecture notes and helpful info after class."
excerpt: "Lecture notes and helpful info after class."
date: 2023-06-15
lastmod: "2023-06-16"
draft: false
images: []
categories: ["Class notes"]
tags: []
contributors: ["Monica Thieu"]
pinned: false
homepage: false
---

## Lecture notes

This isn’t the exact *order* that we practiced these concepts in class, but I’ve reordered these into an order that reads/flows more sensibly after the fact.

You can read more about these in [Chapter 4 of R for Data Science](https://r4ds.hadley.nz/data-transform.html). For more technical, in-depth information about dataframe manipulation tools, check out the package docs for [dplyr,](https://dplyr.tidyverse.org), the package these functions come from.

First, we need to read in the data used for the demonstrations below.

``` r
big5_raw <- read_tsv(here::here("static", "datasets", "BIG5_20140518", "data.csv"))
```

    ## Rows: 19719 Columns: 57
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: "\t"
    ## chr  (1): country
    ## dbl (56): race, age, engnat, gender, hand, source, E1, E2, E3, E4, E5, E6, E...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

### `|>`: the pipe

The pipe does one simple, but key, thing: **takes the object on the left-hand side and feeds it into the first argument of the function on the right-hand side.** This means that:

`a |> foo()` is equivalent to `foo(a)`. Fine and good.

`a |> foo() %>% bar(arg = TRUE)` is equivalent to `bar(foo(a), arg = TRUE)`. Now, nested function calls read left-to-right!

Generally, a block of code like:

    df_new <- df_old |> 
      foo() |> 
      bar(arg = TRUE) |> 
      baz()

is equivalent to:

    df_new <- baz(bar(foo(df_old), arg = TRUE))

Now, you can chain a series of commands to operate on a dataframe all at once, and easily read those commands from left-to-right and top-to-bottom, in the same order that R executes them.

In order to use a function in a \|\> chain, the function must:

- take as their first argument the object (usually a dataframe) to be operated upon
- return the same object, but with the operation completed

Essentially all the functions from the `tidyverse` package that we will use are pipe-safe, but bear this in mind when trying to use functions from other packages in pipe chains.

### `mutate()`: modify columns of the data

Use `mutate()` to add new columns to the data, or change/overwrite existing columns.

The general syntax inside `mutate()` is `new_col = function_of(old_col)`, or `old_col = function_of(old_col)` if you want to overwrite an existing column.

You can modify any kind of column: text data, numeric data, or otherwise, as long as you use the right functions that match that kind of data. For example, you can only `fct_recode()` on text columns, and you can only do math operations on numeric columns.

You can use many kinds of functions to modify these columns, as long as those functions work in a *one-to-one* way: The function needs to be able to take a column as input, and return a column *of the same length* as output.

``` r
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
         )
```

Using some special syntax, use the function `across()` inside of `mutate()` to apply the same function to multiple columns at once. By default, using `across()` will overwrite those columns in place, but there are more advanced argument settings you can specify to return new columns with different names.

### `filter()`: choose a subset of *rows*

Use `filter()` to return a shortened version of your dataframe, choosing rows based on relational statements with columns in the dataframe.

``` r
big5_modified |> 
  filter(hand_char == "left")
```

    ## # A tibble: 1,724 × 60
    ##     race   age engnat gender  hand source country    E1    E2    E3    E4    E5
    ##    <dbl> <dbl>  <dbl>  <dbl> <dbl>  <dbl> <chr>   <dbl> <dbl> <dbl> <dbl> <dbl>
    ##  1     3    21      1      1     2      2 CA          3     3     4     3     4
    ##  2     3    25      1      1     2      4 US          4     4     4     4     4
    ##  3     3    15      1      2     2      1 GB          1     2     3     2     3
    ##  4     3    46      1      2     2      3 US          1     4     3     3     3
    ##  5     3    18      1      2     2      2 US          2     3     3     3     4
    ##  6     3    19      1      2     2      1 US          1     2     2     2     1
    ##  7     3    23      2      1     2      1 SE          2     5     4     4     2
    ##  8     5    35      1      2     2      1 US          4     5     5     3     5
    ##  9     5    47      1      2     2      2 US          1     1     2     1     1
    ## 10     6    16      2      2     2      1 BR          2     4     3     3     3
    ## # ℹ 1,714 more rows
    ## # ℹ 48 more variables: E6 <dbl>, E7 <dbl>, E8 <dbl>, E9 <dbl>, E10 <dbl>,
    ## #   N1 <dbl>, N2 <dbl>, N3 <dbl>, N4 <dbl>, N5 <dbl>, N6 <dbl>, N7 <dbl>,
    ## #   N8 <dbl>, N9 <dbl>, N10 <dbl>, A1 <dbl>, A2 <dbl>, A3 <dbl>, A4 <dbl>,
    ## #   A5 <dbl>, A6 <dbl>, A7 <dbl>, A8 <dbl>, A9 <dbl>, A10 <dbl>, C1 <dbl>,
    ## #   C2 <dbl>, C3 <dbl>, C4 <dbl>, C5 <dbl>, C6 <dbl>, C7 <dbl>, C8 <dbl>,
    ## #   C9 <dbl>, C10 <dbl>, O1 <dbl>, O2 <dbl>, O3 <dbl>, O4 <dbl>, O5 <dbl>, …

Remember to spell column names and comparison terms correctly, otherwise you’ll filter for the wrong thing!

Here are the relational operators you can use:

- `>` (greater than)
- `<` (less than)
- `>=` (greater than or equal to)
- `<=` (less than or equal to)
- `==` (is equal to–remember that it is *two* equals signs)
- `!=` (is not equal to)
- `%in%` (is contained by; this is useful when you need to see whether the element on the left matches any of a vector of elements on the right)

To use `%in%`, you must specify the vector of potential values using `c()` to group them together:

``` r
big5_modified |> 
  filter(hand_char %in% c("left", "both"))
```

    ## # A tibble: 2,195 × 60
    ##     race   age engnat gender  hand source country    E1    E2    E3    E4    E5
    ##    <dbl> <dbl>  <dbl>  <dbl> <dbl>  <dbl> <chr>   <dbl> <dbl> <dbl> <dbl> <dbl>
    ##  1     5    39      1      2     3      4 US          3     5     5     5     5
    ##  2     3    26      1      2     3      5 GB          2     3     4     3     1
    ##  3     3    21      1      1     2      2 CA          3     3     4     3     4
    ##  4     3    25      1      1     2      4 US          4     4     4     4     4
    ##  5     3    15      1      2     2      1 GB          1     2     3     2     3
    ##  6     3    46      1      2     2      3 US          1     4     3     3     3
    ##  7     3    18      1      2     2      2 US          2     3     3     3     4
    ##  8     3    19      1      2     2      1 US          1     2     2     2     1
    ##  9     3    23      2      1     2      1 SE          2     5     4     4     2
    ## 10     5    35      1      2     2      1 US          4     5     5     3     5
    ## # ℹ 2,185 more rows
    ## # ℹ 48 more variables: E6 <dbl>, E7 <dbl>, E8 <dbl>, E9 <dbl>, E10 <dbl>,
    ## #   N1 <dbl>, N2 <dbl>, N3 <dbl>, N4 <dbl>, N5 <dbl>, N6 <dbl>, N7 <dbl>,
    ## #   N8 <dbl>, N9 <dbl>, N10 <dbl>, A1 <dbl>, A2 <dbl>, A3 <dbl>, A4 <dbl>,
    ## #   A5 <dbl>, A6 <dbl>, A7 <dbl>, A8 <dbl>, A9 <dbl>, A10 <dbl>, C1 <dbl>,
    ## #   C2 <dbl>, C3 <dbl>, C4 <dbl>, C5 <dbl>, C6 <dbl>, C7 <dbl>, C8 <dbl>,
    ## #   C9 <dbl>, C10 <dbl>, O1 <dbl>, O2 <dbl>, O3 <dbl>, O4 <dbl>, O5 <dbl>, …

You can modify/combine relational statements using these logical operators:

- `!` (NOT operator; this returns the opposite of whatever follows it)
- `&` (AND operator; this returns `TRUE` if both statements on either side are both true
- `|` (Pipe, or shift-backslash–OR operator; this returns `TRUE` if *at least one* of the statements on either side is true)

If you use multiple relational statements one after the other separated by a comma, `filter()` combines them with **AND** by default.

``` r
big5_modified |> 
  filter(hand_char %in% c("left", "both"),
         E1 > 3)
```

    ## # A tibble: 577 × 60
    ##     race   age engnat gender  hand source country    E1    E2    E3    E4    E5
    ##    <dbl> <dbl>  <dbl>  <dbl> <dbl>  <dbl> <chr>   <dbl> <dbl> <dbl> <dbl> <dbl>
    ##  1     3    25      1      1     2      4 US          4     4     4     4     4
    ##  2     5    35      1      2     2      1 US          4     5     5     3     5
    ##  3    13    13      1      2     3      1 US          5     5     5     1     5
    ##  4     3    20      1      1     2      1 GB          4     5     5     4     5
    ##  5     3    19      1      2     2      1 CA          5     4     5     5     5
    ##  6     3    29      1      1     3      1 US          5     5     5     3     5
    ##  7     3    39      1      2     2      1 AU          5     5     5     5     5
    ##  8     3    22      2      2     2      3 DK          4     5     5     4     4
    ##  9     3    33      2      1     2      1 SE          5     5     5     1     5
    ## 10     3    38      1      2     2      1 US          4     4     4     4     4
    ## # ℹ 567 more rows
    ## # ℹ 48 more variables: E6 <dbl>, E7 <dbl>, E8 <dbl>, E9 <dbl>, E10 <dbl>,
    ## #   N1 <dbl>, N2 <dbl>, N3 <dbl>, N4 <dbl>, N5 <dbl>, N6 <dbl>, N7 <dbl>,
    ## #   N8 <dbl>, N9 <dbl>, N10 <dbl>, A1 <dbl>, A2 <dbl>, A3 <dbl>, A4 <dbl>,
    ## #   A5 <dbl>, A6 <dbl>, A7 <dbl>, A8 <dbl>, A9 <dbl>, A10 <dbl>, C1 <dbl>,
    ## #   C2 <dbl>, C3 <dbl>, C4 <dbl>, C5 <dbl>, C6 <dbl>, C7 <dbl>, C8 <dbl>,
    ## #   C9 <dbl>, C10 <dbl>, O1 <dbl>, O2 <dbl>, O3 <dbl>, O4 <dbl>, O5 <dbl>, …

If you want to use logical **OR,** you must use the bar: `|`.

### `select()`: choose a subset of *columns*

Use `select()` to return a narrow version of your dataframe, choosing a subset of columns.

Generally, enter the column names you want to keep, *in the order you want them in the resulting dataframe.*

``` r
big5_modified |> 
  select(gender_char, hand_char, engnat_char, A1, A2, A3, A4, A5)
```

    ## # A tibble: 19,719 × 8
    ##    gender_char hand_char engnat_char    A1    A2    A3    A4    A5
    ##    <fct>       <fct>     <fct>       <dbl> <dbl> <dbl> <dbl> <dbl>
    ##  1 male        right     yes             5     5     5     5     4
    ##  2 female      right     yes             5     3     3     4     2
    ##  3 female      right     no              1     1     1     5     5
    ##  4 female      right     no              4     5     2     4     3
    ##  5 female      right     no              1     5     3     5     5
    ##  6 female      right     yes             4     2     3     4     3
    ##  7 female      right     yes             1     5     5     5     5
    ##  8 male        right     no              4     5     5     4     3
    ##  9 female      both      yes             5     5     5     5     5
    ## 10 female      right     yes             4     3     5     4     4
    ## # ℹ 19,709 more rows

If you want to keep a range of adjacent columns, use the colon `:` to select “through”. If you use the colon, you need to know what order the columns start off in so you know which columns land in the range!

``` r
# Same result as the syntax above
# You can read that last bit as "A1 through A5"
big5_modified |> 
  select(gender_char, hand_char, engnat_char, A1:A5)
```

    ## # A tibble: 19,719 × 8
    ##    gender_char hand_char engnat_char    A1    A2    A3    A4    A5
    ##    <fct>       <fct>     <fct>       <dbl> <dbl> <dbl> <dbl> <dbl>
    ##  1 male        right     yes             5     5     5     5     4
    ##  2 female      right     yes             5     3     3     4     2
    ##  3 female      right     no              1     1     1     5     5
    ##  4 female      right     no              4     5     2     4     3
    ##  5 female      right     no              1     5     3     5     5
    ##  6 female      right     yes             4     2     3     4     3
    ##  7 female      right     yes             1     5     5     5     5
    ##  8 male        right     no              4     5     5     4     3
    ##  9 female      both      yes             5     5     5     5     5
    ## 10 female      right     yes             4     3     5     4     4
    ## # ℹ 19,709 more rows

You can also use [helper functions](https://tidyselect.r-lib.org/reference/language.html) to select columns based on their names.

``` r
big5_modified |> 
  select(ends_with("char"), A1:A5)
```

    ## # A tibble: 19,719 × 8
    ##    gender_char hand_char engnat_char    A1    A2    A3    A4    A5
    ##    <fct>       <fct>     <fct>       <dbl> <dbl> <dbl> <dbl> <dbl>
    ##  1 male        right     yes             5     5     5     5     4
    ##  2 female      right     yes             5     3     3     4     2
    ##  3 female      right     no              1     1     1     5     5
    ##  4 female      right     no              4     5     2     4     3
    ##  5 female      right     no              1     5     3     5     5
    ##  6 female      right     yes             4     2     3     4     3
    ##  7 female      right     yes             1     5     5     5     5
    ##  8 male        right     no              4     5     5     4     3
    ##  9 female      both      yes             5     5     5     5     5
    ## 10 female      right     yes             4     3     5     4     4
    ## # ℹ 19,709 more rows

### `rename()`: rename columns

While you can use `select()` to rename columns, use `rename()` if the *only* thing you want to do is rename columns (without dropping any columns or changing the column order).

They both use the syntax `new_col = old_col`. (When we `select()` without specifying a new column name, we are effectively just saying `old_col`.)

``` r
big5_modified |> 
  # I looked, and I know E1 to O10 will capture all 50 personality columns
  select(ends_with("char"), E1:O10) |> 
  rename(gender = gender_char,
         hand = hand_char,
         engnat = engnat_char)
```

    ## # A tibble: 19,719 × 53
    ##    gender hand  engnat    E1    E2    E3    E4    E5    E6    E7    E8    E9
    ##    <fct>  <fct> <fct>  <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
    ##  1 male   right yes        4     4     5     4     5     5     4     3     5
    ##  2 female right yes        2     4     3     3     3     3     1     1     1
    ##  3 female right no         5     5     1     2     5     5     1     1     5
    ##  4 female right no         2     1     2     2     3     2     3     2     4
    ##  5 female right no         3     5     3     3     3     5     3     5     3
    ##  6 female right yes        1     1     2     2     1     3     2     2     1
    ##  7 female right yes        5     5     5     5     5     5     5     2     4
    ##  8 male   right no         4     3     5     3     5     5     4     3     4
    ##  9 female both  yes        3     5     5     5     5     5     5     4     5
    ## 10 female right yes        1     2     2     1     2     2     1     2     1
    ## # ℹ 19,709 more rows
    ## # ℹ 41 more variables: E10 <dbl>, N1 <dbl>, N2 <dbl>, N3 <dbl>, N4 <dbl>,
    ## #   N5 <dbl>, N6 <dbl>, N7 <dbl>, N8 <dbl>, N9 <dbl>, N10 <dbl>, A1 <dbl>,
    ## #   A2 <dbl>, A3 <dbl>, A4 <dbl>, A5 <dbl>, A6 <dbl>, A7 <dbl>, A8 <dbl>,
    ## #   A9 <dbl>, A10 <dbl>, C1 <dbl>, C2 <dbl>, C3 <dbl>, C4 <dbl>, C5 <dbl>,
    ## #   C6 <dbl>, C7 <dbl>, C8 <dbl>, C9 <dbl>, C10 <dbl>, O1 <dbl>, O2 <dbl>,
    ## #   O3 <dbl>, O4 <dbl>, O5 <dbl>, O6 <dbl>, O7 <dbl>, O8 <dbl>, O9 <dbl>, …

(Now that you’ve seen `rename()` in the lecture notes, I’ll re-create the dataframe `big5_modified`, but renaming those columns and *saving them* into the new object. Now, the new `big5_modified` will have the new column changes.)

``` r
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

### `count()`: summarize by counting the number of rows

`count()` does one *specific* type of summarizing: Counting the number of observations at each level of a categorical variable.

`count()` returns a *summarized* dataframe. Instead of having one row per observation, you get one row per *level of the counted variable.*

``` r
big5_modified |> 
  count(engnat)
```

    ## # A tibble: 3 × 2
    ##   engnat     n
    ##   <fct>  <int>
    ## 1 yes    12379
    ## 2 no      7270
    ## 3 <NA>      70

If you `count()` the levels of multiple variables at once, the `n` column gives you the observations at each possible combination of the counted variables:

``` r
big5_modified |> 
  count(gender, engnat)
```

    ## # A tibble: 12 × 3
    ##    gender engnat     n
    ##    <fct>  <fct>  <int>
    ##  1 male   yes     4392
    ##  2 male   no      3188
    ##  3 male   <NA>      28
    ##  4 female yes     7904
    ##  5 female no      4041
    ##  6 female <NA>      40
    ##  7 other  yes       69
    ##  8 other  no        32
    ##  9 other  <NA>       1
    ## 10 <NA>   yes       14
    ## 11 <NA>   no         9
    ## 12 <NA>   <NA>       1

{{< alert icon="🤔" >}}
Food for thought: What happens when you try to count() the observations in a continuous variable? What does that tell you about how count() works? Try it for yourself and see!
{{< /alert >}}

### `summarize()`: summarize using functions of your choice

`summarize()` is the general function for summarizing any columns in your data using any summary functions that satisfy the following rule:

A summarizing function must take *all* the values in the column as inputs, and returns *one* summary value as an output.

For example, we can use summary statistics functions like `mean()` and `sd()` to calculate the mean and standard deviation of some columns.

When we want to name our summarized output columns (which we almost always do), we use some familiar syntax: `summarized_col = function_of(input_col)`.

``` r
big5_modified |> 
  summarize(mean_E1 = mean(E1),
            sd_E1 = sd(E1),
            mean_E2 = mean(E2),
            sd_E2 = sd(E2))
```

    ## # A tibble: 1 × 4
    ##   mean_E1 sd_E1 mean_E2 sd_E2
    ##     <dbl> <dbl>   <dbl> <dbl>
    ## 1    2.63  1.23    3.24  1.31

## Peek ahead

- **`select()` tricks:** How can we use `select()` to *drop* columns (keep everything EXCEPT the columns specified)? Or *reorder* columns without dropping any columns?
- **Summarizing for different groups:** How can you use the new function `group_by()` before `summarize()` so that instead of getting one summary value for the entire column, you get sub-summary values for *each level of a categorical variable?* (For example, how can I get the mean value of E1 separately for men, women, and genderqueer participants, instead of across everyone?)
- **More knowledge through data manipulation:** How can you use `mutate()` to calculate each participant’s *sum score* for each of the Big 5 personality factors? For example, the total extraversion score is the sum of scores for E1 through E10. Once we have those sum scores, we can actually analyze patterns of personality scores across the dataset!
