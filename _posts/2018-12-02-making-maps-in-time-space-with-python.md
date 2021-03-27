---
layout: post
title: "Making Maps in 'Time-Space' with Python"
date: 2018-12-02
tags: [geography, python]
splash_img_source: /assets/images/blogPosts/stlCityCounty.jpg
permalink: "blog/making-maps-in-time-space-with-python"
---

_You can find the code for this on [my Github](https://github.com/nequals30/city-timespace). I walk through it there._

How can we make maps by travel-time, rather than distance?

This project was inspired by [this analysis](http://gradientmetrics.com/new-york-city-in-timespace) from [Gradient Metrics](http://gradientmetrics.com/) in which they re-projected Manhattan Island into “Timespace”, which shows how long it takes to get from any two points (via public transportation, in their case).

Unfortunately, they didn’t share their code. I’ve tried to do the same analysis in Python, and generalize it so it works with any place (e.g. city, county, state, etc). For example, here is St. Louis City and County:

![STL City and County](/assets/images/blogPosts/stlCityCounty.jpg)

On the left is the St. Louis area in “__distance space__“, where the dots are all equally spaced on a map. On the right are the same dots in “__time space__“, where the distances between the points represent driving time, instead of geographic distance.

The take-aways from this map are not super obvious, but we can begin to see a few things:

* Getting places is slower as you approach the city (the right side of the map)
* There are some places deep in the county (bottom left) that are remote
You can see the points clump together in lines. These are highways — it’s easier to get between two points on the same highway.

We could, of course, make the same map by walking travel-time, rather than driving. The results should not surprise you:

![STL City and County (Walking)](/assets/images/blogPosts/stlCityCounty_walking.jpg)

Yep — it takes about the same amount of time to walk between equally spaced points. This shows that takes roughly 600 minutes (10 hours) to walk across the entire St. Louis area.

Finally, here’s an example of another city, Washington DC. This is at 8pm on a Saturday, but you can imagine it would look very different during rush hour.

![DC in Time-Space](/assets/images/blogPosts/dc.jpg)

### How It Works

1. I download [shapefiles from the census](https://www.census.gov/geo/maps-data/data/cbf/cbf_counties.html) to draw the boundaries of the areas.
2. The script calculates evenly spaced points (latitude/longitude) within the boundary.
3. It uses the [Bing Maps API](https://msdn.microsoft.com/en-us/library/mt827298.aspx) to pull down all of the driving times between all of the points. Then it turns them into a [distance matrix](https://en.wikipedia.org/wiki/Distance_matrix). I would of used Google Maps, but I’m not sure it’s possible anymore.
4. It uses [multidimensional scaling](https://en.wikipedia.org/wiki/Multidimensional_scaling) (aka Principal Coordinates Analysis) to arrange the points in “Time-Space”.
5. It attempts to align the “distance-space” coordinates with the “time-space” coordinates using regular Principal Component Analysis.
6. It uses [Matplotlib](https://en.wikipedia.org/wiki/Matplotlib) to plot the results.

### Future Improvements
* __More points__. Right now that’s limited by the size of the query to Bing Maps API, but it can be fixed by making multiple queries.
* __Better visualization__. I threw this together quickly in Matplotlib, but I think the data can be presented better.
* __Public transportation__. Currently Bing Maps API doesn’t handle this well.
* __Travel-time at certain hours__. As you can imagine, the map would look very different during rush hour. Currently, Bing Maps API isn’t very good at giving you data during alternate hours, but one can imagine a GIF which shows how the suburbs get “farther away” in time-space during a heavy commute.

Thanks for reading! If you have any questions, hit me up at (this website name)@gmail.com

