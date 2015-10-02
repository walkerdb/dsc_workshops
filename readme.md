
## Web scraping
#### Draft in progress

0. take a look at the site's data use terms if you can find them
0. stop to check if you can use an api instead
1. 

#### Setup
We'll be using two 3rd-party modules for this: one for requesting the web pages (requests), and another to read them (beautiful soup 4). To install these, just run ```pip install requests``` and ```pip install bs4``` from the command line.

First we'll import the libraries we'll be using:

```python
from bs4 import BeautifulSoup
import requests
```

#### Using requests
Retrieving web data is super easy with the requests module:

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

```

But what we're really interested here is the ```.text``` option, which will give us the full html of the page -- this is what we need to create the BeautifulSoup object.

#### Basic BeautifulSoup usage
We'll use the raw html (retrieved through requests' ```.text``` method) to make a new BeautifulSoup object for searching and retrieving data:

```python
soup = BeautifulSoup(data.text)
```

Search syntax is super simple. If you want a list of all the "a" tags on a given page, all you need is 

```python
soup("a")
```

To search by tag attribute instead, just search with the attribute as a keyword variable. eg:

```python
soup(id="start-of-content")
```

Since ```class``` is already a python keyword, to find things in the "class" attribute you'll need to add a trailing underscore to the keyword:

```python
soup(class_="container")
```

To search tag text instead of attributes or tag types, use the "string" keyword:

```python
soup(string="yolo")
```

The above will only ever match if the entire tag text is exactly the given value. To do partial searches, you'll need to write out the search term as a regular expression:

```python
import re
soup(string=re.compile("hi!"))
```

To use multiple facets, just chain them:
```pythttps://github.com/walkerdb/dsc_workshops/edit/master/readme.md#hon
soup("span", class_="octicon octicon-x", string=re.compile("yo"))
```

To retrieve the plaintext from a tag, all you need to do is access the ".text" value of the tag object:
```python
>>> first_p_tag = soup("p")[0]
>>> first_p_tag.text
"hello"
```

**getting text from tags**

### in progress
