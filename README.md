PM566 Midterm Project
================
Flemming Wu
2022-10-11

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
names(fs_d1) <- make.names(names(fs_d1))

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
# create new column for this so it can still be ordered
fs_d1[, `:=`(intake_day_cat, fifelse(intake_day_of_the_week ==
    1, "Sunday", fifelse(intake_day_of_the_week == 2, "Monday",
    fifelse(intake_day_of_the_week == 3, "Tuesday", fifelse(intake_day_of_the_week ==
        4, "Wednesday", fifelse(intake_day_of_the_week == 5,
        "Thursday", fifelse(intake_day_of_the_week == 6, "Friday",
            "Saturday")))))))]

# table(fs_d1$`did_you_eat_this_meal_at_home?`) #1, 2, and
# 9 are the only answers
fs_d1[, `:=`(did_you_eat_this_meal_at_home., fifelse(did_you_eat_this_meal_at_home. ==
    1, "yes", fifelse(did_you_eat_this_meal_at_home. == 2, "no",
    "dont_know")))]
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

### Preliminary Results

#### Exploratory Data Analysis

Check dimensions, headers, and footers

``` r
# First merge all the data tables into one

df <- merge(x = merge(x = fs_d1, y = demographic, by.x = "respondent_sequence_number",
    by.y = "respondent_sequence_number"), y = food_categories,
    by.x = "usda_food_code", by.y = "food_code")

df <- as.data.table(df)
```

``` r
dim(df)
```

    ## [1] 171744     47

