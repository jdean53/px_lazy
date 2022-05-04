## Imports


```python
'''
Package Import
---
* Operational
* Data Gather
* Data Management
* Time
'''
import glob
import os
import shutil
from tqdm import tqdm

from sec_edgar_downloader import Downloader
import pandas_datareader as pdr 

from bs4 import BeautifulSoup
import re
import numpy as np 
import pandas as pd

import datetime as dt
from time import sleep
```

## File/Folder Creation


```python
'''
creates file structure, initiates downloader, pulls ticker list
---
* Creates file structure - Sorts by filing type (Q or K) and year ('10 or '20)
* Initiates filing downloader (dl for 10-K, dq for 10-Q)
* Pulls ticker list (for 2010 and 2020)
'''
os.makedirs("input", exist_ok=True)                                         # creates the input folder
os.makedirs("10k_files/2010/", exist_ok=True)                               # creates the 10k_files folder with the 2010 folder within
os.makedirs("10k_files/2020/", exist_ok=True)                               # creates the 10k_files folder with the 2020 folder within
os.makedirs("10q_files/2010/", exist_ok=True)                               # creates the 10q_files folder with the 2010 folder within
os.makedirs("10q_files/2020/", exist_ok=True)                               # creates the 10q_files folder with the 2020 folder within

dl = Downloader('10k_files')                                                # puts the downloader focused on the 10k_files folder
dq = Downloader('10q_files')                                                # puts the downloader focused on the 10k_files folder

sample_csv_2010 = "input/sp500_firms_2010.csv"                              # focuses the sample csv on the 2010 list of S&P500 companies
sample_2010 = pd.read_csv(sample_csv_2010)                                  # reads the csv file into the sample variable
sample_csv_2020 = "input/sp500_firms.csv"                                   # creates the sample_csv path for the sp500 firms

if not os.path.exists(sample_csv_2020):                                     # if the sample csv file does not already exist
    url = "https://en.wikipedia.org/wiki/List_of_S%26P_500_companies"       # the url of the current list of sp500 companies
    sample_2020 = pd.read_html(url)[0]                                      # reads the html
    sample_2020.to_csv(sample_csv_2020, index=False)                        # makes it into the csv file and names it with the path of sample_csv provided above 
else:
    sample_2020 = pd.read_csv(sample_csv_2020)                              # reads the csv file into the sample variable
```

## Combined 10K Download 2009/2010 with CSV File


```python
'''
Premptive data cleaning
---
List of filler words we wish to remove from the word counts
These are constant across all filings & years
'''
fill_words = ['and', 'the', 'of', 'to', 'in', 'a', 'for', 'on', 'or', 'as', 'is', 'this']
```


```python
'''
Pulls 2010 10-K filings
---
for given firm list
    - create a folder for each firm
    - pull filing for firms and store
    - remove txt filings
    - read html filing
        * store word count df as a csv in the same file folder
    - delete the html file
store the filings in the approprate folder for the year
'''
if(os.path.isdir("10k_files/2010/sec-edgar-filings/") == True):                                                             # if the 10K files are already in the 2010 folder
    shutil.rmtree("10k_files/2010/sec-edgar-filings/")                                                                      # then deletes the sec-edgar-filings folder
for firm in tqdm(list(sample_2010['ticker'])[354:543]):                                                                         # for loop that runs through a specified number of the symbols in the sp500
    firm_folder = "10k_files/sec-edgar-filings/" + firm                                                                     # assigns the path for each firm within the 10k_files folder
    html_files = glob.glob(firm_folder  + '/**/*.html', recursive=True)                                                     # assigns the html_files to the html files that are in the path assigned above

    if len(html_files) == 0:                                                                                                # if statement that runs if the length of the html file is 0 meaning it doesn't exist
        dl.get("10-K", firm, after="2009-01-01", before="2010-12-20")                                                       # gets the 10q for the specified dates (adjusted to June due to ABMD with 3/31 FY)
    
    html_files = glob.glob(firm_folder  + '/**/*.html', recursive=True)                                                     # assigns the html_files to the html files that are in the path assigned above
    
    for f in glob.glob(firm_folder  + '/**/*.txt', recursive=True):                                                         # Removes unnecessary text file
        os.remove(f)
    
    if(os.path.isdir("10k_files/sec-edgar-filings/" + firm + "/10-K") == True):                                             # Error trap: does path exist?
        num_folder = os.listdir("10k_files/sec-edgar-filings/" + firm + "/10-K")
        counter = 0
        for file in html_files:
            with open(file, encoding ='utf-8', errors='ignore') as f:                                                                        # Opens file
                doc = BeautifulSoup(f.read()).get_text()                                                                    # Reads file to text
                doc = re.sub(r'\W',' ',doc)                                                                                 # removes white space
                doc = re.sub(r'\s+',' ',doc)                                                                                # lowercase
            
            word_df = pd.value_counts(np.array(doc.split()))                                                                # splits doc from 1 long string to dataframe of words with their frequency
            word_df = word_df.drop(fill_words)                                                                              # removes filler words from word count df
            sample_csv = "10k_files/sec-edgar-filings/" + firm + "/10-K/" + num_folder[counter] + "/bag_o_words.csv"        # creates path for word count csv
            if not os.path.exists(sample_csv):                                                                              # if the sample csv file does not already exist
                word_df.to_csv(sample_csv)                                                                                  # creates a csv of the word count
            counter = counter + 1
        
        for f in glob.glob(firm_folder  + '/**/*.html', recursive=True):                                                    # for the html file provided
            os.remove(f)                                                                                                    # remove the html file

cwd = os.getcwd()                                                                                                           # get current working directory
path_sec_edgar = cwd + "/10k_files/sec-edgar-filings/"                                                                      # move files from here
path_destination = cwd + "/10k_files/2010"                                                                                  # to here
shutil.move(path_sec_edgar, path_destination)                                                                               # using this
```

    100%|████████████████████████████████████████████████████████████████████████████████| 188/188 [41:04<00:00, 13.11s/it]
    




    'C:\\Users\\mattm\\Desktop\\FIN377\\project-px_lazy/10k_files/2010\\sec-edgar-filings'



