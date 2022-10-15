PM566 Midterm Project
================
Flemming Wu
2022-10-15

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

### Preliminary Results

Summary tables

| Day of Food Intake | Average Total Sugar Consumption | Standard Deviation of Total Sugar Consumption | Average Total Saturated Fatty Acid Consumption | Standard Deviation of Total Saturated Fatty Acid Consumption | Number of Observations |
|:-------------------|--------------------------------:|----------------------------------------------:|-----------------------------------------------:|-------------------------------------------------------------:|-----------------------:|
| Sunday             |                        7.801538 |                                      15.43017 |                                       2.014210 |                                                     4.013946 |                  29413 |
| Monday             |                        7.304740 |                                      14.37943 |                                       1.835125 |                                                     3.531646 |                  13925 |
| Tuesday            |                        7.350899 |                                      14.26861 |                                       1.830120 |                                                     3.769581 |                  14489 |
| Wednesday          |                        7.739192 |                                      16.26911 |                                       1.890072 |                                                     3.748256 |                  14084 |
| Thursday           |                        7.221043 |                                      14.29067 |                                       1.815898 |                                                     3.562960 |                  16431 |
| Friday             |                        7.694802 |                                      14.95740 |                                       1.894253 |                                                     3.824790 |                  44868 |
| Saturday           |                        7.869872 |                                      15.29819 |                                       2.017774 |                                                     3.952384 |                  38534 |

From the summary table above, it can be observed that average sugar and
saturated fat consumption is slightly higher on the weekends, as well as
Friday and Wednesday.

| Hour of the Day (0 = 12:00 AM - 12:59 AM, 23 = 11:00 PM - 11:59 PM) | Average Total Sugar Consumption | Standard Deviation of Total Sugar Consumption | Average Total Saturated Fatty Acid Consumption | Standard Deviation of Total Saturated Fatty Acid Consumption | Number of Observations |
|--------------------------------------------------------------------:|--------------------------------:|----------------------------------------------:|-----------------------------------------------:|-------------------------------------------------------------:|-----------------------:|
|                                                                   0 |                       10.424403 |                                      23.19728 |                                       2.174412 |                                                     5.122779 |                    452 |
|                                                                   1 |                       10.857127 |                                      19.23448 |                                       2.697064 |                                                     7.184363 |                    362 |
|                                                                   2 |                        8.827192 |                                      17.78924 |                                       1.892840 |                                                     4.596523 |                    349 |
|                                                                   3 |                        7.545575 |                                      11.56730 |                                       1.461445 |                                                     2.778165 |                    339 |
|                                                                   4 |                        7.990772 |                                      25.78593 |                                       1.202934 |                                                     2.451188 |                    518 |
|                                                                   5 |                        7.782802 |                                      16.46515 |                                       1.354409 |                                                     3.081780 |                   1281 |
|                                                                   6 |                        7.108148 |                                      15.52001 |                                       1.256766 |                                                     2.605827 |                   3451 |
|                                                                   7 |                        7.356812 |                                      13.57788 |                                       1.361875 |                                                     2.999766 |                   8227 |
|                                                                   8 |                        7.271421 |                                      14.34040 |                                       1.451280 |                                                     2.827272 |                  10510 |
|                                                                   9 |                        8.048842 |                                      17.56882 |                                       1.639531 |                                                     3.171391 |                   9365 |
|                                                                  10 |                        8.579764 |                                      17.94100 |                                       1.695341 |                                                     3.297806 |                   8563 |
|                                                                  11 |                        7.477384 |                                      14.64267 |                                       1.748477 |                                                     3.362429 |                  10316 |
|                                                                  12 |                        6.885942 |                                      13.97966 |                                       1.856575 |                                                     3.709916 |                  15844 |
|                                                                  13 |                        7.139820 |                                      15.98766 |                                       1.894265 |                                                     3.702045 |                  10893 |
|                                                                  14 |                        7.858531 |                                      14.44776 |                                       1.813887 |                                                     3.512866 |                   8798 |
|                                                                  15 |                        8.348775 |                                      14.65888 |                                       1.841782 |                                                     3.794892 |                   8130 |
|                                                                  16 |                        8.366923 |                                      15.39613 |                                       2.033539 |                                                     4.288626 |                   7989 |
|                                                                  17 |                        6.911473 |                                      13.57440 |                                       2.128911 |                                                     4.146229 |                  11635 |
|                                                                  18 |                        6.743814 |                                      13.43673 |                                       2.187405 |                                                     4.203664 |                  15324 |
|                                                                  19 |                        7.269543 |                                      13.92301 |                                       2.256668 |                                                     4.307055 |                  13974 |
|                                                                  20 |                        8.134775 |                                      14.69946 |                                       2.288643 |                                                     4.155776 |                  11343 |
|                                                                  21 |                        9.124306 |                                      16.83863 |                                       2.430521 |                                                     4.661732 |                   7347 |
|                                                                  22 |                        9.603860 |                                      17.01041 |                                       2.365811 |                                                     4.602343 |                   4368 |
|                                                                  23 |                        8.734024 |                                      14.78982 |                                       2.170832 |                                                     4.285383 |                   2366 |

