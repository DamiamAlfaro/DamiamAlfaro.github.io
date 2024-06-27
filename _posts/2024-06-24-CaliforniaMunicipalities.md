---
layout: post
title: "Web Scraping California Municipalities"
date: 2024-06-25 10:00:00 -0000
categories: blog
---

Currently I find myself working on a project involving all Municipalities of California. 
Then I began to wonder, where can I find a list of all municipalities? Then I thought about the greatest
enclyclopedia found in Earth (at least the one that I know of); [Wikipedia](https://www.wikipedia.org/). Free, public, and trustworthy.
In it, I found what I was looking for:

[List of municipalities in California](https://en.wikipedia.org/wiki/List_of_municipalities_in_California) 

![WikiPageExample](/assets/images/WebScrapingCaliforniaMunicipalities1.png)

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
We are using [Beautifulsoup](https://beautiful-soup-4.readthedocs.io/en/latest/) to webscrap (I am suprised webscrap is not an official word
in the English dictionary still???) from the website. 

There is a total of 4 [<table>](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/table) elements in the html webpage, we are going to be focusing on the second one, in which
the information of all [municipalities](https://en.wikipedia.org/wiki/Municipality) of California is found.

```python
table = soup.find_all('table') # Total of 4
result = table[1]
```
In order to extract the data from each of the rows of the table (because ultimately, a [table](https://en.wikipedia.org/wiki/Table_(information)) is a table regardless of being
static in a webpage; i.e. it has rows, columns, headings, variables, etc) one has to seek for the [<tr>](https://www.w3schools.com/tags/tag_tr.asp) attribute within the <table> 
html element, as it is that element the one containing the information of each row, i.e. in this case, each municipality.

```python
tablerows = result.find_all('tr')
```
Now, once we find the source of the raw datum, we extract it:

```python
municipalities = []
for muni in range(len(tablerows)):
  municipalityRaw = [tablerows[muni].find_all('th'),tablerows[muni].find_all('td')]
  municipalityAtt = []
  for i in municipalityRaw[0]:
    x = i.text[:-1]
    if x[-1].isalpha() == False:
      municipalityAtt.append(x[:-1])
    else:
      municipalityAtt.append(x)
  for rest in municipalityRaw[1]:
    if "\xa0" in rest.text:
      newString = rest.text.replace('\xa0', ' ')
      municipalityAtt.append(newString[:-1])
    elif municipalityRaw[1].index(rest) == 4:
      if rest.text[0] == "-":
        newString = f"({rest.text[:-2]})"
        municipalityAtt.append(newString)
      else:
        municipalityAtt.append(rest.text[1:-2])
    else:
      municipalityAtt.append(rest.text[:-1])
  municipalities.append(municipalityAtt)
```
Let me break this down:




