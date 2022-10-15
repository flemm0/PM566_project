PM566 Midterm Project
================
Flemming Wu
2022-10-15

Main source of data:

<https://www.ars.usda.gov/northeast-area/beltsville-md-bhnrc/beltsville-human-nutrition-research-center/food-surveys-research-group/docs/wweia-documentation-and-data-sets/>

Questions:  
Insulin resistance and diabetes is a growing health issue for Americans.
When foods with a high glycemic index (causing a rapid rise in blood
sugar) are consumed, the pancreas must pump insulin to move sugar from
the blood back into the cells. Over time, if these foods are constantly
being consumed, the cells stop responding to insulin and the normal
blood sugar level rises. This leads to weight gain, as excess blood
sugar is sent to be stored as body fat, and sets the stage for
prediabetes and type 2 diabetes.  
While there are many other factors that influence the development of
insulin resistance and diabetes such as lifestyle, environmental
factors, and genetics, in this project, I will only be investigating
factors affecting our food choices using the NHANES (National Health and
Nutrition Examination Survey) data. More specifically, I examined the
data that was collected in What We Eat in America (WWEIA), the dietary
interview component of the NHANES. I will use the data to investigate
the following questions that I have asked:  

-   What time and/or day of the week do people generally eat foods high
    in sugar or saturated fatty acids (FA)?
-   Does sugar or saturated fa consumption vary by age, ethnicity, or
    gender?
-   Does the source of food or whether the meal was eaten at home have
    an effect on sugar or saturated fa consumption?

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
saturated fat consumption is slightly higher on the weekends, and are
closely followed by Friday and Wednesday.

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

From the summary table above, which groups sugar and saturated fat
consumption by the hours that they were consumed, it can be seen that
higher sugar and saturated fat consumption occurs between the hours of
8PM to 2AM. Additionally, 3PM and 4PM also had slightly higher sugar and
saturated fat consumption.

| Eating Occasion      | Average Total Sugar Consumption | Standard Deviation of Total Sugar Consumption | Average Total Saturated Fatty Acid Consumption | Standard Deviation of Total Saturated Fatty Acid Consumption | Number of Observations |
|:---------------------|--------------------------------:|----------------------------------------------:|-----------------------------------------------:|-------------------------------------------------------------:|-----------------------:|
| Don’t know           |                       20.900000 |                                      0.000000 |                                      1.0370000 |                                                     0.000000 |                      2 |
| Extended consumption |                       15.721413 |                                     50.865800 |                                      0.6547581 |                                                     3.519036 |                   2534 |
| Bocadillo            |                       10.968052 |                                     16.025106 |                                      1.7000406 |                                                     3.950243 |                    616 |
| Snack                |                       10.751874 |                                     16.167832 |                                      2.0776491 |                                                     4.102707 |                  29089 |
| Tentempie            |                        9.664634 |                                     10.323859 |                                      1.7000976 |                                                     2.823390 |                     41 |
| Merienda             |                        9.470025 |                                     14.209530 |                                      1.4602024 |                                                     3.090206 |                   1186 |
| Botana               |                        9.186853 |                                     12.589935 |                                      1.4895198 |                                                     3.466799 |                    429 |
| Drink                |                        8.943371 |                                     18.033214 |                                      0.4381726 |                                                     1.939871 |                   8423 |
| Entre comida         |                        8.335434 |                                     14.733579 |                                      1.3515717 |                                                     3.758018 |                    495 |
| Infant feeding       |                        7.545321 |                                      6.038356 |                                      1.5679247 |                                                     1.438459 |                   2988 |
| Brunch               |                        7.512050 |                                     13.522190 |                                      2.3093810 |                                                     4.393188 |                   2005 |
| Breakfast            |                        7.498159 |                                     12.440818 |                                      1.6602312 |                                                     3.106925 |                  31741 |
| Supper               |                        6.799386 |                                     13.952740 |                                      2.5479036 |                                                     4.750105 |                   9388 |
| Cena                 |                        6.745963 |                                     12.785199 |                                      1.9265675 |                                                     4.209928 |                   2992 |
| Bebida               |                        6.648814 |                                     14.167280 |                                      0.3213970 |                                                     1.120434 |                    801 |
| Desayano             |                        6.601639 |                                     10.990466 |                                      1.4365567 |                                                     2.791270 |                   3032 |
| Lunch                |                        6.444327 |                                     13.067584 |                                      2.0005485 |                                                     3.759927 |                  34174 |
| Dinner               |                        6.088390 |                                     12.764289 |                                      2.3426095 |                                                     4.315730 |                  37289 |
| Almuerzo             |                        6.040902 |                                     12.888273 |                                      1.8254938 |                                                     3.930899 |                   2351 |
| Comida               |                        5.996550 |                                     12.452777 |                                      1.9596370 |                                                     4.220748 |                   2168 |

