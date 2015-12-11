# Cleaning data with OpenRefine + Python

### Intro
OpenRefine is an incredibly useful open-source program meant to simplify the process of cleaning, normalizing, and transforming data. It should be one of the first tools in the toolbelt of anyone who works with metadata - be they academics, librarians, archivists or programmers. In this tutorial we'll work through some of the basics on how OpenRefine can be used with Python to clean a typical dataset, and hopefully give enough base knowledge that you can feel comfortable using the tool in your own work.

This is intended to be an active tutorial, so please follow along with your own copy of OpenRefine as we move along! Each section will begin with a description of some kind of functionality, possibly with a few examples, then close with a task that you should try on your own. Tasks will be marked like this:

> ```yaml
-> Do the haka!
```

###Table of contents

1. [Setup](https://github.com/walkerdb/workshop_drafts/blob/master/README.md#setup)
2. [Creating an OpenRefine project](https://github.com/walkerdb/workshop_drafts/blob/master/README.md#creating-a-project)
    1. [Dealing with character encodings](https://github.com/walkerdb/workshop_drafts/blob/master/README.md#dealing-with-character-encodings)
3. [Editing data](https://github.com/walkerdb/workshop_drafts/blob/master/README.md#editing-data)
    1. [Prebuilt tranformations](https://github.com/walkerdb/workshop_drafts/blob/master/README.md#prebuilt-transformations)
        1. [Filling down data](https://github.com/walkerdb/workshop_drafts/blob/master/README.md#filling-down-data)
        2. [Splitting up columns](https://github.com/walkerdb/workshop_drafts/blob/master/README.md#splitting-one-column-into-several-columns)
        3. [Cleaning up whitespace](https://github.com/walkerdb/workshop_drafts/blob/master/README.md#cleaning-up-whitespace)
        4. [Normalization through clustering](https://github.com/walkerdb/workshop_drafts/blob/master/README.md#normalization-through-clustering)
        5. [Undo/Redo](https://github.com/walkerdb/workshop_drafts/blob/master/README.md#the-undoredo-tab)
    2. [Custom transformations](https://github.com/walkerdb/workshop_drafts/blob/master/README.md#custom-transformations)
        1. [Text editing with Python](https://github.com/walkerdb/workshop_drafts/blob/master/README.md#text-editing-with-python)
            1. [Concatenating strings](https://github.com/walkerdb/workshop_drafts/blob/master/README.md#concatenating-strings)
            2. [Splitting strings](https://github.com/walkerdb/workshop_drafts/blob/master/README.md#splitting-strings)
            3. [Stripping strings](https://github.com/walkerdb/workshop_drafts/blob/master/README.md#stripping-strings)
            4. [Replacing words in a string](https://github.com/walkerdb/workshop_drafts/blob/master/README.md#replacing-words-in-a-string)
        2. [Applying this to OpenRefine](https://github.com/walkerdb/workshop_drafts/blob/master/README.md#applying-this-to-openrefine)
            1. [Setting the language](https://github.com/walkerdb/workshop_drafts/blob/master/README.md#setting-the-language)
            2. [Using custom code](https://github.com/walkerdb/workshop_drafts/blob/master/README.md#using-custom-code)
            3. [Try your own!](https://github.com/walkerdb/workshop_drafts/blob/master/README.md#try-your-own)
4. [Saving the results](https://github.com/walkerdb/workshop_drafts/blob/master/README.md#saving-the-results)

## Setup
OpenRefine is really easy to set up. All you need to do is download the latest version ([here](https://github.com/OpenRefine/OpenRefine/releases/download/2.6-rc.2/openrefine-mac-2.6-rc.2.dmg) for mac, [here](https://github.com/OpenRefine/OpenRefine/releases/download/2.6-rc.2/openrefine-win-2.6-rc.2.zip) for windows, and [here](https://github.com/OpenRefine/OpenRefine/releases/download/2.6-rc.2/openrefine-linux-2.6-rc.2.tar.gz) for linux), extract to wherever you'd like to keep it, then open up the "OpenRefine" program.

You'll also want to make sure you have a current version of Python installed.

> ```yaml
-> Download and open OpenRefine
```

Running OpenRefine should open a command line (which you don't need to worry about) as well as a browser window, which serves as the program's GUI. You can think of it just as an app living in your browser. If you ever accidentally close your browser window, you can get back to the app by entering ```localhost:3333``` in the browser's address bar.


## Using OpenRefine
For the rest of this workshop, we will be working with [this example metadata file](https://github.com/walkerdb/dsc_workshops/raw/master/open_refine/BSO_archive_dataset.csv), taken from a real project. (right-click and select "save as" to download). We'd like to use it in a research project, but at the moment it's too messy to reliably use. We technically _could_ clean it by hand, but with thousands of rows it would take days to do manually. Here we'll do it in just a few minutes.

> ```yaml
-> Download the dataset!
```


## Creating a project
To start a new project, click the "create project" button on the top-left of the page, choose an input file to use, then click "next".

> ```yaml
-> Create a new project and import the dataset
```

You should now be on a page that shows a preview of what OpenRefine thinks the data should look like, along with a set of options. Here you can set things like what kind of input type the file is and some of the specifics on how to read each of those types. For the most part the program should automatically pick the best settings, but you should always take a look at the preview to make sure things look like they should.

### dealing with character encodings

This step is __absolutely essential__. The history of text encoding is [actually pretty fascinating](http://www.joelonsoftware.com/articles/Unicode.html), but the end result today is that there are countless standards for how to encode what an "a" looks like in binary, and most of them are incompatible with each other. 

Have you ever seen characters that look like ```Ã¤``` when you know they should look like ```ä```? Whatever is displaying the text is trying to decode the raw data using the wrong standard. This happens all the time, and is generally a perennial pain when dealing with textual data. 

One of the first few preview cells of our OpenRefine page should have a few special characters, but at the moment they likely look totally out of place. This usually means that the default decoding standard the program is using is wrong. To change it, you can click in the "character encoding" box, select one of the options, and see if that improves anything. Most of the time __UTF-8__ should be your go-to option, but if that doesn't work you can play around with some of the other standards and see what happens.

Generally, you'll always want to ensure the proper encoding is selected here before moving on to the next steps, or risk mangling any text that uses a character beyond the English basics.

Once you're convinced things are as they should be, you can finalize things by clicking the "create project" button on the top-right of the page.

> ```yaml
-> Set the character encoding to "UTF-8"; finish data setup
```


##Editing data

You see those little upside-down triangles on top of each of the data columns? They are your portal to performing almost every possible action on the dataset. Clicking on one of them brings up a host of options, any of which if selected will be applied to that entire column of data. There are two primary kinds of operations you can access through these menus:

1. Predefined common-use transformations
2. Custom transformation commands, which you can write in one of the several scripting languages supported by OpenRefine. __Python__ is one of them.

We'll go through these two categories in more depth below.

###Prebuilt transformations
OpenRefine supplies a number of pre-built commands that cover a number of common use-cases. Here are a few of the more common options:

 

####Filling down data
There are a number of columns where, for whatever reason, only the first row in a related set of rows contains any value. We want full notated data for every row, so how can we make that happen? Easy: just select ```Edit cells -> Fill down```. This will go down the column and fill in each blank cell with the most recent value that came before it, which is exactly what we want. Just be careful that you don't fill down columns that shouldn't be filled.

> ```yaml
-> try filling down all the columns that need filling
-> [note: do not fill down the Soloist/Instrument column]
```

 

####Splitting one column into several columns
You'll notice the "Composer: Work" column is really holding two different categories of data: both composers and piece titles. We'd like there to be one column for each of those parts. OpenRefine can do this for you if you select ```Edit column -> Split into several columns...```. This will bring up a dialog box asking for a few options: what string to split the column on, whether you'd like to keep the old column afterwards, the maximum number of columns to split the data into, etc. Here's how this works:

If I have a column with cells in this form:
```
thing 1; thing 2; thing 3
```

and I wanted to split that into its individual parts, I could just use "; " as the string to split by and leave the rest of the options unchanged.

But if I had a column with cells like this:
```
thing 1: thing 2 title: subtitle
```
I wouldn't want to blindly split by the ": " character, since I'd also end up splitting parts of the second thing that should stay together. Here is where the option for max number of split columns comes in: putting "2" into that field would ensure that I only split on the first instance of ": " while ignoring the rest, safely giving me one column with ```thing 1``` and another with ```thing 2 title: subtitle```.

Once you've split a column, you can give a new name to each resulting column by selecting ```Edit column -> Rename this column```

> ```yaml
-> Try splitting the "Composer: Work" column into two columns, and give them new names
```

 

####Cleaning up whitespace
Excess whitespace is a very common and sometimes hard to diagnose problem: a program will categorize ```"Emily Dickinson"```, ```"Emily Dickinson "```, and ```"Emily  Dickinson"``` as three different strings, but these kinds of differences can be difficult for a human to spot. 

OpenRefine has a few functions to automate cleaning these issues: ```Edit cells -> Common transformations -> Trim leading and trailing whitespace``` and ```Edit cells -> Common transformations -> Collapse consecutive whitespace```.

> ```yaml
-> Try these out on the new "Composer" column.
```

 

####Normalization through clustering
This is, to me, by far the coolest functionality in OpenRefine. The basic gist is it runs one of several clustering algorithms to find strings that are slightly different but are still likely to be the same entity, then provides a simple interface to manually choose which version to normalize each instance into. You can find it via ```Edit cells -> Cluster and edit```.

Here's what the clustering interface should look like:

<img src="http://i.imgur.com/ZzoqaSB.png" width=800/>

In this case it has found clusters of related strings using the "fingerprint" clustering method. To normalize each entry in a given cluster to one value, all you need to do is click on the name you'd like to change them all to for every cluster in the list, then press the "Merge selected and re-cluster" button when you're done.

I've personally found OpenRefine's clustering to be one of its most useful functions: To give just one example, I was able to normalize all of the control-access terms used in the Bentley Historical Library's EAD finding-aids (~30,000 entries) in only a few hours, and have used it countless times in personal projects. It's especially useful when dealing with hand-entered data (which is likely to have typos), or when reconciling terms used across multiple datasets.

> ```yaml
-> Try running the cluster/edit function on the Composer column and the Work column to clean them up a bit. 
-> Does anything change when you try different Keying methods?
```

 

####The Undo/Redo tab
If you look over to the upper-left side of the page, you should see a tab called "Undo / Redo". If you select this you can see your full edit history from the beginning of the project, and you should be able to seamlessly roll back to any previous point in the project. This is incredibly useful if you ever decide you want to undo an edit, or see how much things have changed from a given point.

 

### Custom transformations

In addition to the prebuilt options, OpenRefine allows you to write your own code defining how to transform a given cell. It supports a few different scripting languages, but for now we'll be using python. Before diving into writing code directly in OpenRefine, let's review some of python's basic text manipulation capabilities (if you already feel like you're comfortable dealing with strings in python, feel free to jump ahead):

 

#### Text editing with Python
One of Python's greatest strengths lies in its ability to easily manipulate text, and it provides a number of built-in functions to cover common tasks. We'll be exploring some of these using __IDLE__, a prettified python interpreter that should have come bundled with your python installation. If you can't find a copy of IDLE on your computer by searching for it, you can instead open up a command line or terminal and type ```python```. This will give you the same basic functionality, just not in color.

> ```yaml
-> Open up IDLE, or, failing that, start up python via a command line. This is where you'll be trying out the code below.
```

 

#####Concatenating strings
In python a segment of text, called a _string_, is designated by enclosing some selection of text in quotes, like ```"this"```. Strings can have all sorts of operations called on them. You can join multiple strings together by adding them:

```python
>>> "string 1" + ": " + "string 2"
"string 1: string 2"
```

 

#####Splitting strings
You can split one string into a list of multiple strings with the ```.split()``` command. If you don't git the command any arguments it splits the string by its whitespace, otherwise it splits by whatever string you feed into it:
```python
>>> "I love turtles".split()
["I", "love", "turtles"]

>>> "You know nothing, Jon Snow".split(", ")
["You know nothing", "Jon Snow"]
```

> ```yaml
-> In IDLE, try splitting the following string by its punctuation:
"Shaken; not stirred"
```

When you split a string, you get back a python ```list```, which you can think of as a container for individual pieces of data. To get at specific pieces of data in a list, you access them by using their "index" in the list, which starts at 0 for the first element and goes up from there:

```python
>>> split_string = "You know why you never see elephants hiding up in trees? Because they're really good at it".split("? ")

>>> split_string[0]
"You know why you never see elephants hiding up in trees"

>>> split_string[1]
"Because they're really good at it"
```

 

#####Stripping strings
You can strip an arbitrary selection of characters from the ends of a string with the ```.strip()``` command. By default the command only removes whitespace characters from the left and right ends of a string, but if you pass it another string it will instead strip each of the characters in the passed string individually. For example:

```python
>>>" surrounded by whitespace ".strip()
"surrounded by whitespace"

>>>"0100101Robot invasion!0100010".strip("01")
"Robot invasion!"
```

There are also variants of strip that work only on the left or right side of a string: ```lstrip()``` and ```rstrip()```.

> ```yaml
-> Try stripping the excess punctuation from this string:
",-What do you call a fake noodle? An impasta..."
```

 

#####Replacing words in a string
The ```.replace()``` command can replace all instances of one word in a string with another. Here's how to use it:

```python
>>> "We are at war with Eurasia".replace("Eurasia", "Oceania")
"We are at war with Oceania"
```

> ```yaml
-> Try replacing "Blueberry" with the item of your choice:
"Blueberry pie is clearly the best of the pies."
```

 

####Applying this to OpenRefine
Back to OpenRefine -- if you select ```Edit cells -> tranform...``` from any of the columns you should see a new form that looks something like this:

#####Setting the language
Your language is likely currently set to OpenRefine's own custom scripting language, the "General Refine Expression Language (GREL)". If you click the language dropdown, you should see "Python / Jython" as an option - you'll want to select that to be able to use python methods in your code.

##### Using custom code
The "expression" text box is where you'll write your code. Whatever you ```return``` from your function will be set as the new value for that cell.

There are a few pre-defined variables that you can make use of, but the most important one is ```value```, which holds the original value of the cell you are currently editing. If you wanted to return the original value with periods stripped off, you would have something like

```python
new_value = value.strip(".")
return new_value
```

If you just wanted to change every cell to "yolo", the command would look like this:

```python
return "yolo"
```

If you ever want to use the data from another cell in the same row in your method, use the following variable exactly, replacing "column name" with the name of the column you're looking for: ```cells["column name"]["value"]```. Also, if you ever want to repeat code you used in some other transformation, you should be able to reload it by clicking the "history" link in the transform box and selecting one of the past commands.


#####Try your own!
You'll notice that the name of the first Conductor is misspelled. Can you write a transformation that replaces the misspelled name in every cell with the proper name? ("Hcnschel" -> "Henschel")

> ```yaml
-> Use a transformation to fix the misspelling in the conductor name
```

It looks like the conductors also all end with a period. Can you fix that too?

> ```yaml
-> Use a transformation to remove the trailing periods
```

Finally, since we are rational human beings who believe in standards and things making any logical sense, we'd like to change the date format for the Date column from MM/DD/YYYY to YYYY-MM-DD.

> ```yaml
-> Transform the date format from MM/DD/YYYY to YYYY-MM-DD
```

That one is a little more tricky, but totally doable. Don't be afraid to ask for help if you're running into a wall.

## Saving the results
Once you've finished cleaning up your dataset, just press "Export" in the upper-right and select the data format you'd like to write things out in. CSV (Comma-separated value) tends to be one of the most universal formats - it can easily be opened in excel or google sheets, or can even be processed straight into a python script using the ```csv``` library for any further processing. There are also [addons for things like RDF export](http://refine.deri.ie/) if that is your format of choice.

The file will be saved to your downloads folder. 

And that's it! We hope you've come out of this with a better idea of how to use OpenRefine, and possibly with a few thoughts about how you'd apply it to your own work Thanks for coming!

------------------------------------------------------

We'd love to hear your feedback -- let us know what you thought about the workshop here!
