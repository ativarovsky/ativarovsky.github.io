---
title: Visualizing Motor Vehicle Fatalities with ggplot2
author: "alice"
date: '2020-07-17'
layout: single
htmlwidgets: true
toc: true
permalink: /car-crashes/
tags:
  - R
  - data visualization
  - ggplot2
  - injuries
---

An analysis of car fatalities by state using data from the Big Cities Health Initiative. 

_"This is my favorite part about analytics: Taking boring flat data and bringing it to life through visualization." -- John Tukey_

## Motivation

The last two posts were a bit dense so we'll try to keep it simple here. 

Let's start with a morbid stat: if you're younger than 30 and live in the United States, your most probable cause of death is a car crash. 

You may have already known that. But here's a kicker - your risk of dying in a car crash is heavily influenced by the city you live in. And although metropolitan density is an obvious factor, other variables including weather, state policy, type of car involved, and emergency response capabilities play a [huge role](https://www.iihs.org/topics/fatality-statistics/detail/state-by-state#fn1ref1).

In this post, we'll be using `ggplot2` - R's de-facto plotting package [^1] - to explore the relationship between city and motor vehicle fatalities in the US between 2010 and 2016.

We'll be using data compiled from the [Big Cities Health Coalition](https://www.bigcitieshealth.org), a partnership organization comprised of public health leaders and thinkers from America's largest cities. I'd never heard of BCHC prior to this and I was genuinely impressed with the wealth of data and analyses published on their site. 

## Background

