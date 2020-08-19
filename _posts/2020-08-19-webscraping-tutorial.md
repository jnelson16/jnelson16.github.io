---
layout: blog
title: "Tutorial: Building a Web Scraper in Python"
date: 2020-08-19
background: '/img/blog/post 2.jpg'
---

In the [previous post](/2020/08/12/webscraping.html), we discussed some of the basics of web scraping using Python. Here, we will actually build a full web scraper using the tools from the previous post.

*Author's note: I put together a Jupyter notebook to more interactively follow along with this tutorial. You can find that in my GitHub repository [here](https://github.com/jnelson16/tutorials/tree/master/notebooks). In addition, I wrote a full python script [here](https://github.com/jnelson16/tutorials/tree/master/scripts), with more functions and a command line interface you can play around with.*

### Determining what to scrape

For the sake of simplicity (and the opportunity to be meta), this tutorial will scrape the very website where this post lives. We will start at the homepage, and navigate through the links, scraping the text and data as we go, ending by putting the data in to a pandas dataframe.

*N.B.: As this website evolves, some parts of this post may be less accurate, though I will attempt to update it as the website changes.*

Let's begin by simply going to the homepage of jonathannelson.io in a browser to see what the page looks like. Below is a screenshot of what you should see:

![homepage](/img/blog/homepage.png)

We can play around with the website to get an idea of what is scrapable and what links we can use to traverse the website. At the top, we see three links that lead to the blog, the resume, and the contact form. In the middle we see a button with a link that will also take us to the resume. In the footer of the page, we see the three social media links we already scraped in the previous post. For purposes of scraping the website, the links at the top seem to be the most useful, so let's look more into those.

### Using inspect

In order to scrape the website, we will need to know a bit about the HTML of the site. If you are using Google Chrome (why you would use anything else? (don't @ me Firefox users)), you can utilize the inspect feature to look at the HTML of the site. Right click on the "blog" link at the top and left click inspect. You should now see something like the screenshot below:

![inspect](/img/blog/inspect.png)

We can see here that each of the navigation links are contained in an `<li>` element with a class attribute "nav-item" and the link itself is an `<a>` element with class attribute "nav-link." For purposes of scraping, either will do, but let's go with the `<li>` elements.

### Parsing with requests and BeautifulSoup

Now that we know what we want to scrape and what the HTML looks like, we can finally start coding. Borrowing code from the previous post, we can get the raw HTML of the homepage and convert it into a BeautifulSoup object:

```
import bs4
import requests
url = 'http://jonathannelson.io'
content = requests.get(url).content
soup = bs4.BeautifulSoup(content, 'lxml')
```

To make things easier, let's put this code into a function so we don't have to type this all out every time we want to get a soup object from a URL:

```
def get_soup(url):
    return bs4.BeautifulSoup(requests.get(url).content, 'lxml')
```

Then we can get the soup object simply by typing ```soup = get_soup(url)```.

### Navigating to other pages

Now that we have the soup object, we can parse the HTML using the element tags we discussed above. We can utilize the class attribute "nav-item" to iterate only through `<li>` elements we want. The following code will get us the three links to get to the rest of the website, and then get soup objects of each of the three pages:

```
import urllib
for page in soup.find_all('li', class_="nav-item"):
    page_url = urllib.parse.urljoin(url, page.find('a')['href'])
    page_soup = get_soup(page_url)
```

The URL for each page comes from "href" attribute of the `<a>` element within the `<li>` element. But the URLs here only include the name of the page, not the full address. We can use the **urllib** library to join the page URLs with the original URL from above to get the full web address, and pass that new URL into the `get_soup()` function. Because this function both uses a get request and parses the HTML into a soup object, it is almost like we are clicking on the link and loading that new page.

### Scraping text and data from a page

The first page we visit using the scraper is the blog page. In order to navigate through this page, we should inspect it just like we did for the homepage. You should see something like this:

![inspect the blog](/img/blog/inspect-blog.png)

We can see that each blog post is represented in the HTML as an `<article>` element with a class attribute "post-preview." We can use this element to get both the links for each blog post as well as some metadata associated with them (which will eventually be added to the empty list called `data`):

```
import pandas as pd
data = []
for post in page_soup.find_all(class_="post-preview"):
    post_title = post.find(class_="post-title").text
    post_date = pd.to_datetime(
        post.find(class_="post-meta").text.split(' · ')[0]).date()
    post_url = urllib.parse.urljoin(url, post.find('a')['href'])
    post_soup = get_soup(post_url)
```

Here we have collected not only the URL for each blog post, but the title and date of the posts as well. *Note that we use the pandas function `to_datetime().date()` to get the date in a clean format.* We can eventually put these metadata into a pandas dataframe, but first let's collect the actual text from the page of the posts.

The main content of each blog post is contained in the third `div` element with a class attribute "container" on the page, so we can get the text using:

```
post_text = post_soup.find_all('div', class_="container")[2].get_text('\n')
```

For full text like this, it is helpful to use `.get_text('\n')` instead of `.text` just in case the text elements run into each other. Now, let's write this text to file so we don't lose it (using tools from the **pathlib** library):

```
from pathlib import Path
outpath = Path('blogs').joinpath(
    post_date, post_title).with_suffix('.txt')
# Create directory structure if it does not exist
if not outpath.parent.exists():
    outpath.parent.mkdir(parents=True)
# Write text to file
outpath.write_text(post_text)
```

This will write the text into a folder called "blogs," further organized by date and the title of the post. The text from these posts could then be analyzed using natural language processing (see future posts for that).

### Putting data into pandas

We now have both the text from the blogs and metadata associated with them. Let's put that metadata into a dataframe. In the for-loop for each post we can append each post's metadata to the list created above:

```
data.append((post_date, post_title, post_url, outpath))
```

Then, outside of the for-loop, we can put this data into a pandas dataframe, and write this data into a csv:

```
df = pd.DataFrame(
    data=data,
    columns=['date', 'title', 'url', 'filepath'])
df.to_csv('blog_metadata.csv', index=False)
```

Since we have the filepath, we could use this csv later as metadata for a natural language processing project using the blog text.

### Summary

Putting it all together, we get code that should look something like this (not including the imports and function definition):

```
url = 'http://jonathannelson.io'
soup = get_soup(url)
data = []
for link in soup.find_all('li', class_="nav-item"):
    page_url = urllib.parse.urljoin(url, link.find('a')['href'])
    page_soup = get_soup(page_url)
    for post in page_soup.find_all(class_="post-preview"):
        post_title = post.find(class_="post-title").text
        post_date = pd.to_datetime(
            post.find(class_="post-meta").text.split(' · ')[0]).date()
        post_url = urllib.parse.urljoin(url, post.find('a')['href'])
        post_soup = get_soup(post_url)
        post_text = post_soup.find_all(
            'div', class_="container")[2].get_text('\n')
        outpath = Path('blogs').joinpath(
            str(post_date), post_title).with_suffix('.txt')
        if not outpath.parent.exists():
            outpath.parent.mkdir(parents=True)
        outpath.write_text(post_text)
        data.append((
            post_date, post_title, post_url, outpath
        ))
df = pd.DataFrame(data, columns=['date', 'title', 'url', 'filepath'])
df.to_csv('blog_metadata.csv')
```

Unfortunately, we only had time to scrape the first of the three pages on this website in this post, but try to scrape data or text from the other two on your own. Feel free to [contact me](/contact) or reach out on social media (links below) for questions or other feedback!