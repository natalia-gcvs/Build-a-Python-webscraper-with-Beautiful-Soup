# Build-a-Python-webscraper-with-Beautiful-Soup

In this tutorial, I discuss web scraping technique using BeautifulSoup, which is the Python library for parsing HTML and XML documents.

# Table of contents

1. What’s webscraping?	
2. Why use webscraping	
3. Webscrapping Libraries in Python	
4. Scraping data from H&M website	

  4.1 Downloading and parsing the page  
  4.2 Extracting price and type details	
  4.3 Extracting color details	  
  4.4 Extracting Composition details
  4.5 Joining the tables
  
  


What’s webscraping?

Webscraping is a technique used to collect data from the internet. Webscrapers are programmed to go to websites, get the relevant pages and extract the information needed. The automation of this process allows tons of data to be extracted at a high speed. 
Why use webscraping 

Some websites offer datasets that are downloadable in CSV format, or accessible via an Application Programming Interface (API). But many websites with useful data don’t offer these convenient options. Therefore, to get it we need to make use of webscraping.
Webscrapping Libraries in Python

Webscraping tools vary from simple ones where no coding experience is required to advanced ones such as simulating browser interaction. Below are some of the webscraping libraries in Python. 
•	Scrapy,
•	Selenium-Python
•	Beautiful Soup
•	lxml
•	HTQL
•	Mechanize

Scraping data from H&M website

In this tutorial, we will build a web scraper using the libraries Beautiful Soup and Requests in Python to extract the following data from H&M website.
•	Product price 
•	Product type
•	Product color
•	Product composition

Downloading and parsing the page 

We start by downloading the pages using the method get in the Python requests library. It sends a request to the specified URL. The headers argument informs the web server who is making the request, so our request is not denied. 

 


Once the request was successful, we want to convert it into a string using .text. The output is a huge mess, HTML content that we cannot make sense of. That’s when Beautiful Soup Python library comes into play.

 

BeautifulSoup is a Python library for parsing structured data. It makes it possible navigating and retrieving data from HTML tags. The library contains a couple of functions that can be used to explore the HTML received. 
When you add the last line of code, a Beautiful Soup object is created. It takes page.text, which is the HTML content you scraped earlier, as its input. The second argument, "html.parser", makes sure that you use the right parser for HTML content. 

 

One thing to be noted is, the URL above directs us to the webpage shown below. Looking at the end of the webpage, we see more products to be loaded. The first URL gives us only part of the items we need.

 

Right-clicking on the number of items allows us to see the page source code shown below. Where there are 36 items per page and 91 in total. Now we can automate the process of loading more products. 

 

Clicking to load more products gives us the following URL https://www2.hm.com/en_us/men/products/jeans.html?sort=stock&image-size=small&image=model&offset=0&page-size=72. 
To better understand how it works let’s deconstruct the above URL into three main parts:
Start: The beginning of the query parameters is represented by a question mark (?).
Information: The details representing one query parameter are encoded in key-value pairs, where related keys and values are joined together by an equal sign (key=value).
Separator: Every URL can have multiple query parameters, separated by an ampersand symbol (&).
What we need to do is to get the page size that contains all items. To do that we first get the total number of items by using the find_all method. As we only want the first item in the list, we select the item 0 and get the element data-total that holds the total number of items. Then, we divide it by 36 (number of items per page), the output is the total number of pages rounded up (the function ceil from the library math does that). Finally, we can concatenate the URL with the page size that gives us all items.

 

Now that we’ve downloaded the page we needed, it’s time to retrieve the data we want. We basically have 4 details we need, price, type, color and composition
Before starting to code, let’s take a quick look at the webpage structure. Each image is a product, clicking on it leads us to another page. The product type and price can be extracted from this page, but to get color and composition we have to go to each individual product page.

 

As we will be using different URL to retrieve all the data, we can write a function to save us time. 
 
 
Before we go any further let’s look at the Beautiful Soup Functions used in this tutorial.
find_all – returns a list containing all results matching the search criteria defined.
find – returns the first result matching a search criterion that we applied on a Beautiful Soup object.
get_text – returns all the text in a document or beneath a tag, as a single unicode string

Extracting price and type details 

Clicking on inspect product type reveals the following code. The HTML <li> defines separate items in the <ul>  list. To retrieve the data, we can either use the find method passing the tag <ul> class=”products-listing small”  and, then the method find_all passing the tag <li> class=”product-item” or simply the later one.

 

We want to extract the product id (data-articlecode) that’s inside the tag <li> class=”product-item” so later we can join tables based on that, the product URL (href) that will be needed to extract other data in the next step,  the product type and product price. 
We first declare a list where the data will be stored. An object soup is passed to the function get_type( ). The output of the method find_all is stored in a variable. We then use a for loop to iterate over the items_list  to get just the data needed and store them in a dictionary where each detail is a key. The output is a list of dictionaries containing the product url, id, name, and price that we can finally convert into a dataframe. 

 

We now have a dataframe containing 89 product types and prices.

 

Extracting color details

Similarly, to type and price the tag <ul class=”inputlist clearfix”> is a list containing all tags <li class=”list-item”> . So, first we use the find method passing <ul class=”inputlist clearfix”>  as argument and retrieve all tags inside that tag. Then, by using find_all method we obtain a list of tags <li class=”list-item”> where we are able to iterate over and get the data highlighted below. 

 

The code below demonstrates how the data were retrieved and stored into a dataframe.


 

Some product types are shown more than once on the main page in different colors causing us to collect the same product type and its colors more than once. To guarantee all the products are unique we use pandas drop_duplicates( ) method.

 
Extracting Composition details

The composition details are inside a <div> tag as shown below

 


To get it, we retrieve all the data enclosed inside the tag < div class=”details parbase” using the find method. We then pass ‘script’ as an argument to the find_all method that returns a list with all the scripts inside that tag. As we saw on the previous figure, the script we need is the second one, we select it using square brackets and entering 1. Then, iterate over the list, extract the data needed and store it into a dictionary.  As some of the products have a different structure, a try except raise was added.
 Next, we iterate over the pages where we append each json object to a list so we it can be converted into a dataframe later. The data requires some “cleaning” since part of it is in a different format, so a function was created to help with that. Once we have the dataframe we can use regular expressions to make the final adjustments. 

 
 

The result can be seen below, a dataframe with 169 rows and 1 column.

 

Joining the tables

We’ve collected all the data needed and stored them in different dataframes, now it’s time to join them all together. Starting from color and composition dataframes that have the same length and order we use the concatenate function on the axis 1,  the final output is shown below.

 

The next step is to merge color_comp with df_type. As those two dataframes have different lengths we can use the merge function where the color_comp is the left table merged with df_type on the id column. It’s like a SQL left outer join that finds all the rows from the first table including the matching rows from the right table. The output is a dataframe with some missing values in the columns that belongs to the smaller dataset.

 

To fix that, we create a new column called style_id, note that the product id has the same number sequence for each product type changing only the three last numbers according to the color, so we get all the number but the last three. The price varies only along with product type, meaning that once we have all product prices, which we do, we can fill the missing values with those prices. We do that by creating a dataframe containing style_id, price and type, dropping nan’s and duplicates to guarantee all values are unique. Finally, we use the function merge that executes to fetch all the rows from the left table (product_details) along with some common rows if matched from the right table (style_id) form the two tables. The result is two columns price and type the latest ones have no missing values, so we drop the first ones and style_id since we no longer need it.

 

The final output is the following table, ready for the analysis. 

 

The compiled code can be found on my GitHub page linked below.




                                                                 
                                              


 
