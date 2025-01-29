# **Building Spokane Tech: Part 6**

Welcome to part 6 of the "Building Spokane Tech" series! In this article, we'll look at how to scrape group and event data from meetup.com and eventbrite.com and populate our database accordingly. 


## **Source Data**
There are a couple platforms that tech groups are currently utilizing, with the majority being on meetup.com and eventbrite.com. We'll focus on collecting group and event data from those two platforms and address data sourcing from additional platforms as future needs dictate. 

Getting data from these platforms from their sites into ours involves a couple steps, including capturing the data, parsing the data, and storing it in our database for use on the website.


## **Data Collection**
Data collection for eventbrite and meetup will vary as eventbrite includes an API we can utilize for free, but meetup only has API access available for a fee. Do to this, we'll be web scraping. 

The first step is to actually capture HTML from a page. In some cases this can be as simple as using the requests library like so:

```html_content = requests.get("www.some_url.com")```

This works okay for some pages, but not of others, where a timeout occurs or the expected html content is not found. This could be due to rate limiting, bot detection, or other server side measures. To make web capture more reliable, we're utilizing Playwright, an automation library for controlling websites. Playwright creates a browser instance for a more real-life interaction with data on a website. It also provides some more advance tools like network waits, and detection of html elements. This provides a more reliable means of collecting data, but is slower and requires a number of heavier dependencies. 


## **Data Parsing**

Once we've captured html content from a web page we'll need to parse out specific pieces of data. To accomplish this we're using BeautifulSoup, a Python library for pulling data out of HTML files.

For events on meetup we perform two capture and parse steps, one to get the list of events from the group page where links to future events are listed, and another to open and capture data from the detail page of each individual event. 


## **Code/Utilities**

Under the web app we have a utilities subdirectory with scrapper files for eventbrite and meetup. Utilities for additional platforms may be added in the future. There's also a html_utils file that contains code to capture html content with requests and playwright.
The directory structure looks like this:

```plaintext
.
├── src
│   ├── django_project
│   │   ├── core
│   │   │   └── ...
│   │   ├── tests
│   │   │   └── ...
│   │   ├── web
│   │   │   ├── utilities
│   │   │   │     └── scrapers
│   │   │   │         ├── eventbrite.py
│   │   │   │         └── meetup.py
│   │   │   ├── html_utils.py
│   │   │   └── ...
```
