PM566 Midterm Project
================
Flemming Wu
2022-10-10

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
library(dtplyr)
library(httr)
library(xml2)
library(rvest)
```

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
fs_d1_labels <- gsub(" ", "_", fs_d1_labels$label)
fs_d1_labels <- gsub("\\(|\\)", "", fs_d1_labels) %>%
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

Create age category variable for demographic table.

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

Categorize the answers in the food survey. Need day of the week, name of
eating occasion, source of food, and if the food was eaten at home
categorized.

``` r
fs_d1[, `:=`(intake_day_of_the_week, fifelse(intake_day_of_the_week ==
    1, "Sunday", fifelse(intake_day_of_the_week == 2, "Monday",
    fifelse(intake_day_of_the_week == 3, "Tuesday", fifelse(intake_day_of_the_week ==
        4, "Wednesday", fifelse(intake_day_of_the_week == 5,
        "Thursday", fifelse(intake_day_of_the_week == 6, "Friday",
            "Saturday")))))))]

# table(fs_d1$`did_you_eat_this_meal_at_home?`) #1, 2, and
# 9 are the only answers
fs_d1[, `:=`(`did_you_eat_this_meal_at_home?`, fifelse(`did_you_eat_this_meal_at_home?` ==
    1, "yes", fifelse(`did_you_eat_this_meal_at_home?` == 2,
    "no", "dont_know")))]
```

``` r
# for the next categorization steps, I will be reading in
# tables from the CDC website because they contain many
# possible values

# categorize the name of eating occasion read in name of
# eating occasion table from CDC website using html and
# full Xpath
eating_occasion <- read_html("https://wwwn.cdc.gov/NCHS/nhanes/2017-2018/P_DR1IFF.htm#DR1DAY") %>%
    html_nodes(xpath = "/html/body/div[2]/div[4]/div[15]/table") %>%
    html_table() %>%
    as.data.frame() %>%
    select(Code.or.Value, Value.Description)

# merge table into original data table
fs_d1 <- merge(x = eating_occasion, y = fs_d1, by.x = "Code.or.Value",
    by.y = "name_of_eating_occasion")

# the merge replaced the old column
# 'name_of_eating_occasion' with 'Code.or.Value' from the
# new table I only need the 'Value.Description' column so
# rename it and remove 'Code.or.Value' column
setnames(fs_d1, "Value.Description", "eating_occasion")
fs_d1 <- fs_d1[-c(1)]

######################################

# categorize source of food
source_of_food <- read_html("https://wwwn.cdc.gov/NCHS/nhanes/2017-2018/P_DR1IFF.htm#DR1DAY") %>%
    html_nodes(xpath = "/html/body/div[2]/div[4]/div[16]/table") %>%
    html_table() %>%
    as.data.frame() %>%
    select(Code.or.Value, Value.Description)

fs_d1 <- merge(x = source_of_food, y = fs_d1, by.x = "Code.or.Value",
    by.y = "source_of_food")

