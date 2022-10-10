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

Change categorical columns from number codes to categorical variables in
demographic table

``` r
demographic[, `:=`(`interview/examination_status`, fifelse(`interview/examination_status` ==
    1, "interview_only", "interview_and_mec_examined"))]

demographic[, `:=`(gender, fifelse(gender == 1, "male", "female"))]

demographic[, `:=`(`race/hispanic_origin`, fifelse(`race/hispanic_origin` ==
    1, "mexican_american", fifelse(`race/hispanic_origin` ==
    2, "other_hispanic", fifelse(`race/hispanic_origin` == 3,
    "non-hispanic_white", fifelse(`race/hispanic_origin` == 4,
        "non-hispanic_black", "other_race_incl_multiracial")))))]

demographic[, `:=`(`race/hispanic_origin_w/_nh_asian`, fifelse(`race/hispanic_origin_w/_nh_asian` ==
    1, "mexican_american", fifelse(`race/hispanic_origin_w/_nh_asian` ==
    2, "other_hispanic", fifelse(`race/hispanic_origin_w/_nh_asian` ==
    3, "non-hispanic_white", fifelse(`race/hispanic_origin_w/_nh_asian` ==
    4, "non-hispanic_black", fifelse(`race/hispanic_origin_w/_nh_asian` ==
    6, "non-hispanic_asian", "other_race_incl_multiracial"))))))]

demographic[, `:=`(six_month_time_period, fifelse(six_month_time_period ==
    1, "nov1-apr30", fifelse(six_month_time_period == 2, "may1-oct31",
    NA_character_)))]

demographic[, `:=`(country_of_birth, fifelse(country_of_birth ==
    1, "united_states", fifelse(country_of_birth == 2, "other",
    fifelse(country_of_birth == 77, "refused", "dont_know"))))]

demographic[, `:=`(length_of_time_in_us, fifelse(length_of_time_in_us ==
    1, "less_than_5", fifelse(length_of_time_in_us == 2, "between_5_and_15",
    fifelse(length_of_time_in_us == 3, "between_15_and_30", fifelse(length_of_time_in_us ==
        4, "30_or_more", fifelse(length_of_time_in_us == 77,
        "refused", fifelse(length_of_time_in_us == 99, "dont_know",
            NA_character_)))))))]

demographic[, `:=`(pregnancy_status_at_exam, fifelse(pregnancy_status_at_exam ==
    1, "pregnant", fifelse(pregnancy_status_at_exam == 2, "not_pregnant",
    fifelse(pregnancy_status_at_exam == 3, "cannot_ascertain",
        NA_character_))))]
```

``` r
demographic[, `:=`(age_category, fifelse(age_in_years_at_screening %between%
    c(1, 3), "1-3", fifelse(age_in_years_at_screening %between%
    c(4, 8), "4-8", fifelse(age_in_years_at_screening %between%
    c(9, 13), "9-13", fifelse(age_in_years_at_screening %between%
    c(14, 18), "14-18", fifelse(age_in_years_at_screening %between%
    c(19, 30), "19-30", fifelse(age_in_years_at_screening %between%
    c(31, 50), "31-50", fifelse(age_in_years_at_screening %between%
    c(51, 70), "51-70", "70+"))))))))]
```

Categorize the answers in the food survey.

``` r
fs_d1[, `:=`(intake_day_of_the_week, fifelse(intake_day_of_the_week ==
    1, "Sunday", fifelse(intake_day_of_the_week == 2, "Monday",
    fifelse(intake_day_of_the_week == 3, "Tuesday", fifelse(intake_day_of_the_week ==
        4, "Wednesday", fifelse(intake_day_of_the_week == 5,
        "Thursday", fifelse(intake_day_of_the_week == 6, "Friday",
            "Saturday")))))))]
```
