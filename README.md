PM566 Midterm Project
================
Flemming Wu
2022-10-12

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
demographic[, `:=`(age_category, fifelse(age_in_years_at_screening ==
    0, "<1", fifelse(age_in_years_at_screening %between% c(1,
    3), "1-3", fifelse(age_in_years_at_screening %between% c(4,
    8), "4-8", fifelse(age_in_years_at_screening %between% c(9,
    13), "9-13", fifelse(age_in_years_at_screening %between%
    c(14, 18), "14-18", fifelse(age_in_years_at_screening %between%
    c(19, 30), "19-30", fifelse(age_in_years_at_screening %between%
    c(31, 50), "31-50", fifelse(age_in_years_at_screening %between%
    c(51, 70), "51-70", "70+")))))))))]
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
head(df)
tail(df)
```

Check variable types

``` r
str(df)
```

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
    knitr::kable(col.names = c("Day of Food Intake", "Averge Total Sugar Consumption",
        "Standard Deviation of Total Sugar Consumption", "Average Total Saturated Fatty Acid Consumption",
        "Standard Deviation of Total Saturated Fatty Acid Consumption",
        "Number of Observations"))
```

| Day of Food Intake | Averge Total Sugar Consumption | Standard Deviation of Total Sugar Consumption | Average Total Saturated Fatty Acid Consumption | Standard Deviation of Total Saturated Fatty Acid Consumption | Number of Observations |
|:-------------------|-------------------------------:|----------------------------------------------:|-----------------------------------------------:|-------------------------------------------------------------:|-----------------------:|
| Sunday             |                       7.801538 |                                      15.43017 |                                       2.014210 |                                                     4.013946 |                  29413 |
| Monday             |                       7.304740 |                                      14.37943 |                                       1.835125 |                                                     3.531646 |                  13925 |
| Tuesday            |                       7.350899 |                                      14.26861 |                                       1.830120 |                                                     3.769581 |                  14489 |
| Wednesday          |                       7.739192 |                                      16.26911 |                                       1.890072 |                                                     3.748256 |                  14084 |
| Thursday           |                       7.221043 |                                      14.29067 |                                       1.815898 |                                                     3.562960 |                  16431 |
| Friday             |                       7.694802 |                                      14.95740 |                                       1.894253 |                                                     3.824790 |                  44868 |
| Saturday           |                       7.869872 |                                      15.29819 |                                       2.017774 |                                                     3.952384 |                  38534 |

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
    knitr::kable(col.names = c("Hour of the Day (0 = 12:00 AM - 12:59 AM, 23 = 11:00 PM - 11:59 PM)",
        "Averge Total Sugar Consumption", "Standard Deviation of Total Sugar Consumption",
        "Average Total Saturated Fatty Acid Consumption", "Standard Deviation of Total Saturated Fatty Acid Consumption",
        "Number of Observations"))