From the summary table above that groups sugar and saturated fat
consumption by the hours that they were consumed, it can be seen that
higher sugar and saturated fat consumption occurs between the hours of
8PM to 2AM. 3PM and 4PM also had slightly higher sugar and saturated fat
consumption.

| Eating Occasion      | Average Total Sugar Consumption | Standard Deviation of Total Sugar Consumption | Average Total Saturated Fatty Acid Consumption | Standard Deviation of Total Saturated Fatty Acid Consumption | Number of Observations |
|:---------------------|--------------------------------:|----------------------------------------------:|-----------------------------------------------:|-------------------------------------------------------------:|-----------------------:|
| Comida               |                        5.996550 |                                     12.452777 |                                      1.9596370 |                                                     4.220748 |                   2168 |
| Almuerzo             |                        6.040902 |                                     12.888273 |                                      1.8254938 |                                                     3.930899 |                   2351 |
| Dinner               |                        6.088390 |                                     12.764289 |                                      2.3426095 |                                                     4.315730 |                  37289 |
| Lunch                |                        6.444327 |                                     13.067584 |                                      2.0005485 |                                                     3.759927 |                  34174 |
| Desayano             |                        6.601639 |                                     10.990466 |                                      1.4365567 |                                                     2.791270 |                   3032 |
| Bebida               |                        6.648814 |                                     14.167280 |                                      0.3213970 |                                                     1.120434 |                    801 |
| Cena                 |                        6.745963 |                                     12.785199 |                                      1.9265675 |                                                     4.209928 |                   2992 |
| Supper               |                        6.799386 |                                     13.952740 |                                      2.5479036 |                                                     4.750105 |                   9388 |
| Breakfast            |                        7.498159 |                                     12.440818 |                                      1.6602312 |                                                     3.106925 |                  31741 |
| Brunch               |                        7.512050 |                                     13.522190 |                                      2.3093810 |                                                     4.393188 |                   2005 |
| Infant feeding       |                        7.545321 |                                      6.038356 |                                      1.5679247 |                                                     1.438459 |                   2988 |
| Entre comida         |                        8.335434 |                                     14.733579 |                                      1.3515717 |                                                     3.758018 |                    495 |
| Drink                |                        8.943371 |                                     18.033214 |                                      0.4381726 |                                                     1.939871 |                   8423 |
| Botana               |                        9.186853 |                                     12.589935 |                                      1.4895198 |                                                     3.466799 |                    429 |
| Merienda             |                        9.470025 |                                     14.209530 |                                      1.4602024 |                                                     3.090206 |                   1186 |
| Tentempie            |                        9.664634 |                                     10.323859 |                                      1.7000976 |                                                     2.823390 |                     41 |
| Snack                |                       10.751874 |                                     16.167832 |                                      2.0776491 |                                                     4.102707 |                  29089 |
| Bocadillo            |                       10.968052 |                                     16.025106 |                                      1.7000406 |                                                     3.950243 |                    616 |
| Extended consumption |                       15.721413 |                                     50.865800 |                                      0.6547581 |                                                     3.519036 |                   2534 |
| Don’t know           |                       20.900000 |                                      0.000000 |                                      1.0370000 |                                                     0.000000 |                      2 |

| Age Range | Average Total Sugar Consumption | Standard Deviation of Total Sugar Consumption | Average Total Saturated Fatty Acid Consumption | Standard Deviation of Total Saturated Fatty Acid Consumption | Number of Observations |
|:----------|--------------------------------:|----------------------------------------------:|-----------------------------------------------:|-------------------------------------------------------------:|-----------------------:|
| \<1       |                        6.603717 |                                      5.933466 |                                       1.322429 |                                                     1.438592 |                   4073 |
| 1-3       |                        6.259825 |                                      9.227262 |                                       1.304175 |                                                     2.180682 |                  11804 |
| 4-8       |                        8.109404 |                                     11.806742 |                                       1.789826 |                                                     3.208890 |                  14622 |
| 9-13      |                        9.324612 |                                     14.373753 |                                       2.227405 |                                                     3.970915 |                  14622 |
| 14-18     |                       10.275529 |                                     18.375623 |                                       2.646329 |                                                     4.879290 |                  12122 |
| 19-30     |                        8.728195 |                                     16.794859 |                                       2.398415 |                                                     4.754211 |                  18209 |
| 31-50     |                        7.769942 |                                     18.048073 |                                       1.995524 |                                                     4.070046 |                  35534 |
| 51-70     |                        6.763384 |                                     14.683986 |                                       1.755763 |                                                     3.651541 |                  42803 |
| 70+       |                        6.073965 |                                     11.764628 |                                       1.625322 |                                                     3.279340 |                  17955 |