``` r
head(df)
```

    ##    usda_food_code respondent_sequence_number                 food_source
    ## 1:       11100000                     111646 Store - grocery/supermarket
    ## 2:       11100000                     123605  Cafeteria in a K-12 school
    ## 3:       11100000                     124271  Restaurant fast food/pizza
    ## 4:       11100000                     117655 Store - grocery/supermarket
    ## 5:       11100000                     113672 Store - grocery/supermarket
    ## 6:       11100000                     110233      From someone else/gift
    ##    eating_occasion dietary_day_one_sample_weight dietary_two.day_sample_weight
    ## 1:           Snack                      72694.03                     76938.883
    ## 2:           Lunch                      15028.82                     15574.121
    ## 3:           Snack                      28329.94                     22901.447
    ## 4:       Breakfast                      13925.45                     11752.330
    ## 5:       Breakfast                      33181.05                     33630.143
    ## 6:           Snack                      10796.28                      9495.814
    ##    food.individual_component_number dietary_recall_status interviewer_id_code
    ## 1:                                6                     1                  49
    ## 2:                                5                     1                  86
    ## 3:                                7                     1                  73
    ## 4:                                2                     1                  73
    ## 5:                                2                     1                  88
    ## 6:                                2                     1                  81
    ##    breast.fed_infant_either_day number_of_days_of_intake
    ## 1:                            2                        2
    ## 2:                            2                        2
    ## 3:                            2                        2
    ## 4:                            2                        2
    ## 5:                            2                        2
    ## 6:                            2                        2
    ##    X._of_days_b.w_intake_and_hh_interview intake_day_of_the_week
    ## 1:                                      1                      3
    ## 2:                                     22                      4
    ## 3:                                      7                      6
    ## 4:                                     12                      1
    ## 5:                                     NA                      5
    ## 6:                                      0                      7
    ##    language_respondent_used_mostly combination_food_number
    ## 1:                               1                       3
    ## 2:                               3                       0
    ## 3:                               1                       5
    ## 4:                               1                       5
    ## 5:                               2                       1
    ## 6:                               1                       0
    ##    combination_food_type time_of_eating_occasion_hh.mm
    ## 1:                     1                      17:00:00
    ## 2:                     0                      11:15:00
    ## 3:                     1                      14:30:00
    ## 4:                     2                      09:20:00
    ## 5:                     1                      10:00:00
    ## 6:                     0                      11:00:00
    ##    did_you_eat_this_meal_at_home.  grams energy_kcal protein_gm carbohydrate_gm
    ## 1:                            yes  10.00           5       0.33            0.49
    ## 2:                             no 244.00         123       8.15           11.89
    ## 3:                            yes  28.75          15       0.96            1.40
    ## 4:                            yes 213.51         108       7.13           10.40
    ## 5:                            yes  91.50          46       3.06            4.46
    ## 6:                             no 244.00         123       8.15           11.89
    ##    total_sugars_gm dietary_fiber_gm total_fat_gm total_saturated_fatty_acids_gm
    ## 1:            0.49                0         0.20                          0.116
    ## 2:           11.94                0         4.86                          2.840
    ## 3:            1.41                0         0.57                          0.335
    ## 4:           10.45                0         4.26                          2.485
    ## 5:            4.48                0         1.82                          1.065
    ## 6:           11.94                0         4.86                          2.840
    ##    total_monounsaturated_fatty_acids_gm total_polyunsaturated_fatty_acids_gm
    ## 1:                                0.043                                0.007
    ## 2:                                1.039                                0.159
    ## 3:                                0.122                                0.019
    ## 4:                                0.910                                0.139
    ## 5:                                0.390                                0.059
    ## 6:                                1.039                                0.159
    ##    cholesterol_mg vitamin_e_as_alpha.tocopherol_mg intake_day_cat
    ## 1:              1                             0.00        Tuesday
    ## 2:             20                             0.08      Wednesday
    ## 3:              2                             0.01         Friday
    ## 4:             18                             0.07         Sunday
    ## 5:              8                             0.03       Thursday
    ## 6:             20                             0.08       Saturday
    ##    data_release_cycle interview/examination_status gender
    ## 1:                 66   interview_and_mec_examined female
    ## 2:                 66   interview_and_mec_examined female
    ## 3:                 66   interview_and_mec_examined female
    ## 4:                 66   interview_and_mec_examined   male
    ## 5:                 66   interview_and_mec_examined female
    ## 6:                 66   interview_and_mec_examined   male
    ##    age_in_years_at_screening age_in_months_at_screening_-_0_to_24_mos
    ## 1:                        30                                       NA
    ## 2:                        10                                       NA
    ## 3:                        63                                       NA
    ## 4:                        15                                       NA
    ## 5:                        54                                       NA
    ## 6:                         5                                       NA
    ##           race/hispanic_origin race/hispanic_origin_w/_nh_asian
    ## 1: other_race_incl_multiracial               non-hispanic_asian
    ## 2:            mexican_american                 mexican_american
    ## 3:          non-hispanic_white               non-hispanic_white
    ## 4:          non-hispanic_black               non-hispanic_black
    ## 5:            mexican_american                 mexican_american
    ## 6:          non-hispanic_white               non-hispanic_white
    ##    six_month_time_period country_of_birth length_of_time_in_us
    ## 1:            nov1-apr30            other          less_than_5
    ## 2:            may1-oct31    united_states                 <NA>
    ## 3:            nov1-apr30    united_states                 <NA>
    ## 4:            nov1-apr30    united_states                 <NA>
    ## 5:            may1-oct31            other    between_15_and_30
    ## 6:            nov1-apr30    united_states                 <NA>
    ##    education_level_-_adults_20+ marital_status pregnancy_status_at_exam
    ## 1:                            5              1             not_pregnant
    ## 2:                           NA             NA                     <NA>
    ## 3:                            5              2                     <NA>
    ## 4:                           NA             NA                     <NA>
    ## 5:                            1              3                     <NA>
    ## 6:                           NA             NA                     <NA>
    ##    age_category short_food_code_description long_food_code_description
    ## 1:        19-30                   MILK, NFS                  Milk, NFS
    ## 2:         9-13                   MILK, NFS                  Milk, NFS
    ## 3:        51-70                   MILK, NFS                  Milk, NFS
    ## 4:        14-18                   MILK, NFS                  Milk, NFS
    ## 5:        51-70                   MILK, NFS                  Milk, NFS
    ## 6:          4-8                   MILK, NFS                  Milk, NFS