```

| Hour of the Day (0 = 12:00 AM - 12:59 AM, 23 = 11:00 PM - 11:59 PM) | Averge Total Sugar Consumption | Standard Deviation of Total Sugar Consumption | Average Total Saturated Fatty Acid Consumption | Standard Deviation of Total Saturated Fatty Acid Consumption | Number of Observations |
|--------------------------------------------------------------------:|-------------------------------:|----------------------------------------------:|-----------------------------------------------:|-------------------------------------------------------------:|-----------------------:|
|                                                                   0 |                      10.424403 |                                      23.19728 |                                       2.174412 |                                                     5.122779 |                    452 |
|                                                                   1 |                      10.857127 |                                      19.23448 |                                       2.697064 |                                                     7.184363 |                    362 |
|                                                                   2 |                       8.827192 |                                      17.78924 |                                       1.892840 |                                                     4.596523 |                    349 |
|                                                                   3 |                       7.545575 |                                      11.56730 |                                       1.461445 |                                                     2.778165 |                    339 |
|                                                                   4 |                       7.990772 |                                      25.78593 |                                       1.202934 |                                                     2.451188 |                    518 |
|                                                                   5 |                       7.782802 |                                      16.46515 |                                       1.354409 |                                                     3.081780 |                   1281 |
|                                                                   6 |                       7.108148 |                                      15.52001 |                                       1.256766 |                                                     2.605827 |                   3451 |
|                                                                   7 |                       7.356812 |                                      13.57788 |                                       1.361875 |                                                     2.999766 |                   8227 |
|                                                                   8 |                       7.271421 |                                      14.34040 |                                       1.451280 |                                                     2.827272 |                  10510 |
|                                                                   9 |                       8.048842 |                                      17.56882 |                                       1.639531 |                                                     3.171391 |                   9365 |
|                                                                  10 |                       8.579764 |                                      17.94100 |                                       1.695341 |                                                     3.297806 |                   8563 |
|                                                                  11 |                       7.477384 |                                      14.64267 |                                       1.748477 |                                                     3.362429 |                  10316 |
|                                                                  12 |                       6.885942 |                                      13.97966 |                                       1.856575 |                                                     3.709916 |                  15844 |
|                                                                  13 |                       7.139820 |                                      15.98766 |                                       1.894265 |                                                     3.702045 |                  10893 |
|                                                                  14 |                       7.858531 |                                      14.44776 |                                       1.813887 |                                                     3.512866 |                   8798 |
|                                                                  15 |                       8.348775 |                                      14.65888 |                                       1.841782 |                                                     3.794892 |                   8130 |
|                                                                  16 |                       8.366923 |                                      15.39613 |                                       2.033539 |                                                     4.288626 |                   7989 |
|                                                                  17 |                       6.911473 |                                      13.57440 |                                       2.128911 |                                                     4.146229 |                  11635 |
|                                                                  18 |                       6.743814 |                                      13.43673 |                                       2.187405 |                                                     4.203664 |                  15324 |
|                                                                  19 |                       7.269543 |                                      13.92301 |                                       2.256668 |                                                     4.307055 |                  13974 |
|                                                                  20 |                       8.134775 |                                      14.69946 |                                       2.288643 |                                                     4.155776 |                  11343 |
|                                                                  21 |                       9.124306 |                                      16.83863 |                                       2.430521 |                                                     4.661732 |                   7347 |
|                                                                  22 |                       9.603860 |                                      17.01041 |                                       2.365811 |                                                     4.602343 |                   4368 |
|                                                                  23 |                       8.734024 |                                      14.78982 |                                       2.170832 |                                                     4.285383 |                   2366 |

From the summary table above that groups sugar and saturated fat
consumption by the hours that they were consumed, it can be seen that
higher sugar and saturated fat consumption occurs between the hours of
8PM to 2AM. 3PM and 4PM also had slightly higher sugar and saturated fat
consumption.

``` r
df[, .(avg_total_sugar = mean(total_sugars_gm), std_total_sugar = sd(total_sugars_gm),
    avg_total_sat_fa = mean(total_saturated_fatty_acids_gm),
    std_total_sat_fa = sd(total_saturated_fatty_acids_gm), num_observations = .N,
    min(age_in_years_at_screening)), by = "age_category"][order(V6)][,
    !c("V6")] %>%
    knitr::kable(col.names = c("Age Range", "Averge Total Sugar Consumption",
        "Standard Deviation of Total Sugar Consumption", "Average Total Saturated Fatty Acid Consumption",
        "Standard Deviation of Total Saturated Fatty Acid Consumption",
        "Number of Observations"))
```

| Age Range | Averge Total Sugar Consumption | Standard Deviation of Total Sugar Consumption | Average Total Saturated Fatty Acid Consumption | Standard Deviation of Total Saturated Fatty Acid Consumption | Number of Observations |
|:----------|-------------------------------:|----------------------------------------------:|-----------------------------------------------:|-------------------------------------------------------------:|-----------------------:|
| \<1       |                       6.603717 |                                      5.933466 |                                       1.322429 |                                                     1.438592 |                   4073 |
| 1-3       |                       6.259825 |                                      9.227262 |                                       1.304175 |                                                     2.180682 |                  11804 |
| 4-8       |                       8.109404 |                                     11.806742 |                                       1.789826 |                                                     3.208890 |                  14622 |
| 9-13      |                       9.324612 |                                     14.373753 |                                       2.227405 |                                                     3.970915 |                  14622 |
| 14-18     |                      10.275529 |                                     18.375623 |                                       2.646329 |                                                     4.879290 |                  12122 |
| 19-30     |                       8.728195 |                                     16.794859 |                                       2.398415 |                                                     4.754211 |                  18209 |
| 31-50     |                       7.769942 |                                     18.048073 |                                       1.995524 |                                                     4.070046 |                  35534 |
| 51-70     |                       6.763384 |                                     14.683986 |                                       1.755763 |                                                     3.651541 |                  42803 |
| 70+       |                       6.073965 |                                     11.764628 |                                       1.625322 |                                                     3.279340 |                  17955 |

``` r
df[, .(avg_total_sugar = mean(total_sugars_gm), std_total_sugar = sd(total_sugars_gm),
    avg_total_sat_fa = mean(total_saturated_fatty_acids_gm),
    std_total_sat_fa = sd(total_saturated_fatty_acids_gm), num_observations = .N),
    by = "gender"] %>%
    knitr::kable(col.names = c("Gender", "Averge Total Sugar Consumption",
        "Standard Deviation of Total Sugar Consumption", "Average Total Saturated Fatty Acid Consumption",
        "Standard Deviation of Total Saturated Fatty Acid Consumption",
        "Number of Observations"))
