Title: Web Scraping 101 with Python
Date: 2013-03-03
Slug: web-scraping-101-with-python
Tags: scraping, python, data, tutorial
Description: A beginner's guide to getting started with web scraping using Python and BeautifulSoup.
Affiliate_Link: <iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//ws-na.amazon-adsystem.com/widgets/q?ServiceVersion=20070822&OneJS=1&Operation=GetAdHtml&MarketPlace=US&source=ac&ref=tf_til&ad_type=product_link&tracking_id=gregrecom-20&marketplace=amazon&region=US&placement=1491910291&asins=1491910291&linkId=PJJ5LM64VYPKSWVT&show_border=true&link_opens_in_new_window=true"></iframe>

_After you're done reading, check out my follow-up to this post [here](/2013/04/29/more-web-scraping-with-python/)._

Yea, yea, I know I said I was going to [write more](http://www.gregreda.com/2013/01/23/translating-sql-to-pandas-part1/) on [pandas](http://pandas.pydata.org), but recently I've had a couple friends ask me if I could teach them how to scrape data.  While they said they were able to find a ton of resources online, all assumed some level of knowledge already.  Here's my attempt at assuming a very minimal knowledge of programming.

#### Getting Setup
We're going to be using Python 2.7, [BeautifulSoup](http://www.crummy.com/software/BeautifulSoup/), and [lxml](http://lxml.de/).  If you don't already have Python 2.7, you'll want to download the proper version for your OS [here](http://python.org/download/releases/2.7.3/).

To check if you have Python 2.7 on OSX, open up [Terminal](http://en.wikipedia.org/wiki/Terminal_(OS_X)) and type _python --version_.  You should see something like this:

![What Terminal should looks like](/images/python-version.png)

Next, you'll need to install [BeautifulSoup](http://www.crummy.com/software/BeautifulSoup/).  If you're on OSX, you'll already have [setuptools](http://pypi.python.org/pypi/setuptools) installed.  Let's use it to install [pip](http://www.pip-installer.org/en/latest/) and use that for package management instead.

In Terminal, run _sudo easy_install pip_.  You'll be prompted for your password - type it in and let it run.  Once that's done, again in Terminal, _sudo pip install BeautifulSoup4_.  Finally, you'll need to [install lxml](http://lxml.de/installation.html).

#### A few scraping rules
Now that we have the packages we need, we can start scraping.  But first, a couple of rules.

1. You should check a site's terms and conditions before you scrape them.  It's their data and they likely have some rules to govern it.
2. Be nice - A computer will send web requests much quicker than a user can.  Make sure you space out your requests a bit so that you don't hammer the site's server.
3. Scrapers break - Sites change their layout all the time.  If that happens, be prepared to rewrite your code.
4. Web pages are inconsistent - There's sometimes some manual clean up that has to happen even after you've gotten your data.

#### Finding your data
For this example, we're going to use the [Chicago Reader's Best of 2011](http://www.chicagoreader.com/chicago/best-of-chicago-2011/BestOf?oid=4100483) list.  Why?  Because I think it's a great example of terrible data presentation on the web.  Go ahead and browse it for a bit.

All you want to see is a list of the category, winner, and maybe the runners-up, right?  But you have to continuously click link upon link, slowly navigating your way through the list.

Hopefully in your clicking you noticed the important thing though - all the pages are structured the same.

#### Planning your code
In looking at the [Food and Drink](http://www.chicagoreader.com/chicago/best-of-chicago-2011-food-drink/BestOf?oid=4106228) section of the Best of 2011 list, we see that all the categories are a link.  Each of those links has the winner, maybe some information about the winner (like an address), and the runners-up.  It's probably a good idea to break these things into separate functions in our code.

To start, we need to take a look at the HTML that displays these categories.  If you're in Chrome or Firefox, highlight "Readers' Poll Winners", right-click, and select Inspect Element.

![Inspect Element](/images/inspect-element.png)

This opens up the browser's Developer Tools (in Firefox, you might now have to click the HTML button on the right side of the developer pane to fully show it).  Now we'll be able to see the page layout.  The browser has brought us directly to the piece of HTML that's used to display the "Readers' Poll Winners" _`<dt>`_ element.

![Inspect Element some more](/images/inspect-element-more.png)

This seems to be the area of code where there's going to be some consistency in how the category links are displayed.  See that _`<dl class="boccat">`_ just above our "Readers' Poll Winners" line?  If you mouse over that line in your browser's dev tools, you'll notice that it highlights the __entire section__ of category links we want.  And every category link is within a _`<dd>`_ element.  Perfect!  Let's get all of them.

![Inspect Element mouse over](/images/inspect-element-mouseover.png)

#### Our first function - getting the category links
Now that we know we know the _`<dl class="boccat">`_ section holds all the links we want, let's write some code to find that section, and then grab all of the links within the _`<dd>`_ elements of that section.

```python
from bs4 import BeautifulSoup
from urllib2 import urlopen

BASE_URL = "http://www.chicagoreader.com"

def get_category_links(section_url):
    html = urlopen(section_url).read()
    soup = BeautifulSoup(html, "lxml")
    boccat = soup.find("dl", "boccat")
    category_links = [BASE_URL + dd.a["href"] for dd in boccat.findAll("dd")]
    return category_links
```

Hopefully this code is relatively easy to follow, but if not, here's what we're doing:

1. Loading the urlopen function from the urllib2 library into our local [namespace](http://en.wikipedia.org/wiki/Namespace_(computer_science)).
2. Loading the BeautifulSoup class from the bs4 (BeautifulSoup4) library into our local namespace.
3. Setting a variable named _BASE_URL_ to "http://www.chicagoreader.com".  We do this because the links used through the site are relative - meaning they do not include the base domain.  In order to store our links properly, we need to concatenate the base domain with each relative link.
4. Define a function named _get_category_links_.
    1. The function requires a parameter of _section_url_.  In this example, we're going to use the [Food and Drink](http://www.chicagoreader.com/chicago/best-of-chicago-2011-food-drink/BestOf?oid=4106228) section of the BOC list, however we could use a different section URL - for instance, the [City Life](http://www.chicagoreader.com/chicago/best-of-chicago-2011-city-life/BestOf?oid=4106233) section's URL.  We're able to create just one generic function because each section page is structured the same.
    2. Open the section_url and read it in the _html_ object.
    3. Create an object called _soup_ based on the BeautifulSoup class.  The _soup_ object is an [instance](http://en.wikipedia.org/wiki/Instance_(computer_science)) of the BeautifulSoup class.  It is initialized with the html object and parsed with [lxml](http://lxml.de/).
    4. In our BeautifulSoup instance (which we called _soup_), find the _`<dl>`_ element with a class of "boccat" and store that section in a variable called _boccat_.
    5. This is a [list comprehension](http://docs.python.org/2/tutorial/datastructures.html#list-comprehensions).  For every _`<dd>`_ element found within our _boccat_ variable, we're getting the href of its _`<a>`_ element (our category links) and concatenating on our _BASE_URL_ to make it a complete link.  All of these links are being stored in a list called _category_links_.  You could also write this line with a [for loop](http://docs.python.org/2/tutorial/controlflow.html#for-statements), but I prefer a list comprehension here because of its simplicity.
    6. Finally, our function returns the _category_links_ list that we created on the previous line.

#### Our second function - getting the category, winner, and runners-up
Now that we have our list of category links, we'd better start going through them to get our winners and runners-up.  Let's figure out which elements contain the parts we care about.

If we look at the [Best Chef](http://www.chicagoreader.com/chicago/best-chef/BestOf?oid=4088191) category, we can see that our category is in _`<h1 class="headline">`_.  Shortly after that, we find our winner and runners-up stored in _`<h2 class="boc1">`_ and _`<h2 class="boc2">`_, respectively.

![Finding our winners and runners-up](/images/winners-and-runners-up.png)

Let's write some code to get all of them.

```python
def get_category_winner(category_url):
    html = urlopen(category_url).read()
    soup = BeautifulSoup(html, "lxml")
    category = soup.find("h1", "headline").string
    winner = [h2.string for h2 in soup.findAll("h2", "boc1")]
    runners_up = [h2.string for h2 in soup.findAll("h2", "boc2")]
    return {"category": category,
            "category_url": category_url,
            "winner": winner,
            "runners_up": runners_up}
```
It's very similar to our last function, but let's walk through it anyway.

1. Define a function called _get_category_winner_.  It requires a _category_url_.
2. Lines two and three are actually exactly the same as before - we'll come back to this in the next section.
3. Find the string within the _`<h1 class="headline">`_ element and store it in a variable named category.
4. Another list comprehension - store the string within every _`<h2 class="boc1">`_ element in a list called _winner_.  But shouldn't there be only one winner?  You'd think that, but some have multiple (e.g. [Best Bang for your Buck](http://www.chicagoreader.com/chicago/best-bang-for-your-buck/BestOf?oid=4088018)).
5. Same as the previous line, but this time we're getting the runners-up.
6. Finally, return a [dictionary](http://docs.python.org/2/tutorial/datastructures.html#dictionaries) with our data.

#### DRY - Don't Repeat Yourself
As mentioned in the previous section, lines two and three of our second function mirror lines in our first function.

Imagine a scenario where we want to change the parser we're passing into our BeautifulSoup instance (in this case, lxml).  With the way we've currently written our code, we'd have to make that change in two places.  Now imagine you've written many more functions to scrape this data - maybe one to get addresses and another to get [paragraphs of text about the winner](http://www.chicagoreader.com/chicago/best-new-food-truckfood/BestOf?oid=4101387) - you've likely repeated those same two lines of code in these functions and you now have to remember to make changes in four different places.  That's not ideal.

A good principle in writing code is [DRY - Don't Repeat Yourself](http://en.wikipedia.org/wiki/Don't_repeat_yourself).  When you notice that you've written the same lines of code a couple times throughout your script, it's probably a good idea to step back and think if there's a better way to structure that piece.

In our case, we're going to write another function to simply process a URL and return a BeautifulSoup instance.  We can then call this function in our other functions instead of duplicating our logic.

```python
def make_soup(url):
    html = urlopen(url).read()
    return BeautifulSoup(html, "lxml")
```

We'll have to change our other functions a bit now, but it's pretty minor - we just need to replace our duplicated lines with the following:

```python
soup = make_soup(url) # where url is the url we're passing into the original function
```

#### Putting it all together
Now that we have our main functions written, we can write a script to output the data however we'd like.  Want to write to a CSV file?  Check out Python's [DictWriter](http://docs.python.org/2/library/csv.html#csv.DictWriter) class.  Storing the data in a database?  Check out the [sqlite3](http://docs.python.org/2/library/sqlite3.html) or [other various database libraries](http://wiki.python.org/moin/DatabaseInterfaces).  While both tasks are somewhat outside of my intentions for this post, if there's interest, let me know in the comments and I'd be happy to write more.

Hopefully you found this post useful.  I've put a final example script in [this gist](http://bit.ly/13yd9ng).