``` r
tail(df)
```

    ##    usda_food_code respondent_sequence_number                 food_source
    ## 1:       95342000                     118682  Restaurant fast food/pizza
    ## 2:       95342000                     121978    Store - convenience type
    ## 3:       95342000                     124594 Store - grocery/supermarket
    ## 4:       95342000                     120913 Store - grocery/supermarket
    ## 5:       95342000                     111219  Restaurant fast food/pizza
    ## 6:       95342000                     124594 Store - grocery/supermarket
    ##    eating_occasion dietary_day_one_sample_weight dietary_two.day_sample_weight
    ## 1:           Snack                     14836.302                     11571.881
    ## 2:           Lunch                     24661.067                     26024.486
    ## 3:           Lunch                      3541.713                      7509.941
    ## 4:       Breakfast                      8515.995                     11167.895
    ## 5:           Lunch                      9635.711                      8366.493
    ## 6:           Snack                      3541.713                      7509.941
    ##    food.individual_component_number dietary_recall_status interviewer_id_code
    ## 1:                                5                     1                  89
    ## 2:                                9                     1                  81
    ## 3:                                2                     1                  88
    ## 4:                                1                     1                  90
    ## 5:                                5                     1                  91
    ## 6:                                3                     1                  88
    ##    breast.fed_infant_either_day number_of_days_of_intake
    ## 1:                            2                        2
    ## 2:                            2                        2
    ## 3:                            2                        2
    ## 4:                            2                        2
    ## 5:                            2                        2
    ## 6:                            2                        2
    ##    X._of_days_b.w_intake_and_hh_interview intake_day_of_the_week
    ## 1:                                     -1                      6
    ## 2:                                      7                      5
    ## 3:                                     15                      6
    ## 4:                                     13                      3
    ## 5:                                      3                      6
    ## 6:                                     15                      6
    ##    language_respondent_used_mostly combination_food_number
    ## 1:                               1                       2
    ## 2:                               1                       0
    ## 3:                               1                       0
    ## 4:                               1                       0
    ## 5:                               1                       3
    ## 6:                               1                       0
    ##    combination_food_type time_of_eating_occasion_hh.mm
    ## 1:                     1                      10:30:00
    ## 2:                     0                      12:00:00
    ## 3:                     0                      12:30:00
    ## 4:                     0                      08:00:00
    ## 5:                    10                      13:00:00
    ## 6:                     0                      13:00:00
    ##    did_you_eat_this_meal_at_home.  grams energy_kcal protein_gm carbohydrate_gm
    ## 1:                             no 185.54         115       1.54           23.80
    ## 2:                             no 496.00         308       4.12           63.64
    ## 3:                            yes 325.50         202       2.70           41.76
    ## 4:                             no  46.50          29       0.39            5.97
    ## 5:                             no  54.25          34       0.45            6.96
    ## 6:                            yes 325.50         202       2.70           41.76
    ##    total_sugars_gm dietary_fiber_gm total_fat_gm total_saturated_fatty_acids_gm
    ## 1:           20.59              2.2         1.54                              0
    ## 2:           55.06              6.0         4.12                              0
    ## 3:           36.13              3.9         2.70                              0
    ## 4:            5.16              0.6         0.39                              0
    ## 5:            6.02              0.7         0.45                              0
    ## 6:           36.13              3.9         2.70                              0
    ##    total_monounsaturated_fatty_acids_gm total_polyunsaturated_fatty_acids_gm
    ## 1:                                    0                                    0
    ## 2:                                    0                                    0
    ## 3:                                    0                                    0
    ## 4:                                    0                                    0
    ## 5:                                    0                                    0
    ## 6:                                    0                                    0
    ##    cholesterol_mg vitamin_e_as_alpha.tocopherol_mg intake_day_cat
    ## 1:              0                            20.61         Friday
    ## 2:              0                            55.11       Thursday
    ## 3:              0                            36.16         Friday
    ## 4:              0                             5.17        Tuesday
    ## 5:              0                             6.03         Friday
    ## 6:              0                            36.16         Friday
    ##    data_release_cycle interview/examination_status gender
    ## 1:                 66   interview_and_mec_examined female
    ## 2:                 66   interview_and_mec_examined   male
    ## 3:                 66   interview_and_mec_examined female
    ## 4:                 66   interview_and_mec_examined   male
    ## 5:                 66   interview_and_mec_examined female
    ## 6:                 66   interview_and_mec_examined female
    ##    age_in_years_at_screening age_in_months_at_screening_-_0_to_24_mos
    ## 1:                        40                                       NA
    ## 2:                        52                                       NA
    ## 3:                        18                                       NA
    ## 4:                         2                                       NA
    ## 5:                        31                                       NA
    ## 6:                        18                                       NA
    ##           race/hispanic_origin race/hispanic_origin_w/_nh_asian
    ## 1: other_race_incl_multiracial      other_race_incl_multiracial
    ## 2: other_race_incl_multiracial      other_race_incl_multiracial
    ## 3:          non-hispanic_black               non-hispanic_black
    ## 4: other_race_incl_multiracial      other_race_incl_multiracial
    ## 5:            mexican_american                 mexican_american
    ## 6:          non-hispanic_black               non-hispanic_black
    ##    six_month_time_period country_of_birth length_of_time_in_us
    ## 1:            may1-oct31    united_states                 <NA>
    ## 2:            nov1-apr30    united_states                 <NA>
    ## 3:            nov1-apr30    united_states                 <NA>
    ## 4:            nov1-apr30    united_states                 <NA>
    ## 5:            may1-oct31    united_states                 <NA>
    ## 6:            nov1-apr30    united_states                 <NA>
    ##    education_level_-_adults_20+ marital_status pregnancy_status_at_exam
    ## 1:                            5              3             not_pregnant
    ## 2:                            4              2                     <NA>
    ## 3:                           NA             NA                     <NA>
    ## 4:                           NA             NA                     <NA>
    ## 5:                            5              1             not_pregnant
    ## 6:                           NA             NA                     <NA>
    ##    age_category short_food_code_description long_food_code_description
    ## 1:        31-50     FRUIT JUICE, ACAI BLEND    Fruit juice, acai blend
    ## 2:        51-70     FRUIT JUICE, ACAI BLEND    Fruit juice, acai blend
    ## 3:        14-18     FRUIT JUICE, ACAI BLEND    Fruit juice, acai blend
    ## 4:          1-3     FRUIT JUICE, ACAI BLEND    Fruit juice, acai blend
    ## 5:        31-50     FRUIT JUICE, ACAI BLEND    Fruit juice, acai blend
    ## 6:        14-18     FRUIT JUICE, ACAI BLEND    Fruit juice, acai blend

