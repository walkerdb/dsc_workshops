
# Web scraping
### Intro
So here is the problem: we want to get at some large set of data online -- bibliographic records, movie reviews, wikipedia entries, blog posts, what-have-you -- but it's all mixed up in html and distributed through thousands or even hundreds of thousands of web pages. Retrieving the data by hand is not feasible, so what can you do? The answer: scrape it!

At a high level, doing this programmatically is actually a pretty simple two-step process: step one is to request a web page through code; then step two is to use some tool for searching through html to extract and save the info we're looking for. After repeating this for all the web pages we're interested in (using either some pattern we've uncovered in how the site is structured, or some other list of links from an index), we'll have all the data we're looking for in some kind of structured form, all in a single place.

Here's how to do that:

## Setup
We'll be using two 3rd-party modules for this: one for requesting the web pages [(_requests_)](https://github.com/kennethreitz/requests), and another to read them [(_BeautifulSoup4_)](http://www.crummy.com/software/BeautifulSoup/bs4/doc/). To install these, just run 
```
pip install requests
``` 
and 
```
pip install bs4
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
Remember the requests object we made earlier? To get BeautifulSoup up and running we just need to feed it the raw html text of the page we requested:

```python
soup = BeautifulSoup(data.text)
```

## finding tags

##### searching by tag type
Search syntax is super simple. If you want a list of all the "a" tags on a given page, all you need is 

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

##### partial searches; multiple facets
The above will only ever match if the entire tag text is exactly the given value. To do partial searches, you'll need to write out the search term as a regular expression:

```python
import re
soup(string=re.compile("hi!"))
```

To use multiple facets, just chain them:
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

## examples!

### song lyrics

### papyrus datasets?
