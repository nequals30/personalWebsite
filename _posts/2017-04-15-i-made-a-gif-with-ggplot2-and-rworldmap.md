---
layout: post
title: "I made a GIF with ggplot2 and rworldmap"
date: 2017-04-15
tags: [geography, R]
splash_img_source: /assets/images/blogPosts/nequals30_popDens.gif
splash_img_caption: Population Density GIF
permalink: "blog/gif-with-ggplot-2-and-rworldmap"
---
In my last post, I mentioned that I was interested in exploring the mapping capabilities of ggplot2.

Well, shortly after I wrote that, I was searching the internet for “ggplot2 animated gif”, and I stumbled across [this excellent walkthrough](https://urbandemographics.blogspot.com/2017/03/creating-animated-world-map-of-life.html) by [Rafael Periera](https://twitter.com/UrbanDemog), which he had posted just minutes earlier! His blog, [Urban Demographics](https://urbandemographics.blogspot.com/) is coincidentally already one of my favorites.

I wanted to make an animated map of world population density (such as this one on [Wikipedia](https://upload.wikimedia.org/wikipedia/commons/d/d0/Countries_by_Population_Density_in_2015.svg)).

Using data from the World Bank, I was able to make this GIF with only a modicum of code (on GitHub [here](https://github.com/nequals30/worldBank_population_test/blob/master/map_worldBank_population.R)).

Here is the final product:

![A GIF of Population Density](/assets/images/blogPosts/nequals30_popDens.gif)

So… Maybe the change in the population density over the last 50 years isn’t that exciting. Mostly countries population density hasn’t significantly changed (with the exception of a few outliers such as Nigeria, Uganda, and Pakistan). The density measure is the [population](https://data.worldbank.org/indicator/SP.POP.TOTL) divided by the [land area](https://data.worldbank.org/indicator/AG.LNDS.TOTL.K2), both stats from the World Bank. I tried to write the code in such a way that I will be able to easily expand it to other country-based data later.