Check variable types

``` r
str(df)
```

    ## Classes 'data.table' and 'data.frame':   171744 obs. of  47 variables:
    ##  $ usda_food_code                          : num  11100000 11100000 11100000 11100000 11100000 11100000 11100000 11100000 11100000 11100000 ...
    ##  $ respondent_sequence_number              : num  111646 123605 124271 117655 113672 ...
    ##  $ food_source                             : chr  "Store - grocery/supermarket" "Cafeteria in a K-12 school" "Restaurant fast food/pizza" "Store - grocery/supermarket" ...
    ##  $ eating_occasion                         : chr  "Snack" "Lunch" "Snack" "Breakfast" ...
    ##  $ dietary_day_one_sample_weight           : num  72694 15029 28330 13925 33181 ...
    ##  $ dietary_two.day_sample_weight           : num  76939 15574 22901 11752 33630 ...
    ##  $ food.individual_component_number        : num  6 5 7 2 2 2 1 3 11 4 ...
    ##  $ dietary_recall_status                   : num  1 1 1 1 1 1 1 1 1 1 ...
    ##  $ interviewer_id_code                     : num  49 86 73 73 88 81 88 49 73 86 ...
    ##  $ breast.fed_infant_either_day            : num  2 2 2 2 2 2 2 2 2 2 ...
    ##  $ number_of_days_of_intake                : num  2 2 2 2 2 2 2 1 2 2 ...
    ##  $ X._of_days_b.w_intake_and_hh_interview  : num  1 22 7 12 NA 0 -5 13 8 -22 ...
    ##  $ intake_day_of_the_week                  : num  3 4 6 1 5 7 7 7 5 5 ...
    ##  $ language_respondent_used_mostly         : num  1 3 1 1 2 1 1 1 1 2 ...
    ##  $ combination_food_number                 : num  3 0 5 5 1 0 0 1 3 1 ...
    ##  $ combination_food_type                   : num  1 0 1 2 1 0 0 1 2 1 ...
    ##  $ time_of_eating_occasion_hh.mm           : 'hms' num  17:00:00 11:15:00 14:30:00 09:20:00 ...
    ##   ..- attr(*, "units")= chr "secs"
    ##  $ did_you_eat_this_meal_at_home.          : chr  "yes" "no" "yes" "yes" ...
    ##  $ grams                                   : num  10 244 28.8 213.5 91.5 ...
    ##  $ energy_kcal                             : num  5 123 15 108 46 123 111 11 62 32 ...
    ##  $ protein_gm                              : num  0.33 8.15 0.96 7.13 3.06 8.15 7.11 0.71 4.07 2.03 ...
    ##  $ carbohydrate_gm                         : num  0.49 11.89 1.4 10.4 4.46 ...
    ##  $ total_sugars_gm                         : num  0.49 11.94 1.41 10.45 4.48 ...
    ##  $ dietary_fiber_gm                        : num  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ total_fat_gm                            : num  0.2 4.86 0.57 4.26 1.82 4.86 4.57 0.42 2.43 1.31 ...
    ##  $ total_saturated_fatty_acids_gm          : num  0.116 2.84 0.335 2.485 1.065 ...
    ##  $ total_monounsaturated_fatty_acids_gm    : num  0.043 1.039 0.122 0.91 0.39 ...
    ##  $ total_polyunsaturated_fatty_acids_gm    : num  0.007 0.159 0.019 0.139 0.059 0.159 0.149 0.014 0.079 0.043 ...
    ##  $ cholesterol_mg                          : num  1 20 2 18 8 20 19 2 10 5 ...
    ##  $ vitamin_e_as_alpha.tocopherol_mg        : num  0 0.08 0.01 0.07 0.03 0.08 0.07 0.01 0.04 0.02 ...
    ##  $ intake_day_cat                          : chr  "Tuesday" "Wednesday" "Friday" "Sunday" ...
    ##  $ data_release_cycle                      : num  66 66 66 66 66 66 66 66 66 66 ...
    ##  $ interview/examination_status            : chr  "interview_and_mec_examined" "interview_and_mec_examined" "interview_and_mec_examined" "interview_and_mec_examined" ...
    ##  $ gender                                  : chr  "female" "female" "female" "male" ...
    ##  $ age_in_years_at_screening               : num  30 10 63 15 54 5 13 66 71 76 ...
    ##  $ age_in_months_at_screening_-_0_to_24_mos: num  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ race/hispanic_origin                    : chr  "other_race_incl_multiracial" "mexican_american" "non-hispanic_white" "non-hispanic_black" ...
    ##  $ race/hispanic_origin_w/_nh_asian        : chr  "non-hispanic_asian" "mexican_american" "non-hispanic_white" "non-hispanic_black" ...
    ##  $ six_month_time_period                   : chr  "nov1-apr30" "may1-oct31" "nov1-apr30" "nov1-apr30" ...
    ##  $ country_of_birth                        : chr  "other" "united_states" "united_states" "united_states" ...
    ##  $ length_of_time_in_us                    : chr  "less_than_5" NA NA NA ...
    ##  $ education_level_-_adults_20+            : num  5 NA 5 NA 1 NA NA 5 4 1 ...
    ##  $ marital_status                          : num  1 NA 2 NA 3 NA NA 1 1 2 ...
    ##  $ pregnancy_status_at_exam                : chr  "not_pregnant" NA NA NA ...
    ##  $ age_category                            : chr  "19-30" "9-13" "51-70" "14-18" ...
    ##  $ short_food_code_description             : chr  "MILK, NFS" "MILK, NFS" "MILK, NFS" "MILK, NFS" ...
    ##  $ long_food_code_description              : chr  "Milk, NFS" "Milk, NFS" "Milk, NFS" "Milk, NFS" ...
    ##  - attr(*, ".internal.selfref")=<externalptr>

