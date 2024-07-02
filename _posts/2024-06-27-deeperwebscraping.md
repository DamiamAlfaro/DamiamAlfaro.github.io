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
Now that we have access to every single bid within the tabulation, we can jump right into each bid to start scraping.
First, we will get the general information: i.e. Bid date, bid time, bid amount, scope of work, bid title, etc.
```python3
# Input: bids list after scroll and each's general information
def getting_bid_general_information(self):
	bids_general_info = []
	for index in range(len(self)):
		try:
			# We need to refer to the initial bid tabulation each time in order to forbid
			# going astray
			current_bids = driver.find_element(By.CLASS_NAME,"table-overflow-container")
			bids = current_bids.find_elements(By.TAG_NAME,"tr")[2:4]

			# Just checking you know
			print(f"{index}: {bids[index].text}")

			# Clicks every <tr> element within the tabulation in the main page
			bids[index].click()

			# This is just a signal that checks if the click() was successful
			# The 'h3' is the first printable element found in the clicked tab
			WebDriverWait(driver,10).until(EC.presence_of_element_located((By.TAG_NAME,"h3")))
			try:
				# Apparently this is where all general info text is found...
				element = driver.find_element(By.CLASS_NAME,"bid-detail-wrapper")

				# Storing them into a list	
				bids_general_info.append(element.text.split("\n"))

			# We want to see if something is wrong without stoping the code, right?
			except Exception as exc:
				print("\n!!!")
				print("\nThere is something wrong with reading the general info\n")
				print("\n!!!\n")
				print(exc)

			# Remember that we need to go back for the next iteration to work
			driver.back()

			# Again, this is to check that each action worked
			WebDriverWait(driver,10).until(EC.presence_of_element_located((By.CLASS_NAME,'table-overflow-container')))
		except Exception as exe:
			print("\nThere is something wrong with getting_bid_general_information\n")

	# The list that will later be use for analysis
	return bids_general_info
```
The idea is to use the list it returns and store it into a csv.
Now we extract each bid line items in order to analyse patterns later on:
```python3
# Input: bid line items
def getting_bid_line_items(self):

	# Where bid line items will be stored separated by a "\n"
	bids_line_items = []
	for index in range(len(self)):
		try:
			current_bids = driver.find_element(By.CLASS_NAME,"table-overflow-container")
			bids = current_bids.find_elements(By.TAG_NAME,"tr")[2:4]
			print(f"{index}: {bids[index].text}")

			# Clicks a bid <tr> element
			bids[index].click()

			# This is just a signal to make sure the click() worked
			WebDriverWait(driver,10).until(EC.presence_of_element_located((By.TAG_NAME,"h3")))

			# Why "try"? Because we are trying to collect every bid data, we want to be notified 
			# in case of any issues without breaking the code mon ami...
			try:
				# 
				element = driver.find_element(By.CLASS_NAME,"bidLineItems")
				element.click()
				element2 = driver.find_element(By.CLASS_NAME,"bid-line-items")
				bids_line_items.append(element2.text.split("\n"))

			except Exception as exc:
				print("\n!!!")
				print("\nThere is something wrong with getting the line items")
				print("\n!!!\n")
				print(exc)

			# Double because we are clicking twice; basically, every click(), one back()
			driver.back()
			driver.back()
			WebDriverWait(driver,10).until(EC.presence_of_element_located((By.CLASS_NAME,'table-overflow-container')))
		except Exception as exe:
			print("\nThere is soemthing wrong with getting_bid_line_items")

	return bids_line_items
```



