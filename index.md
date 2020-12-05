# How to do Sentiment Analysis with Amazon's Reviews

You can use the [editor on GitHub](https://github.com/yikeliu-echo/yikeliu-echo.github.io/edit/main/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

## Amazon Review Scrapping and Cleaning

Since my English name is Echo. I would like to scrap the review data that customers left for Echo Show - HD smart display with Alexa.
The number of comments is , which would be enough for trainning model.

## Data Scrapping
```markdown
# %%%% Preliminaries and library loading
import datetime
import os
import pandas as pd
import time
import random

# libraries to crawl websites
from bs4 import BeautifulSoup
from selenium import webdriver

pd.set_option('display.max_rows', 10)
pd.set_option('display.max_columns', 5)
pd.set_option('display.width',800)
path = 'C:/Users/Admin/Downloads/chromedriver_win32'
os.chdir(path)

driver = webdriver.Chrome()


# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/yikeliu-echo/yikeliu-echo.github.io/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