Check key variables and provide summary statistics in tabular form.

First, check the predicted variables.

``` r
quantile(df$total_sugars_gm, seq(0, 1, 0.1))
```

    ##      0%     10%     20%     30%     40%     50%     60%     70%     80%     90% 
    ##   0.000   0.000   0.050   0.260   0.770   1.825   3.660   6.830  12.200  21.570 
    ##    100% 
    ## 690.230

``` r
quantile(df$total_saturated_fatty_acids_gm, seq(0, 1, 0.1))
```

    ##       0%      10%      20%      30%      40%      50%      60%      70% 
    ##   0.0000   0.0000   0.0020   0.0130   0.0710   0.3680   0.9148   1.6820 
    ##      80%      90%     100% 
    ##   3.0270   5.6380 131.1270

Some food items seem to be high in sugars and saturated fats, lets see
what it makes sense

``` r
df[total_sugars_gm > 200, .(unique(long_food_code_description),
    total_sugars_gm)] %>%
    head(n = 20)
```

    ## Warning in as.data.table.list(jval, .named = NULL): Item 1 has 35 rows but
    ## longest item has 78; recycled with remainder.

    ##                                                                         V1
    ##  1:                    Cake or cupcake, chocolate with white icing, bakery
    ##  2:                Cake or cupcake, chocolate with chocolate icing, bakery
    ##  3:                                                Cake or cupcake, marble
    ##  4:                                                            Cake, pound
    ##  5:                                     Cake, pound, with icing or filling
    ##  6:                                                        Cookie, coconut
    ##  7:                                                        Tart, all types
    ##  8:                                                      Pie, sweet potato
    ##  9: Orange juice, 100%, with calcium added, canned, bottled or in a carton
    ## 10:                                                      Apple juice, 100%
    ## 11:                                                      Grape juice, 100%
    ## 12:                                       Sugar, white, granulated or lump
    ## 13:                                                          Pancake syrup
    ## 14:                                   Fruit leather and fruit snacks candy
    ## 15:                                                           Candy, taffy
    ## 16:                    Tea, iced, instant, black, pre-sweetened with sugar
    ## 17:                     Tea, iced, brewed, black, pre-sweetened with sugar
    ## 18:                                              Tea, iced, bottled, black
    ## 19:                                                       Soft drink, cola
    ## 20:                                                Soft drink, pepper type
    ##     total_sugars_gm
    ##  1:          209.20
    ##  2:          217.38
    ##  3:          314.35
    ##  4:          218.30
    ##  5:          219.04
    ##  6:          286.10
    ##  7:          241.14
    ##  8:          203.46
    ##  9:          270.96
    ## 10:          214.00
    ## 11:          204.75
    ## 12:          329.74
    ## 13:          301.47
    ## 14:          313.13
    ## 15:          210.64
    ## 16:          598.80
    ## 17:          224.55
    ## 18:          201.60
    ## 19:          212.55
    ## 20:          235.23

