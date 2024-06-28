---
layout: post
title: "Deeper Webscraping"
date: 2024-06-27 10:00:00 -0000
categories: blog
---
In order to extend the data bank for [WeSonder](https://wesonder.com/) (thereby increasing its accuracy and reliance) we ought to increase the sources of what that data is extracted from, 
and when it comes of construction, a great source of users and entities is [Planetbids](https://home.planetbids.com/).
The problem is that unlike my [prior post](https://damiamalfaro.github.io/blog/2024/06/25/CaliforniaMunicipalities.html), Planetbids cannot be scraped by solely [BeautifulSoup](https://beautiful-soup-4.readthedocs.io/en/latest/). You see,
here we have to be creative and perhaps smarter and use something stronger. Why? Planetbids is full of [Javascript](https://www.simplilearn.com/html-vs-javascript-article#:~:text=The%20most%20basic%20difference%20is,the%20behavior%20of%20the%20page.) and not solely HTML & CSS, which is harder to scrap from.
Which is why we will be using [Selenium](https://www.selenium.dev/documentation/).

```python3
from selenium import webdriver
from selenium.webdriver.common.by import By

driver = webdriver.Chrome()
```
The main difference I've realized between BeautifulSoup and Selenium is that selenium actually interacts with the webpage and does more than extracting the
pure static text found in the webpage (as it is the case with BeautifulSoup).
We actually [interact with elements](https://www.selenium.dev/documentation/webdriver/elements/interactions/) such as buttoms, clicks, table drawdowns, etc.
We are going to use the [Planetbids City of Rialto](https://vendors.planetbids.com/portal/28159/bo/bo-search) webpage as an example in our "Selenium Journey".
```python3
driver.get("https://vendors.planetbids.com/portal/28159/bo/bo-search")
title = driver.title
driver.implicitly_wait(100)
# This will tell us how many bids are found in the Municipality's Planetbids webpage currently, doing so we check if a new bid was recently added.
totalMunicipalityBids = driver.find_element(By.CLASS_NAME, 'bids-table-filter-message')
print(element.text)
```
```python3
Found 652 bids
```
