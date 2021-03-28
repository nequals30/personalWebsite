---
layout: post
title: "Baby Naming Algorithm (Python)"
date: 2019-10-27
tags: [python]
permalink: "blog/baby-naming-algorithm"
---
To help name my coworkers babies, I wrote this script. It works as follows:

1. Popularity:  This is done via a [beta distribution](https://en.wikipedia.org/wiki/Beta_distribution). An alpha parameter of 1 prioritizes the most common names, and higher numbers produce less common names.
2. Alliteration: Prioritizes first names to match the first letter of the last name.
3. Syllable count: Lowers priority of names in which the first name has the same number of syllables as the last name. It uses the [syllables](https://pypi.org/project/syllables/) python package.

To run it, you will need the top baby name list for a given year (as a txt file), which you can get from this [zip file](https://www.ssa.gov/oact/babynames/names.zip) from the [Social Security website](https://www.ssa.gov/oact/babynames/limits.html).

This is the code:

``` python
import pandas as pd
import syllables
from scipy.stats import beta
 
# inputs
gender = 'M'
lastName = 'Matthews'
alphaParam = 1 # commonness(1 = very common, 1.5 = medium common 100 = very uncommon)
 
# parameters
betaParam = 2 # how much commonness matters (probably dont change)
wgt_common = 1000
wgt_allit = 0
wgt_badSyl = 1
 
# pull data and filter gender
data = pd.read_csv('yob2018.txt', sep=",", header=None, names=['name','gender','count'])
data = data.loc[data.gender==gender,]
 
# Compute a commonality score
data['pctle'] = (data['count']/sum(data['count']))[::-1].cumsum()[::-1]
data['beta'] = beta.pdf(abs(data['pctle']-1),alphaParam,betaParam)
data['common'] = data['beta']/sum(data['beta'])
 
# Compute alliteration score
data['allit'] = data['name'].str[:1] == lastName[:1].upper()
 
# Compute syllable score
data['sylcount'] = data['name'].apply(syllables.estimate)
data['badSyl'] = data['sylcount'] == syllables.estimate(lastName)
 
# Compute total score
data['score'] = data.common*wgt_common + data.allit*wgt_allit - data.badSyl*wgt_badSyl
data = data.sort_values(by=['score'],ascending=False)
print(data.name[:20])
```