From the summary table above, we can see that eating occasions that are
more considered to be more formal meals, such as dinner, lunch,
almuerzo, desayano, supper, etc. generally involve less consumption of
sugar than do informal eating occasions such as snacks. The reverse is
true for saturated fatty acid consumption, as the average grams consumed
for these are higher in more formal eating occasions.

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

The summary table above shows that average sugar and fatty acid
consumption increases from birth until the ages of 14-18, and then
decreases after age 18.

| Gender | Average Total Sugar Consumption | Standard Deviation of Total Sugar Consumption | Average Total Saturated Fatty Acid Consumption | Standard Deviation of Total Saturated Fatty Acid Consumption | Number of Observations |
|:-------|--------------------------------:|----------------------------------------------:|-----------------------------------------------:|-------------------------------------------------------------:|-----------------------:|
| female |                        6.843890 |                                      13.02220 |                                       1.680465 |                                                     3.289837 |                  87533 |
| male   |                        8.487986 |                                      16.88883 |                                       2.178095 |                                                     4.305765 |                  84211 |

From the table above, it can be seen that males consume higher amounts
of sugar and saturated fatty acids on average.

| Race                        | Average Total Sugar Consumption | Standard Deviation of Total Sugar Consumption | Average Total Saturated Fatty Acid Consumption | Standard Deviation of Total Saturated Fatty Acid Consumption | Number of Observations |
|:----------------------------|--------------------------------:|----------------------------------------------:|-----------------------------------------------:|-------------------------------------------------------------:|-----------------------:|
| non-hispanic_asian          |                        5.518388 |                                      10.54705 |                                       1.481301 |                                                     3.024665 |                  18008 |
| mexican_american            |                        7.306042 |                                      13.27055 |                                       1.878062 |                                                     4.020707 |                  22247 |
| non-hispanic_white          |                        7.863279 |                                      16.19630 |                                       2.024360 |                                                     3.871168 |                  61381 |
| non-hispanic_black          |                        8.394050 |                                      15.73418 |                                       2.038325 |                                                     4.074282 |                  42203 |
| other_race_incl_multiracial |                        8.607685 |                                      16.77969 |                                       2.084585 |                                                     3.934074 |                  10509 |
| other_hispanic              |                        7.160682 |                                      14.03439 |                                       1.717158 |                                                     3.441850 |                  17396 |

The groups with the highest average sugar and saturated fatty acid
consumption are “other race including multiracial” and “non-hispanic
black”, and they are closely followed by “non-hispanic white”.

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

High average sugar consumption can be seen to come from food sources
such as vending machines and convenience stores, likely due to the sale
of sugar-sweetened beverages. High saturated fatty acid consumption can
be seen to come from food sources such as fish (due to naturally
occurring omega fatty acids in fish) as well as fast food restaurants
and recreational facilities.

| Meal Eaten at Home | Average Total Sugar Consumption | Standard Deviation of Total Sugar Consumption | Average Total Saturated Fatty Acid Consumption | Standard Deviation of Total Saturated Fatty Acid Consumption | Number of Observations |
|:-------------------|--------------------------------:|----------------------------------------------:|-----------------------------------------------:|-------------------------------------------------------------:|-----------------------:|
| no                 |                        7.890357 |                                     15.566681 |                                       1.952841 |                                                     3.828746 |                  52001 |
| yes                |                        7.546146 |                                     14.842989 |                                       1.912270 |                                                     3.830932 |                 119689 |
| dont_know          |                        6.498519 |                                      9.579175 |                                       1.636796 |                                                     2.444655 |                     54 |

The table above reveals that food not eaten at home is generally
slightly higher in average sugars and saturated fatty acids.

### Data Visualization

