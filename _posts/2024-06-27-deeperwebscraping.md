---
layout: post
title: "Planetbids Webscraping"
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
Now that we know that we can interact with the webpage, we need to find our crux, where we ought to focus on, which in this case
is each individual bids. Thing is that each bid is found within a <tr> element, therefore we have to go over each of the
layers and find each individual <tr>:
```python3
# Tabulation of bids
total_bids = driver.find_element(By.CLASS_NAME,"bids-table-filter-message") # int

current_bids = start.find_element(By.CLASS_NAME,"table-overflow-container")

# Only displays the first 30
bids = current_bids.find_elements(By.TAG_NAME,"tr") # int
```
This code allow us to visualize which bids we have, but there is one problem: some planetbids portals have multiple
bids post and not just the first default 30 showed in the main page. 
```python3
bids = current_bids.find_elements(By.TAG_NAME,"tr") # int
print(len(bids))
```
```
32
```
Why 32? Because there are two tabulations (tr elements) that are counted as tr elements, the header and subheader. Afterwards
the first 30 bids start to display.
In order to solve this issue we will need to [scroll down](https://stackoverflow.com/questions/20986631/how-can-i-scroll-a-web-page-using-selenium-webdriver-in-python) using
selenium's method. However, keep in mind that we need to scroll down not in the webpage itself, but in the table container, as in the case of planetbids website design it is
the table element that contains all of the bids, thereby the element to scroll down within. Let's create a function to do so:
```python3
# Scrolls down within the table container
def scroll_table_container(container, scroll_pause_time=1):
    last_height = driver.execute_script("return arguments[0].scrollHeight", container)
    
    while True:
        # Scroll down by a small amount within the container
        driver.execute_script("arguments[0].scrollTop = arguments[0].scrollHeight", container)
        
        # Wait to load the new content
        time.sleep(scroll_pause_time)
        
        # Calculate new scroll height and compare with last scroll height
        new_height = driver.execute_script("return arguments[0].scrollHeight", container)
        if new_height == last_height:
            break
        last_height = new_height
```
This function takes the table container as input:
```python3
current_bids = start.find_element(By.CLASS_NAME,"table-overflow-container")
```
Now that we have all our bids acknowledged, we can interact with each '<tr>' element and begin scraping from it:
```python3
bids = current_bids.find_elements(By.TAG_NAME,"tr")
print(len(bids))
```
```python3
652
```
However, we are going to proceed with a different approach now. The reason for this is due to the limitations of my poor wifi power.
I am currently using a general collective conventional wifi provider, which limites the power of loading, refreshing, and
updating webpages at the optimal level. Nevertheless, each obstacle in life is a test to ourselves (and our versitality) in order
to observe if one can land in one's feet. One should always be capable of doing so, our versitality should always be up to date. Why?
The volatility of life is ruthless; landing on one's feet is the only ability that will secure safety on the face of adversity. We are
going to be creating a function for each of the tabs found in each bid <tr> item, and loop through all bids <tr> items in chunks
of 10. Sure, this will be time consuming but functional nonetheless. In my contemporary world time is the most expensive and valuable
asset that money can buy. I am not quite there yet...
![PlanetbidsBidHeader](/assets/images/planetbids_header.png)


