![pic](https://www.google.com/imgres?imgurl=https://wordstream-files-prod.s3.amazonaws.com/s3fs-public/styles/simple_image/public/images/how-to-get-amazon-reviews1.png?8aTNb_tMNas_DNOFaOnVfIuJxQo47zXv%26itok%3DbCeELZ5Z&imgrefurl=https://www.wordstream.com/blog/ws/2014/04/10/amazon-reviews&tbnid=vxNpQFN6NDSwHM&vet=1&docid=A2hUtvqKO6uX_M&w=512&h=275&source=sh/x/im.png)  
# 1. Introduction
## 1.1 What is Sentiment Analysis?
Sentiment analysis, also known as severity analysis and opinion mining, is the process of analyzing, processing, summarizing and reasoning subjective texts with emotional colors. The purpose of sentiment analysis is to judge the text with positive, negative and neutral judgment. 
## 1.2 Why do sentiment analysis on Amazon's reviews?
With the popularity of online shopping, competition among major e-commerce companies is fierce. In order to improve the quality of customer service, in addition to price wars, it is more and more important to understand customers' needs and listen to their voices.

# 2. Data Preparation
## 2.1 Data Scrapping
Since my English name is Echo, I would like to scrap the review data that customers left for Echo Show - HD smart display with Alexa.
The number of comments is 10,048, which would be enough for trainning model. 
https://www.amazon.com/Echo-Show-8/product-reviews/B07PF1Y28C/ref=cm_cr_dp_d_show_all_btm?ie=UTF8&reviewerType=all_reviews

### 2.1.1 Library and driver loading 
```python
#Preliminaries and library loading
import datetime
import os
import pandas as pd
import time
import random
from bs4 import BeautifulSoup
from selenium import webdriver

pd.set_option('display.max_rows', 10)
pd.set_option('display.max_columns', 5)
pd.set_option('display.width',800)
path = 'C:/Users/Admin/Downloads/chromedriver_win32' 
os.chdir(path)

driver = webdriver.Chrome()
```
### 2.1.2 Find the link to scrap
```python
links_to_scrape = "https://www.amazon.com/Echo-Show-8/product-reviews/B07PF1Y28C/ref=cm_cr_dp_d_show_all_btm?ie=UTF8&reviewerType=all_reviews"
driver.get(links_to_scrape)
```
### 2.1.3 Find all the reviews in the website
```python
condition = True
reviews_one_store = []

while (condition):
    reviews           = driver.find_elements_by_xpath("//div[@class='a-section review aok-relative']")
    r                 = 0
    for r in range(len(reviews)):
        one_review                   = {}
        one_review['url']            = driver.current_url
        soup                         = BeautifulSoup(reviews[r].get_attribute('innerHTML'),  "html.parser")

        try:
            one_review_raw = reviews[r].get_attribute('innerHTML')
        except:
            one_review_raw = ""
        one_review['review_raw'] = one_review_raw
    
        #get the comment content
        try:
            one_review_text = soup.find('span', attrs={'class':'a-size-base review-text review-text-content'}).text
        except:
            one_review_text = ""
        one_review['review_text'] = one_review_text
        
        #get the number of people who think this comment is useful
        try:
            one_review_helpful_statement = soup.find('span', attrs={'class':'a-size-base a-color-tertiary cr-vote-text'}).text
        except:
            one_review_helpful_statement = ""
        one_review['review_helpful_statement'] = one_review_helpful_statement
        
        #get the keyword of this comment(Amazon's highlight)
        try:
            one_review_keyword = soup.find('a', attrs={'class':'a-size-base a-link-normal review-title a-color-base review-title-content a-text-bold'}).text
        except:
            one_review_keyword = ""
        one_review['review_keyword'] = one_review_keyword
        
        
        reviews_one_store.append(one_review)
        
    driver.find_element_by_xpath("//li[@class='a-last']").click()
    time.sleep(random.randint(3,5)) 
```

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/yikeliu-echo/yikeliu-echo.github.io/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
