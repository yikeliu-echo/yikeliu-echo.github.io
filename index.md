# How to do Sentiment Analysis with Amazon's Reviews

You can use the [editor on GitHub](https://github.com/yikeliu-echo/yikeliu-echo.github.io/edit/main/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

## 1. Amazon Review Scrapping and Cleaning

Since my English name is Echo, I would like to scrap the review data that customers left for Echo Show - HD smart display with Alexa.
The number of comments is , which would be enough for trainning model. Also, data cleaning work is necessary to get the useful information.

### 1.1 Data Scrapping

```javascript

# Preliminaries and library loading
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

# Decide the link to scrap
links_to_scrape = "https://www.amazon.com/Echo-Show-8/product-reviews/B07PF1Y28C/ref=cm_cr_dp_d_show_all_btm?ie=UTF8&reviewerType=all_reviews"
driver.get(links_to_scrape)

# Find all the reviews in the website and bringing them to python
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
        
# click to get more reviews)
    driver.find_element_by_xpath("//li[@class='a-last']").click()
    time.sleep(random.randint(3,5)) 

'''

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/yikeliu-echo/yikeliu-echo.github.io/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