setnames(fs_d1, "Value.Description", "food_source")
fs_d1 <- fs_d1[-c(1)]
```

``` r
str(fs_d1)
```

    ## 'data.frame':    171744 obs. of  30 variables:
    ##  $ food_source                          : chr  "Store - grocery/supermarket" "Store - grocery/supermarket" "Store - grocery/supermarket" "Store - grocery/supermarket" ...
    ##  $ eating_occasion                      : chr  "Breakfast" "Breakfast" "Desayano" "Breakfast" ...
    ##  $ respondent_sequence_number           : num  122474 119677 123644 112441 109587 ...
    ##  $ dietary_day_one_sample_weight        : num  12511 3463 6272 4751 36013 ...
    ##  $ dietary_two-day_sample_weight        : num  15417 2762 6218 0 35149 ...
    ##  $ food/individual_component_number     : num  1 3 2 2 7 2 4 8 1 1 ...
    ##  $ dietary_recall_status                : num  1 1 1 1 1 1 1 1 1 1 ...
    ##  $ interviewer_id_code                  : num  89 81 90 86 91 86 88 81 91 81 ...
    ##  $ breast-fed_infant_either_day         : num  2 2 2 2 2 2 2 2 2 2 ...
    ##  $ number_of_days_of_intake             : num  2 2 2 1 2 2 2 2 2 1 ...
    ##  $ #_of_days_b/w_intake_and_hh_interview: num  1 13 33 NA -2 -6 43 23 -1 15 ...
    ##  $ intake_day_of_the_week               : chr  "Monday" "Saturday" "Tuesday" "Saturday" ...
    ##  $ language_respondent_used_mostly      : num  1 1 2 1 1 1 1 2 1 1 ...
    ##  $ combination_food_number              : num  0 0 1 1 3 1 2 0 1 1 ...
    ##  $ combination_food_type                : num  0 0 1 2 90 1 1 0 1 2 ...
    ##  $ time_of_eating_occasion_hh:mm        : 'hms' num  07:00:00 09:00:00 08:30:00 08:30:00 ...
    ##   ..- attr(*, "units")= chr "secs"
    ##  $ did_you_eat_this_meal_at_home?       : chr  "no" "yes" "yes" "yes" ...
    ##  $ usda_food_code                       : num  22600200 54325000 12210210 11112210 63200100 ...
    ##  $ grams                                : num  32 18 60 122 150 ...
    ##  $ energy_kcal                          : num  155 75 151 52 62 2 48 382 4 297 ...
    ##  $ protein_gm                           : num  11.97 1.7 0.41 4.12 1.01 ...
    ##  $ carbohydrate_gm                      : num  0.61 13.33 21.04 6.33 14.53 ...
    ##  $ total_sugars_gm                      : num  0.5 0.23 19.82 6.05 10.09 ...
    ##  $ dietary_fiber_gm                     : num  0 0.5 0.7 0 3 0 0 2.4 0 4.9 ...
    ##  $ total_fat_gm                         : num  11.45 1.56 8.1 1.16 0.38 ...
    ##  $ total_saturated_fatty_acids_gm       : num  3.93 0.298 1.581 0.693 0.012 ...
    ##  $ total_monounsaturated_fatty_acids_gm : num  5.013 0.357 2.401 0.256 0.021 ...
    ##  $ total_polyunsaturated_fatty_acids_gm : num  1.938 0.87 3.761 0.039 0.066 ...
    ##  $ cholesterol_mg                       : num  32 0 0 6 0 0 0 44 0 0 ...
    ##  $ vitamin_e_as_alpha-tocopherol_mg     : num  0.13 0.21 0.95 0.02 0.56 0.02 0 0.21 0 1.55 ...

``` r
str(demographic)
```

    ## Classes 'data.table' and 'data.frame':   15560 obs. of  15 variables:
    ##  $ respondent_sequence_number              : num  109263 109264 109265 109266 109267 ...
    ##   ..- attr(*, "label")= chr "Respondent sequence number"
    ##  $ data_release_cycle                      : num  66 66 66 66 66 66 66 66 66 66 ...
    ##   ..- attr(*, "label")= chr "Data release cycle"
    ##  $ interview/examination_status            : chr  "interview_and_mec_examined" "interview_and_mec_examined" "interview_and_mec_examined" "interview_and_mec_examined" ...
    ##  $ gender                                  : chr  "male" "female" "male" "female" ...
    ##  $ age_in_years_at_screening               : num  2 13 2 29 21 18 2 11 49 0 ...
    ##   ..- attr(*, "label")= chr "Age in years at screening"
    ##  $ age_in_months_at_screening_-_0_to_24_mos: num  NA NA NA NA NA NA NA NA NA 3 ...
    ##   ..- attr(*, "label")= chr "Age in months at screening - 0 to 24 mos"
    ##  $ race/hispanic_origin                    : chr  "other_race_incl_multiracial" "mexican_american" "non-hispanic_white" "other_race_incl_multiracial" ...
    ##  $ race/hispanic_origin_w/_nh_asian        : chr  "non-hispanic_asian" "mexican_american" "non-hispanic_white" "non-hispanic_asian" ...
    ##  $ six_month_time_period                   : chr  "may1-oct31" "may1-oct31" "may1-oct31" "may1-oct31" ...
    ##  $ country_of_birth                        : chr  "united_states" "united_states" "united_states" "other" ...
    ##  $ length_of_time_in_us                    : chr  NA NA NA "between_5_and_15" ...
    ##  $ education_level_-_adults_20+            : num  NA NA NA 5 4 NA NA NA 2 NA ...
    ##   ..- attr(*, "label")= chr "Education level - Adults 20+"
    ##  $ marital_status                          : num  NA NA NA 3 3 NA NA NA 3 NA ...
    ##   ..- attr(*, "label")= chr "Marital status"
    ##  $ pregnancy_status_at_exam                : chr  NA NA NA "not_pregnant" ...
    ##  $ age_category                            : chr  "1-3" "9-13" "1-3" "19-30" ...
    ##  - attr(*, ".internal.selfref")=<externalptr>

### Preliminary Results

Check key variables and provide summary statistics in tabular form.

``` r
quantile(fs_d1$total_sugars_gm, seq(0, 1, 0.1))
```

    ##      0%     10%     20%     30%     40%     50%     60%     70%     80%     90% 
    ##   0.000   0.000   0.050   0.260   0.770   1.825   3.660   6.830  12.200  21.570 
    ##    100% 
    ## 690.230

#### What time and/or day of the week do people generally eat foods high in sugar/saturated fats?

Plot sugar consumption versus time.

``` r
# make names for food survey table and convert back to data
# table
names(fs_d1) <- make.names(names(fs_d1))
fs_d1 <- as.data.table(fs_d1)

