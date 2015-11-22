
# Web scraping 101
### Intro
The problem: we want to get at some large set of data online -- bibliographic records, movie reviews, wikipedia entries, blog posts, what-have-you -- but it's all mixed up in html and distributed through thousands or even hundreds of thousands of web pages. Retrieving the data by hand is not feasible, so what can you do? The answer: scrape it!

At a high level, doing this programmatically is actually a pretty simple two-step process: step one is to request a web page through code; then step two is to use some tool for searching through html to extract and save the info we're looking for. After repeating this for all the web pages we're interested in (using either some pattern we've uncovered in how the site is structured, or some other list of links from an index), we'll have all the data we're looking for in some kind of structured form, all in a single place.

This tutorial shows one way to do that.

## Setup
We'll be using python and two 3rd-party python modules: one for requesting the web pages ([_requests_](https://github.com/kennethreitz/requests)), and another to search through them ([_BeautifulSoup4_](http://www.crummy.com/software/BeautifulSoup/bs4/doc/)). To install these, just run 
```
pip install requests
``` 
and 
```
pip install beautifulsoup4
``` 
from the command line. If you're on a mac and get some kind of permissions error, try throwing a in a ```sudo``` at the beginning. 

If you're still getting errors, your python probably isn't configured properly. If you're on windows, you can try the following from the command-line (replace "27" with whatever python version you have installed - likely either 27 or 34 or 35):
```
C:\python27\python.exe -m pip install requests beautifulsoup4
```
If that doesn't work, then I'd probably just recommend googling things related to "pip not working windows" or the same for mac and going from there.

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
  
 Delays are really easy to code, using python's built-in ```time``` library - just import the library and add ```time.sleep(2)``` or however many seconds you'd like to delay for somewhere in your scraping loop. You can see this in action in the full example scraper code down at the bottom of this document.

2. __Always make sure your scraper is identified as a scraper__
  * this is done using what is known as the "user-agent" header -- more on this further below


## Using requests
Requests makes retrieving web pages super easy. First, import the libary:

```python
import requests
```

then just make the page request:

```python
data = requests.get("https://github.com")
```

Since we're consciensious scrapers, we should also identify where these requests are coming from. To do this, you'll need to make a new dictionary, with "user-agent" as a key and the name of your scraper as the value, then pass that dictionary into requests.get()'s headers option:

```python
headers = {"user-agent": "Name of your scraper"}  # you can call this whatever you want
data = requests.get("https://github.com", headers=headers)
```

Now whenever the administrators of the target site look at their access logs, our entries will include the name of our scraper. Some folks also include a contact email address in the user-agent text so that administrators can contact them if there is a problem.

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
from bs4 import BeautifulSoup

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
soup("span", class_="octicon octicon-x", string=re.compile("hi"))
```

This example will find all the tags of type ```span```, with a class attribute whose value is exactly ```octicon octicon-x```, whose text contains the string ```hi```. For example, it would match to this tag:

```html
<span class_="octicon octicon-x">octocat says hi</span>
```

but not this tag:
```html
<span class_="octicon octicon-x">text</span>
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
So, now that we know the basics of how to use the tools, how do we go about applying them? After doing this for a while, I've settled on a workflow for scraping that works pretty well for me. It boils down to this:

1. Look at the source of a target page
2. Play around with a prototype to narrow down bs4 search terms
3. Write out the code for a single page
4. Write out the code to loop through all pages
5. Write the results to a file

And that's it! Here are those steps in more detail:

### Step 1: Look at the source

Let's say we're doing a research project on early Italian poetry, and we need the raw text for every poem in a particular manuscript. Once we've identified a potential data source (in this instance, wikisource), step one is to take a look at the source code of [one of the target pages](https://it.wikisource.org/wiki/Canzoniere_\(Rerum_vulgarium_fragmenta\)/Lasciato_%C3%A0i,_Morte,_senza_sole_il_mondo) to see if you can eyeball some ways you might use beautifulsoup to get at the data you want.

Looking directly through the source code can be a bit of a slog, but fortunately there's a better way: if you right-click on the specific element you want to extract things from in either Firefox or Chrome, you can bring up an interactive view of that exact html by selecting "inspect element":

<img src="http://i.imgur.com/ODHXIgR.gif" width=800/>

If you're using Safari you'll need to manually enable this by going into settings -> advanced and checking the right check mark. 

Once the inspect pane comes up you can do a little exploration to see if there are any obvious ways to point BeautifulSoup at the right data. The most ideal case is when the tag holding the data you want has an "id" attribute, since this is unique, and can be used to narrow down the search immediately.

In this case we can see that the poem text has a parent ```<div>``` tag that has a "poem" class attribute. This seems like a promising entry point.

### Step 2: play around with a prototype
Once you have a general idea of how you might try to get at the data, it's often helpful to try some quick and dirty prototyping in IDLE to narrow down the exact details. 

So just fire up your interpreter, import requests and beautifulsoup, then just try out some sample searches and see what comes up. Here's what that might look like with the "poem" class idea:

<img src="http://i.imgur.com/kdUhtji.gif" width=800/>

### Step 3: write out code for a single page
Now that we know exactly what BeautifulSoup searches to run to get at the data we want, we can formally write out that code in a new python file. Based on our prototyping in IDLE, here is what that might look like (we'll call the file poem_scraper.py):
```python
import requests
from bs4 import BeautifulSoup

# setting identifying headers to be used whenever we make a request:
headers = {"user-agent": "Poem scraper v.0.1 - please contact email@address.com if there are any issues"}

# we're putting the scraping code for the single page into a function, so that we can use it multiple times
def get_poem_text_from_page(web_address):

    # making the request, with proper identifiers
    data = requests.get(web_address, headers=headers)
  
    # make the BeautifulSoup object so that you can do the search
    soup = BeautifulSoup(data.text)
  
    # remember that the soup search always returns a list, even when you know there will only be
    # one result, so to get at the actual object just access the first element in the list:
    poem_text = soup(class_="poem")[0].text
    
    # since there are likely special characters in the text, we'll need to make sure they're handled properly:
    poem_text = poem_text.encode("utf-8")
    
    return poem_text
  
# print out the text for a specific page, just to show it worked
print(get_poem_text_from_page("https://it.wikisource.org/wiki/Canzoniere_(Rerum_vulgarium_fragmenta)/Lasciato_%C3%A0i,_Morte,_senza_sole_il_mondo"))
```

### Step 4: loop through every page, __with a delay__, and add each result to a results list
We'll need to somehow get at the list of links to every poem in the publication, which we can do by scraping all the right ```<a href="...">``` tags in the publication page. Then we can apply the poem extraction code to each of those, with a delay in between each request. 

There are usually two main ways to go about doing this. The first and often easiest method is to take a look at the web address of one of the pages we'll be scraping from to see if there is any kind of pattern we could take away from it to programmatically generate the addresses of each page we're looking for. For example, if you have a web address like [this](http://www.loc.gov/jukebox/recordings/detail/id/1425) (from the National Jukebox):

```
http://www.loc.gov/jukebox/recordings/detail/id/1425
```

you'll notice that it ends with a number. This should be your first hint that you might be able to write some code to automatically generate all the addresses for material in that collection. To check that, try going in and changing that number, and see what happens.

In that example, you could write a script that just appends increasing sequential numbers to the base address, and you'd be able to get at every record in the collection.

In our current example, the poetry texts, it doesn't look like there is any simple pattern like that. The alternative option is to try to find an index somewhere online that has the links to all the material we're looking for, and scrape the addresses from that index page. In this case, we can look at the page for the publication as a whole, [here](https://it.wikisource.org/wiki/Canzoniere_\(Rerum_vulgarium_fragmenta\))


So, we'll go into that page and use the same methods we used in step one and two to narrow down a way to extract just the links we're looking for.

After first trying some exploration and prototyping to find ways to get at those links, this is what that might look like:
```python
import re  # for making partial matches with BeautifulSoup
import time  # for making delays between requests

# get the publication page source, and make the soup
publication_page_data = requests.get("https://it.wikisource.org/wiki/Canzoniere_(Rerum_vulgarium_fragmenta)", headers=headers)

soup = BeautifulSoup(publication_page_data.text)

# grab all the tags linking to individual poems by finding all of the tags with an href-value that contains the match text
a_tags = soup(href=re.compile("wiki\/Canzoniere"))

# The first link we matched isn't actually to a poem, so we'll just remove it
a_tags.pop(0)

# make a results list to store each poem's text in
results = []

# run through every matching link found in the parent page
for a_tag in a_tags:

    # the links were relative, so we need to reconstruct the absolute value
    link = "https://it.wikisource.org" + a_tag["href"] 
  
    poem_text = get_poem_text_from_page(link)
  
    # Add the results, along with the link used to get there, to the results list
    results.append([link, poem_text])
    
    # wait one second before continuing
    time.sleep(1)

```

### Step 5: write out the results!
You can do this however you like, but I recommend using the csv format - it's really versatile, and can be imported into excel or google sheets if you want your data there. Here is some skeleton code for how you might do that:

```python
# python has a library for reading and writing csv files - let's import it first
import csv

# note: if you're using python 3, replace the next line with the following:
# with open("output.csv.", mode="w", newline="") as f:
with open("output.csv", mode="wb") as f:
    writer = csv.writer(f)
    headers = ["source link", "text"]
    
    writer.writerow(headers)
    writer.writerows(results)

```

__note__: In this case, the file will be written to whatever directory you're running the script from. If you want to write to somewhere else, you'll need to use an absolute filepath: instead of just ```"output.csv"``` you would need to do something like ```r"C:\Users\username\Desktop\output.csv"``` on windows or ```"/Users/username/Desktop/output.csv"``` on a mac.

And that's it! Here is what that looks like all put together, with minimal comments:

```python
# import all the libraries
import re
import csv
import time

import requests
from bs4 import BeautifulSoup

# headers, to identify the scraper to the target site
headers = {"user-agent": "Poem scraper v.0.1 - please contact email@address.com if there are any issues"}

# code to extract data from a single page
def get_poem_text_from_page(web_address):
    data = requests.get(web_address, headers=headers)
    soup = BeautifulSoup(data.text)
    poem_text = soup(class_="poem")[0].text
  
    return poem_text.encode("utf-8")
    
# get links to all the pages we want to extract data from
publication_page_data = requests.get("https://it.wikisource.org/wiki/Canzoniere_(Rerum_vulgarium_fragmenta)", headers=headers)

soup = BeautifulSoup(publication_page_data.text)
a_tags = soup(href=re.compile("wiki/Canzoniere"))
a_tags.pop(0) # removing an incorrect tag

# extract data from all pages we got the links to
results = []
for a_tag in a_tags:
    link = "https://it.wikisource.org" + a_tag["href"]
    poem_text = get_poem_text_from_page(link)
    
    results.append([link, poem_text])
    
    time.sleep(1)
    

# write the results to file

# note: if you're using python 3, replace the next line with the following:
# with open("output.csv.", mode="w", newline="") as f:
with open("output.csv", mode="wb") as f:
    writer = csv.writer(f)
    header = ["source link", "text"]
    
    writer.writerow(header)
    writer.writerows(results)
```

## try your own!
If any of the following seem interesting, give it a try!

* grab course numbers, titles, and descriptions from the public SI [course catalog](https://www.si.umich.edu/programs/courses/catalog)

-------------------------

* Extract all the comments from [this hackernews article](https://news.ycombinator.com/item?id=6097155) and save them in a .CSV file (just the comment texts only - no need to preserve any of the comment hierarchies)

------------------------------

* Find the audio file on [this page](http://www.library.ucsb.edu/OBJID/Cylinder9861) and save it to your computer, using only python.
  * _hint: you can use the request.get() method on files, too -- just use ```.content``` instead of ```.text``` to get at its actual data_

### API challenge
* See if you can work out how to use _requests_ and the [HathiTrust bibliographic API](https://www.hathitrust.org/bib_api) to retrieve json-formatted bibliographic records for one of the books on [this page](http://catalog.hathitrust.org/Record/009212663)
  * _hint: try looking at the links in that page to see if you can extract an item's HathiTrust ID number_