``` r
df[total_saturated_fatty_acids_gm > 50, .(unique(long_food_code_description),
    total_saturated_fatty_acids_gm)] %>%
    head(n = 20)
```

    ## Warning in as.data.table.list(jval, .named = NULL): Item 1 has 39 rows but
    ## longest item has 47; recycled with remainder.

    ##                                                     V1
    ##  1:                                        Milk, whole
    ##  2:                                       Cream, heavy
    ##  3:                                Sour cream, regular
    ##  4:                                           Tiramisu
    ##  5:                                    Cheese, Cheddar
    ##  6:                                   Cheese, Monterey
    ##  7:        Beef, shortribs, cooked, lean and fat eaten
    ##  8:        Pork, spareribs, cooked, lean and fat eaten
    ##  9: Chicken "wings" with hot sauce, from other sources
    ## 10:                                  Seafood thermidor
    ## 11:                                   Pot pie, chicken
    ## 12:                      Coconut milk, used in cooking
    ## 13:                 Roll, sweet, cinnamon bun, frosted
    ## 14:                                  Cheesecake, plain
    ## 15:                                  Cheesecake, fruit
    ## 16:                                    Cookie, coconut
    ## 17:    Cookie, butter or sugar, with fruit and/or nuts
    ## 18:                                  Pie, sweet potato
    ## 19:          Popcorn, movie theater, with added butter
    ## 20:            Popcorn, movie theater, no butter added
    ##     total_saturated_fatty_acids_gm
    ##  1:                         72.614
    ##  2:                        110.554
    ##  3:                         60.840
    ##  4:                         67.958
    ##  5:                         78.336
    ##  6:                         52.224
    ##  7:                         56.054
    ##  8:                         50.264
    ##  9:                         82.703
    ## 10:                         68.177
    ## 11:                         60.118
    ## 12:                         57.947
    ## 13:                         50.736
    ## 14:                         96.765
    ## 15:                         51.205
    ## 16:                         67.883
    ## 17:                         52.672
    ## 18:                        120.594
    ## 19:                         58.760
    ## 20:                         93.884

Next, check the predictor variables.

``` r
# fs_d1[, .(mean(total_sugars_gm)), by =
# 'time_of_eating_occasion_hh.mm']
unique(df$food_source)
```

    ##  [1] "Store - grocery/supermarket"                 
    ##  [2] "Cafeteria in a K-12 school"                  
    ##  [3] "Restaurant fast food/pizza"                  
    ##  [4] "From someone else/gift"                      
    ##  [5] "Child/Adult care center"                     
    ##  [6] "Restaurant with waiter/waitress"             
    ##  [7] "Don't know"                                  
    ##  [8] "Grown or caught by you or someone you know"  
    ##  [9] "Store - convenience type"                    
    ## [10] "Cafeteria NOT in a K-12 school"              
    ## [11] "Store - no additional info"                  
    ## [12] "Community food program - other"              
    ## [13] "Child/Adult home care"                       
    ## [14] "Restaurant no additional information"        
    ## [15] "Meals on Wheels"                             
    ## [16] "Common coffee pot or snack tray"             
    ## [17] "Soup kitchen/shelter/food pantry"            
    ## [18] "Mail order purchase"                         
    ## [19] "Sport, recreation, or entertainment facility"
    ## [20] "Residential dining facility"                 
    ## [21] "Bar/tavern/lounge"                           
    ## [22] "Street vendor, vending truck"                
    ## [23] "Vending machine"                             
    ## [24] "Fundraiser sales"                            
    ## [25] "Community program no additional information" 
    ## [26] "Fish caught by you or someone you know"

