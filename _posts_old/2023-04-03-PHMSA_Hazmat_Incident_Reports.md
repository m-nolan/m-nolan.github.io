# PHMSA Hazmat Reports - Work with the Data Liberation Project

Michael Nolan

## Hazmat Incident Reporting
I recently had the opportunity to work with the [Data Liberation Project](https://www.data-liberation-project.org/) on a web scraping project. The Pipeline and Hazardous Materials Safety Administration, a division of the U.S. Department of Transportation, regularly publishes hazmat incident reports. However, data for each incident is not saved to a collected source for easy access.

DLP went ahead and made a scraper to collect that incident data into a single set of csv files that can be found through [this project description page](https://www.data-liberation-project.org/datasets/phmsa-hazmat-incident-reports/). I was able to contribute to the project by building a script that regularly updates and publishes an RSS feed of all incidents. A separate feed for each U.S. state is also available. All feeds can be found here in the [project's github repo](https://github.com/data-liberation-project/phmsa-hazmat-incident-reports/tree/main/data/processed/feeds).