## Combined 10K Download 2020 with CSV File


```python
'''
Pulls 2020 10-K filings
---
NOTE: Same loop as 2010 filings
for given firm list
    - create a folder for each firm
    - pull filing for firms and store
    - remove txt filings
    - read html filing
        * store word count df as a csv in the same file folder
    - delete the html file
store the filings in the approprate folder for the year
'''
if(os.path.isdir("10k_files/2020/sec-edgar-filings/") == True):                                                             # if the 10K files are already created in the 2020 folder
    shutil.rmtree("10k_files/2020/sec-edgar-filings/")                                                                      # then deletes the sec-edgar-filings folder
for firm in tqdm(sample_2020['Symbol'][:505]):                                                                              # for loop that runs through a specified number of the symbols in the sp500
    firm_folder = "10k_files/sec-edgar-filings/" + firm                                                                     # assigns the path for each firm within the 10k_files folder
    html_files = glob.glob(firm_folder  + '/**/*.html', recursive=True)                                                     # assigns the html_files to the html files that are in the path assigned above
    
    if len(html_files) == 0:                                                                                                # if statement that runs if the length of the html file is 0 meaning it doesn't exist
        dl.get("10-K", firm, after="2020-01-01", before="2021-12-20")                                                       # gets the 10q for the specified dates (adjusted to June due to ABMD with 3/31 FY)
    
    html_files = glob.glob(firm_folder  + '/**/*.html', recursive=True)                                                     # assigns the html_files to the html files that are in the path assigned above
    
    for f in glob.glob(firm_folder  + '/**/*.txt', recursive=True):                                                         # removes unnecessary text file
        os.remove(f)
    
    if(os.path.isdir("10k_files/sec-edgar-filings/" + firm + "/10-K") == True):                                             # Error trap: does path exist?
        num_folder = os.listdir("10k_files/sec-edgar-filings/" + firm + "/10-K")
        counter = 0
        for file in html_files:
            with open(file, encoding ='utf-8', errors='ignore') as f:                                                                        # Opens file
                doc = BeautifulSoup(f.read()).get_text()                                                                    # Reads file to text
                doc = re.sub(r'\W',' ',doc)                                                                                 # Removes white space
                doc = re.sub(r'\s+',' ',doc)                                                                                # lowercase
            
            word_df = pd.value_counts(np.array(doc.split()))                                                                # splits doc from 1 long string to dataframe of words with their frequency
            word_df = word_df.drop(fill_words)                                                                              # removes filler words from word count df
            sample_csv = "10k_files/sec-edgar-filings/" + firm + "/10-K/" + num_folder[counter] + "/bag_o_words.csv"        # creates path for word count csv
            if not os.path.exists(sample_csv):                                                                              # if the sample csv file does not already exist
                word_df.to_csv(sample_csv)                                                                                  # creates csv of the word count
            counter = counter + 1
        
        for f in glob.glob(firm_folder  + '/**/*.html', recursive=True):                                                    # for the html file provided
            os.remove(f)                                                                                                    # remove the html file

cwd = os.getcwd()                                                                                                           # Get current working directory
path_sec_edgar = cwd + "/10k_files/sec-edgar-filings/"                                                                      # move files from here
path_destination = cwd + "/10k_files/2020"                                                                                  # to here
shutil.move(path_sec_edgar, path_destination)                                                                               # using this
```

