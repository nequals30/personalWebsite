---
layout: post
title: "Markdown Calendar"
date: 2019-06-09
tags: [python]
splash_img_source: /assets/images/blogPosts/blog_calendar.png
permalink: "blog/markdown-calendar"
---

Are you somebody who likes the latest and greatest apps for productivity? Perhaps this is not for you.

I didn’t really like the calendar apps in iPhone or Android — I have certain requirements which those apps don’t satisfy.

Up till now, I’ve been using a printed paper calendar and a pen, but I figured it was time to upgrade to a (only slightly) less kludgy solution. So I made [my own calendar system](https://github.com/nequals30/MDcalendar).

![Markdown Calendar](/assets/images/blogPosts/blog_calendar.png)

It has the following features:

* Your calendar is stored as one plain markdown document which is completely readable like an agenda
* You can keep that document anywhere you want, and it’s fully version-controllable
* It also works as a to-do list, and can handle recurring events
* It can be easily turned into HTML, but it could be extended into other forms (e.g. a CLI calendar or an phone app)

I wrote [this script](https://github.com/nequals30/MDcalendar/blob/master/mdcalendar.py) to generate the HTML calendar. Features:

* Shows all events up-front, without having to click on the day
* Fully customizable appearance via a CSS file
* It also cleans up and organizes the markdown “agenda” file

You can find the code for this on [my Github](https://github.com/nequals30/MDcalendar).
