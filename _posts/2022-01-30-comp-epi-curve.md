---
title: Plotting Comparative Covid-19 Incidence with 'ggplot2'
author: "alice"
date: '2022-01-30'
excerpt: "A DIY Epi Curve, Just for Fun"
layout: redirected
toc: true
permalink: /comp-epi-curve/
redirect_to: https://www.alicetivarovsky.com/blog/2022-01-30-comp-epi-curve/
tags:
  - Epidemics
  - Modeling
--- 

## Motivation

Covid-19 - the bug that won't go away. It's hard to believe we've been here for over 2 years. Perhaps like you, I went from checking the Epi curves daily back in 2019, to loneliness and pandemic fatigue, to baking sourdough and planting stuff, to kind of, sort of, almost going back to normal. And then... Omicron. So here we are, back in the familiar bubble.

Speaking of Epi curves, it's very easy to make one of your own - certainly easier than sourdough starter. Here's one using `ggplot2` and the Covid-19 global dataset maintained by [Our World in Data, OWID](https://ourworldindata.org/coronavirus-source-data). OWID compiles this dataset from several sources including the Center for Systems Science and Engineering (CSSE) at Johns Hopkins University, national government reports, and the Human Mortality Database (2021). 


{% highlight r %}
library(tidyverse)
{% endhighlight %}

## Data Preparation

First, let's import and examine the .csv from the "Our World in Data" Github [repo](https://github.com/owid/covid-19-data/tree/master/public/data). 


{% highlight r %}
raw_data = read.csv("../data/epi_curves/owid-covid-data.csv")
str(raw_data)
{% endhighlight %}



{% highlight text %}
## 'data.frame':	157936 obs. of  67 variables:
##  $ iso_code                                  : chr  "AFG" "AFG" "AFG" "AFG" ...
##  $ continent                                 : chr  "Asia" "Asia" "Asia" "Asia" ...
##  $ location                                  : chr  "Afghanistan" "Afghanistan" "Afghanistan" "Afghanistan" ...
##  $ date                                      : chr  "2020-02-24" "2020-02-25" "2020-02-26" "2020-02-27" ...
##  $ total_cases                               : num  5 5 5 5 5 5 5 5 5 5 ...
##  $ new_cases                                 : num  5 0 0 0 0 0 0 0 0 0 ...
##  $ new_cases_smoothed                        : num  NA NA NA NA NA 0.714 0.714 0 0 0 ...
##  $ total_deaths                              : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ new_deaths                                : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ new_deaths_smoothed                       : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ total_cases_per_million                   : num  0.126 0.126 0.126 0.126 0.126 0.126 0.126 0.126 0.126 0.126 ...
##  $ new_cases_per_million                     : num  0.126 0 0 0 0 0 0 0 0 0 ...
##  $ new_cases_smoothed_per_million            : num  NA NA NA NA NA 0.018 0.018 0 0 0 ...
##  $ total_deaths_per_million                  : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ new_deaths_per_million                    : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ new_deaths_smoothed_per_million           : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ reproduction_rate                         : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ icu_patients                              : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ icu_patients_per_million                  : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ hosp_patients                             : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ hosp_patients_per_million                 : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ weekly_icu_admissions                     : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ weekly_icu_admissions_per_million         : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ weekly_hosp_admissions                    : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ weekly_hosp_admissions_per_million        : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ new_tests                                 : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ total_tests                               : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ total_tests_per_thousand                  : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ new_tests_per_thousand                    : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ new_tests_smoothed                        : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ new_tests_smoothed_per_thousand           : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ positive_rate                             : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ tests_per_case                            : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ tests_units                               : chr  "" "" "" "" ...
##  $ total_vaccinations                        : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ people_vaccinated                         : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ people_fully_vaccinated                   : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ total_boosters                            : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ new_vaccinations                          : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ new_vaccinations_smoothed                 : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ total_vaccinations_per_hundred            : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ people_vaccinated_per_hundred             : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ people_fully_vaccinated_per_hundred       : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ total_boosters_per_hundred                : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ new_vaccinations_smoothed_per_million     : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ new_people_vaccinated_smoothed            : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ new_people_vaccinated_smoothed_per_hundred: num  NA NA NA NA NA NA NA NA NA NA ...
##  $ stringency_index                          : num  8.33 8.33 8.33 8.33 8.33 ...
##  $ population                                : num  39835428 39835428 39835428 39835428 39835428 ...
##  $ population_density                        : num  54.4 54.4 54.4 54.4 54.4 ...
##  $ median_age                                : num  18.6 18.6 18.6 18.6 18.6 18.6 18.6 18.6 18.6 18.6 ...
##  $ aged_65_older                             : num  2.58 2.58 2.58 2.58 2.58 ...
##  $ aged_70_older                             : num  1.34 1.34 1.34 1.34 1.34 ...
##  $ gdp_per_capita                            : num  1804 1804 1804 1804 1804 ...
##  $ extreme_poverty                           : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ cardiovasc_death_rate                     : num  597 597 597 597 597 ...
##  $ diabetes_prevalence                       : num  9.59 9.59 9.59 9.59 9.59 9.59 9.59 9.59 9.59 9.59 ...
##  $ female_smokers                            : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ male_smokers                              : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ handwashing_facilities                    : num  37.7 37.7 37.7 37.7 37.7 ...
##  $ hospital_beds_per_thousand                : num  0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 ...
##  $ life_expectancy                           : num  64.8 64.8 64.8 64.8 64.8 ...
##  $ human_development_index                   : num  0.511 0.511 0.511 0.511 0.511 0.511 0.511 0.511 0.511 0.511 ...
##  $ excess_mortality_cumulative_absolute      : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ excess_mortality_cumulative               : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ excess_mortality                          : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ excess_mortality_cumulative_per_million   : num  NA NA NA NA NA NA NA NA NA NA ...
{% endhighlight %}



{% highlight r %}
head(raw_data)
{% endhighlight %}



{% highlight text %}
##   iso_code continent    location       date total_cases new_cases new_cases_smoothed total_deaths
## 1      AFG      Asia Afghanistan 2020-02-24           5         5                 NA           NA
## 2      AFG      Asia Afghanistan 2020-02-25           5         0                 NA           NA
## 3      AFG      Asia Afghanistan 2020-02-26           5         0                 NA           NA
## 4      AFG      Asia Afghanistan 2020-02-27           5         0                 NA           NA
## 5      AFG      Asia Afghanistan 2020-02-28           5         0                 NA           NA
## 6      AFG      Asia Afghanistan 2020-02-29           5         0              0.714           NA
##   new_deaths new_deaths_smoothed total_cases_per_million new_cases_per_million
## 1         NA                  NA                   0.126                 0.126
## 2         NA                  NA                   0.126                 0.000
## 3         NA                  NA                   0.126                 0.000
## 4         NA                  NA                   0.126                 0.000
## 5         NA                  NA                   0.126                 0.000
## 6         NA                  NA                   0.126                 0.000
##   new_cases_smoothed_per_million total_deaths_per_million new_deaths_per_million
## 1                             NA                       NA                     NA
## 2                             NA                       NA                     NA
## 3                             NA                       NA                     NA
## 4                             NA                       NA                     NA
## 5                             NA                       NA                     NA
## 6                          0.018                       NA                     NA
##   new_deaths_smoothed_per_million reproduction_rate icu_patients icu_patients_per_million
## 1                              NA                NA           NA                       NA
## 2                              NA                NA           NA                       NA
## 3                              NA                NA           NA                       NA
## 4                              NA                NA           NA                       NA
## 5                              NA                NA           NA                       NA
## 6                              NA                NA           NA                       NA
##   hosp_patients hosp_patients_per_million weekly_icu_admissions weekly_icu_admissions_per_million
## 1            NA                        NA                    NA                                NA
## 2            NA                        NA                    NA                                NA
## 3            NA                        NA                    NA                                NA
## 4            NA                        NA                    NA                                NA
## 5            NA                        NA                    NA                                NA
## 6            NA                        NA                    NA                                NA
##   weekly_hosp_admissions weekly_hosp_admissions_per_million new_tests total_tests
## 1                     NA                                 NA        NA          NA
## 2                     NA                                 NA        NA          NA
## 3                     NA                                 NA        NA          NA
## 4                     NA                                 NA        NA          NA
## 5                     NA                                 NA        NA          NA
## 6                     NA                                 NA        NA          NA
##   total_tests_per_thousand new_tests_per_thousand new_tests_smoothed new_tests_smoothed_per_thousand
## 1                       NA                     NA                 NA                              NA
## 2                       NA                     NA                 NA                              NA
## 3                       NA                     NA                 NA                              NA
## 4                       NA                     NA                 NA                              NA
## 5                       NA                     NA                 NA                              NA
## 6                       NA                     NA                 NA                              NA
##   positive_rate tests_per_case tests_units total_vaccinations people_vaccinated
## 1            NA             NA                             NA                NA
## 2            NA             NA                             NA                NA
## 3            NA             NA                             NA                NA
## 4            NA             NA                             NA                NA
## 5            NA             NA                             NA                NA
## 6            NA             NA                             NA                NA
##   people_fully_vaccinated total_boosters new_vaccinations new_vaccinations_smoothed
## 1                      NA             NA               NA                        NA
## 2                      NA             NA               NA                        NA
## 3                      NA             NA               NA                        NA
## 4                      NA             NA               NA                        NA
## 5                      NA             NA               NA                        NA
## 6                      NA             NA               NA                        NA
##   total_vaccinations_per_hundred people_vaccinated_per_hundred people_fully_vaccinated_per_hundred
## 1                             NA                            NA                                  NA
## 2                             NA                            NA                                  NA
## 3                             NA                            NA                                  NA
## 4                             NA                            NA                                  NA
## 5                             NA                            NA                                  NA
## 6                             NA                            NA                                  NA
##   total_boosters_per_hundred new_vaccinations_smoothed_per_million new_people_vaccinated_smoothed
## 1                         NA                                    NA                             NA
## 2                         NA                                    NA                             NA
## 3                         NA                                    NA                             NA
## 4                         NA                                    NA                             NA
## 5                         NA                                    NA                             NA
## 6                         NA                                    NA                             NA
##   new_people_vaccinated_smoothed_per_hundred stringency_index population population_density
## 1                                         NA             8.33   39835428             54.422
## 2                                         NA             8.33   39835428             54.422
## 3                                         NA             8.33   39835428             54.422
## 4                                         NA             8.33   39835428             54.422
## 5                                         NA             8.33   39835428             54.422
## 6                                         NA             8.33   39835428             54.422
##   median_age aged_65_older aged_70_older gdp_per_capita extreme_poverty cardiovasc_death_rate
## 1       18.6         2.581         1.337       1803.987              NA               597.029
## 2       18.6         2.581         1.337       1803.987              NA               597.029
## 3       18.6         2.581         1.337       1803.987              NA               597.029
## 4       18.6         2.581         1.337       1803.987              NA               597.029
## 5       18.6         2.581         1.337       1803.987              NA               597.029
## 6       18.6         2.581         1.337       1803.987              NA               597.029
##   diabetes_prevalence female_smokers male_smokers handwashing_facilities hospital_beds_per_thousand
## 1                9.59             NA           NA                 37.746                        0.5
## 2                9.59             NA           NA                 37.746                        0.5
## 3                9.59             NA           NA                 37.746                        0.5
## 4                9.59             NA           NA                 37.746                        0.5
## 5                9.59             NA           NA                 37.746                        0.5
## 6                9.59             NA           NA                 37.746                        0.5
##   life_expectancy human_development_index excess_mortality_cumulative_absolute
## 1           64.83                   0.511                                   NA
## 2           64.83                   0.511                                   NA
## 3           64.83                   0.511                                   NA
## 4           64.83                   0.511                                   NA
## 5           64.83                   0.511                                   NA
## 6           64.83                   0.511                                   NA
##   excess_mortality_cumulative excess_mortality excess_mortality_cumulative_per_million
## 1                          NA               NA                                      NA
## 2                          NA               NA                                      NA
## 3                          NA               NA                                      NA
## 4                          NA               NA                                      NA
## 5                          NA               NA                                      NA
## 6                          NA               NA                                      NA
{% endhighlight %}

The dataset is clean and in wide format. Countries are contained in the `location` variable (with corresponding values for `iso_code` given), and we have daily values for quite a few variables including `new_cases` (we'll use this to plot disease incidence), `new_cases_per_million`, `new_tests`, `total_vaccinations` and booster data is now available as `total_boosters`.

## Analysis

Let's first plot disease incidence using the `new_cases` variable.

{% highlight r %}
us_data = 
  raw_data %>% 
  filter(location == "United States") %>% 
  mutate(date = as.Date(date)) 

ggplot(us_data, aes(x = date, y = new_cases)) +
  geom_point()
{% endhighlight %}

![center](/figs/2022-01-30-comp-epi-curve/new_cases plot-1.png)

It looks a little crazy, almost like there are several superimposed curves. What's really happening is that we're seeing a lot of noise from day-to-day fluctuation. Data aren't reported uniformly, so it's not uncommon to observe a "spike" in reports on, say, Mondays when reporting for the weekend is done. 

For this reason, we used smoothed data, usually amounting to a 7-day average. This technique allows us to eliminate the noise attributable to reporting fluctuations through the week. OWID gives us a smoothed version of `new_cases` which achieves just that. 


{% highlight r %}
us_plot = 
  ggplot(us_data, aes(x = date, y = new_cases_smoothed)) +
  geom_point() + 
  labs(title = "Covid-19 Incidence in the United States")

us_plot
{% endhighlight %}

![center](/figs/2022-01-30-comp-epi-curve/us plot-1.png)

Much better. Let's now compare the US to a few other (arbitrarily chosen) countries. Here's Peru. 


{% highlight r %}
peru_plot = 
  raw_data %>% 
  filter(location == "Peru") %>% 
  mutate(date = as.Date(date)) %>% 
  ggplot(aes(x = date, y = new_cases_smoothed)) +
  geom_point() + 
  labs(title = "Covid-19 Incidence in Peru")

peru_plot
{% endhighlight %}

![center](/figs/2022-01-30-comp-epi-curve/peru-1.png)


And here's Japan. 


{% highlight r %}
japan_plot = 
  raw_data %>% 
  filter(location == "Japan") %>% 
  mutate(date = as.Date(date)) %>% 
  ggplot(aes(x = date, y = new_cases_smoothed)) +
  geom_point() + 
  labs(title = "Covid-19 Incidence in Japan")

japan_plot
{% endhighlight %}

![center](/figs/2022-01-30-comp-epi-curve/japan plot-1.png)


Here's Australia. 


{% highlight r %}
aus_plot = 
  raw_data %>% 
  filter(location == "Australia") %>% 
  mutate(date = as.Date(date)) %>% 
  ggplot(aes(x = date, y = new_cases_smoothed)) +
  geom_point() + 
  labs(title = "Covid-19 Incidence in Australia")

aus_plot
{% endhighlight %}

![center](/figs/2022-01-30-comp-epi-curve/australia plot-1.png)

Here's Zimbabwe. 


{% highlight r %}
zimbabwe_plot = 
  raw_data %>% 
  filter(location == "Zimbabwe") %>% 
  mutate(date = as.Date(date)) %>% 
  ggplot(aes(x = date, y = new_cases_smoothed)) +
  geom_point() + 
  labs(title = "Covid-19 Incidence in Zimbabwe")

zimbabwe_plot
{% endhighlight %}

![center](/figs/2022-01-30-comp-epi-curve/zimbabwe plot-1.png)


Finally, here's Turkey. 


{% highlight r %}
turkey_plot = 
  raw_data %>% 
  filter(location == "Turkey") %>% 
  mutate(date = as.Date(date)) %>% 
  ggplot(aes(x = date, y = new_cases_smoothed)) +
  geom_point() + 
  labs(title = "Covid-19 Incidence in Turkey")

turkey_plot
{% endhighlight %}

![center](/figs/2022-01-30-comp-epi-curve/turkey plot-1.png)

It's evident that Omicron has a foothold all over the world, but to really compare the penetration we'll need to scale the case loads to the population. Again, OWID gives us a variable where cases/population is already calculated: `new_cases_per_million` and `new_cases_smoothed_per_million`. We'll used the smoothed version and combine the incidence of all six countries onto one plot. 


{% highlight r %}
combined_plot = 
  raw_data %>% 
  filter(location %in% c("United States", "Peru", "Japan", "Australia", "Zimbabwe", "Turkey")) %>% 
  mutate(date = as.Date(date)) %>% 
  ggplot(aes(x = date, y = new_cases_smoothed_per_million, group = location)) +
  geom_line(aes(color = location))

combined_plot
{% endhighlight %}

![center](/figs/2022-01-30-comp-epi-curve/new cases per population combined-1.png)

Interestingly, Australia has had the highest Omicron peak, almost double that of the US. This is particularly unfortunate, given Australia's [ultra-high vaccination rate](https://www.aljazeera.com/news/2022/1/10/australia-to-push-through-omicron-as-total-cases-hit-1-million). 

Now, having learned something, let's polish up the curves. 


{% highlight r %}
final_plot = 
  combined_plot +
  labs(title = "Covid-19 Incidence in Australia, Japan, Peru, Turkey, the United States, and Zimbabwe, Cases per Million, Smoothed 7-Day Average", 
       x = "Date", 
       y = "Cases per Million")

final_plot + theme_minimal()
{% endhighlight %}

![center](/figs/2022-01-30-comp-epi-curve/final plot-1.png)


## Conclusion

While you can certainly just google Covid-19 in any country that interests you, getting to an accurate, scaled comparison between countries probably requires a bit more effort. Thankfully, Our World in Data and `ggplot2` make the task entirely painless. 