In the 1950's, the United States mortality rate due to motor vehicle crashes was about [22 per 100,000](https://web.archive.org/web/20110921222129/http://www.saferoads.org/federal/2004/TrafficFatalities1899-2003.pdf) population. That proportion peaked at about 26 in the early 1970's and has since been steadily falling. Despite a somewhat alarming uptick in 2015, the proportion today hovers at [11 per 100,000](https://web.archive.org/web/20110921222129/http://www.saferoads.org/federal/2004/TrafficFatalities1899-2003.pdf). 

While that's undoubtedly huge progress, it still means that more than [35,000](https://www.cdc.gov/winnablebattles/report/motor.html) people die as a result of a motor vehicle crash every year. According to the [CDC](https://www.cdc.gov/vitalsigns/motor-vehicle-safety/index.html), about 1/3 of fatalities involve drunk drivers, another 1/3 involve drivers or passengers where proper restraints were not used (meaning seat belts, car seats, etc.), and another 1/3 involve drivers that were speeding. 

Another somewhat sad statistic is that the reduction in motor vehicle fatalities in the U.S. has not kept pace with that of other high income countries. On average, high income countries reduced crash deaths by 56% between 2000 and 2013, compared with 31% in the U.S [CDC](https://www.cdc.gov/vitalsigns/motor-vehicle-safety/index.html). Today, the mortality rate in the U.S. (the 11 per 100,000 mentioned above) is by far the highest among high-income countries. 

Interestingly, states that are predominantly rural tend to have higher vehicle mortality rates. Mississippi (22.5 deaths/100,000 population), South Carolina (20.4 deaths/100,000 population), and Alabama (19.5 deaths/100,000 population) represent the highest rates, while New York (4.8 deaths/100,000 population), Massachusetts (5.2 deaths/100,000 population), and New Jersey (6.3 deaths/100,000 population) represent three of the lowest [IIHS](https://www.iihs.org/topics/fatality-statistics/detail/state-by-state). However, this divide is much less clear when we drill down to individual cities. Cary, NC, for instance, is one of the safest cities for drivers (1.3 fatal accidents per 100,000 population), while Detroit is one of the most dangerous (16.2 fatal accidents per 100,000 population) [Nerdwallet](https://www.nerdwallet.com/blog/insurance/dangerous-cities-car-drivers-2016/). 

Ok, enough sad statistics. Let's look at some data. 


## Data Preparation


### Data Source

These data were taken from the [data platform page](https://bchi.bigcitieshealth.org/indicators/1872/searches/34543) on the Big Cities Health Coalition [website](https://www.bigcitieshealth.org).

### Libraries

```r
library(tidyverse)
library(viridis)
library(plotly)
```


### Data Import

According to the website, this dataset was updated in March 2019. As loaded, the dataset contains 34,492 rows and 15 variables. We filter for only the motor vehicle fatality portion, which leaves us with 880 observations. 


```r
drive_df = read.csv(file = "../data/BCHI_dataset.csv") %>% 
  janitor::clean_names() %>% 
  filter(indicator == "Motor Vehicle Mortality Rate (Age-Adjusted; Per 100,000 people)")
```

### Data Tidy

We want to make sure that the variable types R gives us are correct, variables are appropriately named, and that everything we expected in the dataframe is indeed there. We also do a quick check to make sure the data are in wide format, wherein each observation is represented by a row and each variable measured for that observation is given in a column. 


```r
str(drive_df)
```

```
## 'data.frame':	880 obs. of  15 variables:
##  $ indicator_category        : Factor w/ 13 levels "Behavioral Health/Substance Abuse",..: 9 9 9 9 9 9 9 9 9 9 ...
##  $ indicator                 : Factor w/ 61 levels "AIDS Diagnoses Rate (Per 100,000 people)",..: 21 21 21 21 21 21 21 21 21 21 ...
##  $ year                      : int  2010 2010 2010 2010 2010 2010 2010 2010 2010 2010 ...
##  $ sex                       : Factor w/ 3 levels "Both","Female",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ race_ethnicity            : Factor w/ 9 levels "All","All ","American Indian/Alaska Native",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ value                     : num  2.6 3.9 4.3 5.5 5.6 5.8 6.1 6.5 6.7 6.7 ...
##  $ place                     : Factor w/ 31 levels "Austin, TX","Baltimore, MD",..: 27 29 17 21 31 5 26 19 13 22 ...
##  $ bchc_requested_methodology: Factor w/ 184 levels "Age-adjusted rate of asthma-related ED visits using ICD-9-CM Codes: 493.0, 493.1, 493.2, 493.8, 493.0. (Numerat"| __truncated__,..: 133 134 133 134 133 133 133 133 133 133 ...
##  $ source                    : Factor w/ 659 levels "","(500 Cities Project)",..: 1 649 292 33 1 1 93 324 306 495 ...
##  $ methods                   : Factor w/ 291 levels "","\"Vaccinated for flu in the past 12 months\"",..: 1 1 31 275 1 1 86 237 1 1 ...
##  $ notes                     : Factor w/ 483 levels ""," Records where the value is blank in the data table indicate that the data are suppressed due to small counts, "| __truncated__,..: 472 1 1 1 1 1 1 1 1 1 ...
##  $ x90_confidence_level_low  : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ x90_confidence_level_high : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ x95_confidence_level_low  : num  2 2.5 NA 3.5 NA NA NA NA 5.1 5.4 ...
##  $ x95_confidence_level_high : num  3.4 6 NA 8.3 NA NA NA NA 8.7 7.9 ...
```

It looks like things are pretty clean. Most of the categorical variables, including race, sex, and city were read in as factors, while the mortality values and corresponding confidence intervals were read in using the double class. The only thing to fix is the `source`, `methods`, and `notes` variables, which should technically be strings since they don't contain any logical levels. I'm also going to change `year` to a categorical variable so R doesn't get confused trying to analyze it continuously. 


```r
drive_df = 
  drive_df %>% 
  mutate(year = as.factor(year), 
         source = as.character(source), 
         methods = as.character(methods),
         notes = as.character(notes)
  )
```


### Variables Used

The variables we care about here are: 
- `year`: 2010 through 2016
- `sex`: Male, female, and the combined value under Both
- `race_ethnicity`: American Indian/Alaska Native,  Asian/PI, Black, Hispanic, Other, White, and combined valued (All)
- `value`: the mortality value we'll be analyzing (i.e. our dependent variable); the details are given under `bchc_requested_methodology`, which explains that `value` is equal to motor vehicle deaths per 100,000 population using 2010 US Census figures, age adjusted to the 2000 census standard population. [^2]
- `x90_confidence_level_low` and `x90_confidence_level_high`: 90% confidence interval around `value`
- `x95_confidence_level_low` and `x95_confidence_level_high`: 95% confidence interval around `value`

## Visualizing Motor Vehicle Fatalities

`ggplot2` works by layering graphical elements on top of one another. These layers are referred to as "geoms" and they include things like axes, grid lines, color schemes points representing your data, trend-lines connecting your data, shaded regions representing confidence intervals, and plot labels, titles, and legends. Creating a final graph requires you to define each element individually and "add" it to the `ggplot()` call. 

The best way to explain `ggplot2` is, of course, to demonstrate it. Let's first just apply the function to our dataframe `drive_df` to see what happens. 


```r
drive_df %>% 
  ggplot()
```

![plot of chunk unnamed-chunk-1](/figs/2020-07-17-car-crashes/unnamed-chunk-1-1.png)

We get this lovely gray square. Who doesn't love gray squares? 

This may seem silly but the point is to demonstrate that `ggplot2` needs you to state what you want. 

The first thing we need to do is tell `ggplot2` what our dependent and independent variables are. We're interested in motor vehicle mortality year over year, so let's try the following:


```r
drive_df %>% 
  ggplot(x = year, y = value)
```

![plot of chunk unnamed-chunk-2](/figs/2020-07-17-car-crashes/unnamed-chunk-2-1.png)

Wait what?

And this is one of the first lessons of `ggplot2`. In order to control how things look within a geom, you have to set what are known as __aesthetics__.

### Aesthetics

Aesthetics set within the ggplot function itself (i.e. before geoms are defined) are global aesthetics that apply to the entire plot. Aesthetics set within a geom apply only to that geom. Therefore, when we define axes, we want to set them globally within a `ggplot(aes()` call: 


```r
drive_df %>% 
  ggplot(aes(x = year, y = value))
```

![plot of chunk unnamed-chunk-3](/figs/2020-07-17-car-crashes/unnamed-chunk-3-1.png)

Ah, now we're getting somewhere. 

Let's attempt to create a plot of motor vehicle mortality as a function of year. We know that we have data from 2010 through 2016 and that the data are collected (or compiled) annually. In order to get some points for each year, we'll need to "add" a point geom to `ggplot2`: 


```r
drive_df %>% 
  ggplot(aes(x = year, y = value)) + 
  geom_point()
```

![plot of chunk unnamed-chunk-4](/figs/2020-07-17-car-crashes/unnamed-chunk-4-1.png){: height="600px" width="600px"}

Clearly this is a pretty bad plot and if you ever see something like this published, please assume that the author doesn't know much about plots. For one thing, using a scatterplot doesn't make sense because the year variable was measured categorically rather than continuously. BUT... this does show us something important in the data: __outliers__. On the left, we have points hovering at 70-75 deaths per 100,000. Recall that in the 1970s, when motor vehicle deaths peaked, the average values reached into the mid-20s. So clearly, 75 is astronomical. Let's take a detour and investigate this a bit more. I'm going to go back to the dataframe and sort in order of decreasing `value`: 


```r
drive_df %>% 
  arrange(desc(value)) %>% 
  head()
```

```
##   indicator_category
## 1    Injury/Violence
## 2    Injury/Violence
## 3    Injury/Violence
## 4    Injury/Violence
## 5    Injury/Violence
## 6    Injury/Violence
##                                                         indicator year  sex
## 1 Motor Vehicle Mortality Rate (Age-Adjusted; Per 100,000 people) 2011 Both
## 2 Motor Vehicle Mortality Rate (Age-Adjusted; Per 100,000 people) 2012 Both
## 3 Motor Vehicle Mortality Rate (Age-Adjusted; Per 100,000 people) 2010 Both
## 4 Motor Vehicle Mortality Rate (Age-Adjusted; Per 100,000 people) 2012 Both
## 5 Motor Vehicle Mortality Rate (Age-Adjusted; Per 100,000 people) 2016 Both
## 6 Motor Vehicle Mortality Rate (Age-Adjusted; Per 100,000 people) 2012 Both
##                  race_ethnicity value           place
## 1                         Other  74.4      Denver, CO
## 2                         Other  74.4      Denver, CO
## 3 American Indian/Alaska Native  70.5 Minneapolis, MN
## 4 American Indian/Alaska Native  39.4 Minneapolis, MN
## 5                      Hispanic  35.6 Minneapolis, MN
## 6                      Hispanic  34.1 Minneapolis, MN
##                                                                                                                                                                                                                                                                             bchc_requested_methodology
## 1 Motor vehicle deaths per 100,000 population using 2010 US Census figures, age adjusted to the year 2000 standard population. ICD-10 Codes: V02-V04, V09.0, V09.2, V12-V14, V19.0-V19.2, V19.4-V19.6, V20-V79, V80.3-V80.5, V81.0-V81.1, V82.0-V82.1, V83-V86, V87.0-V87.8, V88.0-V88.8, V89.0, V89.2
## 2 Motor vehicle deaths per 100,000 population using 2010 US Census figures, age adjusted to the year 2000 standard population. ICD-10 Codes: V02-V04, V09.0, V09.2, V12-V14, V19.0-V19.2, V19.4-V19.6, V20-V79, V80.3-V80.5, V81.0-V81.1, V82.0-V82.1, V83-V86, V87.0-V87.8, V88.0-V88.8, V89.0, V89.2
## 3 Motor vehicle deaths per 100,000 population using 2010 US Census figures, age adjusted to the year 2000 standard population. ICD-10 Codes: V02-V04, V09.0, V09.2, V12-V14, V19.0-V19.2, V19.4-V19.6, V20-V79, V80.3-V80.5, V81.0-V81.1, V82.0-V82.1, V83-V86, V87.0-V87.8, V88.0-V88.8, V89.0, V89.2
## 4 Motor vehicle deaths per 100,000 population using 2010 US Census figures, age adjusted to the year 2000 standard population. ICD-10 Codes: V02-V04, V09.0, V09.2, V12-V14, V19.0-V19.2, V19.4-V19.6, V20-V79, V80.3-V80.5, V81.0-V81.1, V82.0-V82.1, V83-V86, V87.0-V87.8, V88.0-V88.8, V89.0, V89.2
## 5 Motor vehicle deaths per 100,000 population using 2010 US Census figures, age adjusted to the year 2000 standard population. ICD-10 Codes: V02-V04, V09.0, V09.2, V12-V14, V19.0-V19.2, V19.4-V19.6, V20-V79, V80.3-V80.5, V81.0-V81.1, V82.0-V82.1, V83-V86, V87.0-V87.8, V88.0-V88.8, V89.0, V89.2
## 6 Motor vehicle deaths per 100,000 population using 2010 US Census figures, age adjusted to the year 2000 standard population. ICD-10 Codes: V02-V04, V09.0, V09.2, V12-V14, V19.0-V19.2, V19.4-V19.6, V20-V79, V80.3-V80.5, V81.0-V81.1, V82.0-V82.1, V83-V86, V87.0-V87.8, V88.0-V88.8, V89.0, V89.2
##                              source
## 1 Colorado Vital Records Death Data
## 2 Colorado Vital Records Death Data
## 3        Minnesota Vital Statistics
## 4        Minnesota Vital Statistics
## 5        Minnesota Vital Statistics
## 6        Minnesota Vital Statistics
##                                                                                                                         methods
## 1                                                            2011-2013 years are the most recently available data at this time.
## 2                                                            2011-2013 years are the most recently available data at this time.
## 3 Rates per 100,000 calculated using the Minneapolis 2010 Census population; age-adjusted using the 2000 US standard population
## 4 Rates per 100,000 calculated using the Minneapolis 2010 Census population; age-adjusted using the 2000 US standard population
## 5                                                                                                                              
## 6 Rates per 100,000 calculated using the Minneapolis 2010 Census population; age-adjusted using the 2000 US standard population
##   notes x90_confidence_level_low x90_confidence_level_high
## 1                             NA                        NA
## 2                             NA                        NA
## 3                             NA                        NA
## 4                             NA                        NA
## 5                             NA                        NA
## 6                             NA                        NA
##   x95_confidence_level_low x95_confidence_level_high
## 1                       NA                        NA
## 2                       NA                        NA
## 3                       NA                        NA
## 4                       NA                        NA
## 5                       NA                        NA
## 6                       NA                        NA
```

The top three outliers belong to 2011 Denver, CO (74.4 deaths), 2012 Denver, CO (also 74.4 deaths), and 2010 Minneapolis, MN (70.5 deaths). What's important is that these are race-stratified. While all three represent total values for both males and females, the Denver numbers represent data for race labeled "Other" and the Minneapolis number belongs to the American Indian/Alaska Native population. If we wanted to dig around, we could perhaps find the original Denver and Minneapolis data and double check these numbers. They might be correct or they might be attributed to typos. 

In this case, we're not interested in racial distribution for the moment, so we're going to restrict our dataset to the collapsed values, meaning those representing the averages of both sexes and all races. Then we'll re-attempt the pseudo-scatterplot. 


```r
drive_df %>% 
  filter(sex == "Both", race_ethnicity == "All") %>% 
  ggplot(aes(x = year, y = value)) + 
  geom_point()
```

![plot of chunk unnamed-chunk-6](/figs/2020-07-17-car-crashes/unnamed-chunk-6-1.png)

Ok, that's a little better, but it still doesn't tell us anything interesting. Each year has many data points representing an individual city and on this plot, it's impossible to tell which city corresponds to which point. We'll come back to this question in a moment, but for now, let's try to extract something useful from the plot. Let's try to answer the question: __on average, did motor vehicle fatalities in the U.S. increase, decrease, or remain stable between 2010 and 2016?__ To visualize the answer, we'll need a plot that can show us the center value of each year, and maybe some indication of the distribution within that year. Enter the boxplot. 

### Boxplots

I love a good boxplot (perhaps even more than I love a good gray square). Here's a wonderful TDS [post](https://towardsdatascience.com/understanding-boxplots-5e2df7bcbd51) on boxplots. 

Let's quickly toss in a boxplot geom: 


```r
drive_df %>% 
  filter(sex == "Both", race_ethnicity == "All") %>% 
  ggplot(aes(x = year, y = value)) + 
    geom_boxplot()
```

![plot of chunk unnamed-chunk-7](/figs/2020-07-17-car-crashes/unnamed-chunk-7-1.png){: height="600px" width="600px"}

Brilliant! Immediately we gleaned a useful nugget of information: there was an alarming spike in the data. Things were chugging along and then something happened in 2015. If you do a news search on this topic, you'll see that public health and safety folks [noticed](https://www.npr.org/sections/thetwo-way/2016/02/18/467230965/2015-traffic-fatalities-rose-by-largest-percent-in-50-years-safety-group-says) this, too. It looks like fuel prices and job growth were potential culprits, but either way, this is clearly a very useful insight.   

Let's spice this plot up a bit. Let's say we want to compare men and women in the same time frame. We will need to specify a color code within the aesthetic for `geom_boxplot` and we'll also need to filter out the "Both" values for sex, since we just want male vs female. 


```r
drive_df %>% 
  filter(race_ethnicity == "All", !sex == "Both") %>% 
  ggplot(aes(x = year, y = value, fill = sex)) + 
    geom_boxplot()
```

![plot of chunk unnamed-chunk-8](/figs/2020-07-17-car-crashes/unnamed-chunk-8-1.png){: height="600px" width="600px"}

NICE. These are meaty results. There's an obvious disparity in mortality between males and females. Most years, male fatality averages are more than double that of females, and some years (2015, for instance) it's more like triple. The female boxplot for 2016 also looks suspiciously narrow. It's worth investigating but for now let's move on. 

I'm not a huge fan of that color scheme, so let's use some nicer colors. For two-tone plots, I like to just manually choose the colors (there are a million color tools online, but [this one](https://coolors.co) is my favorite). For three or more colors, I normally use a package like [`RColorBrewer`](https://cran.r-project.org/web/packages/RColorBrewer/RColorBrewer.pdf) (built-in) or the extra-fancy [`viridis`](https://cran.r-project.org/web/packages/viridis/vignettes/intro-to-viridis.html) package, which you'd need to install. I'm also going to get rid of the gray background using a theme and change the opacity of the fill color using an `alpha` statement (the latter is just personal preference). Finally, I'll capitalize the axis labels, and add a title and caption. 


```r
drive_df %>% 
  filter(race_ethnicity == "All", !sex == "Both") %>% 
  ggplot(aes(x = year, y = value, fill = sex)) + 
    #change outline color to gray
    geom_boxplot(alpha = 0.7) +
    scale_fill_manual(values=c("#996699", "#99CC99")) + 
    theme_bw() +
    labs(
      title = "Motor Vehicle Fatality Rate 2010-2016 by Sex",
      x = "Year",
      y = "Fatality (death/100,000 population)",
      caption = "Data source: Big Cities Health Initiative"
    )
```

![plot of chunk unnamed-chunk-9](/figs/2020-07-17-car-crashes/unnamed-chunk-9-1.png){: height="600px" width="600px"}

Great, we wound up with a nice-looking boxplot with some useful information. 

### Violin Plots

Now let's take a look at a cousin of the boxplot: the violin plot. All you really need to know to understand how it works is shown in this figure, taken from another [TDS post](https://towardsdatascience.com/violin-plots-explained-fb1d115e023d).

<figure class="half">
    <img src="/assets/images/violin.png">
</figure>

There was a time a few years back when everyone thought violin plots were the best thing since sliced bagels. The trend seems to have somewhat faded but I think they're nice so let's make one. 


```r
drive_df %>% 
  filter(sex == "Both", race_ethnicity == "All") %>%
  ggplot(aes(x = year, y = value)) + 
    geom_violin()
```

![plot of chunk unnamed-chunk-10](/figs/2020-07-17-car-crashes/unnamed-chunk-10-1.png)

Is it just me or do these look like a Dr. Seuss version of stick-figures? 

Anyway, these aren't very "violin-y" but the point is that higher frequencies of values lead to wider parts of the plots and vice-versa. If we want to see a more violin shape, we can combine some of the years to get more data points per 'violin". Let's do that here: 


```r
drive_df %>% 
  filter(sex == "Both", race_ethnicity == "All") %>%
  mutate(year_group = if_else(year %in% c(2010, 2011, 2012), "2010-2012", "2013-2016")) %>% 
  ggplot(aes(x = year_group, y = value)) + 
    geom_violin()
```

![plot of chunk unnamed-chunk-11](/figs/2020-07-17-car-crashes/unnamed-chunk-11-1.png)

Much more violin-like! Clearly, the 2010-2012 group had most of its data points in the 6-6.5 range, while 2013-2016 was a bit more spread out across its range of values and its median is a bit higher (probably driven by that alarming 2015 number).

What if we split out males vs females like we did in our boxplots? 

```r
violin = 
drive_df %>% 
  filter(!sex == "Both", race_ethnicity == "All") %>%
  mutate(year_group = if_else(year %in% c(2010, 2011, 2012), "2010-2012", "2013-2016")) %>% 
  ggplot(aes(x = year_group, y = value, fill = sex)) + 
    geom_violin(alpha = 0.7)

violin
```

![plot of chunk unnamed-chunk-12](/figs/2020-07-17-car-crashes/unnamed-chunk-12-1.png){: height="600px" width="600px"}

Cute, right? The gender difference we saw in the boxplots is even more detailed here. 

Let's clean this up a bit and then we're going to do something __really__ exciting. 

```r
violin_1 = 
  violin +
  scale_fill_manual(values=c("#996699", "#99CC99")) + 
  theme_bw() +
  labs(
      title = "Motor Vehicle Fatality Rates, Early vs Mid 2000s by Sex",
      x = "Year",
      y = "Fatality (death/100,000 population)",
      caption = "Data source: Big Cities Health Initiative"
    )

violin_1
```

![plot of chunk unnamed-chunk-13](/figs/2020-07-17-car-crashes/unnamed-chunk-13-1.png){: height="600px" width="600px"}

All right, let's get extra fancy. We're going to use a function within `ggplot2` called `stat_summary` to give us some extra detail. 

#### `stat_summary()`

`stat_summary` does just what it sounds like - it summarizes stats. We've been using stats within our geoms this whole time, we just didn't know about it because `ggplot2` does it in the background. For instance, the boxplot geom calculates the median, 25th, and 75th percentile stats. 

Here, we'll add dots to indicate the median in each violin. The `position_dodge` function moves the dots inside the violins, as opposed to positioning them directly on the gridline. 


```r
violin_1 +
  stat_summary(fun = median, geom="point", position = position_dodge(width = 0.9))
```

![plot of chunk unnamed-chunk-14](/figs/2020-07-17-car-crashes/unnamed-chunk-14-1.png){: height="600px" width="600px"}


```r
violin_1 +
  stat_summary(fun.data = mean_sdl, geom = "pointrange", position = position_dodge(width = 0.9))
```

![plot of chunk unnamed-chunk-15](/figs/2020-07-17-car-crashes/unnamed-chunk-15-1.png){: height="600px" width="600px"}

There are lots of other stats you can layer on. You can even embed little boxplots within the violin plots. [This page](http://www.sthda.com/english/wiki/ggplot2-violin-plot-quick-start-guide-r-software-and-data-visualization) gives a great summary, but for our purposes we're going to end our mini-tour of violin plots here and move on. 

### Points and Lines 

Now let's come back to our initial question of how motor vehicle fatalities differ across cities in the U.S. Recall that we started with the scatterplot below: 


```r
drive_df %>% 
  filter(sex == "Both", race_ethnicity == "All") %>% 
  ggplot(aes(x = year, y = value)) + 
  geom_point()
```

![plot of chunk unnamed-chunk-16](/figs/2020-07-17-car-crashes/unnamed-chunk-16-1.png)

Every dot represents a city and every city has a value for every year of measurement, so of course we want to know which dot belongs to which city.

Let's start with some basic color-coding. 


```r
drive_df %>% 
  filter(sex == "Both", race_ethnicity == "All") %>% 
  ggplot(aes(x = year, y = value)) + 
  geom_point(aes(color = place))
```

![plot of chunk unnamed-chunk-17](/figs/2020-07-17-car-crashes/unnamed-chunk-17-1.png){: height="600px" width="600px"}

Yikes. It's basically impossible to distinguish any of these colors from one another, or to follow the trajectory of any city year to year. We need lines rather than dots. We also need `ggplot2` to know that each city should be treated as a grouping (this latter point is not super intuitive to me, but just know that it doesn't work otherwise).


```r
drive_df %>% 
  filter(sex == "Both", race_ethnicity == "All") %>% 
  ggplot(aes(x = year, y = value, group = place)) + 
  geom_line()  
```

![plot of chunk unnamed-chunk-18](/figs/2020-07-17-car-crashes/unnamed-chunk-18-1.png){: height="600px" width="600px"}

Does this bring back fond [Etch A Sketch](https://etchasketch.com) memories? 

What we've created here is called a __spaghetti plot__ for reasons that should be obvious. Clearly we need colors though. Let's drop a color aesthetic into our line geom. 


```r
spaghetti_plot = 
drive_df %>% 
  filter(sex == "Both", race_ethnicity == "All") %>% 
  ggplot(aes(x = year, y = value, group = place)) + 
  geom_line(aes(color = place)) 

spaghetti_plot
```

![plot of chunk unnamed-chunk-19](/figs/2020-07-17-car-crashes/unnamed-chunk-19-1.png){: height="600px" width="600px"}

Double yikes. Too many similar colors. Also, too much legend. 


```r
spaghetti_plot_1 = 
  spaghetti_plot +
  scale_color_viridis(discrete = TRUE) + 
  theme_bw() + 
  theme(legend.position = "bottom", legend.title = element_text(size = 8),
  legend.text = element_text(size = 6) 
  ) +
  labs(
      title = "Motor Vehicle Fatality Rates by City",
      x = "Year",
      y = "Fatality (death/100,000 population)",
      caption = "Data source: Big Cities Health Initiative"
    )

spaghetti_plot_1
```

![plot of chunk unnamed-chunk-20](/figs/2020-07-17-car-crashes/unnamed-chunk-20-1.png){: height="600px" width="600px"}

That's a lot better, but there's still no way anyone would be able to distinguish Houston from Indianapolis, or even be able to infer which city has the highest crash fatality burden versus lowest. And this is where interactivity comes in handy. 

A word of warning, it's easy to over-do interactivity and go nuts with things like `plotly` because it's fun and flashy. Don't do it. If it doesn't add value to visualizing patterns in the data, it doesn't have any place on your plot. 

### `ggplotly`

In this case, we definitely want some interactivity to help us understand the data. We're going to use [`ggplotly`](https://www.rdocumentation.org/packages/plotly/versions/4.9.2.1/topics/ggplotly), a convenient marriage of `plotly` and `ggplot2`. You can hover on a line to see which city it represents, or double any city in the legend to isolate it on the plot.


```r
spaghetti_plot_1 %>% 
  ggplotly()
```

<div class="figure">
<!--html_preserve--><div id="htmlwidget-81a9e2a8210bea495c06" style="width:650px;height:650px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-81a9e2a8210bea495c06">{"x":{"data":[{"x":[1,3,4,5,6],"y":[null,5,4.7,4.3,3.8],"text":["place: Boston, MA<br />year: 2010<br />value:   NA<br />place: Boston, MA","place: Boston, MA<br />year: 2012<br />value:  5.0<br />place: Boston, MA","place: Boston, MA<br />year: 2013<br />value:  4.7<br />place: Boston, MA","place: Boston, MA<br />year: 2014<br />value:  4.3<br />place: Boston, MA","place: Boston, MA<br />year: 2015<br />value:  3.8<br />place: Boston, MA"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(68,1,84,1)","dash":"solid"},"hoveron":"points","name":"Boston, MA","legendgroup":"Boston, MA","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1,2,3,4],"y":[5.8,5.7,6.5,6.3],"text":["place: Chicago, Il<br />year: 2010<br />value:  5.8<br />place: Chicago, Il","place: Chicago, Il<br />year: 2011<br />value:  5.7<br />place: Chicago, Il","place: Chicago, Il<br />year: 2012<br />value:  6.5<br />place: Chicago, Il","place: Chicago, Il<br />year: 2013<br />value:  6.3<br />place: Chicago, Il"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(71,16,99,1)","dash":"solid"},"hoveron":"points","name":"Chicago, Il","legendgroup":"Chicago, Il","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[3,4,5],"y":[7.3,7.1,6.3],"text":["place: Cleveland, OH<br />year: 2012<br />value:  7.3<br />place: Cleveland, OH","place: Cleveland, OH<br />year: 2013<br />value:  7.1<br />place: Cleveland, OH","place: Cleveland, OH<br />year: 2014<br />value:  6.3<br />place: Cleveland, OH"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(72,29,111,1)","dash":"solid"},"hoveron":"points","name":"Cleveland, OH","legendgroup":"Cleveland, OH","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1,2,3,4,6,7],"y":[9.4,10.7,10.5,8.3,12.1,12.8],"text":["place: Columbus, OH<br />year: 2010<br />value:  9.4<br />place: Columbus, OH","place: Columbus, OH<br />year: 2011<br />value: 10.7<br />place: Columbus, OH","place: Columbus, OH<br />year: 2012<br />value: 10.5<br />place: Columbus, OH","place: Columbus, OH<br />year: 2013<br />value:  8.3<br />place: Columbus, OH","place: Columbus, OH<br />year: 2015<br />value: 12.1<br />place: Columbus, OH","place: Columbus, OH<br />year: 2016<br />value: 12.8<br />place: Columbus, OH"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(72,41,122,1)","dash":"solid"},"hoveron":"points","name":"Columbus, OH","legendgroup":"Columbus, OH","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1,2,3,4,5,6,7],"y":[8.5,7.2,6.5,8.4,7.7,10.7,13.5],"text":["place: Denver, CO<br />year: 2010<br />value:  8.5<br />place: Denver, CO","place: Denver, CO<br />year: 2011<br />value:  7.2<br />place: Denver, CO","place: Denver, CO<br />year: 2012<br />value:  6.5<br />place: Denver, CO","place: Denver, CO<br />year: 2013<br />value:  8.4<br />place: Denver, CO","place: Denver, CO<br />year: 2014<br />value:  7.7<br />place: Denver, CO","place: Denver, CO<br />year: 2015<br />value: 10.7<br />place: Denver, CO","place: Denver, CO<br />year: 2016<br />value: 13.5<br />place: Denver, CO"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(69,53,129,1)","dash":"solid"},"hoveron":"points","name":"Denver, CO","legendgroup":"Denver, CO","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[2,3,4,5],"y":[9,12.9,12.9,13.2],"text":["place: Detroit, MI<br />year: 2011<br />value:  9.0<br />place: Detroit, MI","place: Detroit, MI<br />year: 2012<br />value: 12.9<br />place: Detroit, MI","place: Detroit, MI<br />year: 2013<br />value: 12.9<br />place: Detroit, MI","place: Detroit, MI<br />year: 2014<br />value: 13.2<br />place: Detroit, MI"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(66,65,134,1)","dash":"solid"},"hoveron":"points","name":"Detroit, MI","legendgroup":"Detroit, MI","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1,2,3,4,5],"y":[9.3,10.9,9.9,11.6,9.8],"text":["place: Fort Worth (Tarrant County), TX<br />year: 2010<br />value:  9.3<br />place: Fort Worth (Tarrant County), TX","place: Fort Worth (Tarrant County), TX<br />year: 2011<br />value: 10.9<br />place: Fort Worth (Tarrant County), TX","place: Fort Worth (Tarrant County), TX<br />year: 2012<br />value:  9.9<br />place: Fort Worth (Tarrant County), TX","place: Fort Worth (Tarrant County), TX<br />year: 2013<br />value: 11.6<br />place: Fort Worth (Tarrant County), TX","place: Fort Worth (Tarrant County), TX<br />year: 2014<br />value:  9.8<br />place: Fort Worth (Tarrant County), TX"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(61,77,138,1)","dash":"solid"},"hoveron":"points","name":"Fort Worth (Tarrant County), TX","legendgroup":"Fort Worth (Tarrant County), TX","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1,2,3],"y":[10.1,9.8,9.7],"text":["place: Houston, TX<br />year: 2010<br />value: 10.1<br />place: Houston, TX","place: Houston, TX<br />year: 2011<br />value:  9.8<br />place: Houston, TX","place: Houston, TX<br />year: 2012<br />value:  9.7<br />place: Houston, TX"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(56,87,140,1)","dash":"solid"},"hoveron":"points","name":"Houston, TX","legendgroup":"Houston, TX","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1,2,3,4,5,7],"y":[6.7,7.3,5.9,6.5,5.4,8.9],"text":["place: Indianapolis (Marion County), IN<br />year: 2010<br />value:  6.7<br />place: Indianapolis (Marion County), IN","place: Indianapolis (Marion County), IN<br />year: 2011<br />value:  7.3<br />place: Indianapolis (Marion County), IN","place: Indianapolis (Marion County), IN<br />year: 2012<br />value:  5.9<br />place: Indianapolis (Marion County), IN","place: Indianapolis (Marion County), IN<br />year: 2013<br />value:  6.5<br />place: Indianapolis (Marion County), IN","place: Indianapolis (Marion County), IN<br />year: 2014<br />value:  5.4<br />place: Indianapolis (Marion County), IN","place: Indianapolis (Marion County), IN<br />year: 2016<br />value:  8.9<br />place: Indianapolis (Marion County), IN"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(52,97,141,1)","dash":"solid"},"hoveron":"points","name":"Indianapolis (Marion County), IN","legendgroup":"Indianapolis (Marion County), IN","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1,2,3,4,5,6,7],"y":[12.6,9.8,10.7,9,9.4,10.8,11.5],"text":["place: Kansas City, MO<br />year: 2010<br />value: 12.6<br />place: Kansas City, MO","place: Kansas City, MO<br />year: 2011<br />value:  9.8<br />place: Kansas City, MO","place: Kansas City, MO<br />year: 2012<br />value: 10.7<br />place: Kansas City, MO","place: Kansas City, MO<br />year: 2013<br />value:  9.0<br />place: Kansas City, MO","place: Kansas City, MO<br />year: 2014<br />value:  9.4<br />place: Kansas City, MO","place: Kansas City, MO<br />year: 2015<br />value: 10.8<br />place: Kansas City, MO","place: Kansas City, MO<br />year: 2016<br />value: 11.5<br />place: Kansas City, MO"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(47,107,142,1)","dash":"solid"},"hoveron":"points","name":"Kansas City, MO","legendgroup":"Kansas City, MO","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1,2,3,4,5,6],"y":[8.7,7.8,8.8,9.1,8.8,10.6],"text":["place: Las Vegas (Clark County), NV<br />year: 2010<br />value:  8.7<br />place: Las Vegas (Clark County), NV","place: Las Vegas (Clark County), NV<br />year: 2011<br />value:  7.8<br />place: Las Vegas (Clark County), NV","place: Las Vegas (Clark County), NV<br />year: 2012<br />value:  8.8<br />place: Las Vegas (Clark County), NV","place: Las Vegas (Clark County), NV<br />year: 2013<br />value:  9.1<br />place: Las Vegas (Clark County), NV","place: Las Vegas (Clark County), NV<br />year: 2014<br />value:  8.8<br />place: Las Vegas (Clark County), NV","place: Las Vegas (Clark County), NV<br />year: 2015<br />value: 10.6<br />place: Las Vegas (Clark County), NV"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(43,116,142,1)","dash":"solid"},"hoveron":"points","name":"Las Vegas (Clark County), NV","legendgroup":"Las Vegas (Clark County), NV","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[2,3,4,5],"y":[4.5,6.1,3.9,4.4],"text":["place: Long Beach, CA<br />year: 2011<br />value:  4.5<br />place: Long Beach, CA","place: Long Beach, CA<br />year: 2012<br />value:  6.1<br />place: Long Beach, CA","place: Long Beach, CA<br />year: 2013<br />value:  3.9<br />place: Long Beach, CA","place: Long Beach, CA<br />year: 2014<br />value:  4.4<br />place: Long Beach, CA"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(39,126,142,1)","dash":"solid"},"hoveron":"points","name":"Long Beach, CA","legendgroup":"Long Beach, CA","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1,2,3],"y":[4.3,3.7,4.1],"text":["place: Los Angeles, CA<br />year: 2010<br />value:  4.3<br />place: Los Angeles, CA","place: Los Angeles, CA<br />year: 2011<br />value:  3.7<br />place: Los Angeles, CA","place: Los Angeles, CA<br />year: 2012<br />value:  4.1<br />place: Los Angeles, CA"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(36,135,142,1)","dash":"solid"},"hoveron":"points","name":"Los Angeles, CA","legendgroup":"Los Angeles, CA","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[2,3,4,5],"y":[10.3,9.9,8.8,10.1],"text":["place: Miami (Miami-Dade County), FL<br />year: 2011<br />value: 10.3<br />place: Miami (Miami-Dade County), FL","place: Miami (Miami-Dade County), FL<br />year: 2012<br />value:  9.9<br />place: Miami (Miami-Dade County), FL","place: Miami (Miami-Dade County), FL<br />year: 2013<br />value:  8.8<br />place: Miami (Miami-Dade County), FL","place: Miami (Miami-Dade County), FL<br />year: 2014<br />value: 10.1<br />place: Miami (Miami-Dade County), FL"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(33,144,140,1)","dash":"solid"},"hoveron":"points","name":"Miami (Miami-Dade County), FL","legendgroup":"Miami (Miami-Dade County), FL","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1,2,3,4,5,6,7],"y":[6.5,3.3,4.6,6.6,2.8,3.3,6.6],"text":["place: Minneapolis, MN<br />year: 2010<br />value:  6.5<br />place: Minneapolis, MN","place: Minneapolis, MN<br />year: 2011<br />value:  3.3<br />place: Minneapolis, MN","place: Minneapolis, MN<br />year: 2012<br />value:  4.6<br />place: Minneapolis, MN","place: Minneapolis, MN<br />year: 2013<br />value:  6.6<br />place: Minneapolis, MN","place: Minneapolis, MN<br />year: 2014<br />value:  2.8<br />place: Minneapolis, MN","place: Minneapolis, MN<br />year: 2015<br />value:  3.3<br />place: Minneapolis, MN","place: Minneapolis, MN<br />year: 2016<br />value:  6.6<br />place: Minneapolis, MN"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(31,153,138,1)","dash":"solid"},"hoveron":"points","name":"Minneapolis, MN","legendgroup":"Minneapolis, MN","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[2,4,5,6],"y":[3.4,3.6,3.1,2.9],"text":["place: New York City, NY<br />year: 2011<br />value:  3.4<br />place: New York City, NY","place: New York City, NY<br />year: 2013<br />value:  3.6<br />place: New York City, NY","place: New York City, NY<br />year: 2014<br />value:  3.1<br />place: New York City, NY","place: New York City, NY<br />year: 2015<br />value:  2.9<br />place: New York City, NY"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(31,162,135,1)","dash":"solid"},"hoveron":"points","name":"New York City, NY","legendgroup":"New York City, NY","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1,2,3,4,5,6,7],"y":[5.5,6.7,12.9,7.7,5.9,10.3,8.6],"text":["place: Oakland (Alameda County), CA<br />year: 2010<br />value:  5.5<br />place: Oakland (Alameda County), CA","place: Oakland (Alameda County), CA<br />year: 2011<br />value:  6.7<br />place: Oakland (Alameda County), CA","place: Oakland (Alameda County), CA<br />year: 2012<br />value: 12.9<br />place: Oakland (Alameda County), CA","place: Oakland (Alameda County), CA<br />year: 2013<br />value:  7.7<br />place: Oakland (Alameda County), CA","place: Oakland (Alameda County), CA<br />year: 2014<br />value:  5.9<br />place: Oakland (Alameda County), CA","place: Oakland (Alameda County), CA<br />year: 2015<br />value: 10.3<br />place: Oakland (Alameda County), CA","place: Oakland (Alameda County), CA<br />year: 2016<br />value:  8.6<br />place: Oakland (Alameda County), CA"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(37,172,130,1)","dash":"solid"},"hoveron":"points","name":"Oakland (Alameda County), CA","legendgroup":"Oakland (Alameda County), CA","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1,2,3,4,5,7],"y":[6.7,6.8,6.8,7,6.7,7.2],"text":["place: Philadelphia, PA<br />year: 2010<br />value:  6.7<br />place: Philadelphia, PA","place: Philadelphia, PA<br />year: 2011<br />value:  6.8<br />place: Philadelphia, PA","place: Philadelphia, PA<br />year: 2012<br />value:  6.8<br />place: Philadelphia, PA","place: Philadelphia, PA<br />year: 2013<br />value:  7.0<br />place: Philadelphia, PA","place: Philadelphia, PA<br />year: 2014<br />value:  6.7<br />place: Philadelphia, PA","place: Philadelphia, PA<br />year: 2016<br />value:  7.2<br />place: Philadelphia, PA"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(49,181,123,1)","dash":"solid"},"hoveron":"points","name":"Philadelphia, PA","legendgroup":"Philadelphia, PA","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[3,5],"y":[8.7,10],"text":["place: Phoenix, AZ<br />year: 2012<br />value:  8.7<br />place: Phoenix, AZ","place: Phoenix, AZ<br />year: 2014<br />value: 10.0<br />place: Phoenix, AZ"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(64,188,114,1)","dash":"solid"},"hoveron":"points","name":"Phoenix, AZ","legendgroup":"Phoenix, AZ","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[2,3,4,5,6,7],"y":[4.5,6,7.6,5.7,6.2,null],"text":["place: Portland (Multnomah County), OR<br />year: 2011<br />value:  4.5<br />place: Portland (Multnomah County), OR","place: Portland (Multnomah County), OR<br />year: 2012<br />value:  6.0<br />place: Portland (Multnomah County), OR","place: Portland (Multnomah County), OR<br />year: 2013<br />value:  7.6<br />place: Portland (Multnomah County), OR","place: Portland (Multnomah County), OR<br />year: 2014<br />value:  5.7<br />place: Portland (Multnomah County), OR","place: Portland (Multnomah County), OR<br />year: 2015<br />value:  6.2<br />place: Portland (Multnomah County), OR","place: Portland (Multnomah County), OR<br />year: 2016<br />value:   NA<br />place: Portland (Multnomah County), OR"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(83,197,105,1)","dash":"solid"},"hoveron":"points","name":"Portland (Multnomah County), OR","legendgroup":"Portland (Multnomah County), OR","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1,2,3,4,5,6],"y":[8.9,6.4,7.8,7.6,10.6,12],"text":["place: San Antonio, TX<br />year: 2010<br />value:  8.9<br />place: San Antonio, TX","place: San Antonio, TX<br />year: 2011<br />value:  6.4<br />place: San Antonio, TX","place: San Antonio, TX<br />year: 2012<br />value:  7.8<br />place: San Antonio, TX","place: San Antonio, TX<br />year: 2013<br />value:  7.6<br />place: San Antonio, TX","place: San Antonio, TX<br />year: 2014<br />value: 10.6<br />place: San Antonio, TX","place: San Antonio, TX<br />year: 2015<br />value: 12.0<br />place: San Antonio, TX"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(103,204,92,1)","dash":"solid"},"hoveron":"points","name":"San Antonio, TX","legendgroup":"San Antonio, TX","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1,2,3,4,5,6,7],"y":[6.1,6.1,6.7,6.1,7,7.1,7.3],"text":["place: San Diego County, CA<br />year: 2010<br />value:  6.1<br />place: San Diego County, CA","place: San Diego County, CA<br />year: 2011<br />value:  6.1<br />place: San Diego County, CA","place: San Diego County, CA<br />year: 2012<br />value:  6.7<br />place: San Diego County, CA","place: San Diego County, CA<br />year: 2013<br />value:  6.1<br />place: San Diego County, CA","place: San Diego County, CA<br />year: 2014<br />value:  7.0<br />place: San Diego County, CA","place: San Diego County, CA<br />year: 2015<br />value:  7.1<br />place: San Diego County, CA","place: San Diego County, CA<br />year: 2016<br />value:  7.3<br />place: San Diego County, CA"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(127,211,78,1)","dash":"solid"},"hoveron":"points","name":"San Diego County, CA","legendgroup":"San Diego County, CA","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1,4],"y":[2.6,3.8],"text":["place: San Francisco, CA<br />year: 2010<br />value:  2.6<br />place: San Francisco, CA","place: San Francisco, CA<br />year: 2013<br />value:  3.8<br />place: San Francisco, CA"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(151,216,63,1)","dash":"solid"},"hoveron":"points","name":"San Francisco, CA","legendgroup":"San Francisco, CA","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[2,3,4],"y":[5.9,6.1,7.1],"text":["place: San Jose, CA<br />year: 2011<br />value:  5.9<br />place: San Jose, CA","place: San Jose, CA<br />year: 2012<br />value:  6.1<br />place: San Jose, CA","place: San Jose, CA<br />year: 2013<br />value:  7.1<br />place: San Jose, CA"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(177,221,46,1)","dash":"solid"},"hoveron":"points","name":"San Jose, CA","legendgroup":"San Jose, CA","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1,2,3,4,5],"y":[3.9,2.9,5.2,3,3.6],"text":["place: Seattle, WA<br />year: 2010<br />value:  3.9<br />place: Seattle, WA","place: Seattle, WA<br />year: 2011<br />value:  2.9<br />place: Seattle, WA","place: Seattle, WA<br />year: 2012<br />value:  5.2<br />place: Seattle, WA","place: Seattle, WA<br />year: 2013<br />value:  3.0<br />place: Seattle, WA","place: Seattle, WA<br />year: 2014<br />value:  3.6<br />place: Seattle, WA"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(203,225,30,1)","dash":"solid"},"hoveron":"points","name":"Seattle, WA","legendgroup":"Seattle, WA","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1,2,3,4,5,6],"y":[11.3,11.1,11.4,10.9,10.8,11.4],"text":["place: U.S. Total, U.S. Total<br />year: 2010<br />value: 11.3<br />place: U.S. Total, U.S. Total","place: U.S. Total, U.S. Total<br />year: 2011<br />value: 11.1<br />place: U.S. Total, U.S. Total","place: U.S. Total, U.S. Total<br />year: 2012<br />value: 11.4<br />place: U.S. Total, U.S. Total","place: U.S. Total, U.S. Total<br />year: 2013<br />value: 10.9<br />place: U.S. Total, U.S. Total","place: U.S. Total, U.S. Total<br />year: 2014<br />value: 10.8<br />place: U.S. Total, U.S. Total","place: U.S. Total, U.S. Total<br />year: 2015<br />value: 11.4<br />place: U.S. Total, U.S. Total"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(229,228,25,1)","dash":"solid"},"hoveron":"points","name":"U.S. Total, U.S. Total","legendgroup":"U.S. Total, U.S. Total","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1,2,3],"y":[5.6,5.4,6.6],"text":["place: Washington, DC<br />year: 2010<br />value:  5.6<br />place: Washington, DC","place: Washington, DC<br />year: 2011<br />value:  5.4<br />place: Washington, DC","place: Washington, DC<br />year: 2012<br />value:  6.6<br />place: Washington, DC"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(253,231,37,1)","dash":"solid"},"hoveron":"points","name":"Washington, DC","legendgroup":"Washington, DC","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null}],"layout":{"margin":{"t":40.8401826484018,"r":7.30593607305936,"b":37.2602739726027,"l":37.2602739726027},"plot_bgcolor":"rgba(255,255,255,1)","paper_bgcolor":"rgba(255,255,255,1)","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"title":{"text":"Motor Vehicle Fatality Rates by City","font":{"color":"rgba(0,0,0,1)","family":"","size":17.5342465753425},"x":0,"xref":"paper"},"xaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[0.4,7.6],"tickmode":"array","ticktext":["2010","2011","2012","2013","2014","2015","2016"],"tickvals":[1,2,3,4,5,6,7],"categoryorder":"array","categoryarray":["2010","2011","2012","2013","2014","2015","2016"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(235,235,235,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"y","title":{"text":"Year","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[2.055,14.045],"tickmode":"array","ticktext":["5","10"],"tickvals":[5,10],"categoryorder":"array","categoryarray":["5","10"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(235,235,235,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"x","title":{"text":"Fatality (death/100,000 population)","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":"transparent","line":{"color":"rgba(51,51,51,1)","width":0.66417600664176,"linetype":"solid"},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":true,"legend":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"font":{"color":"rgba(0,0,0,1)","family":"","size":7.97011207970112},"y":0.955005624296963},"annotations":[{"text":"place","x":1.02,"y":1,"showarrow":false,"ax":0,"ay":0,"font":{"color":"rgba(0,0,0,1)","family":"","size":10.6268161062682},"xref":"paper","yref":"paper","textangle":-0,"xanchor":"left","yanchor":"bottom","legendTitle":true}],"hovermode":"closest","barmode":"relative"},"config":{"doubleClick":"reset","showSendToCloud":false},"source":"A","attrs":{"4417112570ae":{"colour":{},"x":{},"y":{},"type":"scatter"}},"cur_data":"4417112570ae","visdat":{"4417112570ae":["function (y) ","x"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot","plotly_sunburstclick"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->
</div>

Isn't this amazing? I'm blown away that one additional line of code did all that. 

### Faceting

Ok, one last trick. Say we wanted to give each city its own little plot to understand time-trends within the city. We can certainly get this information from the spaghetti plot but it would undoubtedly cause a headache. Let's play with `facet_wrap` and `facet_grid` instead. 

The difference between these two isn't super clear to start with, but basically, `facet_wrap` is good if you're just faceting on one variable (in our case, city). If you want to end up with a separate mini-plot for each combination of two variables (for instance, a mini-plot for female fatalities in Denver, another plot for male fatalities in Denver, and so on for every combination of city and sex), you want `facet_grid` (more info [here](https://ggplot2-book.org/facet.html)).

We're going to start with `facet_wrap`.


```r
drive_df %>% 
  filter(sex == "Both", race_ethnicity == "All") %>% 
  ggplot(aes(x = year, y = value, group = place)) + 
  geom_line() + 
  facet_wrap(vars(place))
```

![plot of chunk unnamed-chunk-22](/figs/2020-07-17-car-crashes/unnamed-chunk-22-1.png){: height="650px" width="650px"}

Aside from the fact that the x-axis labels are a bit messed up, this is actually pretty nice. 

Now let's try a grid for combinations of place and sex.


```r
drive_df %>% 
  filter(!sex == "Both", race_ethnicity == "All") %>% 
  ggplot(aes(x = year, y = value, group = place)) + 
  geom_line() + 
  facet_grid(sex ~ place)
```

![plot of chunk unnamed-chunk-23](/figs/2020-07-17-car-crashes/unnamed-chunk-23-1.png){: height="650px" width="650px"}

Oh boy. Let's go back to `facet_wrap` and just do some color-coding for sex rather than trying to make sense of this crazy grid. 


```r
drive_df %>% 
  filter(!sex == "Both", race_ethnicity == "All") %>% 
  ggplot(aes(x = year, y = value, group = sex)) + 
  geom_line(aes(color = sex)) + 
  facet_wrap(vars(place))
```

![plot of chunk unnamed-chunk-24](/figs/2020-07-17-car-crashes/unnamed-chunk-24-1.png){: height="650px" width="650px"}

NIIIIICE. If we fix the formatting, this will actually be an ok plot. First off, we should minimize the number of plots displayed horizontally using `ncol`. 


```r
drive_df %>% 
  filter(!sex == "Both", race_ethnicity == "All") %>% 
  ggplot(aes(x = year, y = value, group = sex)) + 
  geom_line(aes(color = sex)) + 
  facet_wrap(vars(place), ncol = 3)
```

![plot of chunk unnamed-chunk-25](/figs/2020-07-17-car-crashes/unnamed-chunk-25-1.png){: height="650px" width="650px"}

Next, let's fix up the colors, axes, legend, etc. 


```r
drive_df %>% 
  filter(!sex == "Both", race_ethnicity == "All") %>% 
  ggplot(aes(x = year, y = value, group = sex)) + 
  geom_line(aes(color = sex)) + 
  scale_color_manual(values=c("#996699", "#99CC99")) +
  facet_wrap(vars(place), ncol = 3)  + 
    theme_bw() +
    labs(
      title = "Motor Vehicle Fatality Rate 2010-2016 by Sex",
      x = "Year",
      y = "Fatality (death/100,000 population)",
      caption = "Data source: Big Cities Health Initiative"
    )
```

![plot of chunk unnamed-chunk-26](/figs/2020-07-17-car-crashes/unnamed-chunk-26-1.png){: height="650px" width="650px"}

I'm pretty happy with that. It's clear we have an issue with Indianapolis (perhaps they only provide the combined male/female figure but not the individual values). It's also clear that Boston had only 2 years of data and other cities have some missing data as well. But a more interesting finding is comparing Denver to Detroit, which are conveniently side-by-side. Detroit saw a leveling off, whereas Denver saw a sharp increase in male fatalities. San Jose sticks out, too, as the only city where female fatalities surpassed male fatalities (and sharply so...) If I were an analyst looking at traffic safety, I'd put some focus on Denver and San Jose and figure out what's going on there. 

## Conclusions

I hope you agree that we did some fun stuff and learned a bit about car crash fatalities. My takeaways were: 
- There is a huge disparity between male and female motor vehicle fatalities in nearly every big city in the U.S., males experiencing double and sometimes triple the female average fatality rate.
- Detroit has historically had the highest burden of motor vehicle fatalities, but Denver has recently caught up. 
- `ggplot2` is very cool. `ggplotly` is also very cool.

This is absolutely just a surface-scratch of all the things `ggplot2` can do, so please take a look at the links below for more complete package exploration. 

## Further Reading 

- A handy [cheat sheet](https://rstudio.com/wp-content/uploads/2015/03/ggplot2-cheatsheet.pdf)
- A beautiful [collection](https://www.r-graph-gallery.com/index.html) of R data viz inspiration

## References

- Michonneau F, Teal T, Fournier A, Seok B, Obeng A, Pawlik AN, Conrado AC, Woo K, Lijnzaad P, Hart T, White EP, Marwick B, Bolker B, Jordan KL, Ashander J, Dashnow H, Hertweck K, Cuesta SM, Becker EA, Guillou S, Shiklomanov A, Klinges D, Odom GJ, Jean M, Mislan KAS, Johnson K, Jahn N, Mannheimer S, Pederson S, Pletzer A, Fouilloux A, Switzer C, Bahlai C, Li D, Kerchner D, Rodriguez-Sanchez F, Rajeg GPW, Ye H, Tavares H, Leinweber K, Peck K, Lepore ML, Hancock S, Sandmann T, Hodges T, Tirok K, Jean M, Bailey A, von Hardenberg A, Theobold A, Wright A, Basu A, Johnson C, Voter C, Hulshof C, Bouquin D, Quinn D, Vanichkina D, Wilson E, Strauss E, Bledsoe E, Gan E, Fishman D, Boehm F, Daskalova G, Tavares H, Kaupp J, Dunic J, Keane J, Stachelek J, Herr JR, Millar J, Lotterhos K, Cranston K, Direk K, Tylén K, Chatzidimitriou K, Deer L, Tarkowski L, Chiapello M, Burle M, Ankenbrand M, Czapanskiy M, Moreno M, Culshaw-Maurer M, Koontz M, Weisner M, Johnston M, Carchedi N, Burge OR, Harrison P, Humburg P, Pauloo R, Peek R, Elahi R, Cortijo S, sfn_brt, Umashankar S, Goswami S, Sumedh, Yanco S, Webster T, Reiter T, Pearse W, Li Y (2020). “datacarpentry/R-ecology-lesson: Data Carpentry: Data Analysis and Visualization in R for Ecologists, June 2019.” doi: 10.5281/zenodo.3264888, https://datacarpentry.org/R-ecology-lesson/.


[^1]: There are people who would trash me for making that statement, given that R has an actual base graphics [package](https://www.rdocumentation.org/packages/graphics/versions/3.6.2) and from what I've seen, it works just fine. Data people on the internet sometimes [argue](https://flowingdata.com/2016/03/22/comparing-ggplot2-and-r-base-graphics/) against `ggplot2` since it seems the base graphics can get you similar results with less code. I have no position on the matter - I use `ggplot2` because I'm a sheep and that's what everyone else seems to use (and hence, what you can find a lot more online help for).

[^2]: Age adjustment is a method used in epidemiology when comparing rates of disease or mortality between populations with different age breakdowns. Roughly speaking, if the population of Austin TX, for instance, is much younger than the average U.S. population, it doesn't make sense to compare mortality rates directly. We know that older people die more frequently than younger people, so if we want an apples-to-apples comparison of two populations we have to use a weighting scheme. And that's all age standardization is - a weighting scheme that assigns more weight to certain age categories to "equalize" their weight to that of the reference population. In our example, older people in Austin would get weighted more than younger people. 

