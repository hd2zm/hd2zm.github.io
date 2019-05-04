---
layout: post
title:  "Faster Web Scraping with Multithreading"
date:   2019-04-22 05:11:03 +0700
categories: [Web Scraping]
---

For one of my projects, I analyzed body measurements of NBA Players and predicted whether this had an impact on getting nominated for the all stars. You can leran more about the project [here]({% link _project/NBA-AllStar.md %})

One of the biggest challenges was web scraping the data. I gathered data from 3 different sources: [All-Star](https://en.wikipedia.org/wiki/List_of_NBA_All-Stars) from Wikipedia (whether an NBA player is an All-Star), [ESPN NBA Value Added](http://insider.espn.com/nba/hollinger/statistics), and [NBA Draft Body Measurements](https://stats.nba.com/draft/combine-anthro/). I used Beautiful Soup to exctract data from All-Star data and ESPN NBA Value Added data, but had to use Selenium in addition to extract NBA Draft Body Measurements. Below is a sample method I used to web scrape NBA Draft Body Measurements with Selenium. 

```python
def get_measurements_content(url, measurements_data):
    
    measurements_data = []
    #Set as Beautiful Soup Object
    driver = webdriver.Chrome("/usr/local/bin/chromedriver")
    
    driver.get(url)
    measurements_soup = BeautifulSoup(driver.page_source)
    driver.quit()
    # Go to the section of interest
    measurements_summary = measurements_soup.find('div', attrs={'class':'nba-stat-table'})

    # Find the tables in the HTML
    measurements_tables = measurements_summary.find_all('table')
        
    # Set rows as first indexed object in tables with a row
    rows = measurements_tables[0].findAll('tr')
    
    # now grab every HTML cell in every row
    for tr in rows:
        cols = tr.findAll('td')
        # Check to see if text is in the row
        measurements_data.append([])
        for td in cols:
            text = td.find(text=True) 
            measurements_data[-1].append(text)
```

I wanted to get NBA Measurements from 2003-2018. So that is 16 urls I have for NBA Body Measurements (https://stats.nba.com/draft/combine-anthro/2003-04, https://stats.nba.com/draft/combine-anthro/2004-05, etc). I got my urls and called the above function for each url. 

**Problem?** Because I'm using Selenium, I'm opening up Google chrome, extract the data, and exiting the page. I repeat the process 15 more times. It took me 1 minute and 30 seconds to scrape that data. While this is fine for 15 requests, it'll be problematic if I want to extend the code to include players from 1990s or 1980s. 

**Solution: Multithreading**

From this definition [here](https://beginnersbook.com/2013/03/multithreading-in-java/). 

> Before we talk about multithreading, let’s discuss threads. A thread is a
> light-weight smallest part of a process that can run concurrently with  
> the other parts(other threads) of the same process. Threads are 
> independent because they all have separate path of execution that’s the 
> reason if an exception occurs in one thread, it doesn’t affect the 
> execution of other threads. All threads of a process share the common 
> memory. The process of executing multiple threads simultaneously is known 
> as multithreading."

> The main purpose of multithreading is to provide simultaneous execution 
> of two or more parts of a program to maximum utilize the CPU time. A 
> multithreaded program contains two or more parts that can run 
> concurrently. Each such part of a program called thread.

I want to execute `get_measurements_content(url)` simultaneously 16 times. I imported `threading` to help me do this. I then added this code to create 16 different threads.

```python

def get_measurements(start_year, end_year):

    #Generating list of urls
    measurements_base_url = 'https://stats.nba.com/draft/combine-anthro/'
    measurements_data = []
    
    urls=[]
    while start_year < end_year:
        urls.append(measurements_base_url + '#!?SeasonYear=' + str(start_year) + '-' + str(start_year+1)[-2:])
        start_year = start_year + 1

    threads = []
    
    for url in urls:
        process = threading.Thread(target=get_measurements_content, args=(url, measurements_data))
        process.start()
        threads.append(process) 
     
    for process in threads:
        process.join()

    print(measurements_data)
```

On execution, 16 web browsers loaded simultaneously, each parsing the data simultaneously. This cut my time down to 40 seconds.  

**Problem?** If I wanted to scale and include data starting from 1990s, I would have around 30 web browsers running at once! Take it back to 1970s and I have around 50. This is too much overhead for a computer, and I risk crashing my program. To streamline this smoothly, I want to run a select number of threads at once. Once a thread is finished, I can reuse that same thread for another task.   

**Solution: Thread Pooling**

Thread pooling is what I want to accomplish. A thread pool is a group of pre-instantiated, idle threads which stand ready to be given work. With many small tasks needed to be executed, thread pooling is a prefered alternative over creating new threads. This prevents overhead of creating a thread multiple times. 

I want to instantiate 6 threads at once and reuse them whenever they're finished. I rewrote my two functions `get_measurement_content` and `get_measurements` to account for ThreadPooling. Below is the new code

```python
from bs4 import BeautifulSoup
import requests
from selenium import webdriver
from multiprocessing.dummy import Pool as ThreadPool

def get_measurements_content(url):
    
    measurements_data = []
    #Set as Beautiful Soup Object
    driver = webdriver.Chrome("/usr/local/bin/chromedriver")
    
    driver.get(url)
    measurements_soup = BeautifulSoup(driver.page_source)
    driver.quit()
    # Go to the section of interest
    measurements_summary = measurements_soup.find('div', attrs={'class':'nba-stat-table'})
    
    # Find the tables in the HTML
    measurements_tables = measurements_summary.find_all('table')
        
    # Set rows as first indexed object in tables with a row
    rows = measurements_tables[0].findAll('tr')
    
    # now grab every HTML cell in every row
    for tr in rows:
        cols = tr.findAll('td')
        # Check to see if text is in the row
        measurements_data.append([])
        for td in cols:
            text = td.find(text=True) 
            measurements_data[-1].append(text)
            
    return measurements_data

def get_measurements(start_year, end_year):

    # Create urls
    measurements_base_url = 'https://stats.nba.com/draft/combine-anthro/'
    measurements_data = []
    
    urls=[]
    while start_year < end_year:
        urls.append(measurements_base_url + '#!?SeasonYear=' + str(start_year) + '-' + str(start_year+1)[-2:])
        start_year = start_year + 1

    # Instantiate thread pool with 6 threads
    pool = ThreadPool(6)
    measurements_data = pool.map(get_measurements_content, urls)
    pool.close()
    pool.join()
    
    measurements_df = pd.DataFrame() 

    for measurement in measurements_data:
        measurements_df = measurements_df.append(measurement[1:])
    
    
    #Hidden code that deals with data frame parsing/manipulation
    ...
    
    return measurements_df

```

My code executes in 40 seconds, so same as multithreading. But it is better for scalability. 

Code for project can be found [here](https://github.com/hd2zm/Data-Science-Projects/blob/master/NBA-Analytics/All-Star-Analysis/NBA_Measurements_Value_Scraper.py).