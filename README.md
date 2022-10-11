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
fs_d1[, `:=`(intake_day_of_the_week, fifelse(intake_day_of_the_week ==
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

``` r
str(fs_d1)
```

    ## 'data.frame':    171744 obs. of  30 variables:
    ##  $ food_source                           : chr  "Store - grocery/supermarket" "Store - grocery/supermarket" "Store - grocery/supermarket" "Store - grocery/supermarket" ...
    ##  $ eating_occasion                       : chr  "Breakfast" "Breakfast" "Desayano" "Breakfast" ...
    ##  $ respondent_sequence_number            : num  122474 119677 123644 112441 109587 ...
    ##  $ dietary_day_one_sample_weight         : num  12511 3463 6272 4751 36013 ...
    ##  $ dietary_two.day_sample_weight         : num  15417 2762 6218 0 35149 ...
    ##  $ food.individual_component_number      : num  1 3 2 2 7 2 4 8 1 1 ...
    ##  $ dietary_recall_status                 : num  1 1 1 1 1 1 1 1 1 1 ...
    ##  $ interviewer_id_code                   : num  89 81 90 86 91 86 88 81 91 81 ...
    ##  $ breast.fed_infant_either_day          : num  2 2 2 2 2 2 2 2 2 2 ...
    ##  $ number_of_days_of_intake              : num  2 2 2 1 2 2 2 2 2 1 ...
    ##  $ X._of_days_b.w_intake_and_hh_interview: num  1 13 33 NA -2 -6 43 23 -1 15 ...
    ##  $ intake_day_of_the_week                : chr  "Monday" "Saturday" "Tuesday" "Saturday" ...
    ##  $ language_respondent_used_mostly       : num  1 1 2 1 1 1 1 2 1 1 ...
    ##  $ combination_food_number               : num  0 0 1 1 3 1 2 0 1 1 ...
    ##  $ combination_food_type                 : num  0 0 1 2 90 1 1 0 1 2 ...
    ##  $ time_of_eating_occasion_hh.mm         : 'hms' num  07:00:00 09:00:00 08:30:00 08:30:00 ...
    ##   ..- attr(*, "units")= chr "secs"
    ##  $ did_you_eat_this_meal_at_home.        : chr  "no" "yes" "yes" "yes" ...
    ##  $ usda_food_code                        : num  22600200 54325000 12210210 11112210 63200100 ...
    ##  $ grams                                 : num  32 18 60 122 150 ...
    ##  $ energy_kcal                           : num  155 75 151 52 62 2 48 382 4 297 ...
    ##  $ protein_gm                            : num  11.97 1.7 0.41 4.12 1.01 ...
    ##  $ carbohydrate_gm                       : num  0.61 13.33 21.04 6.33 14.53 ...
    ##  $ total_sugars_gm                       : num  0.5 0.23 19.82 6.05 10.09 ...
    ##  $ dietary_fiber_gm                      : num  0 0.5 0.7 0 3 0 0 2.4 0 4.9 ...
    ##  $ total_fat_gm                          : num  11.45 1.56 8.1 1.16 0.38 ...
    ##  $ total_saturated_fatty_acids_gm        : num  3.93 0.298 1.581 0.693 0.012 ...
    ##  $ total_monounsaturated_fatty_acids_gm  : num  5.013 0.357 2.401 0.256 0.021 ...
    ##  $ total_polyunsaturated_fatty_acids_gm  : num  1.938 0.87 3.761 0.039 0.066 ...
    ##  $ cholesterol_mg                        : num  32 0 0 6 0 0 0 44 0 0 ...
    ##  $ vitamin_e_as_alpha.tocopherol_mg      : num  0.13 0.21 0.95 0.02 0.56 0.02 0 0.21 0 1.55 ...

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

First, check the predicted variables.

``` r
quantile(fs_d1$total_sugars_gm, seq(0, 1, 0.1)) %>%
    knitr::kable(col.names = "grams sugar in food item")
```

|      | grams sugar in food item |
|:-----|-------------------------:|
| 0%   |                    0.000 |
| 10%  |                    0.000 |
| 20%  |                    0.050 |
| 30%  |                    0.260 |
| 40%  |                    0.770 |
| 50%  |                    1.825 |
| 60%  |                    3.660 |
| 70%  |                    6.830 |
| 80%  |                   12.200 |
| 90%  |                   21.570 |
| 100% |                  690.230 |

``` r
quantile(fs_d1$total_saturated_fatty_acids_gm, seq(0, 1, 0.1)) %>%
    knitr::kable(col.names = "grams saturated FA in food item")
```

|      | grams saturated FA in food item |
|:-----|--------------------------------:|
| 0%   |                          0.0000 |
| 10%  |                          0.0000 |
| 20%  |                          0.0020 |
| 30%  |                          0.0130 |
| 40%  |                          0.0710 |
| 50%  |                          0.3680 |
| 60%  |                          0.9148 |
| 70%  |                          1.6820 |
| 80%  |                          3.0270 |
| 90%  |                          5.6380 |
| 100% |                        131.1270 |

Some food items seem to be high in sugars and saturated fats, lets see
what it makes sense

``` r
survey_cats_merged <- merge(x = fs_d1, y = food_categories, by.x = "usda_food_code",
    by.y = "food_code")

survey_cats_merged <- as.data.table(survey_cats_merged)

survey_cats_merged[total_sugars_gm > 200, .(unique(long_food_code_description),
    total_sugars_gm)] %>%
    head(n = 20) %>%
    knitr::kable(col.names = c("Food item", "grams sugar in food item"))
```

    ## Warning in as.data.table.list(jval, .named = NULL): Item 1 has 35 rows but
    ## longest item has 78; recycled with remainder.

| Food item                                                              | grams sugar in food item |
|:-----------------------------------------------------------------------|-------------------------:|
| Cake or cupcake, chocolate with white icing, bakery                    |                   209.20 |
| Cake or cupcake, chocolate with chocolate icing, bakery                |                   217.38 |
| Cake or cupcake, marble                                                |                   218.30 |
| Cake, pound                                                            |                   314.35 |
| Cake, pound, with icing or filling                                     |                   286.10 |
| Cookie, coconut                                                        |                   219.04 |
| Tart, all types                                                        |                   241.14 |
| Pie, sweet potato                                                      |                   203.46 |
| Orange juice, 100%, with calcium added, canned, bottled or in a carton |                   270.96 |
| Apple juice, 100%                                                      |                   214.00 |
| Grape juice, 100%                                                      |                   204.75 |
| Sugar, white, granulated or lump                                       |                   329.74 |
| Pancake syrup                                                          |                   313.13 |
| Fruit leather and fruit snacks candy                                   |                   301.47 |
| Candy, taffy                                                           |                   210.64 |
| Tea, iced, instant, black, pre-sweetened with sugar                    |                   598.80 |
| Tea, iced, brewed, black, pre-sweetened with sugar                     |                   224.55 |
| Tea, iced, bottled, black                                              |                   201.60 |
| Soft drink, cola                                                       |                   212.55 |
| Soft drink, pepper type                                                |                   235.23 |

``` r
survey_cats_merged[total_saturated_fatty_acids_gm > 50, .(unique(long_food_code_description),
    total_saturated_fatty_acids_gm)] %>%
    head(n = 20) %>%
    knitr::kable(col.names = c("Food item", "grams saturated FA in food item"))
```

    ## Warning in as.data.table.list(jval, .named = NULL): Item 1 has 39 rows but
    ## longest item has 47; recycled with remainder.

| Food item                                          | grams saturated FA in food item |
|:---------------------------------------------------|--------------------------------:|
| Milk, whole                                        |                          72.614 |
| Cream, heavy                                       |                         110.554 |
| Sour cream, regular                                |                          60.840 |
| Tiramisu                                           |                          67.958 |
| Cheese, Cheddar                                    |                          78.336 |
| Cheese, Monterey                                   |                          52.224 |
| Beef, shortribs, cooked, lean and fat eaten        |                          56.054 |
| Pork, spareribs, cooked, lean and fat eaten        |                          50.264 |
| Chicken “wings” with hot sauce, from other sources |                          82.703 |
| Seafood thermidor                                  |                          68.177 |
| Pot pie, chicken                                   |                          60.118 |
| Coconut milk, used in cooking                      |                          57.947 |
| Roll, sweet, cinnamon bun, frosted                 |                          50.736 |
| Cheesecake, plain                                  |                          96.765 |
| Cheesecake, fruit                                  |                          67.883 |
| Cookie, coconut                                    |                          51.205 |
| Cookie, butter or sugar, with fruit and/or nuts    |                          52.672 |
| Pie, sweet potato                                  |                         120.594 |
| Popcorn, movie theater, with added butter          |                          58.760 |
| Popcorn, movie theater, no butter added            |                          93.884 |

Next, check the predictor variables.

``` r
# fs_d1[, .(mean(total_sugars_gm)), by =
# 'time_of_eating_occasion_hh.mm']
unique(fs_d1$food_source)
```

    ##  [1] "Store - grocery/supermarket"                 
    ##  [2] "Soup kitchen/shelter/food pantry"            
    ##  [3] "Meals on Wheels"                             
    ##  [4] "Community food program - other"              
    ##  [5] "Community program no additional information" 
    ##  [6] "Vending machine"                             
    ##  [7] "Common coffee pot or snack tray"             
    ##  [8] "From someone else/gift"                      
    ##  [9] "Mail order purchase"                         
    ## [10] "Residential dining facility"                 
    ## [11] "Grown or caught by you or someone you know"  
    ## [12] "Restaurant with waiter/waitress"             
    ## [13] "Fish caught by you or someone you know"      
    ## [14] "Sport, recreation, or entertainment facility"
    ## [15] "Street vendor, vending truck"                
    ## [16] "Fundraiser sales"                            
    ## [17] "Store - convenience type"                    
    ## [18] "Store - no additional info"                  
    ## [19] "Restaurant fast food/pizza"                  
    ## [20] "Bar/tavern/lounge"                           
    ## [21] "Restaurant no additional information"        
    ## [22] "Cafeteria NOT in a K-12 school"              
    ## [23] "Cafeteria in a K-12 school"                  
    ## [24] "Child/Adult care center"                     
    ## [25] "Child/Adult home care"                       
    ## [26] "Don't know"

``` r
unique(fs_d1$eating_occasion)
```

    ##  [1] "Breakfast"            "Desayano"             "Comida"              
    ##  [4] "Almuerzo"             "Snack"                "Lunch"               
    ##  [7] "Supper"               "Dinner"               "Merienda"            
    ## [10] "Infant feeding"       "Cena"                 "Brunch"              
    ## [13] "Entre comida"         "Botana"               "Extended consumption"
    ## [16] "Bocadillo"            "Tentempie"            "Bebida"              
    ## [19] "Drink"                "Don't know"

``` r
unique(fs_d1$did_you_eat_this_meal_at_home.)
```

    ## [1] "no"        "yes"       "dont_know"

``` r
unique(fs_d1$intake_day_of_the_week)
```

    ## [1] "Monday"    "Saturday"  "Tuesday"   "Thursday"  "Friday"    "Sunday"   
    ## [7] "Wednesday"

``` r
summary(demographic$age_in_years_at_screening)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##    0.00   10.00   30.00   33.74   56.00   80.00

``` r
unique(demographic$gender)
```

    ## [1] "male"   "female"

``` r
unique(demographic$`race/hispanic_origin_w/_nh_asian`)
```

    ## [1] "non-hispanic_asian"          "mexican_american"           
    ## [3] "non-hispanic_white"          "other_hispanic"             
    ## [5] "non-hispanic_black"          "other_race_incl_multiracial"

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
