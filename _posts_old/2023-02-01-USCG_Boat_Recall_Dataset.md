# USCG Boat and Watercraft Recall dataset

Michael Nolan

## A web scraping exercise
A few weeks ago, I noticed a call to action in Jeremy Singer-Vine's great newsletter [Data is Plural](https://www.data-is-plural.com/).
The [2023-01-25 post](https://www.data-is-plural.com/archive/2023-01-25-edition/) mentioned that "someone could probably scrape" a collection of semi-tabular boat recall data from the [U.S. Coast Guard (USCG) boating safety information website](https://uscgboating.org/). 
The website has lots of data for each individual recall, but this data is broken out across several web pages. 
They also don't make the data available in a single structured format.

Looking to get some practice in, I saw this as a good opportunity to teach myself how to scrape websites for tabular data.
Using the table scraping features built into the `pandas` package in python, I wrote a simple script to collect the boat recall data into a single CSV table.

The script and table can be found at the following github repo, along with documentation describing each column in the table: [https://github.com/m-nolan/USCG_Boat_Recalls](https://github.com/m-nolan/USCG_Boat_Recalls)

Also, lots of thanks to Jeremy for shouting me out in the most recent newsletter edition.