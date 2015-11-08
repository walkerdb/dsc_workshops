
## Web scraping
#### Intro
So here is the problem: we want to get at some large set of data online -- bibliographic records, movie reviews, wikipedia entries, blog posts, what-have-you -- but it's all mixed up in html and distributed through thousands or even hundreds of thousands of web pages. Retrieving the data by hand is not feasible, so what can you do? The answer: Python!

At a high level, doing this programmatically is actually a pretty simple two-step process: step one is to request a web page with python; then step two is to use a python tool for searching through html to extract and save the info we're looking for. After repeating this for all the web pages we're interested in (using either some pattern we've uncovered in how the site is structured, or some other list of links from an index), we'll have all the data we're looking for in some kind of structured form in a single file.


####Before doing anything!
* take a look at the site's data use terms if you can find them -- this has implications for the ways in which you can present your results.
* stop to check if you can use an api instead - usually this will be faster and easier on the host's bandwidth


#### Setup
We'll be using two 3rd-party modules for this: one for requesting the web pages (_requests_), and another to read them (_BeautifulSoup4_). To install these, just run 
```bash
pip install requests
``` 
and 
```bash
pip install bs4
``` 
from the command line. If you're on a mac and get some kind of permissions error, try throwing a in a ```sudo``` at the beginning.

Now that we have the libraries in place, we'll get started with a web-scraping script. First, make a new python file and import the libraries we'll be using:

```python
from bs4 import BeautifulSoup
import requests
```

#### Using requests
Requests makes retrieving web data is super easy:

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

#### Searching through html with BeautifulSoup...
We'll use the raw html (retrieved through requests' ```.text``` method) to make a new BeautifulSoup object for searching and retrieving data:

```python
soup = BeautifulSoup(data.text)
```

##### a quick review of HTML structure
This is probably review for many of you, but to use BeautifulSoup to its fullest, you'll need to know some of the basics of how HTML data is structured. There are only just a few things you need to know: HTML is made of __tags__, which look like this:
```
<p></p>
```
Every tag has an opening and closing element, and everything in between is considered a "child" of that tag. Tags very often nest inside one another. Tags also can have __attributes__, which all have a value. Attributes look like this:

```html
<p class="description">
```

In this case ```class``` is an attribute of the tag ```p```, with a value of ```description```.

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

##### retrieving text
Once you have a single tag selected, to retrieve its text all you need to do is access the ".text" value of the tag object:
```python
>>> first_p_tag = soup("p")[0]
>>> first_p_tag.text
"hello"
```

**getting text from tags**

### in progress