#### What time and/or day of the week do people generally eat foods high in sugar/saturated fats?

<img src="README_files/figure-gfm/plot sugar and saturated fa consumption against time-1.png" style="display: block; margin: auto;" /><img src="README_files/figure-gfm/plot sugar and saturated fa consumption against time-2.png" style="display: block; margin: auto;" />
The plots above reveal that average sugar consumption increases as the
day progresses, with large spikes at around 4:00 PM, 6:00 PM, 9:00 PM
and 4:00 AM. A similar trend can be observed for saturated fatty acid
consumption, with a large spike at around 12:00 PM and 10:30 PM.

<img src="README_files/figure-gfm/plot sugar and saturated fa consumption against time grouped by day of the week-1.png" style="display: block; margin: auto;" /><img src="README_files/figure-gfm/plot sugar and saturated fa consumption against time grouped by day of the week-2.png" style="display: block; margin: auto;" />
The plots above reveal that sugar consumption is higher towards the
evening for Monday through Wednesday. Additionally, there are spikes in
sugar consumption in the morning around 9:30 AM on Wednesday and 5:30 AM
on Thursday. Averages for sugar consumption tend to be more consistent
throughout the 24 hour period on Friday, Saturday, and Sunday.  
Averages for saturated fatty acid consumption are generally more
consistent than sugar consumption, but there is an interesting large
spike in consumption towards the end of the day on Wednesday. It can
also be noted that there are more spikes in saturated fatty acid
consumption on Saturday and Sunday.

<img src="README_files/figure-gfm/plot sugar and saturated fa consumption by eating occasion-1.png" style="display: block; margin: auto;" /><img src="README_files/figure-gfm/plot sugar and saturated fa consumption by eating occasion-2.png" style="display: block; margin: auto;" />
The plots above reveal that average sugar consumption is lowest in meals
eaten later in the day such as lunch and dinner, increases for earlier
meals such as breakfast or brunch, and is highest in the snack and
extended consumption categories. For average saturated fatty acid
consumption, the trend can be described as an increase in consumption
with more formal meal categories, such as brunch, dinner, and supper.
However, snacking is also associated with higher average saturated fatty
acid consumption. As expected, bebida, or drinks, are associated with
the lowest saturated fatty acid consumption.

#### Does sugar / saturated fatty acid consumption vary by age, ethnicity, gender?

<img src="README_files/figure-gfm/sugar and sat fa consumption by age-1.png" style="display: block; margin: auto;" /><img src="README_files/figure-gfm/sugar and sat fa consumption by age-2.png" style="display: block; margin: auto;" />
The violin plots above reveal several trends in average sugar and
saturated fatty acid consumption: first, that consumption is higher on
average for males than females. Secondly, the trend in average
consumption increases until age 18, and then decreases with age. Lastly,
the range of values tends to increase with increasing age, especially so
for males.

<img src="README_files/figure-gfm/sugar and sat fa consumption by ethnicity-1.png" style="display: block; margin: auto;" /><img src="README_files/figure-gfm/sugar and sat fa consumption by ethnicity-2.png" style="display: block; margin: auto;" />
In grouping average sugar and saturated fatty acid consumption by
ethnicity, there is not much variation between ethnicity groups and
sugar consumption, however, the lowest average sugar and saturated fatty
acid consumption and range of values is lowest in the non hispanic asian
group, and highest in non hispanic black group.

#### Does the source of the food or whether the meal was eaten at home have an effect?

<img src="README_files/figure-gfm/plot meal eaten at home-1.png" style="display: block; margin: auto;" /><img src="README_files/figure-gfm/plot meal eaten at home-2.png" style="display: block; margin: auto;" />
The plots above graph the log normalized distribution of sugar and
saturated fatty acid in foods eaten at home, not at home and unknown. It
can be seen that the distributions of foods eaten at home and not at
home are essentially the same for both sugar and saturated fatty acids.

<img src="README_files/figure-gfm/plot source of food-1.png" style="display: block; margin: auto;" /><img src="README_files/figure-gfm/plot source of food-2.png" style="display: block; margin: auto;" />

![](README_files/figure-gfm/plot%20correlation%20between%20total%20saturated%20fatty%20acid%20consumption%20and%20sugar%20consumption-1.png)<!-- -->

    ## [1] 0.4862645
