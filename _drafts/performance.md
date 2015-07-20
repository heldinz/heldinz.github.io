---
layout: post
title:  "Performance and Usability"
categories: blog
tags: performance pagespeed speed usability user experience
---

This little website of mine is very basic so far. I am planning to add more content over time, but for now the idea is to just do a bit of blogging and use this as my personal playground for trying out some new (or not so new) techniques.

My minimum requirements here are that the website is (a) responsive, (b) usable, and (c) fast. (It is 2015 after all, so this isn't a lot to ask!) The initial version of the site, based on the standard Jekyll templates, wasn't too bad -- it was responsive, but its [PageSpeed][pgspd] score wasn't perfect for either Speed or User Experience on mobile.

The only User Experience issue was one I really should have considered earlier: "Size tap targets appropriately". The issue here was with the links in the footer on small screens. Rather than increasing the size of the links themselves, I opted to increase the space between them by bumping up the margin to 16px. It was a small improvement which made a big difference. My mobile User Experience score is now a perfect 100/100. [Luke W][luke] has collected some information about sizing tap targets if you are interested in reading more about this issue.

As for Speed, I needed to "Eliminate render-blocking JavaScript and CSS in above-the-fold content".


[pgspd]:     https://developers.google.com/speed/pagespeed/insights/
[luke]:      http://www.lukew.com/ff/entry.asp?1085