```

| Gender | Averge Total Sugar Consumption | Standard Deviation of Total Sugar Consumption | Average Total Saturated Fatty Acid Consumption | Standard Deviation of Total Saturated Fatty Acid Consumption | Number of Observations |
|:-------|-------------------------------:|----------------------------------------------:|-----------------------------------------------:|-------------------------------------------------------------:|-----------------------:|
| female |                       6.843890 |                                      13.02220 |                                       1.680465 |                                                     3.289837 |                  87533 |
| male   |                       8.487986 |                                      16.88883 |                                       2.178095 |                                                     4.305765 |                  84211 |

``` r
df[, .(avg_total_sugar = mean(total_sugars_gm), std_total_sugar = sd(total_sugars_gm),
    avg_total_sat_fa = mean(total_saturated_fatty_acids_gm),
    std_total_sat_fa = sd(total_saturated_fatty_acids_gm), num_observations = .N),
    by = "race/hispanic_origin_w/_nh_asian"] %>%
    knitr::kable(col.names = c("Race", "Averge Total Sugar Consumption",
        "Standard Deviation of Total Sugar Consumption", "Average Total Saturated Fatty Acid Consumption",
        "Standard Deviation of Total Saturated Fatty Acid Consumption",
        "Number of Observations"))
```

| Race                        | Averge Total Sugar Consumption | Standard Deviation of Total Sugar Consumption | Average Total Saturated Fatty Acid Consumption | Standard Deviation of Total Saturated Fatty Acid Consumption | Number of Observations |
|:----------------------------|-------------------------------:|----------------------------------------------:|-----------------------------------------------:|-------------------------------------------------------------:|-----------------------:|
| non-hispanic_asian          |                       5.518388 |                                      10.54705 |                                       1.481301 |                                                     3.024665 |                  18008 |
| mexican_american            |                       7.306042 |                                      13.27055 |                                       1.878062 |                                                     4.020707 |                  22247 |
| non-hispanic_white          |                       7.863279 |                                      16.19630 |                                       2.024360 |                                                     3.871168 |                  61381 |
| non-hispanic_black          |                       8.394050 |                                      15.73418 |                                       2.038325 |                                                     4.074282 |                  42203 |
| other_race_incl_multiracial |                       8.607685 |                                      16.77969 |                                       2.084585 |                                                     3.934074 |                  10509 |
| other_hispanic              |                       7.160682 |                                      14.03439 |                                       1.717158 |                                                     3.441850 |                  17396 |

``` r
df[, .(avg_total_sugar = mean(total_sugars_gm), std_total_sugar = sd(total_sugars_gm),
    avg_total_sat_fa = mean(total_saturated_fatty_acids_gm),
    std_total_sat_fa = sd(total_saturated_fatty_acids_gm), num_observations = .N),
    by = "food_source"][order(-avg_total_sugar, -avg_total_sat_fa)] %>%
    knitr::kable(col.names = c("Food Source", "Averge Total Sugar Consumption",
        "Standard Deviation of Total Sugar Consumption", "Average Total Saturated Fatty Acid Consumption",
        "Standard Deviation of Total Saturated Fatty Acid Consumption",
        "Number of Observations"))
