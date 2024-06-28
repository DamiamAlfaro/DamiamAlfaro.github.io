---
layout: post
title: "Deeper Webscraping"
date: 2024-06-27 10:00:00 -0000
categories: blog
---
In order to extend the data bank for [WeSonder](https://wesonder.com/) (increasing its accuracy and reliance) we ought to increase the sources of what that data is extracted from, 
and when it comes of construction, a great source of users and entities is [Planetbids](https://home.planetbids.com/).
The problem is that unlike my [prior post](https://damiamalfaro.github.io/blog/2024/06/25/CaliforniaMunicipalities.html), Planetbids cannot be scraped by solely [Beautifulsoup](https://beautiful-soup-4.readthedocs.io/en/latest/). You see,
here we have to be creative and perhaps smarter and use something stronger. Why? Planetbids is full of [Javascript](https://www.simplilearn.com/html-vs-javascript-article#:~:text=The%20most%20basic%20difference%20is,the%20behavior%20of%20the%20page.) and not solely HTML & CSS, which is harder to scrap from.
Which is why we will be using [Selenium](https://www.selenium.dev/documentation/).

```python
from selenium import webdriver
from selenium.webdriver.common.by import By

driver = webdriver.Chrome()
```
