---
title: "Phthalate Exposure in U.S. Women of Reproductive Age - an NHANES Review, Part I "
author: "Alice Tivarovsky"
date: "5/12/2020"
output: html_document
bibliography: references.bib
---
# Motivation
Perhaps I’m the type of person that worries excessively about things outside of their control, but I’ve long been concerned about plastics leaching into our bodies from things like food packaging and personal care products. 

Some of these anxieties might be justified. A recent analysis of bio-specimen data from the National Health and Nutrition Examination Survey [NHANES](https://www.cdc.gov/nchs/nhanes/index.htm), the largest annual survey of the United States population, found nearly ubiquitous exposure to phthalates (Zota et al, 2014). This is particularly concerning for pregnant women because studies have found adverse effects, including shorter pregnancy duration and decreased head circumference (Polanska et al, 2016), as well as endocrine disrupting effects in males, including decreased anogenital distance (animal study) (Swan et al, 2005) and damage to sperm DNA (Duty et al, 2003). So how concerned should we be? 

After reviewing the literature, I wanted to answer the following questions: 

1. What phthalates are present in women in the US, what are the average levels of these phthalates, and how have these levels changed over time? 
2. Are phthalate levels disproportionately distributed through the population? Namely, is there an association with phthalates and socioeconomic status? 


## Background
Before we dive in to the data, we need a bit of priming on what phthalates are and why they're important. Phthalates are the most common category of industrial plasticizers (chemicals that alter the flexibility and rigidity of plastics) in use today. Due to their widespread presence in manufacturing, the metabolites of these chemicals (i.e the breakdown products when they enter the body) are now ubiquitously detectable in humans in the United States (Zota et al, 2014). Phthalates are of particular concern in women of reproductive age because they can easily cross the placenta and interact with fetal development during critical windows of pregnancy (@Polanska2016). Previous studies have established detrimental defects on reproductive system development in fetuses exposed to phthalates (@Swan2005). 

Applications of phthalates range from building materials, including flooring and adhesives, to personal care products, including nail polish and shampoo (@CDC). High molecular weight phthalates like butylbenzyl phthalate (BBzP), di(2-ethylhexyl) phthalate (DEHP) and diisononyl phthalate (DiNP) are commonly found in building materials. Low molecular weight materials like diethyl phthalate (DEP), di-n-butyl phthalate (DnBP) are more commonly found in personal care products and medications (@Zota2014).

Although several studies have been conducted using cohorts, a powerful tool at our disposal is NHANES. This national survey has been performed annually by the CDC since [1960](https://www.cdc.gov/nchs/nhanes/about_nhanes.htm), provides a large sample size (about 10,000 participants per year), and assesses a very wide range of health factors including demographics, diet, chronic health conditions, and biomarkers in blood and urine. Because of its breadth and large sample size, NHANES provides rich datasets for studying associations between these health attributes. 

# Analysis
Although there is a [library](https://cran.r-project.org/web/packages/RNHANES/index.html) for analyzing NHANES in R (aptly named `RNHANES`), if you work with it long enough, you will notice that only some cycles and variables are available. I spent more time than I care to admit wrangling with this library, and ultimately concluded it would not give me what I needed. Instead, I downloaded the SAS export files from CDC's NHANES [website](https://www.cdc.gov/nchs/nhanes/index.htm) and imported them to R using the `foreign` library. Below are the setup, import, and tidying steps I used. 

## Data Preparation


### Libraries

{% highlight r %}
library(tidyverse)
library(foreign)
library(stats)
library(viridis)
library(survey)
library(plotly)
library(kableExtra)
library(gridExtra)
library(sjPlot)
{% endhighlight %}

### Data Import
First, we read in NHANES cycles 2003 - 2004, 2005 - 2006, 2007 - 2008, 2009 - 2010, 2011 - 2012, 2013 - 2014, and 2015-2016. As of June 2020, phthalate data for 2017 - 2018 was not yet available. The NHANES [datasets](https://wwwn.cdc.gov/nchs/nhanes/default.aspx) used are Demographics (DEMO) and Phthalates and Plasticizers Metabolites - Urine (PHTHTE). For the 2015-2016 cycle we also need the Albumin & Creatinine (ALB_CR_I) file since creatinine data were removed from the phthalates files during this cycle (more on creatinine below).


{% highlight r %}
# reading in 2003 - 2004 data
demo_03_04 = read.xport("./data/2003_2004_DEMO_C.XPT")
{% endhighlight %}



{% highlight text %}
## Error in lookup.xport(file): unable to open file: 'No such file or directory'
{% endhighlight %}



{% highlight r %}
phthalate_03_04 = read.xport("./data/2003_2004_PHTHTE_C.XPT.txt")
{% endhighlight %}



{% highlight text %}
## Error in lookup.xport(file): unable to open file: 'No such file or directory'
{% endhighlight %}



{% highlight r %}
# reading in 2005 - 2006 data
demo_05_06 = read.xport("./data/2005_2006_DEMO_D.XPT")
{% endhighlight %}



{% highlight text %}
## Error in lookup.xport(file): unable to open file: 'No such file or directory'
{% endhighlight %}



{% highlight r %}
phthalate_05_06 = read.xport("./data/2005_2006_PHTHTE_D.XPT.txt")
{% endhighlight %}



{% highlight text %}
## Error in lookup.xport(file): unable to open file: 'No such file or directory'
{% endhighlight %}



{% highlight r %}
# reading in 2007 - 2008 data
demo_07_08 = read.xport("./data/2007_2008_DEMO_E.XPT")
{% endhighlight %}



{% highlight text %}
## Error in lookup.xport(file): unable to open file: 'No such file or directory'
{% endhighlight %}



{% highlight r %}
phthalate_07_08 = read.xport("./data/2007_2008_PHTHTE_E.XPT.txt")
{% endhighlight %}



{% highlight text %}
## Error in lookup.xport(file): unable to open file: 'No such file or directory'
{% endhighlight %}



{% highlight r %}
# reading in 2009 - 2010 data
demo_09_10 = read.xport("./data/2009_2010_DEMO_F.XPT")
{% endhighlight %}



{% highlight text %}
## Error in lookup.xport(file): unable to open file: 'No such file or directory'
{% endhighlight %}



{% highlight r %}
phthalate_09_10 = read.xport("./data/2009_2010_PHTHTE_F.XPT.txt")
{% endhighlight %}



{% highlight text %}
## Error in lookup.xport(file): unable to open file: 'No such file or directory'
{% endhighlight %}



{% highlight r %}
# reading in 2011 - 2012 data
demo_11_12 = read.xport("./data/2011_2012_DEMO_G.XPT.txt")
{% endhighlight %}



{% highlight text %}
## Error in lookup.xport(file): unable to open file: 'No such file or directory'
{% endhighlight %}



{% highlight r %}
phthalate_11_12 = read.xport("./data/2011_2012_PHTHTE_G.XPT.txt")
{% endhighlight %}



{% highlight text %}
## Error in lookup.xport(file): unable to open file: 'No such file or directory'
{% endhighlight %}



{% highlight r %}
# reading in 2013 - 2014 data
demo_13_14 = read.xport("./data/2013_2014_DEMO_H.XPT.txt")
{% endhighlight %}



{% highlight text %}
## Error in lookup.xport(file): unable to open file: 'No such file or directory'
{% endhighlight %}



{% highlight r %}
phthalate_13_14 = read.xport("./data/2013_2014_PHTHTE_H.XPT.txt")
{% endhighlight %}



{% highlight text %}
## Error in lookup.xport(file): unable to open file: 'No such file or directory'
{% endhighlight %}



{% highlight r %}
# reading in 2015 - 2016 data (note change in creatinine source file for this cycle)
demo_15_16 = read.xport("./data/2015_2016_DEMO_I.XPT.txt")
{% endhighlight %}



{% highlight text %}
## Error in lookup.xport(file): unable to open file: 'No such file or directory'
{% endhighlight %}



{% highlight r %}
phthalate_15_16 = read.xport("./data/2015_2016_PHTHTE_I.XPT.txt")
{% endhighlight %}



{% highlight text %}
## Error in lookup.xport(file): unable to open file: 'No such file or directory'
{% endhighlight %}



{% highlight r %}
creat_15_16 = read.xport("./data/2015_2016_ALB_CR_I.XPT.txt")
{% endhighlight %}



{% highlight text %}
## Error in lookup.xport(file): unable to open file: 'No such file or directory'
{% endhighlight %}


### Data Tidy
Next we'll bind the data files for each cycle using left-joins, merging on the unique identifier `SEQN`.


{% highlight r %}
data_03_04 = 
  left_join(demo_03_04, phthalate_03_04, by = "SEQN") %>% 
  mutate(cycle = "2003-2004")
{% endhighlight %}



{% highlight text %}
## Error in left_join(demo_03_04, phthalate_03_04, by = "SEQN"): object 'demo_03_04' not found
{% endhighlight %}



{% highlight r %}
data_05_06 = 
  left_join(demo_05_06, phthalate_05_06, by = "SEQN") %>% 
  mutate(cycle = "2005-2006")
{% endhighlight %}



{% highlight text %}
## Error in left_join(demo_05_06, phthalate_05_06, by = "SEQN"): object 'demo_05_06' not found
{% endhighlight %}



{% highlight r %}
data_07_08 = 
  left_join(demo_07_08, phthalate_07_08, by = "SEQN") %>% 
  mutate(cycle = "2007-2008")
{% endhighlight %}



{% highlight text %}
## Error in left_join(demo_07_08, phthalate_07_08, by = "SEQN"): object 'demo_07_08' not found
{% endhighlight %}



{% highlight r %}
data_09_10 = 
  left_join(demo_09_10, phthalate_09_10, by = "SEQN") %>% 
  mutate(cycle = "2009-2010")
{% endhighlight %}



{% highlight text %}
## Error in left_join(demo_09_10, phthalate_09_10, by = "SEQN"): object 'demo_09_10' not found
{% endhighlight %}



{% highlight r %}
data_11_12 = 
  left_join(demo_11_12, phthalate_11_12, by = "SEQN") %>% 
  mutate(cycle = "2011-2012")
{% endhighlight %}



{% highlight text %}
## Error in left_join(demo_11_12, phthalate_11_12, by = "SEQN"): object 'demo_11_12' not found
{% endhighlight %}



{% highlight r %}
data_13_14 = 
  left_join(demo_13_14, phthalate_13_14, by = "SEQN") %>% 
  mutate(cycle = "2013-2014")
{% endhighlight %}



{% highlight text %}
## Error in left_join(demo_13_14, phthalate_13_14, by = "SEQN"): object 'demo_13_14' not found
{% endhighlight %}



{% highlight r %}
data_15_16 = 
  left_join(demo_15_16, phthalate_15_16, by = "SEQN") %>% 
  left_join(creat_15_16, by = "SEQN") %>% 
  mutate(cycle = "2015-2016") 
{% endhighlight %}



{% highlight text %}
## Error in left_join(demo_15_16, phthalate_15_16, by = "SEQN"): object 'demo_15_16' not found
{% endhighlight %}



{% highlight r %}
all_data = 
  bind_rows(data_03_04, data_05_06, data_07_08, data_09_10, data_11_12, data_13_14, data_15_16) 
{% endhighlight %}



{% highlight text %}
## Error in list2(...): object 'data_03_04' not found
{% endhighlight %}


#### Variables Used
Next, we'll select the variables we want. We will choose 12 phthalates measured in NHANES between 2003 and 2016, as well as urinary [creatinine](https://en.wikipedia.org/wiki/Creatinine) `URXUCR`. The latter is a constant byproduct of metabolic activities and is often used to measure urinary dilution. 

NHANES takes biosample data for about 1/3 of the survey participants, so we will remove observations with missing phthalate data. We will restrict analysis to female respondents between the ages of 20 and 44, which we'll take to mean reproductive age in this analysis. We will also include age and socioeconomic measures to be used in Question 2. 

Below are the NHANES variables used, along with abbreviations for phthalate names. More intuitive variable names are assigned the subsequent code chunk. 

* `SEQN`: Unique NHANES identifier
* `RIAGENDR`: Gender
* `RIDAGEYR`: Age
* `RIDRETH1`: Ethnicity
* `DMDEDUC2`: Education
* `DMDMARTL`: Marital Status
* `INDHHIN2`: Annual household income (cycles 2007-2008, 2009-2010, 2011-2012, 2013-2014, 2015-2016)
* `INDFMIN2`: Annual family income (cycles 2007-2008, 2009-2010, 2011-2012, 2013-2014, 2015-2016)
* `INDHHINC`: Annual household income (cycles 2003-2004, 2005-2006)
* `INDFMINC`: Annual family income (cycles 2003-2004, 2005-2006)
* `WTMEC2YR`, `SDMVPSU`,`SDMVSTRA`: Survey weighting variables, addressed in Question 2 below
* `URXUCR`: Urinary creatinine
* `URXCNP`: Mono(carboxyisononyl) phthalate (MCNP) (ng/mL)
* `URXCOP`: Mono(carboxyisoctyl) phthalate (MCOP) (ng/mL)
* `URXMNP`: Mono-isononyl phthalate (MiNP) (ng/mL)
* `URXECP`: Mono-2-ethyl-5-carboxypentyl phthalate (MECPP) (ng/mL)
* `URXMHH`: Mono-(2-ethyl-5-hydroxyhexyl) phthalate (MEHHP) (ng/mL)
* `URXMHP`: Mono-(2-ethyl)-hexyl phthalate (MEHP) (ng/mL)
* `URXMOH`: Mono-(2-ethyl-5-oxohexyl) phthalate (MEOHP) (ng/mL)
* `URXMBP`: Mono-n-butyl phthalate (MnBP) (ng/mL)
* `URXMEP`: Mono-ethyl phthalate (MEP) (ng/mL)
* `URXMIB`: Mono-isobutyl phthalate (MiBP) (ng/mL)
* `URXMC1`: Mono-(3-carboxypropyl) phthalate (MCPP) (ng/mL)
* `URXMZP`: Mono-benzyl phthalate (MBzP) (ng/mL)



{% highlight r %}
# select variables of interest, drop every observation without biomarker data, filter out women aged 20-44, and consolidate household and family income variables
all_data = 
  all_data %>% 
  select(SEQN, cycle, RIAGENDR, RIDAGEYR, RIDRETH1, DMDEDUC2, DMDMARTL, INDHHINC, INDFMINC, INDHHIN2, INDFMIN2, WTMEC2YR, SDMVPSU, SDMVSTRA, URXUCR, URXECP, URXMBP, URXMC1, URXMEP, URXMHH, URXMHP, URXMIB, URXMOH, URXMZP, URXMNP, URXCOP, URXMNP, URXCNP) %>%
  drop_na(URXMEP) %>% 
  rename(gender = RIAGENDR, age = RIDAGEYR, ethnicity = RIDRETH1, education = DMDEDUC2, marital_status = DMDMARTL, creatinine = URXUCR, mecpp = URXECP, mnbp = URXMBP, mcpp = URXMC1, mep = URXMEP, mehhp = URXMHH, mehp = URXMHP, mibp = URXMIB, meohp = URXMOH, mbzp = URXMZP, mcnp = URXCNP, mcop = URXCOP, minp = URXMNP) %>% 
  filter(gender == 2, age %in% (20:44)) %>% 
  mutate(
    household_income = if_else(cycle %in% c("2003-2004", "2005-2006"), INDHHINC, INDHHIN2), 
    family_income = if_else(cycle %in% c("2003-2004", "2005-2006"), INDFMINC, INDFMIN2)
    ) %>% 
  select(-c(INDHHINC, INDHHIN2, INDFMINC, INDFMIN2))
{% endhighlight %}



{% highlight text %}
## Error in eval(lhs, parent, parent): object 'all_data' not found
{% endhighlight %}

Finally, we will add variables for creatinine-adjusted phthalate concentrations. There is, however, a units mismatch we'll need to deal with. Creatinine is measured in mg/dL and all phthalate biomarkers are measured in ng/mL. Creatinine adjusted measures are reported here (and often in literature) in units of $\mu$g phthalate/g creatinine. To get to these final units, we multiply the phthalate concentration by 100 and divide by creatinine [^1]. Adjusted values are denoted with a `_c`. 
 

{% highlight r %}
all_data_c = 
  all_data %>% 
  mutate(mecpp_c = 100*mecpp/creatinine, 
         mnbp_c = 100*mnbp/creatinine,
         mcpp_c = 100*mcpp/creatinine,
         mep_c = 100*mep/creatinine, 
         mehhp_c = 100*mehhp/creatinine,
         mehp_c = 100*mehp/creatinine, 
         mibp_c = 100*mibp/creatinine, 
         meohp_c = 100*meohp/creatinine, 
         mbzp_c = 100*mbzp/creatinine, 
         mcnp_c = 100*mcnp/creatinine,
         mcop_c = 100*mcop/creatinine, 
         minp_c = 100*minp/creatinine)
{% endhighlight %}



{% highlight text %}
## Error in eval(lhs, parent, parent): object 'all_data' not found
{% endhighlight %}

### Question 1: What phthalates are present in women in the US, what are the average levels of these chemicals, and how have these levels changed over time?

After I set about answering the first question, I realized that the most interesting part is the temporal aspect. If there were spikes in certain phthalates, for instance, that would narrow my focus going forward. Conversely, if phthalates were steadily decreasing in the population, maybe this would assuage my anxieties and I could find something else to worry about. Either way, I decided to tackle this question first. I used a spaghetti plot to visualize phthalates over the seven cycles of data and added `ggplotly` interactivity to help distinguish one line from the other. 


{% highlight r %}
means_c = 
  all_data_c %>% 
  select(-c(gender:mbzp), -c(SEQN, mcnp, mcop, mcnp_c, mcop_c)) %>% 
  pivot_longer(c(mecpp_c:minp_c), names_to = "chemical_c", values_to = "value") %>% 
  drop_na() %>% 
  mutate(chemical_c = recode(chemical_c, 
                           "mecpp_c"= "Mono-2-ethyl-5-carboxypentyl phthalate (MECPP_c)", 
                           "mnbp_c"= "Mono-n-butyl phthalate (MnBP_c)",
                           "mcpp_c"= "Mono-(3-carboxypropyl) phthalate (MCPP_c)",
                           "mep_c"= "Mono-ethyl phthalate (MEP_c)",
                           "mehhp_c"= "Mono-(2-ethyl-5-hydroxyhexyl) phthalate (MEHHP_c)",
                           "mehp_c"= "Mono-(2-ethyl)-hexyl phthalate (MEHP_c)",
                           "mibp_c"= "Mono-isobutyl phthalate (MiBP_c)",
                           "meohp_c"= "Mono-(2-ethyl-5-oxohexyl) phthalate (MEOHP_c)", 
                           "mbzp_c"= "Mono-benzyl phthalate (MBzP_c)", 
                           "mcnp_c"= "Mono(carboxyisononyl) phthalate (MCNP_c)", 
                           "mcop_c"= "Mono(carboxyisoctyl) phthalate (MCOP_c)",
                           "minp_c"= "Mono-isononyl phthalate (MiNP_c)")) %>% 
  group_by(cycle, chemical_c) %>% 
  summarize(mean_c = mean(value), n = n(), sd = sd(value), median_c = median(value), iqr_c = IQR(value))
{% endhighlight %}



{% highlight text %}
## Error in eval(lhs, parent, parent): object 'all_data_c' not found
{% endhighlight %}



{% highlight r %}
sp_plot = 
  means_c %>% 
  ggplot(aes(x = cycle, y = mean_c)) + 
  geom_line(aes(group = chemical_c, color = chemical_c))+
  labs(
    title = "Figure 1: Mean creatinine-adjusted phthalate concentration \n by NHANES cycle in women aged 20 - 44 (n = 2,754)",
    x = "NHANES Cycle",
    y = "Phthalate oncentration (ug/g creatinine)"
  ) +
  theme(text = element_text(size = 9)) + scale_colour_viridis_d(option = "inferno")
{% endhighlight %}



{% highlight text %}
## Error in eval(lhs, parent, parent): object 'means_c' not found
{% endhighlight %}



{% highlight r %}
ggplotly(sp_plot)
{% endhighlight %}



{% highlight text %}
## Error in ggplotly(sp_plot): object 'sp_plot' not found
{% endhighlight %}
Immediately, it's clear that MEP stands out. Peaking during the 2005-2006 cycle at 628.92 $\mu$g/g creatinine, MEP represented more than six times the biomarker concentration of the next highest phthalate, MECPP. Following the peak, the concentration fell sharply in 2007-2008 and continued to decline through next four cycles. All other phthalates also show a general decline over time. 

What's special about MEP and why did it decline so sharply? At first, I thought maybe there was an issue with the data. Perhaps an error in the 2005-2006 cycle, or, since we looked at means, some extremely influential outliers that drove the mean upward. However, calculating the median (a statistic not susceptible to outliers) below confirms that MEP is still the highest phthalate by far, measuring more than 4 times the mean of the next highest phthalate, MECPP.  


{% highlight r %}
means_c %>% 
  filter(cycle == "2005-2006") %>% 
  select(cycle, chemical_c, median_c)
{% endhighlight %}



{% highlight text %}
## Error in eval(lhs, parent, parent): object 'means_c' not found
{% endhighlight %}
Now that we've gone through this sanity check, we're back to figuring out why MEP stands out from the pack. And to be honest, after scouring the bowels of the internet, I didn't find a smoking gun. We do know that MEP is the primary metabolite of Diethyl Phthalate (DEP), which, like other phthalates, is used as a plasticizer for rigid materials including toys and toothbrushes. Unlike other phthalates, however, DEP is also used as a solvent in liquid cosmetics and perfumes [@CDC_DEP]. As such, the route of exposure is not only oral but also topical, perhaps explaining some of this unique trajectory. 

At this point, we might want to pull some information on industrial DEP usage in the US over the past 15 years and perhaps do some research on whether industry/policy changes circa 2005. Ah, but not so fast... After going down this rabbit hole, I learned some enlightening/frustrating information about the history of chemical reporting policies in the US. If this isn't your cup of tea, feel free to skip the following section. 

### __A brief historical interlude__
In 1976, Congress passed the [Toxic Substances Control Act](https://en.wikipedia.org/wiki/Toxic_Substances_Control_Act_of_1976), which sought to measure and regulate industrial usage of chemicals deemed to pose "unreasonable risk" to humans and the environment. The EPA was tasked with administration of the act and since passage, an inventory of about 84,000 chemicals has been established. In short, the [inventory](https://www.epa.gov/chemical-data-reporting) is problematic. First, it's only updated every four years. Second, it gives exemptions to smaller manufacturers and an ever-growing list of exempt chemicals. Finally (and most importantly), quantities are often reported in extremely wide ranges ("1,000,000 - <20,000,000 lbs" was an entry I saw in the 2016 data...) (@Zota2014, @Roundtable2014). Basically, I once assumed that some governing body has a solid grasp on the quantities and types of chemicals used in the US. I no longer assume this. 

In another facet of this regulatory picture, we have the [Consumer Product Safety Improvement Act of 2008](https://www.cpsc.gov/Regulations-Laws--Standards/Statutes/The-Consumer-Product-Safety-Improvement-Act). The act focuses on toxic substances in children's toys and banned presence of certain phthalates (check out this disturbing pile of [dolls](https://www.flickr.com/photos/cbpphotos/10928300625/) seized by US Customs in 2013 due to excess phthalate levels). One might think this would provide some clues on why MEP declined, but the timing doesn't work out- the act went into effect in 2009 and our trend started in 2004/2005. Additionally, DEP is not included in the scope, it's hard to ascribe the act as the root cause. It is possible, however, that the industry responded to pressure from advocacy groups and consumers, or in anticipation of further bans, and undertook formulation changes outside of (and prior to)  passage of the act. 


### Back to the numbers
Up until now, we've explored the temporal trends of common phthalates and did a bit of unsuccessful (but fun?) digging through the exciting world of toxic chemical legislative history. Now we will backtrack and summarize the average levels, as posed by Question 1. 

We proceed to summarizing the raw and creatinine-adjusted values from the 2015-2016 NHANES cycle. Mean and median values for both measures are reported in Table 1 below. Both are shown because there is 
a high degree of right-skew in the data. To illustrate this, here is a quick histogram of unadjusted MEP levels. 


{% highlight r %}
all_data %>% 
  ggplot(aes(x = mep)) +
  geom_histogram(binwidth = 500) +
  labs(x = "Unadjusted MEP (ng/mL)", 
       title = "Figure 2: MEP Histogram")
{% endhighlight %}



{% highlight text %}
## Error in eval(lhs, parent, parent): object 'all_data' not found
{% endhighlight %}

This right-skew is predictable - most people (thankfully) have very low levels of urinary MEP. As such, the median is a better measure of central tendency than the mean. We will look at both since some of the literature I reference below uses mean values and keeping the means will allow for comparison. 



{% highlight r %}
# calculating means and medians for phthalate values, not adjusted for creatinine
means_raw = 
all_data_c %>% 
  filter(cycle == "2015-2016") %>% 
  select(-c(gender:creatinine), -SEQN, -c(mecpp_c:minp_c)) %>% 
  pivot_longer(c(mecpp:mcnp), names_to = "chemical", values_to = "value") %>% 
  drop_na() %>% 
  mutate(chemical = recode(chemical, 
                           "mecpp"= "Mono-2-ethyl-5-carboxypentyl phthalate (MECPP)", 
                           "mnbp"= "Mono-n-butyl phthalate (MnBP)",
                           "mcpp"= "Mono-(3-carboxypropyl) phthalate (MCPP)",
                           "mep"= "Mono-ethyl phthalate (MEP)",
                           "mehhp"= "Mono-(2-ethyl-5-hydroxyhexyl) phthalate (MEHHP)",
                           "mehp"= "Mono-(2-ethyl)-hexyl phthalate (MEHP)",
                           "mibp"= "Mono-isobutyl phthalate (MiBP)",
                           "meohp"= "Mono-(2-ethyl-5-oxohexyl) phthalate (MEOHP)", 
                           "mbzp"= "Mono-benzyl phthalate (MBzP)", 
                           "mcnp"= "Mono(carboxyisononyl) phthalate (MCNP)", 
                           "mcop"= "Mono(carboxyisoctyl) phthalate (MCOP)",
                           "minp"= "Mono-isononyl phthalate (MiNP)")) %>%
  group_by(chemical) %>% 
  summarize(mean = mean(value), median = median(value)) %>% 
  mutate(id = row_number())
{% endhighlight %}



{% highlight text %}
## Error in eval(lhs, parent, parent): object 'all_data_c' not found
{% endhighlight %}



{% highlight r %}
# Calculating means and medians for adjusted values
means_c_15_16 = 
  means_c %>% 
  filter(cycle == "2015-2016") %>% 
  mutate(id = row_number()) %>% 
  select(id, chemical_c, mean_c, median_c)
{% endhighlight %}



{% highlight text %}
## Error in eval(lhs, parent, parent): object 'means_c' not found
{% endhighlight %}



{% highlight r %}
# joining adjusted and un-adjusted into one table
left_join(means_c_15_16, means_raw, by = "id") %>% 
  select(c("cycle", "chemical", "mean", "median", "mean_c", "median_c")) %>% 
  rename("Chemical" = "chemical", "Adjusted Mean" = "mean_c", "Adjusted Median" = "median_c", "Mean" = "mean", "Median" = "median") %>% 
  knitr::kable(digits = 1, caption = "Table 1: Phthalate Concentration in Women Ages 20-44, per NHANES 2015-2016 Cycle.") %>% 
  kable_styling(c("striped", "bordered")) %>%
  add_header_above(c(" " = 2, "Unadjusted (ng/mL)" = 2, "Creatinine-Adjusted (ug/g creatinine)" = 2))
{% endhighlight %}



{% highlight text %}
## Error in left_join(means_c_15_16, means_raw, by = "id"): object 'means_c_15_16' not found
{% endhighlight %}

So what do these values tell us? Are they safe? And what does "safe" even mean in this context?

Figuring out the answers gets us into a gray area. Some (myself included) would argue that no amount of plasticizers should be present in the human body. However, given how frequently we interact with plastics, this is probably not a realistic goal. 

Alternatively, we can draw certain conclusions about threshold values above which detrimental health effects were observed in past studies, and compare those values to what we saw in Table 1. But this, too, is tricky. For one thing, there's no way to ethically study the effects of phthalates in humans using the gold standard of study design - the randomized controlled trial (RCT). You can't just pick a group of pregnant women, split them up, and force one group to eat large amounts of phthalates. 

This leaves us with two options: animal studies and observational human studies. The former has the widely-acknowledged limitation that humans and lab animals (usually rats) are not, in fact, biologically equivalent. The latter has analytic limitations, the major one being unmeasured confounding. Confounding is a huge consideration in health/medical research and many epidemiologists spend their entire careers studying its effects and how it distorts measured relationships between exposures and outcomes (you can find a nice primer [here](https://www.healthknowledge.org.uk/public-health-textbook/research-methods/1a-epidemiology/biases)). For our purposes it's sufficient to say that observational studies of phthalates might tell us, for instance, that women with high levels of urinary MEP gave birth to smaller babies, but we cannot say that phthalates were the cause unless we control for all other potential causes of small babies (randomization accomplishes this, hence the usage of RCTs in humans). Both options have pretty big limitations, but if we combine the knowledge from both, we can perhaps glean conclusions to help us make sense of Table 1. 

Here is what I found: 

* Several studies have established adverse effects in rats, as summarized by Lyche et al (@Lyche2009). The outcomes were primarily related to reproductive effects and included sperm count reduction in males (@Kavlock2006) and delayed puberty in females (@Grande2006). Looking at the doses, however, these effects were observed in the range of 100 - 2000 mg chemical/kg animal per day. If we take a number in this range, say 375 mg/kg animal/day, we get a dose of roughly 150 mg per rat, per day (the average adult rat weighs about [400 grams](http://web.jhu.edu/animalcare/procedures/rat.html
).) If we extrapolate this to humans, using an average weight of 68 kilograms (150 lbs) for women, we get that there are about 170 rats per human (not literally of course), meaning that the human equivalent of 150 mg is 25,500 mg, or 25 grams of phthalate (almost 1 oz) per day. This is pretty huge and I highly doubt that anyone is ingesting this much on a daily basis. 

* Human studies on reproductive outcomes are limited, but Polanska et al (@Polanska2016) found an association between MEP and pregnancy duration in a cohort of Polish women. The other outcomes studied were weight. length, head and chest circumference of the baby and no significant associations were found between these outcomes and any other phthalate. In this study, the median creatinine-adjusted MEP was 22.7 $\mu$g/g creatinine, which is on par with the value calculated here, (33.2 $\mu$g/g creatinine). So barring unmeasured confounding in this study, we would conclude that MEP levels in US women are still too high and pose risk for adverse reproductive outcomes. 

* In a cohort of US women who gave birth to males, Swan et al (@Swan2005) found an association between the boys' [anogenital distance](https://en.wikipedia.org/wiki/Anogenital_distance) and mothers' exposure to MEP, MnBP, MBzP, and MiBP, with MnBP having the biggest effect. In this study, the boys with the shortest anogenital distance (i.e. the most reproductive impairment) had median concentrations of MEP, MnBP, MBzP, and MiBP of 225 ng/mL, 24.5 ng/mL, 16.1 ng/mL, and 4.8 ng/mL, respectively (the authors did not adjust for creatinine). Aside from MiBP, these values are much higher than the median values computed above.  


## Question 2: Are phthalate levels disproportionately distributed through the population? Namely, is there an association with phthalates and socioeconomic status? 

Now that we understand a bit about phthalates, it's interesting to explore whether their distributions vary between certain groups. For instance, do women with higher income have lower exposure to some/all phthalates. Or perhaps vice-versa? What about education? Why might this be the case? Well, it's possible that phthalates like MEP are present predominantly in inexpensive consumer products, thereby increasing risk for women who purchase them. And because manufacturers are not required to disclose usage of phthalates (as discussed in our historical interlude above), it's difficult to track them from the source. Understanding impacted groups gives us clues and insights into the mechanisms of action of these chemicals.  

Socioeconomic status is not a straightforward parameter to quantify. Often, it involves complex relationships between income, inherited wealth, education levels, race, and marital status. Because we are limited to the variables measured in NHANES, we will look at education as one proxy, and income as another. 

To figure out the effects of these variables on phthalate levels, we will do two things. The first is an exploratory visualization (mostly because these are nice to look at), and the second is a more rigorous set of statistical regression models. But first, (of course) we have to do a bit more data manipulation.  

### Recode income and education variables
Income is measured in NHANES using two categorical variables, __annual household income__ and __annual family income__. Both variables are coded using the same (somewhat non-intuitive) scale: 

* 1: $0 - 4,999
* 2: $5,000 - 9,999
* 3: $10,000 - 14,999
* 4: $15,000 - 19,999
* 5: $20,000 - 24,999
* 6: $25,000 - 34,999
* 7: $35,000 - 44,999
* 8: $45,000 - 54,999
* 9: $55,000 - 64,999
* 10: $65,000 - 74,999
* 11: $\ge$ $75,000 (2003-2004 and 2005-2006 cycles only)
* 12: $\ge$ 20,000
* 13: < 20,000
* 14: $75,000 - 99,999 (2007-2008 cycles onward)
* 15: $\ge$ $100,000

It's not immediately clear which variable to use, but one would guess that they're highly correlated. Indeed:


{% highlight r %}
all_data_c %>% 
  select(household_income, family_income) %>% 
  cor(use = "complete.obs", method = "pearson")
{% endhighlight %}



{% highlight text %}
## Error in eval(lhs, parent, parent): object 'all_data_c' not found
{% endhighlight %}

Since the Pearson correlation coefficient is very high, we can choose either. I chose annual family income to account for younger women, or women that reside with family. 

Next, I wanted to simplify analysis, make the levels more intuitive, and represent income levels in rough accordance with US class breakdown (poor, lower-middle class, middle and upper- middle class, and upper class). The income variable was collapsed into four annual income levels: 

* level 1: < $20,000 
* level 2: $20,000 - 45,000 
* level 3: $45,000 - 100,000 
* level 4: > $100,000  

We will recode the education variable per the levels above, drop refused/don't know/missing income observations, and ask R to interpret the variable type as categorical:

{% highlight r %}
all_data_c = 
  all_data_c %>% 
  select(- household_income) %>% 
  mutate(family_income = 
           if_else(family_income %in% c(1:4, 13), 1, 
                   if_else(family_income %in% c(5:7), 2, 
                           if_else(family_income %in% c(8:11, 14), 3, 
                                   if_else(family_income == 15, 4, 20)
                                   )
                           )
                   )
  ) %>%
  filter(!family_income %in% c(20, NA)) %>% 
  mutate(family_income = as.factor(family_income))
{% endhighlight %}



{% highlight text %}
## Error in eval(lhs, parent, parent): object 'all_data_c' not found
{% endhighlight %}

Working with the education variable is a bit more straightforward. The NHANES categories for adults over 20 years of age are as follows: 

* 1: < 9th grade
* 2: 9-11th grade
* 3: High school grad/GED
* 4: Some college/AA degree
* 5: College grad and above

Thus, all we need to do is drop the refuse/don't know/missing education observations, fix the variable type to categorical, and we're ready to roll. 

{% highlight r %}
all_data_c = 
  all_data_c %>% 
  drop_na(education) %>% 
  filter(education %in% c(1:5)) %>% 
  mutate(education = as.factor(education))
{% endhighlight %}



{% highlight text %}
## Error in eval(lhs, parent, parent): object 'all_data_c' not found
{% endhighlight %}

### Visualization 
Here we'll look at some boxplots to eyeball trends. We won't plot every phthalate because there's a lot of them and the intent here is exploratory. Instead we'll choose a handful of common ones, including those from both the low and high molecular weight categories. 


{% highlight r %}
# relabel education and income
box_plot_data = 
  all_data_c %>% 
    mutate(
      education = recode(education, "1" = "< 9th grade", "2" = "< High school", "3" = "High school grad/GED", "4" = "Some college/AA degree", "5" = "College graduate and above", .default = NULL, .missing = NULL), 
      family_income = recode(family_income, "1" = "< $20,000", "2" = "$20,000 - 45,000", "3" = "$45,000 - 100,000 ", "4" = "> $100,000", .default = NULL, .missing = NULL)
      )
{% endhighlight %}



{% highlight text %}
## Error in eval(lhs, parent, parent): object 'all_data_c' not found
{% endhighlight %}



{% highlight r %}
# income boxplots
mep_1 = 
  box_plot_data %>% 
    ggplot(aes(x = family_income, y = log(mep_c))) + 
    geom_boxplot() +
    labs(
      title = "Log MEP vs Annual Family Income",
      x = "Annual Family Income",
      y = "Log MEP"
    )
{% endhighlight %}



{% highlight text %}
## Error in eval(lhs, parent, parent): object 'box_plot_data' not found
{% endhighlight %}



{% highlight r %}
mehp_1 = 
  box_plot_data %>% 
    ggplot(aes(x = family_income, y = log(mehp_c))) + 
    geom_boxplot() +
    labs(
      title = "Log MEHP vs Annual Family Income",
      x = "Annual Family Income",
      y = "Log MEHP"
    )
{% endhighlight %}



{% highlight text %}
## Error in eval(lhs, parent, parent): object 'box_plot_data' not found
{% endhighlight %}



{% highlight r %}
minp_1 = 
  box_plot_data %>% 
    ggplot(aes(x = family_income, y = log(minp_c))) + 
    geom_boxplot() +
    labs(
      title = "Log MINP vs Annual Family Income",
      x = "Annual Family Income",
      y = "Log MINP"
    )
{% endhighlight %}



{% highlight text %}
## Error in eval(lhs, parent, parent): object 'box_plot_data' not found
{% endhighlight %}



{% highlight r %}
grid.arrange(mep_1, mehp_1, minp_1, top = "Figure 3: Income vs Phthalates")
{% endhighlight %}



{% highlight text %}
## Error in arrangeGrob(...): object 'mep_1' not found
{% endhighlight %}



{% highlight r %}
# education boxplots
mep_2 = 
  box_plot_data %>% 
    ggplot(aes(x = education, y = log(mep_c))) + 
    geom_boxplot() +
    labs(
      title = "Log MEP vs Education",
      x = "Annual Family Income",
      y = "Log MEP"
    )
{% endhighlight %}



{% highlight text %}
## Error in eval(lhs, parent, parent): object 'box_plot_data' not found
{% endhighlight %}



{% highlight r %}
mehp_2 = 
  box_plot_data %>% 
    ggplot(aes(x = education, y = log(mehp_c))) + 
    geom_boxplot() +
    labs(
      title = "Log MEHP vs Education",
      x = "Education",
      y = "Log MEHP"
    )
{% endhighlight %}



{% highlight text %}
## Error in eval(lhs, parent, parent): object 'box_plot_data' not found
{% endhighlight %}



{% highlight r %}
minp_2 = 
  box_plot_data %>% 
    ggplot(aes(x = family_income, y = log(minp_c))) + 
    geom_boxplot() +
    labs(
      title = "Log MINP vs Education",
      x = "Education",
      y = "Log MINP"
    )
{% endhighlight %}



{% highlight text %}
## Error in eval(lhs, parent, parent): object 'box_plot_data' not found
{% endhighlight %}



{% highlight r %}
grid.arrange(mep_2, mehp_2, minp_2, top = "Figure 4: Education vs Phthalates")
{% endhighlight %}



{% highlight text %}
## Error in arrangeGrob(...): object 'mep_2' not found
{% endhighlight %}
<br>
Admittedly, these boxplots are not as earth-shattering as I'd hoped. There's definitely a lot of noise and not a definitive consistent pattern. But we can still glean some weak-ish trends. MEP biomarker levels show a slight decline with increasing income and education, MiNP shows the opposite trend, and MEHP fluctuates with both. We'll need more rigorous analysis to pin-point relationships. Which means, it's time to [model](https://en.wikipedia.org/wiki/America%27s_Next_Top_Model). 

### Regression Models
The relationship between SES and phthalate levels is quantified below using multivariable linear regression models (a nice primer on regression modeling can be found [here](https://academic.oup.com/ejcts/article/55/2/179/5265263)). Phthalates are modeled as outcome variables and are log-transformed to address the right-skew discussed above. Income and education are modeled as independent variables, and age is included as a covariate to adjust for potential confounding effects. Creatinine is added as a covariate to adjust for urine dilution.

This is where it gets a little tricky. NHANES uses a complex, stratified survey design (detailed [here](https://www.cdc.gov/nchs/data/series/sr_02/sr02_162.pdf)), along with over-sampling in accordance with the US census. Simply put, the ~10,000 individuals that participate in NHANES do not constitute a perfect random sample of the entire US population. Inevitably, some demographic groups will be over- or under-represented and some might not be represented at all. Therefore, each individual is assigned a sampling weight, which allows us to extrapolate results to the US population.

Because of this, it's not enough to put the variables of interest into a model and hit play. Instead, we must utilize the `survey` package and the three survey variables, `WTMEC2YR, SDMVPSU,SDMVSTRA` that we've ignored up to now. The way these variables work is somewhat complicated and we won't go into detail, but you can read all about it [here](https://www.cdc.gov/nchs/data/series/sr_02/sr02-184-508.pdf). For our purposes, we need two steps: 1. create a survey design object using the survey variables and 2. carry out regression modeling using `svyglm`. [This site](https://stats.idre.ucla.edu/r/faq/how-can-i-do-regression-estimation-with-survey-data/) provides a reference for both. 


{% highlight r %}
# adjust weighting variable for aggregation of 7 cycles
all_data_c$WTMEC14YR = all_data_c$WTMEC2YR/7
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'all_data_c' not found
{% endhighlight %}



{% highlight r %}
# create log variables 
all_data_c = 
  all_data_c %>% 
  mutate(log_mecpp = log(mecpp),
         log_mnbp = log(mnbp),
         log_mcpp = log(mcpp),
         log_mep = log(mep),
         log_mehhp = log(mehhp),
         log_mehp = log(mehp),
         log_mibp = log(mibp),
         log_meohp = log(meohp),
         log_mbzp = log(mbzp),
         log_mcnp = log(mcnp),
         log_mcop = log(mcop),
         log_minp = log(minp)
         )
{% endhighlight %}



{% highlight text %}
## Error in eval(lhs, parent, parent): object 'all_data_c' not found
{% endhighlight %}



{% highlight r %}
# selecting needed variables
all_data_m = 
  all_data_c %>% 
  filter(cycle %in% c("2003-2004","2005-2006","2007-2008","2009-2010","2011-2012", "2013-2014", "2015-2016")) %>%
  select(age, education, family_income, SDMVPSU, SDMVSTRA, WTMEC14YR, creatinine, log_mecpp, log_mnbp, log_mcpp, log_mep, log_mehhp, log_mehp, log_mibp, log_meohp, log_mbzp, log_mcnp, log_mcop, log_minp)
{% endhighlight %}



{% highlight text %}
## Error in eval(lhs, parent, parent): object 'all_data_c' not found
{% endhighlight %}



{% highlight r %}
# create survey design variable
svy_design = svydesign(id = ~SDMVPSU, strata = ~SDMVSTRA, data = all_data_m, weights = ~WTMEC14YR, nest = TRUE)
{% endhighlight %}



{% highlight text %}
## Error in svydesign(id = ~SDMVPSU, strata = ~SDMVSTRA, data = all_data_m, : object 'all_data_m' not found
{% endhighlight %}

Whew. Now we're finally ready to model. Based on the variables we're using, our theoretical model takes the following form. Note that because we have two multi-level categorical variables, we need to create _n-1_ "dummy" variables, n being the number of levels (more on dummy variables [here](https://stats.idre.ucla.edu/other/mult-pkg/faq/general/faqwhat-is-dummy-coding/)). 

$$y_i = \beta_0 +\beta_1*income_1+\beta_2*income_2+\beta_3*income_3+\beta_4*education_1+\beta_5*education_2+\beta_6*education_3+\beta_7*education_4+\beta_8*age+\beta_9*creatinine+\epsilon_i $$
$$income_1 = \{1\ for\ 20k-45k \ vs <20k, \ 0\ otherwise\}$$
$$income_2 = \{1\ for\ 45k-100k \ vs <20k, \ 0\ otherwise\}$$
$$income_3 = \{1\ for\ >100k\ vs <20k, \ 0\ otherwise\}$$
$$education_1 = \{1\ for\ 9-11th\ grade \ vs <9th\ grade, \ 0\ otherwise\}$$
$$education_2 = \{1\ for\ High\ school\ grad/GED \ vs <9th\ grade, \ 0\ otherwise\}$$
$$education_3 = \{1\ for\ Some\ college/AA\ degree \ vs <9th\ grade, \ 0\ otherwise\}$$
$$education_4 = \{1\ for\ College\ and\ above \ vs <9th\ grade, \ 0\ otherwise\}$$

The fitted models are calculated below. It'll be a lot of numbers but bear with me. We'll get through it together.  

{% highlight r %}
mecpp_model = svyglm(log_mecpp ~ age + family_income + education + creatinine, design = svy_design, data = all_data_m) 
{% endhighlight %}



{% highlight text %}
## Error in .svycheck(design): object 'svy_design' not found
{% endhighlight %}



{% highlight r %}
mnbp_model = svyglm(log_mnbp ~ age + family_income + education + creatinine, design = svy_design, data = all_data_m) 
{% endhighlight %}



{% highlight text %}
## Error in .svycheck(design): object 'svy_design' not found
{% endhighlight %}



{% highlight r %}
mcpp_model = svyglm(log_mcpp ~ age + family_income + education + creatinine, design = svy_design, data = all_data_m)
{% endhighlight %}



{% highlight text %}
## Error in .svycheck(design): object 'svy_design' not found
{% endhighlight %}



{% highlight r %}
mep_model = svyglm(log_mep ~ age + family_income + education + creatinine, design = svy_design, data = all_data_m) 
{% endhighlight %}



{% highlight text %}
## Error in .svycheck(design): object 'svy_design' not found
{% endhighlight %}



{% highlight r %}
mehhp_model = svyglm(log_mehhp ~ age + family_income + education + creatinine, design = svy_design, data = all_data_m) 
{% endhighlight %}



{% highlight text %}
## Error in .svycheck(design): object 'svy_design' not found
{% endhighlight %}



{% highlight r %}
mehp_model = svyglm(log_mehp ~ age + family_income + education + creatinine, design = svy_design, data = all_data_m) 
{% endhighlight %}



{% highlight text %}
## Error in .svycheck(design): object 'svy_design' not found
{% endhighlight %}



{% highlight r %}
mibp_model = svyglm(log_mibp ~ age + family_income + education + creatinine, design = svy_design, data = all_data_m) 
{% endhighlight %}



{% highlight text %}
## Error in .svycheck(design): object 'svy_design' not found
{% endhighlight %}



{% highlight r %}
meohp_model = svyglm(log_meohp ~ age + family_income + education + creatinine, design = svy_design, data = all_data_m) 
{% endhighlight %}



{% highlight text %}
## Error in .svycheck(design): object 'svy_design' not found
{% endhighlight %}



{% highlight r %}
mbzp_model = svyglm(log_mbzp ~ age + family_income + education + creatinine, design = svy_design, data = all_data_m) 
{% endhighlight %}



{% highlight text %}
## Error in .svycheck(design): object 'svy_design' not found
{% endhighlight %}



{% highlight r %}
mcnp_model = svyglm(log_mcnp ~ age + family_income + education + creatinine, design = svy_design, data = all_data_m) 
{% endhighlight %}



{% highlight text %}
## Error in .svycheck(design): object 'svy_design' not found
{% endhighlight %}



{% highlight r %}
mcop_model = svyglm(log_mcop ~ age + family_income + education + creatinine, design = svy_design, data = all_data_m) 
{% endhighlight %}



{% highlight text %}
## Error in .svycheck(design): object 'svy_design' not found
{% endhighlight %}



{% highlight r %}
minp_model = svyglm(log_minp ~ age + family_income + education + creatinine, design = svy_design, data = all_data_m) 
{% endhighlight %}



{% highlight text %}
## Error in .svycheck(design): object 'svy_design' not found
{% endhighlight %}



{% highlight r %}
# arranging models into tables
tab_model(mecpp_model, mnbp_model, mcpp_model, show.stat = TRUE, show.aic = FALSE, title = "Table 2: Regression Results for MECPP, MnBP, and MCPP")
{% endhighlight %}



{% highlight text %}
## Error in tab_model(mecpp_model, mnbp_model, mcpp_model, show.stat = TRUE, : object 'mecpp_model' not found
{% endhighlight %}



{% highlight r %}
tab_model(mep_model, mehhp_model, mehp_model, show.stat = TRUE, show.aic = FALSE,  title = "Table 3: Regression Results for MEP, MEHHP, and MEHP")
{% endhighlight %}



{% highlight text %}
## Error in tab_model(mep_model, mehhp_model, mehp_model, show.stat = TRUE, : object 'mep_model' not found
{% endhighlight %}



{% highlight r %}
tab_model(mibp_model, meohp_model, mbzp_model, show.stat = TRUE, show.aic = FALSE,  title = "Table 4: Regression Results for MIBP, MEOHP, and MBZP")
{% endhighlight %}



{% highlight text %}
## Error in tab_model(mibp_model, meohp_model, mbzp_model, show.stat = TRUE, : object 'mibp_model' not found
{% endhighlight %}



{% highlight r %}
tab_model(mcnp_model, mcop_model, minp_model, show.stat = TRUE, show.aic = FALSE,  title = "Table 5: Regression Results for MCNP, MCOP, and MINP")
{% endhighlight %}



{% highlight text %}
## Error in tab_model(mcnp_model, mcop_model, minp_model, show.stat = TRUE, : object 'mcnp_model' not found
{% endhighlight %}

<br>
Let's dissect the results. If we look at income first, we see generally negative values for the parameter estimates. This signals that higher income generally corresponds with lower urinary phthalate concentration. The parameters are pretty small and most are not statistically significant at an alpha level of 0.05. Notably, at the highest family income category, family_income_4 (meaning the highest earners vs the lowest earners), the estimates tend to be higher and most of them __are__ statistically significant. This means that, in general, there is only an association between annual family income and phthalate exposure at the extremes of income. 

Looking at education, we also see very small estimates for the beta parameters and almost all of them are not statistically significant. 

So is there an association between socioeconomic status and phthalate exposure? Eh... not really. (I know, that's a very scientific answer.) There are some weak relationships but based on data gathered from 13 years of observation, phthalates seem to be everywhere and permeate all levels of society. But hey, at least we know they've decreased from the early 2000s. 

# Conlusions 
After all this, I think we've successfully summarized phthalate exposure in US women aged 20-44 between 2003 and 2016. We've learned the following: 

* Overall phthalate levels have decreased between 2003 and 2016. The most dramatic decrease was observed in Mono-ethyl Phthalate, which has unique applications including use in cosmetics and personal care products. The reason for this sharp decline is not entirely clear, but the investigation led to some enlightening insights into the weaknesses of US toxic chemical tracking. 
* The exposure levels, as measured in urine samples, are comparable to previous studies using cohorts. Some of this work established associations between phthalate exposure during pregnancy and adverse effects on fetal reproductive development. The levels observed here, however, are much lower than toxic exposure levels in animal studies. 
* Multivariable regression modeling did not establish strong associations between phthalate exposure and measures of socioeconomic status. 

I hope you agree that we've learned a lot of good stuff. It's clear to me that we don't really understand the full story. Research on phthalates didn't start until the early 2000s and plastics themselves are a relatively young addition to the human toolkit. So there's no question that further work needs to be done. Until then, I feel _a little_ less anxious about phthalates. Still, I will definitely not be sticking plastic in the microwave and if you're pregnant, you probably shouldn't either. 

Thank you for reading. Comments and questions are always welcome. 

# Further Reading
For more than you ever wanted to know about phthalates, check out this publicly available [report](https://www.ncbi.nlm.nih.gov/books/NBK215040/) published by the National Academies and this [document](https://www.epa.gov/sites/production/files/2015-09/documents/phthalates_actionplan_revised_2012-03-14.pdf) published by the EPA. For something a bit more entertaining, The Guardian published this pretty comprehensive [article](https://www.theguardian.com/lifeandstyle/2015/feb/10/phthalates-plastics-chemicals-research-analysis) in 2015. 


[^1]: Since 1 mg/dL = 10,000 ng/mL, we divide phthalate concentration by 10,000 to get the units to match, then take phthalate/creatinine. Most values are in the $1e-6$ (micro) order of magnitude, so we express the final adjusted answer in micrograms phthalate per gram creatinine.


# References