```

| Food Source                                  | Averge Total Sugar Consumption | Standard Deviation of Total Sugar Consumption | Average Total Saturated Fatty Acid Consumption | Standard Deviation of Total Saturated Fatty Acid Consumption | Number of Observations |
|:---------------------------------------------|-------------------------------:|----------------------------------------------:|-----------------------------------------------:|-------------------------------------------------------------:|-----------------------:|
| Vending machine                              |                      19.888186 |                                     24.438825 |                                      1.1177063 |                                                     2.404319 |                    463 |
| Store - convenience type                     |                      14.482878 |                                     26.497933 |                                      1.6245324 |                                                     3.862708 |                   7258 |
| Fundraiser sales                             |                      12.112783 |                                     13.283127 |                                      3.1431913 |                                                     4.049391 |                    115 |
| Sport, recreation, or entertainment facility |                      10.116901 |                                     17.636052 |                                      4.1400352 |                                                     9.959429 |                    710 |
| Street vendor, vending truck                 |                       9.598734 |                                     17.395667 |                                      2.8635253 |                                                     5.213732 |                    474 |
| Store - no additional info                   |                       9.503502 |                                     15.719486 |                                      2.0621958 |                                                     4.277158 |                    674 |
| Cafeteria in a K-12 school                   |                       8.811974 |                                      9.619468 |                                      1.7129800 |                                                     2.793296 |                   3652 |
| Community food program - other               |                       8.289384 |                                      8.525483 |                                      1.5690293 |                                                     2.232993 |                    341 |
| From someone else/gift                       |                       8.261302 |                                     15.242012 |                                      1.9390957 |                                                     3.571431 |                  10488 |
| Residential dining facility                  |                       7.808561 |                                     14.979978 |                                      1.7549773 |                                                     3.474209 |                    132 |
| Soup kitchen/shelter/food pantry             |                       7.744011 |                                     17.791625 |                                      1.7975879 |                                                     3.484726 |                    182 |
| Store - grocery/supermarket                  |                       7.457371 |                                     14.397440 |                                      1.7204906 |                                                     3.520996 |                 112031 |
| Restaurant fast food/pizza                   |                       7.228542 |                                     14.316174 |                                      3.2071811 |                                                     5.148242 |                  18094 |
| Child/Adult home care                        |                       7.223605 |                                     12.475460 |                                      1.5533256 |                                                     3.074922 |                    172 |
| Child/Adult care center                      |                       6.533219 |                                      8.278626 |                                      1.3239741 |                                                     1.993239 |                    848 |
| Bar/tavern/lounge                            |                       6.189798 |                                     15.957909 |                                      1.8428584 |                                                     3.737848 |                    445 |
| Cafeteria NOT in a K-12 school               |                       5.972846 |                                     11.873962 |                                      1.7879406 |                                                     3.272797 |                   1346 |
| Don’t know                                   |                       5.954612 |                                     11.021807 |                                      1.5457670 |                                                     3.117133 |                    618 |
| Meals on Wheels                              |                       5.519375 |                                      8.174597 |                                      2.3256667 |                                                     3.931965 |                     96 |
| Grown or caught by you or someone you know   |                       5.403435 |                                     12.317432 |                                      1.4059732 |                                                     2.596471 |                    821 |
| Common coffee pot or snack tray              |                       5.328894 |                                     14.322616 |                                      0.4903145 |                                                     1.654317 |                    461 |
| Community program no additional information  |                       5.082667 |                                      7.527454 |                                      1.4772667 |                                                     2.338569 |                     15 |
| Restaurant with waiter/waitress              |                       4.997763 |                                     12.892353 |                                      2.2433060 |                                                     4.023191 |                  11379 |
| Mail order purchase                          |                       4.468455 |                                     11.840954 |                                      0.9213269 |                                                     2.642477 |                    835 |
| Restaurant no additional information         |                       3.431039 |                                      7.588165 |                                      1.7768182 |                                                     3.541378 |                     77 |
| Fish caught by you or someone you know       |                       1.707647 |                                      3.441011 |                                      3.7097647 |                                                     5.645766 |                     17 |

``` r
df[, .(avg_total_sugar = mean(total_sugars_gm), std_total_sugar = sd(total_sugars_gm),
    avg_total_sat_fa = mean(total_saturated_fatty_acids_gm),
    std_total_sat_fa = sd(total_saturated_fatty_acids_gm), num_observations = .N),
    by = "did_you_eat_this_meal_at_home."][order(-avg_total_sugar)] %>%
    knitr::kable(col.names = c("Meal Eaten at Home", "Averge Total Sugar Consumption",
        "Standard Deviation of Total Sugar Consumption", "Average Total Saturated Fatty Acid Consumption",
        "Standard Deviation of Total Saturated Fatty Acid Consumption",
        "Number of Observations"))
```

| Meal Eaten at Home | Averge Total Sugar Consumption | Standard Deviation of Total Sugar Consumption | Average Total Saturated Fatty Acid Consumption | Standard Deviation of Total Saturated Fatty Acid Consumption | Number of Observations |
|:-------------------|-------------------------------:|----------------------------------------------:|-----------------------------------------------:|-------------------------------------------------------------:|-----------------------:|
| no                 |                       7.890357 |                                     15.566681 |                                       1.952841 |                                                     3.828746 |                  52001 |
| yes                |                       7.546146 |                                     14.842989 |                                       1.912270 |                                                     3.830932 |                 119689 |
| dont_know          |                       6.498519 |                                      9.579175 |                                       1.636796 |                                                     2.444655 |                     54 |

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
