PM566 Midterm Project
================
Flemming Wu
2022-10-09

Main source of data:

<https://www.ars.usda.gov/northeast-area/beltsville-md-bhnrc/beltsville-human-nutrition-research-center/food-surveys-research-group/docs/wweia-documentation-and-data-sets/>

Questions:  
Insulin resistance and diabetes is a growing health issue for Americans.
Causes of this includes consumption of foods high in carbohydrates and
saturated fats, causing blood glucose levels to spike and insulin is
needed to bring the blood glucose levels back down to normal. Over time,
consistent high blood glucose spikes lead to the body becoming
accustomed to high amounts of sugar, and therefore acquiring insulin
resistance. In this project, I will be investigating what attributes
cause people to eat foods that lead to blood glucose spikes.  

-   What time and/or day of the week do people generally eat foods high
    in sugar/saturated fats?
-   Does having a dietary restriction affect sugar/fat consumption?
-   Does sugar consumption vary by age, ethnicity, gender, or pregnancy
    status?
-   Does source of food or whether the meal was eaten at home have an
    effect?

Load Libraries

``` r
library(haven)
library(data.table)
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.2 ──
    ## ✔ ggplot2 3.3.6      ✔ purrr   0.3.4 
    ## ✔ tibble  3.1.8      ✔ dplyr   1.0.10
    ## ✔ tidyr   1.2.1      ✔ stringr 1.4.1 
    ## ✔ readr   2.1.2      ✔ forcats 0.5.2 
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::between()   masks data.table::between()
    ## ✖ dplyr::filter()    masks stats::filter()
    ## ✖ dplyr::first()     masks data.table::first()
    ## ✖ dplyr::lag()       masks stats::lag()
    ## ✖ dplyr::last()      masks data.table::last()
    ## ✖ purrr::transpose() masks data.table::transpose()

``` r
library(dtplyr)
library(httr)
library(xml2)
library(rvest)
```

    ## 
    ## Attaching package: 'rvest'
    ## 
    ## The following object is masked from 'package:readr':
    ## 
    ##     guess_encoding

Read in data from CDC website

``` r
fs_d1 <- read_xpt("https://wwwn.cdc.gov/NCHS/nhanes/2017-2018/P_DR1IFF.xpt")
demographic <- read_xpt("https://wwwn.cdc.gov/NCHS/nhanes/2017-2018/P_DEMO.xpt")
food_categories <- read_xpt("https://wwwn.cdc.gov/NCHS/nhanes/2017-2018/P_drxfcd.xpt")
```

Rename columns with the column labels provided

``` r
# Get get all of the column labels in a data frame
fs_d1_labels <- purrr::map_df(fs_d1, ~attributes(.x))

demographic_labels <- purrr::map_df(demographic, ~attributes(.x))

fc_labels <- purrr::map_df(food_categories, ~attributes(.x))


# gsub() the spaces to underscores and put into vector
fs_d1_labels <- gsub(" ", "_", fs_d1_labels$label) %>%
    as.vector() %>%
    tolower() %>%
    unique()

demographic_labels <- gsub(" ", "_", demographic_labels$label) %>%
    as.vector() %>%
    tolower()

fc_labels <- gsub(" ", "_", fc_labels$label) %>%
    as.vector() %>%
    tolower()


# reassign names
names(fs_d1) <- fs_d1_labels

names(demographic) <- demographic_labels

names(food_categories) <- fc_labels
```

Convert to data table

``` r
fs_d1 <- as.data.table(fs_d1)
food_categories <- as.data.table(food_categories)
demographic <- as.data.table(demographic)
```

Remove unnecessary columns

``` r
fs_d1 <- fs_d1[, c(1:30)]
demographic <- demographic[, respondent_sequence_number:pregnancy_status_at_exam]
food_categories <- food_categories[, -(former_short_food_code_description:former_long_food_code_description)]
```