| Gender | Average Total Sugar Consumption | Standard Deviation of Total Sugar Consumption | Average Total Saturated Fatty Acid Consumption | Standard Deviation of Total Saturated Fatty Acid Consumption | Number of Observations |
|:-------|--------------------------------:|----------------------------------------------:|-----------------------------------------------:|-------------------------------------------------------------:|-----------------------:|
| female |                        6.843890 |                                      13.02220 |                                       1.680465 |                                                     3.289837 |                  87533 |
| male   |                        8.487986 |                                      16.88883 |                                       2.178095 |                                                     4.305765 |                  84211 |

| Race                        | Average Total Sugar Consumption | Standard Deviation of Total Sugar Consumption | Average Total Saturated Fatty Acid Consumption | Standard Deviation of Total Saturated Fatty Acid Consumption | Number of Observations |
|:----------------------------|--------------------------------:|----------------------------------------------:|-----------------------------------------------:|-------------------------------------------------------------:|-----------------------:|
| non-hispanic_asian          |                        5.518388 |                                      10.54705 |                                       1.481301 |                                                     3.024665 |                  18008 |
| mexican_american            |                        7.306042 |                                      13.27055 |                                       1.878062 |                                                     4.020707 |                  22247 |
| non-hispanic_white          |                        7.863279 |                                      16.19630 |                                       2.024360 |                                                     3.871168 |                  61381 |
| non-hispanic_black          |                        8.394050 |                                      15.73418 |                                       2.038325 |                                                     4.074282 |                  42203 |
| other_race_incl_multiracial |                        8.607685 |                                      16.77969 |                                       2.084585 |                                                     3.934074 |                  10509 |
| other_hispanic              |                        7.160682 |                                      14.03439 |                                       1.717158 |                                                     3.441850 |                  17396 |

| Food Source                                  | Average Total Sugar Consumption | Standard Deviation of Total Sugar Consumption | Average Total Saturated Fatty Acid Consumption | Standard Deviation of Total Saturated Fatty Acid Consumption | Number of Observations |
|:---------------------------------------------|--------------------------------:|----------------------------------------------:|-----------------------------------------------:|-------------------------------------------------------------:|-----------------------:|
| Vending machine                              |                       19.888186 |                                     24.438825 |                                      1.1177063 |                                                     2.404319 |                    463 |
| Store - convenience type                     |                       14.482878 |                                     26.497933 |                                      1.6245324 |                                                     3.862708 |                   7258 |
| Fundraiser sales                             |                       12.112783 |                                     13.283127 |                                      3.1431913 |                                                     4.049391 |                    115 |
| Sport, recreation, or entertainment facility |                       10.116901 |                                     17.636052 |                                      4.1400352 |                                                     9.959429 |                    710 |
| Street vendor, vending truck                 |                        9.598734 |                                     17.395667 |                                      2.8635253 |                                                     5.213732 |                    474 |
| Store - no additional info                   |                        9.503502 |                                     15.719486 |                                      2.0621958 |                                                     4.277158 |                    674 |
| Cafeteria in a K-12 school                   |                        8.811974 |                                      9.619468 |                                      1.7129800 |                                                     2.793296 |                   3652 |
| Community food program - other               |                        8.289384 |                                      8.525483 |                                      1.5690293 |                                                     2.232993 |                    341 |
| From someone else/gift                       |                        8.261302 |                                     15.242012 |                                      1.9390957 |                                                     3.571431 |                  10488 |
| Residential dining facility                  |                        7.808561 |                                     14.979978 |                                      1.7549773 |                                                     3.474209 |                    132 |
| Soup kitchen/shelter/food pantry             |                        7.744011 |                                     17.791625 |                                      1.7975879 |                                                     3.484726 |                    182 |
| Store - grocery/supermarket                  |                        7.457371 |                                     14.397440 |                                      1.7204906 |                                                     3.520996 |                 112031 |
| Restaurant fast food/pizza                   |                        7.228542 |                                     14.316174 |                                      3.2071811 |                                                     5.148242 |                  18094 |
| Child/Adult home care                        |                        7.223605 |                                     12.475460 |                                      1.5533256 |                                                     3.074922 |                    172 |
| Child/Adult care center                      |                        6.533219 |                                      8.278626 |                                      1.3239741 |                                                     1.993239 |                    848 |
| Bar/tavern/lounge                            |                        6.189798 |                                     15.957909 |                                      1.8428584 |                                                     3.737848 |                    445 |
| Cafeteria NOT in a K-12 school               |                        5.972846 |                                     11.873962 |                                      1.7879406 |                                                     3.272797 |                   1346 |
| Don’t know                                   |                        5.954612 |                                     11.021807 |                                      1.5457670 |                                                     3.117133 |                    618 |
| Meals on Wheels                              |                        5.519375 |                                      8.174597 |                                      2.3256667 |                                                     3.931965 |                     96 |
| Grown or caught by you or someone you know   |                        5.403435 |                                     12.317432 |                                      1.4059732 |                                                     2.596471 |                    821 |
| Common coffee pot or snack tray              |                        5.328894 |                                     14.322616 |                                      0.4903145 |                                                     1.654317 |                    461 |
| Community program no additional information  |                        5.082667 |                                      7.527454 |                                      1.4772667 |                                                     2.338569 |                     15 |
| Restaurant with waiter/waitress              |                        4.997763 |                                     12.892353 |                                      2.2433060 |                                                     4.023191 |                  11379 |
| Mail order purchase                          |                        4.468455 |                                     11.840954 |                                      0.9213269 |                                                     2.642477 |                    835 |
| Restaurant no additional information         |                        3.431039 |                                      7.588165 |                                      1.7768182 |                                                     3.541378 |                     77 |
| Fish caught by you or someone you know       |                        1.707647 |                                      3.441011 |                                      3.7097647 |                                                     5.645766 |                     17 |

