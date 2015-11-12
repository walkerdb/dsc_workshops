
# Web scraping
### Intro
So here is the problem: we want to get at some large set of data online -- bibliographic records, movie reviews, wikipedia entries, blog posts, what-have-you -- but it's all mixed up in html and distributed through thousands or even hundreds of thousands of web pages. Retrieving the data by hand is not feasible, so what can you do? The answer: scrape it!

At a high level, doing this programmatically is actually a pretty simple two-step process: step one is to request a web page through code; then step two is to use some tool for searching through html to extract and save the info we're looking for. After repeating this for all the web pages we're interested in (using either some pattern we've uncovered in how the site is structured, or some other list of links from an index), we'll have all the data we're looking for in some kind of structured form, all in a single place.

This tutorial shows one way to do that.

## Scraping etiquette
There are two things to always keep in mind when making a web scraper:

1. __Always put a delay between requests__, usually at least one second
  * If you don't do this, you're at risk of accidentally DDOSing the site. This is bad news.

  It's usually a good idea to take a look at the host's robots.txt file: this will occasionally include the site's preferred delay time. You can usually get to a robots.txt file by just adding "/robots.txt" to the base web address of the host. 

  For example, here is the Library of Congress's robots.txt, from https://www.loc.gov/robots.txt:
  ```
  User-agent: 008
  Disallow: /	
  
  User-agent: *
  Disallow: /cgi-bin/
  Disallow: /web_arch/
  Disallow: /rr/mopic/staff
  Disallow: /loc/volunteers
  Disallow: /ficmanagers
  Disallow: /preserv/extranet/
  Disallow: /myloc
  Disallow: /nationalfilmregistry
  Disallow: /fedsearch
  Disallow: /search
  Crawl-Delay: 2
  ```

  ```Crawl-Delay``` is what we're interested in here -- in this case the LOC prefers a 2-second delay between a scraper's html requests.

2. __Always make sure your scraper is identified as a scraper__
  * this is really easy to do using the user-agent header -- more on this further below

## Setup
We'll be using python and two 3rd-party python modules: one for requesting the web pages ([_requests_](https://github.com/kennethreitz/requests)), and another to read them ([_BeautifulSoup4_](http://www.crummy.com/software/BeautifulSoup/bs4/doc/)). To install these, just run 
```
pip install requests
``` 
and 
```
pip install beautifulsoup4
``` 
from the command line. If you're on a mac and get some kind of permissions error, try throwing a in a ```sudo``` at the beginning.

Now that we have the libraries in place, we'll get started with a web-scraping script. First, make a new python file and import the libraries we'll be using:

```python
from bs4 import BeautifulSoup
import requests
```

## Using requests
Requests makes retrieving web pages super easy:

```python
data = requests.get("https://github.com")
```

Since we're consciensious scrapers, we should also identify where these requests are coming from. This is pretty straightforward:

```python
headers = {"user-agent": "Name of your scraper"}  # you can call this whatever you want
data = requests.get("https://github.com", headers=headers)
```

Now whenever the administrators of the target site look at their access logs, our entries will include the name of the scraper. Some folks also include a contact email address in the user-agent text so that administrators can contact them if there is a problem.

```requests.get``` returns a web-page object that hold all sorts of interesting data. For example, 

```python
>>> data.encoding 
'utf-8'

>>> data.headers['content-type']
'text/html; charset=utf-8'

>>> data.status_code
200

>>> data.text
u'''
<!DOCTYPE html>
<html lang="en" class="">
  <head>
    <meta charset='utf-8'>
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta http-equiv="Content-Language" content="en">
    [...etc.]
'''

```

For our immediate purposes, what we're really interested here is the ```.text``` option, which gives us the raw html text of the page -- this is what we need to create the BeautifulSoup object.

## Using BeautifulSoup
BeautifulSoup gives you a clean and relatively straight-forward interface to find and extract specific parts of an html document. The basic gist is to use various search methods to narrow down to one html tag or set of tags, and from there you can grab the exact data you're looking for.

### a quick review of HTML structure
This is probably review for many of you, but to use BeautifulSoup you'll need to know some of the basics of how HTML data is structured. There are only just a few things you need to know: HTML is made of __tags__, which look like this:
```
<p>text</p>
```
Every tag has an opening and closing element, and between those elements there can either be text, other tags, or both. If a tag is inside of another tag, the first tag is often called a "parent", and the one inside of it a "child". Tags also can have __attributes__, which look like this:

```html
<p class="description">text</p>
```

In this case ```class``` is an attribute of the tag ```p```, with a value of ```description```.

### making the BeautifulSoup object
To get BeautifulSoup up and running we just need to feed it the raw html text of the page we requested, using the web-page object requests created:

```python
soup = BeautifulSoup(data.text)
```

## finding tags

##### searching by tag type
Search syntax is super simple. If you want a list of all the "a" tags on a given page, all you need to do is 

```python
soup("a")
```

##### searching by tag attribute
To search by tag attribute instead, just search with the attribute as a keyword variable. eg:

```python
soup(id="start-of-content")
```

Since ```class``` is already a python keyword, to find things in the "class" attribute you'll need to add a trailing underscore to the keyword:

```python
soup(class_="container")
```

To just search for tags that have a specific attribute, no matter the attribute's value, use
```python
soup(href=True)
```

##### searching by exact tag text
To search tag text instead of attributes or tag types, use the "string" keyword:

```python
soup(string="yolo")
```

##### partial searches
The above will only ever match if the entire tag text is exactly the given value. To do partial searches, you'll need to write out the search term as a regular expression:

```python
import re
soup(string=re.compile("hi!"))
```

_a note about regular expressions in python_: many non-alphabetic characters have special meanings in the regular expression language (for example, a period is a wild-card character and not a period) -- if you aren't familiar with regular expressions in python it may be best to precede all non-alphanumeric characters with a forward-slash ("\"), which will ensure that they mean what you intend them to mean.

##### multiple facets
To use multiple search facets, just chain them:
```python
soup("span", class_="octicon octicon-x", string=re.compile("yo"))
```

## extracting data

Once you've narrowed down a tag to get things from, there are a few things you can do to get at its contents:

##### retrieving text
Once you have a single tag selected, to retrieve its text all you need to do is access the ".text" value of the tag object:
```python
>>> first_p_tag = soup("p")[0]
>>> first_p_tag.text
"hello"
```

##### retrieving attribute values
Attributes are accessed just like values in a python dictionary, with the attribute name as a key. So, if we've selected a tag that looks like this
```html
<a href="http://www.google.com">Link here!</a>
```
and wanted to get to the web address contained in the href attribute, all you need to do is:

```python
>>> selected_a_tag["href"]
"http://www.google.com"
```

## A web-scraping workflow
### Step 1: Look at the source
So, now that we know the basics of how to use the tools, how do we go about applying them?

Let's say we're doing a research project on the evolution of [Taylor Swift lyrics](http://www.azlyrics.com/t/taylorswift.html) over the last decade, and we need the raw text for all of her songs. Once we've identified a potential data source, step one is to take a look at the source code to see if you can eyeball some ways you might use beautifulsoup to get at the data you want.

Looking directly through the source code can be a bit of a slog, but fortunately there's a better way: if you right-click on the specific element you want to extract things from in either Firefox or Chrome, you can bring up an interactive view of that exact html by selecting "inspect element":

<img src="http://i.imgur.com/gqkYLSZ.gif" width=600/>

Once the inspect pane comes up you can do a little exploration to see if there are any obvious ways to point BeautifulSoup at the right data. The most ideal case is when the tag holding the data you want has an "id" attribute, since this is unique, and can be used to narrow down the search immediately.

### Step 2: play around with a prototype
Once you have a general idea of how you might try to get at the data, it's often helpful to try some quick and dirty prototyping in IDLE. [demo!]

### Step 3: write out code for a single page


### Step 4: loop through every page, __with a delay__

### Step 5: write out the results!

[demo]

## try your own!
If any of the following seem interesting, give it a try!

* grab course numbers, titles, and descriptions from the public SI [course catalog](https://www.si.umich.edu/programs/courses/catalog)

-------------------------

* Extract all the comments from [this hackernews article](https://news.ycombinator.com/item?id=6097155) and save them in a .CSV file (just the comment texts only - don't need to preserve any of the comment hierarchies)
  * See this file for an example of how to write a csv file

------------------------------

* Find the audio file on [this page](http://www.library.ucsb.edu/OBJID/Cylinder9861) and save it to your computer, using only python.
  * _hint: you can use the request.get() method on files, too_
  * _see this python file for an example of what this might look like_

### API challenge
* See if you can work out how to use _requests_ and the HathiTrust bib API to retrieve json-formatted bibliographic records for all the books on this page
  * _hint: the oclc numbers might be useful_

### Dealing with dynamic data
* Some websites require some amount of user interaction to load new data. [talk about selenium problem here]

### song lyrics

### papyrus datasets?
