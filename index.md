# How to do Sentiment Analysis for Product Review on Amazon
<img src="https://growtraffic-bc85.kxcdn.com/blog/wp-content/uploads/2019/01/Amazon-5-Star-Review-Illustration.jpg" width="%200" height="%100" />
# 1. Introduction
## 1.1 What is Sentiment Analysis?
Sentiment analysis, also known as severity analysis and opinion mining, is the process of analyzing, processing, summarizing and reasoning subjective texts with emotional colors. The purpose of sentiment analysis is to separate the text with positive, negative and neutral judgment. 
## 1.2 Why do sentiment analysis on Amazon's reviews?
With the popularity of online shopping, competition among major e-commerce companies is fierce. In order to improve the quality of customer service, in addition to price wars, it is more and more important to understand customers' needs and listen to their voices.
## 1.3 Which model to choose?
I would like to try Naive Bayesian Model(NBM) to see whether it has a high accurancy rate.


# 2. Data Preparation
## 2.1 Data Scrapping
Since my English name is Echo, I would like to scrap the review data that customers left for Echo Show - HD smart display with Alexa.
The number of comments is 10,048, which would be enough for trainning model. 
[Review Link](https://www.amazon.com/Echo-Show-8/product-reviews/B07PF1Y28C/ref=cm_cr_dp_d_show_all_btm?ie=UTF8&reviewerType=all_reviews)

[![rFo0UI.jpg](https://s3.ax1x.com/2020/12/10/rFo0UI.jpg)](https://imgchr.com/i/rFo0UI)

### 2.1.1 Library and driver loading 
```python
#Preliminaries and library loading
import os
import numpy as np
import pandas as pd
import time
import random
import matplotlib.pyplot as plt

from bs4                             import BeautifulSoup
from selenium                        import webdriver
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.naive_bayes             import MultinomialNB
from sklearn.metrics                 import confusion_matrix
from wordcloud                       import WordCloud

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

## 2.2 Data Cleaning 
The most useful data would be the raw comment text and the star the customers are given. Also, I've scrapped two added information in case useful:
1. The review conclusion which Amazon given 2. how many people thought this review is useful. 

```python
dta = pd.DataFrame.from_dict(reviews_one_store)

#to check if there is some useful information lost
BeautifulSoup(dta.review_raw.iloc[1]).text
#add rating star information into the dataframe
dta['star'] = dta.review_raw.str.extract('([0-9])(.[0]) (out of 5 stars)').reset_index()[[0]].astype(int)

#take this dataframe into csv
dta.to_csv("AmazonReviewData.csv")   
```

# 3. Model Development
* Model: Naive Bayesian Model, NBM
* Dependent Variable: review star
* Independent Variable: review text

## 3.1 Decide on variables
The dependent variable is review star. We classify emotions according to the score, more than 2 are divided into positive emotions, less than or equal to 2 are divided into negative emotions. Positive emotions are represented by 1, and negative emotions are represented by 0.
```python
dta['star'] = dta.review_raw.str.extract('([0-9])(.[0]) (out of 5 stars)').reset_index()[[0]].astype(int)
dta.loc[dta['star'] >  2,'star_group'] = 1
dta.loc[dta['star'] <= 2,'star_group'] = 0
dta["star_group"].astype(int)
```
The independent variable is review text. We'll separate the sentences into words.

[![rFoy28.png](https://s3.ax1x.com/2020/12/10/rFoy28.png)](https://imgchr.com/i/rFoy28)
```python
corpus          = dta.review_text.to_list()
ngram_range     = (1,1)
max_df          = 0.90
min_df          = 0.01
vectorizer      = CountVectorizer(lowercase   = True,
                                  ngram_range = ngram_range,
                                  max_df      = max_df     ,
                                  min_df      = min_df     );
                                  
X               = vectorizer.fit_transform(corpus)
```
Have a quick look at the word cloud

<img src="https://s3.ax1x.com/2020/12/09/rPYOBt.png" width="%10" height="%10" />

```python
flist = vectorizer.get_feature_names()
f = " ".join(flist)
wordcloud = WordCloud(background_color="white",width=1000, height=860, margin=2).generate(f)
plt.imshow(wordcloud)
plt.axis("off")
plt.show()

wordcloud.to_file('test.png')
```

## 3.2 Data Splliting - Training and Test
The original data is randomly divided into 100 groups. The first 80 groups are used as training data while left 20 groups are used as test data.
```python
dta['ML_group']   = np.random.randint(100,size = dta.shape[0])
dta               = dta.sort_values(by='ML_group')
inx_train         = dta.ML_group <  80                     
inx_test          = dta.ML_group >= 80

Y_train   = dta.star_group[inx_train].to_list()
Y_test    = dta.star_group[inx_test].to_list()

X_train   = X[np.where(inx_train)[0],:]
X_test    = X[np.where(inx_test) [0],:]
```
## 3.3 Naive Bayes Model
```python
clf                           = MultinomialNB().fit(X_train.toarray(), Y_train)
dta['N_star_hat']             = np.concatenate(
        [
                clf.predict(X_train.toarray()),
                clf.predict(X_test.toarray( ))
        ]).round().astype(int)

C2= confusion_matrix(Y_test, dta.N_star_hat.loc[inx_test].to_list())
print(C2)

right = C2.diagonal().sum()
total = C2.sum()
R2 = right/total
print("The Accuracy Rate of Naive Bayes Method is", R2)  
```
The correction rate is 89.02%, which would be a good model.
