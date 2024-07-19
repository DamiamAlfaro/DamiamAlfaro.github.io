---
layout: post
title: "Planetbids Webscraping"
date: 2024-06-27 10:00:00 -0000
categories: blog
---
### Project: [planetbids_extraction.ipynb](https://github.com/DamiamAlfaro/Europe-WeSonder/tree/main/DataSources/Planetbids)

In order to extend the data bank for [WeSonder](https://wesonder.com/) (thereby increasing its accuracy and reliance) we ought to increase the sources of what that data is extracted from, 
and when it comes of construction, a great source of users and entities is [Planetbids](https://home.planetbids.com/).
The problem is that unlike my [prior post](https://damiamalfaro.github.io/blog/2024/06/25/CaliforniaMunicipalities.html), Planetbids cannot be scraped by solely [BeautifulSoup](https://beautiful-soup-4.readthedocs.io/en/latest/). You see,
here we have to be creative and perhaps smarter and use something stronger. Why? Planetbids is full of [Javascript](https://www.simplilearn.com/html-vs-javascript-article#:~:text=The%20most%20basic%20difference%20is,the%20behavior%20of%20the%20page.) and not solely HTML & CSS, which is harder to scrap from.
Which is why we will be using [Selenium](https://www.selenium.dev/documentation/).

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
From now on we are going to be using [Jupyter Notebooks](https://docs.jupyter.org/en/latest/) in order to better visualize our progress and procedures.
I will leave the above's explanations as they are quite useful for someone who is just learning about webscraping, but from now on I will be sharing my procedures
exactly as how they were implemented along with their reasoning.
First of all, we will be importing multiple modules from selenium and assigning our testing website. 

### Explanation Onset

```python3
import sys
import pandas as pd
from pandas.errors import EmptyDataError
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import TimeoutException
import time
```

```python3
url = "https://vendors.planetbids.com/portal/28159/bo/bo-search"
```
We will be using the city of rialto as an example as I recently won a project for my current employeer in that municipality. 

Anyway, as describe above, the first step is to build the function that will scroll down the <table> element within the planet bids portal:

![table element screenshot](/assets/images/planetbids_extraction_image1.png)

As you can see, this <table> element is found within the webpage; the element contains 652 <tr> elements, each of which representing a separate link for a new tab containing a different bid (which we want to scrap). However, as explained before, the function *scroll_table_container()* does the automatic job for us for exchange of a few seconds, fair deal. 

```python3
def scroll_table_container(container, driver,scroll_pause_time=1):
    
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
However, before commencing with the heavy machinery, let's raise the question: what are we looking for here? In a short paragraph, what we are looking for in this program is to extract seven attributes (if applicable) from each bid:
1. Bid Information: address, date, scope of work, estimated value, bid number, duration, among other attributes of lesser hierarchy.
2. Line items: the items that, according to the [Awarding body](https://www.dir.ca.gov/public-works/awarding-bodies.html), are the items that has to be included in the project, or in another words: the scope of work of the project broken down detaily using Unit Of Measures (Square Feet, Linear Feet, Cubic Yards, etc.), Lump Sum items, and/or [alternatives](https://www.minnstate.edu/system/finance/facilities/design-construction/pm_emanual/doc/CD.50%20Bid%20Alternates%209-20-19.pdf).
3. Documents: the provided documents by the awarding body (e.g. Project plans, project technical specifications, geotechnical reports, etc.)
4. Addenda/Emails: any [addenda](https://en.wikipedia.org/wiki/Addendum) or email notifications from the bid and their dates.
5. Q&A: any questions that prospective bidders might've asked before the bid due date (they provide insight on how confusing each of the trait is).
6. Prospective Bidder: who was/is interested in this project? Along with their respective information (address, email, phone number, name, contact, etc.)
7. Bid Results: final bidders that posted a bid for the project, their subcontractors (if available; not always revealed), and their line items (if available; not always revealed).

![bid results seven attributes](/assets/images/planetbids_extraction_image2.png)

With that being set, let's begin: 

### First Stage: Set up

```python
def extraction(url,number):
    beacon = 0
    beacon += number

    # Start Selenium's webdriver
    driver = webdriver.Chrome()
    driver.get(url)
    time.sleep(10)

    # Identifies if the total number of bids in the municipality increased
    total_bids = driver.find_element(By.CLASS_NAME,"bids-table-filter-message")

    # Table container with all bids in the municipality's planetbids portal
    current_bids = driver.find_element(By.CLASS_NAME,"table-overflow-container")

    # Scroll through the table container
    scroll_table_container(current_bids,driver)
    driver.implicitly_wait(40)

    # After scrolling, we put all bids in the webpage into a list; n bids
    bids = current_bids.find_elements(By.TAG_NAME,"tr")

    # Output
    bid_general_info = []
    bid_line_items = []
    bid_documents = []
    bid_addenda = []
    bid_q_and_a = []
    bid_prospective_bidders = []
    bid_results = []
```
The first stage is to set up the [Selenium WebDriver](https://www.selenium.dev/documentation/webdriver/), which is our main motor to navigate the website. The varibale *beacon* will serve as a signaling position within the <table> element, e.g. beacon = 19, that means that the targeted bid is the 19th <tr> element within the aforementioned <table> element. The class element *bids-table-filter-message* displays the text message of the total bids found within the website, which is an useful datum as it will eventually signal an addition of a new bid as we iterate the program. 

![total current bids](/assets/images/planetbids_extraction_image3.png)

Next is the <table> element that we've mentioned quite frequently by now, to which we apply the aforementioned *scroll_table_container()* function and scroll through it. The following logical step is to pinpoint all found <tr> elements into a list in order to iterate through them. Lastly for this stage, we create the lists that will contain our main seven attributes we want to extract that were mentioned previously. 

### Second Stage: General Information

```python3
    '''
    General info
    '''
    try:
        # Assuring stability
        table_container = WebDriverWait(driver,10).until(
            EC.presence_of_element_located((By.CLASS_NAME,"bids-table-container"))
        )
    
        current_bids = WebDriverWait(driver,10).until(
            EC.presence_of_element_located((By.CLASS_NAME,"table-overflow-container"))
        )
    
        # The <tr> element we seek
        targeted_bids = bids[beacon+2]
    
        # Once acknowledged, pinpoint it and click on it, aka open it
        driver.execute_script("arguments[0].scrollIntoView();",targeted_bids)
        time.sleep(2)
        print(targeted_bids.text)
        driver.execute_script("arguments[0].click();",targeted_bids)
        
        # Make sure the element was clicked
        WebDriverWait(driver,10).until(
                EC.presence_of_element_located((By.CLASS_NAME,"bid-detail-wrapper"))
            )
    
        # Pinpoint the general info
        general_info = driver.find_element(By.CLASS_NAME,"bid-detail-wrapper")
    
        # Append the general info into the list
        bid_general_info.append(general_info.text.split("\n"))
            
        driver.implicitly_wait(1)
    except:
        print(f"Problem with the general information at {targeted_bids.text}")
        sys.exit(1)

```
We are using [try & except](https://docs.python.org/3/tutorial/errors.html) statements since we do not want to halt the program if an error for a single tab has ocurred: keep in mind that this program is intended to be executed through long period of times: there are **483** municipalities in California, each of them having one or more planetbids portals.
Beginning with the *targeted_bids* variable: as mentioned above, we will singularly target a bid (aka <tr> element) and extract from it every single interation. It is worth mentioning that I tried iterating through multiple <tr> elements within the same WebDriver session, but the code would halt or produce many errors. Why? My internet is that of the [middle class](https://fox5sandiego.com/news/california-news/heres-how-much-you-have-to-earn-in-california-to-qualify-as-middle-class-study-says/#:~:text=In%20California%2C%20the%20middle%2Dclass,it%20was%20%2440%2C933%20to%20%24122%2C800.), not as powerful as the upperclass, but nevertheless enough and scant for what I want to achieve. Or perhaps I am wrong and *cuando la partera es mala le echa la culpa al culo*. I'd go for the former narrative.
The reason for the *+2* in the *targeted_bids* variable is due to the first two items found within the <tr> list *bids* in the first stage: they are basically the headers of the table, and an empty *\n*. 
Once the targeted <tr> element is identified, we 1) scroll into view onto the element, and 2) click on it. We need to scroll into view since the *scroll_table_container()* function iterates until the bottom of the <table> element (variable *bids*), which forces us to scroll back to our *targeted_bids* <tr> element. 
After clicking on the desired bid, we find ourselves in our intended extraction location: the general information tab. We assure stability within it by checking the element class *"bid-detail-wrapper"*, in where we proceed by extracting the page (general information) text splited by *"\n"* in order to pinpoint each of the repeating items that we will utilize to locate important attributes within the general information. 

### Third Stage: Line Items


