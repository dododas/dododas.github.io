---
title: "Odds ratio computations in Julia"
date: 2023-06-01T20:08:06-07:00
katex: True
categories: ["Statistics"]
tags: ["Julia", "Bootstrap"]
draft: True
# bookComments: false
# bookSearchExclude: false
---

This post explains the concept of an odds ratio and how to compute an odds ratio and its confidence interval using bootstrap resampling. Data from two published studies are analyzed with Julia code. 

<!--more-->

## Odds ratio and relative risk

Odds ratio and relative risk are two related statistics that measure the strength of association between two variables - a risk factor or treatment, such as smoking, or taking a daily dose of aspirin, and an outcome of interest, such as a lung cancer diagnosis or getting a heart attack. Note that both the factor and the outcome are binary events. For example, a subject gets either the placebo or the treatment, and the outcome of interest either happens or not during the study period. 

The odds ratio is the odds of the outcome for the treatment group relative to the odds for the placebo group:
$$ \text{Odds ratio } = \frac{\text{odds} _{\sf{treatment}}}{\text{odds} _{\sf{placebo}}} $$
and the relative risk is the risk of the outcome for the treatment group divided by the risk for the placebo group:
$$ \text{Relative risk } = \frac{\text{risk} _{\sf{treatment}}}{\text{risk} _{\sf{placebo}}} $$


The data needed to compute an odds ratio are commonly summarized in a contingency table. Each cell of the table lists the number of subjects or events in each category, as seen in the two examples below: 

### Example 1: Aspirin, heart attacks and strokes

These data are from a randomized controlled trial that followed 22071 physicians who were randomly assigned to receive either a 325 mg dose of aspirin or a placebo every other day. The number of heart attacks and strokes recorded during the 60 month study period are shown in the contingency table below:

Treatment | Heart attacks | Strokes | Total
--------- | ------------- | ------- | -----   
Placebo   | 239           | 98      | 11034
Aspirin   | 139           | 119     | 11037

These data suggest that taking a daily dose of aspirin is associated with a lower incidence of heart attacks (139 in the aspirin group -vs- 239 in the placebo group). But there is also a small but measurable increase in the number of strokes from 98 in the placebo group to 119 in the aspirin group. 

### Example 2: Smoking and lung cancer

A landmark study by Doll and Hill reported the association between smoking and lung cancer by comparing the proportion of smokers amongst patients diagnosed with lung cancer with an age and sex matched sample of patients who did not have lung cancer. The following contingency table summarizes the data reported for all male patients in this case-control study:

Group       | Smokers | Non-smokers | Total
----------- | ------- | ----------- | -----
Control     | 622     | 27          | 649
Lung cancer | 647     | 2           | 649

Though a majority of the subjects in each group are smokers (this being the 1940s!), it is striking that there are only 2 nonsmokers in the lung cancer group. 

The next section analyzes each of these examples by computing an odds ratio for each case. The estimated odds ratio and its confidence interval is used to evaluate the benefit versus risk of taking aspirin, and to quantify the strength of association between smoking and lung cancer. 

## Computing odds ratio 

**Example calculation**: The odds ratio of getting a heart attack for the aspirin group -vs- placebo is given by:
$$ \begin{aligned} 
\text{OR } & = \frac{ \text{No. of heart attacks} _{\text{aspirin}} / (\text{Total - No. of heart attacks})  _{\text{aspirin}}}{ \text{No. of heart attacks} _{\text{placebo}} / (\text{Total - No. of heart attacks})  _{\text{placebo}} } \\\
& = \frac{139/(11037 - 139)}{239/(11034 - 239} \\\
& = 0.576
\end{aligned} $$

The following block of Julia code uses the `TypedTables.jl` package to create contingency table:
```julia
using TypedTables
t = Table(Treatment    = ["Placebo", "Aspirin"],
          HeartAttacks = [239, 139], 
          Strokes      = [98, 119],
          n            = [11034, 11037])

Table with 4 columns and 2 rows:
     Treatment  HeartAttacks  Strokes  n
   ┌────────────────────────────────────────
 1 │ Placebo    239           98       11034
 2 │ Aspirin    139           119      11037
```

Next, define a function to compute the odds ratio from such a contingency table:
```julia
function OddsRatio(ctable::Table, outcome::Symbol; total::Symbol = :n)
    """Compute odds ratio for an outcome of interest
    from input contingency table"""
    e = getproperty(ctable, outcome)
    n = getproperty(ctable, total)
    odds = e ./ (n - e)
    return odds[2]/odds[1]
end
OddsRatio (generic function with 1 method)
```

Using this function compute the odds ratio of the aspirin group relative to placebo for getting either a heart attack or a stroke: 
```julia
OddsRatio(t, :HeartAttacks)
0.576093191257695

OddsRatio(t, :Strokes)
1.2162876507994662
```

These odds ratios quantify the effect of taking aspirin on each of the outcomes. The odds of getting a heart attack for the aspirin groups is about 40% lower than for the placebo group, but the odds of getting a stroke is about 20% higher. This suggests an overall beneficial effect of taking aspirin. However, these *point estimates* of the odds ratios do not tell the whole story as they don't offer any information about uncertainty, such as a standard error, or a confidence interval. 

The next section describes how to use bootstrap resampling to compute a confidence interval for the odds ratio. 

## Bootstrap confidence interval estimation 

Bootstrap is a computational technique that uses random resampling of observed data to simulate the behavior of a statistic of interest, such as the odds ratio. The 


## References

1. "Final report on the aspirin component of the ongoing Physicians' Health Study", Steering Committee of the Physicians' Health Study Research Group https://pubmed.ncbi.nlm.nih.gov/2664509/

2. "Smoking and Carcinoma of the Lung", Doll and Hill, 