| Meal Eaten at Home | Average Total Sugar Consumption | Standard Deviation of Total Sugar Consumption | Average Total Saturated Fatty Acid Consumption | Standard Deviation of Total Saturated Fatty Acid Consumption | Number of Observations |
|:-------------------|--------------------------------:|----------------------------------------------:|-----------------------------------------------:|-------------------------------------------------------------:|-----------------------:|
| no                 |                        7.890357 |                                     15.566681 |                                       1.952841 |                                                     3.828746 |                  52001 |
| yes                |                        7.546146 |                                     14.842989 |                                       1.912270 |                                                     3.830932 |                 119689 |
| dont_know          |                        6.498519 |                                      9.579175 |                                       1.636796 |                                                     2.444655 |                     54 |

### Data Visualization

#### What time and/or day of the week do people generally eat foods high in sugar/saturated fats?

Plot sugar consumption versus time.

    ## `geom_smooth()` using method = 'loess'

<img src="README_files/figure-gfm/plot sugar and saturated fa consumption against time-1.png" style="display: block; margin: auto;" />

    ## `geom_smooth()` using method = 'loess'

<img src="README_files/figure-gfm/plot sugar and saturated fa consumption against time-2.png" style="display: block; margin: auto;" />

<img src="README_files/figure-gfm/plot sugar and saturated fa consumption against time grouped by day of the week-1.png" style="display: block; margin: auto;" /><img src="README_files/figure-gfm/plot sugar and saturated fa consumption against time grouped by day of the week-2.png" style="display: block; margin: auto;" />

    ##  [1] "Snack"                "Lunch"                "Breakfast"           
    ##  [4] "Supper"               "Desayano"             "Dinner"              
    ##  [7] "Almuerzo"             "Cena"                 "Drink"               
    ## [10] "Brunch"               "Infant feeding"       "Comida"              
    ## [13] "Bebida"               "Bocadillo"            "Merienda"            
    ## [16] "Entre comida"         "Extended consumption" "Botana"              
    ## [19] "Tentempie"            "Don't know"

<img src="README_files/figure-gfm/plot sugar and saturated fa consumption by eating occasion-1.png" style="display: block; margin: auto;" /><img src="README_files/figure-gfm/plot sugar and saturated fa consumption by eating occasion-2.png" style="display: block; margin: auto;" />

#### Does sugar / saturated fatty acid consumption vary by age, ethnicity, gender?

<img src="README_files/figure-gfm/sugar and sat fa consumption by age-1.png" style="display: block; margin: auto;" /><img src="README_files/figure-gfm/sugar and sat fa consumption by age-2.png" style="display: block; margin: auto;" />

<img src="README_files/figure-gfm/sugar and sat fa consumption by ethnicity-1.png" style="display: block; margin: auto;" /><img src="README_files/figure-gfm/sugar and sat fa consumption by ethnicity-2.png" style="display: block; margin: auto;" />

#### Does the source of the food or whether the meal was eaten at home have an effect?

<img src="README_files/figure-gfm/plot meal eaten at home-1.png" style="display: block; margin: auto;" /><img src="README_files/figure-gfm/plot meal eaten at home-2.png" style="display: block; margin: auto;" />

<img src="README_files/figure-gfm/plot source of food-1.png" style="display: block; margin: auto;" /><img src="README_files/figure-gfm/plot source of food-2.png" style="display: block; margin: auto;" />
