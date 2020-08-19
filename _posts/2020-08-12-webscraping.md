---
layout: blog
title: "Introduction to Web Scraping for Data Science Using Python"
date: 2020-08-12
background: '/img/blog/post 2.jpg'
---

One of the hardest parts of data science is getting the data you need. If we are lucky, the data may be available in a csv or similar format on a website (like data from BEA or BLS). But more often, the data we want may be online, but with no obvious way of getting it. Thankfully, we can use Python to automate the retrieval and creation of data from online sources.

In this post, I will explain the best ways to approach web scraping using tools in Python. In the [next post](/2020/08/19/webscraping-tutorial.html), we will write a web scraper to extract data from this website.

## What is web scraping?

Web scraping is the use of programmatic tools like Python to extract data or information from websites. Usually, web scrapers are written and employed when a large amount of data is needed, since it would take far too long to manually click through a website and extract or download all of the desired data.

For example, in my day job at the Mercatus Center, I often write web scrapers to download the text of and extract other information from various regulatory codes, such as the [California Code of Regulations](https://govt.westlaw.com/calregs/Browse/Home/California/CaliforniaCodeofRegulations). In this case, there are far too many regulations to click through and download each one, let alone save the metadata associated with them. Thankfully, web scraping allows us to both download the text and organize it in a way that is useful for analysis down the road.

## Python tools

We can use a variety of different Python tools, or libraries, to make web scraping easy. For a basic scraper, we can use just three libraries: **requests**, **BeautifulSoup** (from the bs4 library), and **pandas**.

Before we begin writing any code, make sure that Python is installed and on your path (type `python which` on your command line to confirm), and that `pip` is installed. Then pip-install the three libraries we will use (along with the **lxml** library, used by BeautifulSoup):

```
pip install bs4 lxml requests pandas
```

#### requests

We use the [requests](https://requests.readthedocs.io/en/master/) library to download the actual content from the website. The most basic requests for our purposes here is the "get" request. The python code for a get request is the following:

```
import requests
url = 'http://jonathannelson.io'
content = requests.get(url).content
```

The "content" variable above will contain the raw HTML from the home page of this website. However, there isn't much we can do with raw HTML on its own, so now we turn to BeautifulSoup.

#### BeautifulSoup

We use BeautifulSoup to parse the raw HTML we got from the get request. BeautifulSoup comes from the [bs4](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) library and can be either used as a submodule or imported explicitly (both shown in the code snippet below). The first step is to convert the content object from requests into a soup object:

```
import bs4
soup = bs4.BeautifulSoup(content, 'lxml')
```
or
```
from bs4 import BeautifulSoup
soup = BeautifulSoup(content, 'lxml')
```

N.B.: The "lxml" argument above refers to the specific parser that BeautifulSoup will use to parse the HTML. "lxml" is the quickest parser, but will not work in every case. If BeautifulSoup returns incoherent results, try pip-installing **html5lib** and replacing "lxml" with "html5lib" in the BeautifulSoup call.

The "soup" variable above can be parsed in a variety of different ways. We can use it to find different HTML elements, such as `<table>`, `<a>`, or `<p>`, which can all be useful for parsing and extracting data and information from websites.

Using the soup above (which is now a parsable version of the homepage of this website), we could extract my social media URLs from the footer, using the following code:

```
urls = []
for link in soup.find('footer').find_all('a'):
    url = link['href']
    urls.append(url)
```

This code will give us a list of the URLs in the footer. Not very useful on its own, but a good starting place. We will go into more detail on BeautifulSoup in the next post, but let's move on to pandas.

#### pandas

The [pandas](https://pandas.pydata.org/docs/getting_started/index.html) library is the most commonly used Python library for creating, cleaning, and organizing data from a variety of sources. It is very useful for creating tabular data from websites.

One of the easiest ways to do this is to organize data into a list of lists, and then pass this list to pandas. For example, using the list of URLs above:

```
import pandas as pd
df = pd.DataFrame(
    data=[urls],
    columns=['github', 'twitter', 'linkedin'])
```

This code will give us a very simple dataframe with three columns, each representing the three social media links in the footer of the website. In practice, you will likely build larger, more complex dataframes. Pandas then can be used to do more intensive data manipulation and analysis (look for more posts on pandas in the future).

#### Other Python tools

There are a variety of other Python libraries that are useful for web scraping. Below is a short list and brief description of libraries I use frequently:

* [**re**](https://docs.python.org/3/library/re.html) - Regular expressions (or regexes) are invaluable for parsing and extracting text based on patterns. They can be used to organize text for use in tabular data. In the [California](https://govt.westlaw.com/calregs/Browse/Home/California/CaliforniaCodeofRegulations) example above, we can use a regex to separately extract the title number and name from the text in the links on the homepage. I often use [regex101.com](https://regex101.com) to write and test my regexes.
* [**urllib**](https://docs.python.org/3/library/urllib.html) - Used to parse URLs the same way a web browser does. One super useful function is `urllib.parse.urljoin()`.
* [**datetime**](https://docs.python.org/3/library/datetime.html) - Used to keep track of dates and times. Since websites change frequently, when collecting data from the web, it is a good idea to include a "date collected" or similar column. Today's date can be found with the code `datetime.date.today()`.
* [**time**](https://docs.python.org/3/library/time.html) - The most useful function in this library for webscraping is `time.sleep()`, which allows the user to cause the code to pause between calls. This is important for websites that may be slow to load or frequently return errors.

## Summary

With only a few libraries, we can use Python write a web scraper to extract text and data from websites. A few other libraries can make our scrapers even more powerful. In the next post, we will write an actual web scraper of using all of the libraries above, and do some quick data analysis using pandas.