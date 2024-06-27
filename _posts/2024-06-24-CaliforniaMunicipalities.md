---
layout: post
title: "Web Scraping California Municipalities"
date: 2024-06-25 10:00:00 -0000
categories: blog
---

Currently I find myself working on a project involving all Municipalities of California. 
Then I began to wonder, where can I find a list of all municipalities? Then I thought about the greatest
enclyclopedia found in Earth (at least the one that I know of); ![Wikipedia](https://www.wikipedia.org/). Free, public, and trustworthy.
In it, I found what I was looking for:

![List of municipalities in California](https://en.wikipedia.org/wiki/List_of_municipalities_in_California) 

![WikiPageTitle](/assets/images/CAMunicipalitiesWikiPageTitle.png)
![WikiPageExample](/assets/images/CAMunicipalitiesTableHead.png)

I suddenly decided to do what I do best, and perhaps one of the things I love the most: materializing my ideas using
programming, mathematics, and critical thinking. 
The idea was to get a csv file as a result from the webscraping of the newly found page, therefore I got to work instantly
after that desired result:

```python
from bs4 import BeautifulSoup
import requests
import pandas as pd


url = "https://en.wikipedia.org/wiki/List_of_municipalities_in_California"

souping = requests.get(url)
soup = BeautifulSoup(souping.content, 'html.parser')
text = soup.get_text() # Pure text without html attributes, useful for strings
```
We are using ![Beautifulsoup](https://beautiful-soup-4.readthedocs.io/en/latest/) to webscrap (I am suprised webscrap is not an official word
in the English dictionary still???) from the website. 