## Combined 10Q Download 2009/2010 with CSV File


```python
'''
Pulls 2010 10-Q filings
---
NOTE: Same loop as 2010 10-K filings
for given firm list
    - create a folder for each firm
    - pull filing for firms and store
    - remove txt filings
    - read html filing
        * store word count df as a csv in the same file folder
    - delete the html file
store the filings in the approprate folder for the year
'''
if(os.path.isdir("10q_files/2010/sec-edgar-filings/") == True):                                                             # if the 10Q downloads are already located in the 2010 folder
    shutil.rmtree("10q_files/2010/sec-edgar-filings/")                                                                      # then deletes the sec-edgar-filings folder
for firm in tqdm(sample_2010['ticker'][2:543]):                                                                               # for loop that runs through a specified number of the symbols in the sp500
    firm_folder = "10q_files/sec-edgar-filings/" + firm                                                                     # assigns the path for each firm within the 10k_files folder
    html_files = glob.glob(firm_folder  + '/**/*.html', recursive=True)                                                     # assigns the html_files to the html files that are in the path assigned above

    if len(html_files) == 0:                                                                                                # if statement that runs if the length of the html file is 0 meaning it doesn't exist
        dq.get("10-Q", firm, after="2009-01-01", before="2010-12-29")                                                       # gets the 10q for the specified dates (adjusted to June due to ABMD with 3/31 FY)
    
    html_files = glob.glob(firm_folder  + '/**/*.html', recursive=True)                                                     # assigns the html_files to the html files that are in the path assigned above
    
    for f in glob.glob(firm_folder  + '/**/*.txt', recursive=True):                                                         # removes unnecessary text file
         os.remove(f)

    if(os.path.isdir("10q_files/sec-edgar-filings/" + firm + "/10-Q") == True):                                             # Error trap: does path exist?
        num_folder = os.listdir("10q_files/sec-edgar-filings/" + firm + "/10-Q")
        counter = 0
        for file in html_files:
            with open(file, encoding ='utf-8', errors='ignore') as f:                                                                        # Opens file
                doc = BeautifulSoup(f.read()).get_text()                                                                    # Reads file to text
                doc = re.sub(r'\W',' ',doc)                                                                                 # Removes white space
                doc = re.sub(r'\s+',' ',doc)                                                                                # lowercase
            
            word_df = pd.value_counts(np.array(doc.split()))                                                                # splits doc from 1 long string to dataframe of words with their frequency
            word_df = word_df.drop(fill_words)                                                                              # removes filler words from word count df
            sample_csv = "10q_files/sec-edgar-filings/" + firm + "/10-Q/" + num_folder[counter] + "/bag_o_words.csv"        # creates path for word count csv
            if not os.path.exists(sample_csv):                                                                              # if the sample csv file does not already exist
                word_df.to_csv(sample_csv)                                                                                  # creates csv of the word count
            counter = counter + 1    
        
        for f in glob.glob(firm_folder  + '/**/*.html', recursive=True):                                                    # for the html file provided
            os.remove(f)                                                                                                    # remove the html file

cwd = os.getcwd()                                                                                                           # Get current working directory
path_sec_edgar = cwd + "/10q_files/sec-edgar-filings/"                                                                      # move files from her
path_destination = cwd + "/10q_files/2010"                                                                                  # to here
shutil.move(path_sec_edgar, path_destination)                                                                               # using this
```

## Combined 10Q Download 2020 with CSV File


