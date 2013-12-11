---
layout: post
title: "JavaScript Date Difference using format : yyyy-mm-dd"
date: 2013-12-11 10:20
comments: true
categories: JavaScript
---

Finding date differences in JavaScript Date objects is trivial but it takes a turn when dealing with multiple timzones. Well, that is up for another discussion but for now let's consider 2 cases of finding difference between two dates using below date formats.

# 1. Using date format - mm/dd/yyyy
{% gist gist_id 7916041 jsdatediff1.js %}

You may notice that the difference from both these ways differs by ``1``. If this is the case and you're using second format [yyyy-mm-dd], then you can solve this using following technique:

{% gist gist_id 7916105 jsdatediff2.js %}

Here, we are considering the timezone offset when a new Date object is created using dateString in  format yyyy-mm-dd.