``` r
unique(df$eating_occasion)
```

    ##  [1] "Snack"                "Lunch"                "Breakfast"           
    ##  [4] "Supper"               "Desayano"             "Dinner"              
    ##  [7] "Almuerzo"             "Cena"                 "Drink"               
    ## [10] "Brunch"               "Infant feeding"       "Comida"              
    ## [13] "Bebida"               "Bocadillo"            "Merienda"            
    ## [16] "Entre comida"         "Extended consumption" "Botana"              
    ## [19] "Tentempie"            "Don't know"

``` r
unique(df$did_you_eat_this_meal_at_home.)
```

    ## [1] "yes"       "no"        "dont_know"

``` r
unique(df$intake_day_of_the_week)
```

    ## [1] 3 4 6 1 5 7 2

``` r
summary(df$age_in_years_at_screening)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##    0.00   13.00   36.00   36.77   59.00   80.00

``` r
unique(df$gender)
```

    ## [1] "female" "male"

``` r
unique(df$`race/hispanic_origin_w/_nh_asian`)
```

    ## [1] "non-hispanic_asian"          "mexican_american"           
    ## [3] "non-hispanic_white"          "non-hispanic_black"         
    ## [5] "other_race_incl_multiracial" "other_hispanic"

Summary tables

``` r
# average and standard deviation of sugar consumption and
# saturated fatty acid consumption by day of the week
df[, .(avg_total_sugar = mean(total_sugars_gm), std_total_sugar = sd(total_sugars_gm),
    avg_total_sat_fa = mean(total_saturated_fatty_acids_gm),
    std_total_sat_fa = sd(total_saturated_fatty_acids_gm), num_observations = .N),
    by = c("intake_day_of_the_week", "intake_day_cat")][order(intake_day_of_the_week)][,
    intake_day_cat:num_observations] %>%
    knitr::kable()
```

| intake_day_cat | avg_total_sugar | std_total_sugar | avg_total_sat_fa | std_total_sat_fa | num_observations |
|:---------------|----------------:|----------------:|-----------------:|-----------------:|-----------------:|
| Sunday         |        7.801538 |        15.43017 |         2.014210 |         4.013946 |            29413 |
| Monday         |        7.304740 |        14.37943 |         1.835125 |         3.531646 |            13925 |
| Tuesday        |        7.350899 |        14.26861 |         1.830120 |         3.769581 |            14489 |
| Wednesday      |        7.739192 |        16.26911 |         1.890072 |         3.748256 |            14084 |
| Thursday       |        7.221043 |        14.29067 |         1.815898 |         3.562960 |            16431 |
| Friday         |        7.694802 |        14.95740 |         1.894253 |         3.824790 |            44868 |
| Saturday       |        7.869872 |        15.29819 |         2.017774 |         3.952384 |            38534 |

From the summary table above, it can be observed that average sugar and
saturated fat consumption is slightly higher on the weekends, as well as
Friday and Wednesday.

``` r
# average and standard deviation of sugar consumption and
# saturated fatty acid consumption versus by time of day

df[, .(avg_total_sugar = mean(total_sugars_gm), std_total_sugar = sd(total_sugars_gm),
    avg_total_sat_fa = mean(total_saturated_fatty_acids_gm),
    std_total_sat_fa = sd(total_saturated_fatty_acids_gm), num_observations = .N),
    by = hour(time_of_eating_occasion_hh.mm)][order(hour)] %>%
    knitr::kable()