```python
'''
Pulls 2020 10-Q filings
---
NOTE: Same loop as 2010 10-K filings
for given firm list
    - create a folder for each firm
    - pull filing for firms and store
    - remove txt filings
    - read html filing
        * store word count df as a csv in the same file folder
    - delete the html file
store the filings in the approprate folder for the year
'''
if(os.path.isdir("10q_files/2020/sec-edgar-filings/") == True):                                                             # if the 10Q downloads are already located in the 2020 folder
    shutil.rmtree("10q_files/2020/sec-edgar-filings/")                                                                      # then deletes the sec-edgar-filings folder
for firm in tqdm(sample_2020['Symbol'][:505]):                                                                               # for loop that runs through a specified number of the symbols in the sp500
    firm_folder = "10q_files/sec-edgar-filings/" + firm                                                                     # assigns the path for each firm within the 10k_files folder
    html_files = glob.glob(firm_folder  + '/**/*.html', recursive=True)                                                     # assigns the html_files to the html files that are in the path assigned above

    if len(html_files) == 0:                                                                                                # if statement that runs if the length of the html file is 0 meaning it doesn't exist
        dq.get("10-Q", firm, after="2020-01-01", before="2021-12-29")                                                       # gets the 10q for the specified dates (adjusted to June due to ABMD with 3/31 FY)
    
    html_files = glob.glob(firm_folder  + '/**/*.html', recursive=True)                                                     # assigns the html_files to the html files that are in the path assigned above
    
    for f in glob.glob(firm_folder  + '/**/*.txt', recursive=True):                                                         # removes unnecessry text file
         os.remove(f)
    
    if(os.path.isdir("10q_files/sec-edgar-filings/" + firm + "/10-Q") == True):                                             # Error trap: does path exist?
        num_folder = os.listdir("10q_files/sec-edgar-filings/" + firm + "/10-Q")
        counter = 0
        for file in html_files:                                                                                             
            with open(file, encoding ='utf-8', errors='ignore') as f:                                                                        # Opens file
                doc = BeautifulSoup(f.read()).get_text()                                                                    # Reads file as text
                doc = re.sub(r'\W',' ',doc)                                                                                 # Removes whitespace
                doc = re.sub(r'\s+',' ',doc)                                                                                # lowercase

            word_df = pd.value_counts(np.array(doc.split()))                                                                # splits doc from 1 long string to dataframe of words with their frequency
            word_df = word_df.drop(fill_words)                                                                              # removes filler words from word count df
            sample_csv = "10q_files/sec-edgar-filings/" + firm + "/10-Q/" + num_folder[counter] + "/bag_o_words.csv"        # creates path for word count csv
            if not os.path.exists(sample_csv):                                                                              # if the sample csv file does not already exist
                word_df.to_csv(sample_csv)                                                                                  # creates csv of word count
            counter = counter + 1    
        
        for f in glob.glob(firm_folder  + '/**/*.html', recursive=True):                                                    # for the html file provided
            os.remove(f)                                                                                                    # remove the html file
            
cwd = os.getcwd()                                                                                                           # gets current working directory
path_sec_edgar = cwd + "/10q_files/sec-edgar-filings/"                                                                      # moves files from here
path_destination = cwd + "/10q_files/2020"                                                                                  # to here
shutil.move(path_sec_edgar, path_destination)                                                                               # using this
```

## Yahoo Finance Returns


```python
symbols_df = [] # initializes the name dataframe that stores the symbols
for firm in tqdm(sample['Symbol'][:5]): # for loop that runs through a specified number of the symbols in the sp500
    symbols_df.append(firm) # adds to the above symbols_df array each firm's symbol
first_df = pdr.DataReader(symbols_df, data_source = 'yahoo', start = '2008-12-21', end = '2012-01-01') # first dataframe that stores the returns for each firm from 2009-2011
second_df = pdr.DataReader(symbols_df, data_source = 'yahoo', start = '2020-01-01', end = '2021-06-01') # second dataframe that stores the returns for each firm from 2020-2021
```

## 2009/2010/2011 10-K Monthly Returns


```python
first_monthly_returns = first_df['Adj Close'].resample('M').ffill().pct_change() # changes the returns dataframe above to just monthly returns for each symbol
first_monthly_returns.drop(['2008-12-31'], inplace = True) # drops the first month because it is calculated as NaN
first_monthly_returns # testing
```

## 2020 Fiscal Year 10-K Monthly Returns


```python
second_monthly_returns = second_df['Adj Close'].resample('M').ffill().pct_change() # changes the returns dataframe above to just monthly returns for each symbol
second_monthly_returns.drop(['2020-05-31'], inplace = True) # drops the first month because it is calculated as NaN
second_monthly_returns.drop(['2021-06-30'], inplace = True) # drops the last month because it is calculated as NaN
second_monthly_returns # testing 
```


```python

```