table(fs_d1$time_of_eating_occasion_hh.mm)
```

    ## 
    ## 00:00:00 00:01:00 00:05:00 00:10:00 00:15:00 00:20:00 00:30:00 00:40:00 
    ##      318        1        5        4       18        1       97        3 
    ## 00:45:00 01:00:00 01:15:00 01:30:00 02:00:00 02:20:00 02:30:00 03:00:00 
    ##        5      284        1       77      274        2       73      246 
    ## 03:30:00 03:45:00 03:50:00 04:00:00 04:15:00 04:20:00 04:23:00 04:30:00 
    ##       85        5        3      242       31        2        2      192 
    ## 04:32:00 04:40:00 04:45:00 04:50:00 05:00:00 05:05:00 05:10:00 05:15:00 
    ##        3        2       42        2      568        2        7       28 
    ## 05:20:00 05:25:00 05:30:00 05:32:00 05:35:00 05:40:00 05:45:00 05:50:00 
    ##        8        6      516        2        9       22       93       18 
    ## 05:51:00 06:00:00 06:03:00 06:07:00 06:10:00 06:15:00 06:16:00 06:20:00 
    ##        2     1543        1        1       10      141        9       24 
    ## 06:24:00 06:25:00 06:30:00 06:35:00 06:40:00 06:45:00 06:47:00 06:50:00 
    ##        1       13     1331        4       63      232        2       73 
    ## 06:55:00 07:00:00 07:01:00 07:02:00 07:05:00 07:06:00 07:07:00 07:10:00 
    ##        3     4082        2        4       10        1        2       57 
    ## 07:12:00 07:13:00 07:14:00 07:15:00 07:18:00 07:20:00 07:21:00 07:25:00 
    ##       11        7       11      337        7      130        5       12 
    ## 07:28:00 07:30:00 07:33:00 07:35:00 07:36:00 07:38:00 07:39:00 07:40:00 
    ##        3     2897        2       20        4       12        4      117 
    ## 07:43:00 07:45:00 07:46:00 07:48:00 07:50:00 07:54:00 07:55:00 07:59:00 
    ##        3      398        3        3       73        5        4        1 
    ## 08:00:00 08:05:00 08:07:00 08:10:00 08:12:00 08:15:00 08:20:00 08:21:00 
    ##     6172       11        2       64        3      373       86        3 
    ## 08:25:00 08:27:00 08:30:00 08:35:00 08:37:00 08:39:00 08:40:00 08:45:00 
    ##        5        3     3398       13        5        4       78      250 
    ## 08:46:00 08:50:00 09:00:00 09:04:00 09:05:00 09:10:00 09:12:00 09:15:00 
    ##        1       39     6407        2        7       27        1      184 
    ## 09:16:00 09:17:00 09:18:00 09:20:00 09:22:00 09:23:00 09:25:00 09:27:00 
    ##        1        6        4       48        2        7        3        4 
    ## 09:30:00 09:35:00 09:40:00 09:45:00 09:50:00 09:52:00 09:53:00 09:55:00 
    ##     2442        9       59      129       17        1        1        4 
    ## 10:00:00 10:01:00 10:05:00 10:10:00 10:15:00 10:17:00 10:20:00 10:25:00 
    ##     5801        1       14       13      152        2       30       10 
    ## 10:30:00 10:32:00 10:35:00 10:40:00 10:43:00 10:45:00 10:46:00 10:50:00 
    ##     2139        5       19       41        3      252        2       63 
    ## 10:55:00 10:56:00 11:00:00 11:03:00 11:05:00 11:06:00 11:07:00 11:08:00 
    ##       13        3     5327        2       44        4        4        7 
    ## 11:10:00 11:11:00 11:12:00 11:14:00 11:15:00 11:16:00 11:20:00 11:21:00 
    ##       41        6        6        1      316        1       94        2 
    ## 11:24:00 11:25:00 11:28:00 11:29:00 11:30:00 11:32:00 11:35:00 11:36:00 
    ##        4       58        9        3     3311        6       42        8 
    ## 11:37:00 11:38:00 11:39:00 11:40:00 11:41:00 11:45:00 11:47:00 11:50:00 
    ##        2        7        4      144        3      710        7      105 
    ## 11:54:00 11:55:00 12:00:00 12:01:00 12:02:00 12:03:00 12:04:00 12:05:00 
    ##        7       31     9560        7        7       14       10       41 
    ## 12:08:00 12:10:00 12:11:00 12:12:00 12:13:00 12:14:00 12:15:00 12:16:00 
    ##        5      110        7        4        9        2      583        6 
    ## 12:17:00 12:18:00 12:19:00 12:20:00 12:24:00 12:25:00 12:27:00 12:30:00 
    ##        3        3        5      148        5       54        4     4549 
    ## 12:31:00 12:32:00 12:33:00 12:34:00 12:35:00 12:36:00 12:38:00 12:40:00 
    ##       11        7        4        2       48        3        1      124 
    ## 12:42:00 12:45:00 12:46:00 12:47:00 12:49:00 12:50:00 12:52:00 12:55:00 
    ##        2      415        5       18        2       52        4        6 
    ## 12:56:00 13:00:00 13:02:00 13:03:00 13:05:00 13:09:00 13:10:00 13:15:00 
    ##        4     7223        5        1        2        3       35      275 
    ## 13:18:00 13:20:00 13:22:00 13:23:00 13:24:00 13:25:00 13:27:00 13:30:00 
    ##        4       78        3        1        1        1        5     3010 
    ## 13:35:00 13:36:00 13:37:00 13:39:00 13:40:00 13:43:00 13:45:00 13:50:00 
    ##        9        1        2        1       57        1      159       12 
    ## 13:53:00 13:55:00 13:57:00 14:00:00 14:03:00 14:05:00 14:10:00 14:12:00 
    ##        2        1        1     6413        2       12        7        7 
    ## 14:14:00 14:15:00 14:17:00 14:20:00 14:23:00 14:30:00 14:32:00 14:33:00 
    ##        1      157        7       26        1     2001        1        3 
    ## 14:35:00 14:40:00 14:43:00 14:45:00 14:50:00 14:57:00 15:00:00 15:05:00 
    ##        9       30        4      105        9        3     5835        6 
    ## 15:06:00 15:08:00 15:10:00 15:11:00 15:15:00 15:18:00 15:20:00 15:21:00 
    ##        4        6       18        1      110        4       46        1 
    ## 15:25:00 15:30:00 15:32:00 15:38:00 15:40:00 15:41:00 15:42:00 15:45:00 
    ##        7     1845        1        1       61        3        2      162 
    ## 15:49:00 15:50:00 15:55:00 16:00:00 16:02:00 16:03:00 16:05:00 16:10:00 
    ##        1       15        1     5517        2        1        5       14 
    ## 16:11:00 16:13:00 16:15:00 16:20:00 16:21:00 16:22:00 16:25:00 16:26:00 
    ##        4        2      171       28        1        5        5        8 
    ## 16:30:00 16:35:00 16:36:00 16:38:00 16:40:00 16:45:00 16:50:00 16:54:00 
    ##     1957       16        4        1       43      170       23        1 
    ## 16:55:00 16:57:00 17:00:00 17:04:00 17:05:00 17:10:00 17:15:00 17:16:00 
    ##        9        2     7060        2        2        8      257        1 
    ## 17:20:00 17:25:00 17:30:00 17:35:00 17:36:00 17:40:00 17:45:00 17:49:00 
    ##       48        7     3833        3       15       48      335        1 
    ## 17:50:00 17:54:00 17:57:00 18:00:00 18:05:00 18:06:00 18:08:00 18:10:00 
    ##        8        4        3     9318        5        6        1       35 
    ## 18:12:00 18:15:00 18:20:00 18:25:00 18:30:00 18:32:00 18:35:00 18:40:00 
    ##        4      358       82        8     4957        1        6       65 
    ## 18:45:00 18:50:00 19:00:00 19:01:00 19:05:00 19:06:00 19:10:00 19:12:00 
    ##      455       23     9035        1        5        3       18        1 
    ## 19:14:00 19:15:00 19:20:00 19:23:00 19:25:00 19:30:00 19:35:00 19:37:00 
    ##        3      227       32        1       10     4217        3        1 
    ## 19:38:00 19:40:00 19:42:00 19:45:00 19:48:00 19:50:00 19:51:00 19:59:00 
    ##        1       62        3      315        4       29        1        2 
    ## 20:00:00 20:01:00 20:04:00 20:05:00 20:10:00 20:12:00 20:13:00 20:15:00 
    ##     7708        2        2        3       19        1        3      190 
    ## 20:17:00 20:18:00 20:20:00 20:21:00 20:23:00 20:25:00 20:29:00 20:30:00 
    ##       18       16       33        1        4       10        3     3051 
    ## 20:35:00 20:40:00 20:45:00 20:47:00 20:48:00 20:50:00 20:51:00 20:55:00 
    ##        7       37      203        2        1       18        5        6 
    ## 21:00:00 21:02:00 21:05:00 21:10:00 21:15:00 21:16:00 21:20:00 21:23:00 
    ##     5108        2        8        5      142        2       50        1 
    ## 21:26:00 21:30:00 21:40:00 21:45:00 21:50:00 21:51:00 21:53:00 21:55:00 
    ##        1     1822       33      141       23        4        2        2 
    ## 21:59:00 22:00:00 22:03:00 22:05:00 22:07:00 22:10:00 22:11:00 22:14:00 
    ##        1     3229        1        5        1        6        1        2 
    ## 22:15:00 22:20:00 22:22:00 22:25:00 22:30:00 22:40:00 22:42:00 22:45:00 
    ##       57       33        1        2      918       15        2       75 
    ## 22:48:00 22:50:00 22:55:00 23:00:00 23:04:00 23:05:00 23:10:00 23:12:00 
    ##        1       16        3     1551        1        3        7        1 
    ## 23:15:00 23:16:00 23:20:00 23:25:00 23:28:00 23:30:00 23:31:00 23:37:00 
    ##       72        2       36        6        1      511        4        1 
    ## 23:40:00 23:45:00 23:49:00 23:50:00 23:52:00 23:55:00 23:58:00 23:59:00 
    ##       13      110        6       14        1        6        1       19

``` r
fs_d1[, mean(total_sugars_gm), by = "time_of_eating_occasion_hh.mm"] %>%
    ggplot() + geom_line(aes(x = time_of_eating_occasion_hh.mm,
    y = V1))
```

![](README_files/figure-gfm/plot%20sugar%20consumption%20against%20time-1.png)<!-- -->