```

| hour | avg_total_sugar | std_total_sugar | avg_total_sat_fa | std_total_sat_fa | num_observations |
|-----:|----------------:|----------------:|-----------------:|-----------------:|-----------------:|
|    0 |       10.424403 |        23.19728 |         2.174412 |         5.122779 |              452 |
|    1 |       10.857127 |        19.23448 |         2.697064 |         7.184363 |              362 |
|    2 |        8.827192 |        17.78924 |         1.892840 |         4.596523 |              349 |
|    3 |        7.545575 |        11.56730 |         1.461445 |         2.778165 |              339 |
|    4 |        7.990772 |        25.78593 |         1.202934 |         2.451188 |              518 |
|    5 |        7.782802 |        16.46515 |         1.354409 |         3.081780 |             1281 |
|    6 |        7.108148 |        15.52001 |         1.256766 |         2.605827 |             3451 |
|    7 |        7.356812 |        13.57788 |         1.361875 |         2.999766 |             8227 |
|    8 |        7.271421 |        14.34040 |         1.451280 |         2.827272 |            10510 |
|    9 |        8.048842 |        17.56882 |         1.639531 |         3.171391 |             9365 |
|   10 |        8.579764 |        17.94100 |         1.695341 |         3.297806 |             8563 |
|   11 |        7.477384 |        14.64267 |         1.748477 |         3.362429 |            10316 |
|   12 |        6.885942 |        13.97966 |         1.856575 |         3.709916 |            15844 |
|   13 |        7.139820 |        15.98766 |         1.894265 |         3.702045 |            10893 |
|   14 |        7.858531 |        14.44776 |         1.813887 |         3.512866 |             8798 |
|   15 |        8.348775 |        14.65888 |         1.841782 |         3.794892 |             8130 |
|   16 |        8.366923 |        15.39613 |         2.033539 |         4.288626 |             7989 |
|   17 |        6.911473 |        13.57440 |         2.128911 |         4.146229 |            11635 |
|   18 |        6.743814 |        13.43673 |         2.187405 |         4.203664 |            15324 |
|   19 |        7.269543 |        13.92301 |         2.256668 |         4.307055 |            13974 |
|   20 |        8.134775 |        14.69946 |         2.288643 |         4.155776 |            11343 |
|   21 |        9.124306 |        16.83863 |         2.430521 |         4.661732 |             7347 |
|   22 |        9.603860 |        17.01041 |         2.365811 |         4.602343 |             4368 |
|   23 |        8.734024 |        14.78982 |         2.170832 |         4.285383 |             2366 |

From the summary table above that groups sugar and saturated fat
consumption by the hours that they were consumed, it can be seen that
higher sugar and saturated fat consumption occurs between the hours of
8PM to 2AM. 3PM and 4PM also had slightly higher sugar and saturated fat
consumption.

``` r
df[, .(avg_total_sugar = mean(total_sugars_gm), std_total_sugar = sd(total_sugars_gm),
    avg_total_sat_fa = mean(total_saturated_fatty_acids_gm),
    std_total_sat_fa = sd(total_saturated_fatty_acids_gm), num_observations = .N),
    by = "age_category"][order(desc(avg_total_sugar))] %>%
    knitr::kable()
```

| age_category | avg_total_sugar | std_total_sugar | avg_total_sat_fa | std_total_sat_fa | num_observations |
|:-------------|----------------:|----------------:|-----------------:|-----------------:|-----------------:|
| 14-18        |       10.275529 |       18.375623 |         2.646329 |         4.879290 |            12122 |
| 9-13         |        9.324612 |       14.373753 |         2.227405 |         3.970915 |            14622 |
| 19-30        |        8.728195 |       16.794859 |         2.398415 |         4.754211 |            18209 |
| 4-8          |        8.109404 |       11.806742 |         1.789826 |         3.208890 |            14622 |
| 31-50        |        7.769942 |       18.048073 |         1.995524 |         4.070046 |            35534 |
| 51-70        |        6.763384 |       14.683986 |         1.755763 |         3.651541 |            42803 |
| 1-3          |        6.259825 |        9.227262 |         1.304175 |         2.180682 |            11804 |
| 70+          |        6.171917 |       10.925403 |         1.569317 |         3.026872 |            22028 |

#### What time and/or day of the week do people generally eat foods high in sugar/saturated fats?

Plot sugar consumption versus time.

``` r
# make names for food survey table and convert back to data
# table
fs_d1 <- as.data.table(fs_d1)


fs_d1[, mean(total_sugars_gm), by = c("time_of_eating_occasion_hh.mm",
    "intake_day_of_the_week")] %>%
    ggplot() + geom_line(aes(x = time_of_eating_occasion_hh.mm,
    y = V1)) + facet_wrap(~intake_day_of_the_week, nrow = 3)
```

![](README_files/figure-gfm/plot%20sugar%20consumption%20against%20time-1.png)<!-- -->
