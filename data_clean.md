# Data Clean

The purpose of this file is to go through the 10K and 10Q downloads of data_grab and clean any data that we cannot do calculations on, while also calculating the returns and cosine similarities of each ticker for the years 2009-2010 and 2020. 



## Imports


```python
'''
packages used throughout project  
---
operational
    * os
    * ssl (used for reading firm list from URL)
data management
    * pandas
    * yfinance
math
    * numpy
plotting
    * matplotlib
    * sns
natural language processing
    * re
    * bs4 (BeautifulSoup for reading HTML)
'''
import os
import ssl
import glob
import pandas as pd
import yfinance as yf
import pandas_datareader as pdr
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import re
from bs4 import BeautifulSoup
from sklearn.metrics.pairwise import cosine_similarity
from sec_edgar_downloader import Downloader
import datetime
```

## Retrieves Data From CSV Files


```python
'''
get ticker lists
'''
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


```python
tickers_2010 = sample_2010['ticker'].to_list()                              # sorts the tickers of 2010 into a list
tickers_2020 = sample_2020['Symbol'].to_list()                              # sorts the tickers of 2020 into a list
```

## 2010 Cosine Similarity Analysis


```python
'''
2010 analyis
'''
    
'''
For data organization and storage
---
cossim_index - index for returned dictionary
cossims - list for storage of cossine similarities
'''
cossim_index = ['10k','q1','q2','q3']                                                                                       # cosine similarity index with 10k, q1, q2, q3
cossims_all_firms = []                                                                                                      # initializes a cossims_all_firms array                                                                          
for ticker in tickers_2010:                                                                                                 # for loop that runs through all of the 2010 tickers
    '''Begin by establishing paths and directories'''
    path_10k = '10k_files\\2010\\sec-edgar-filings\\' + ticker + '\\10-K\\'                                                 # assigns the path of the 2010 10k's
    path_10q = '10q_files\\2010\\sec-edgar-filings\\' + ticker + '\\10-Q\\'                                                 # assigns the path of the 2010 10q's
    '''cosine similarity for each firm'''
    cossims = []                                                                                                            # initializes the cossims array
    '''
    10-K part
    ---
    Loops over filings to get word bags
    stores word bags as 1 df
    computes cosine sim
    '''
    if os.path.exists(path_10k) and (len(os.listdir(path_10k)) >= 2):                                                       # if the path for the 10k's exists and there are greater than or equal to 2 10k's
        filing_list_10k = os.listdir(path_10k)                                                                              # assigns the directory from the path to the filing_list_10k variable
        for filing in filing_list_10k:                                                                                      # for loop that runs through the directories found above
            throw_error = False                                                                                             # initializes throw_error as False
            current_filing = path_10k + filing                                                                              # sets current_filing equal to path_10k with the next directory
            if os.path.exists(current_filing + '\\bag_o_words.csv'):                                                        # if there is a bag of words CSV file
                if filing == filing_list_10k[0]:                                                                            # if the filing is equal to the 0 index of the filing_list_10k
                    word_df = pd.read_csv(path_10k + '\\' + filing + '\\bag_o_words.csv', index_col='Unnamed: 0')           # sets the word_df equal to the path_10k with the filing and the bag of words within
                else:
                    word_df = pd.merge(
                        left = word_df, 
                        right = pd.read_csv(path_10k + '\\' + filing + '\\bag_o_words.csv', index_col='Unnamed: 0'),
                        how = 'inner',
                        left_index=True,
                        right_index=True)                                                                                   # merges with an inner merge the word_df and the newly read bag of words
            else:
                throw_error = True                                                                                          # sets the throw_error equal to True
        if throw_error:                                                                                                     # if the throw_error is equal to True
            cossim_10k = 0                                                                                                  # sets the cossim_10k equal to 0 which we clean later
        else:
            cossim_10k = np.dot(word_df.iloc[:,0], word_df.iloc[:,1]) / (np.linalg.norm(word_df.iloc[:,0]) * np.linalg.norm(word_df.iloc[:,1]))
        cossims.append(cossim_10k)                                                                                          # appends the cossim_10k to the cossims
    else:
        cossims.append(0)                                                                                                   # appends 0 to the cossims

    '''
    10-Q part
    ---
    loops over first 3 10-Qs
        * assuming these are the same fiscal year and second 3 are different year
    sotres bag of words as 1 df for each paired statement
    computes cosine sims
    '''
    if os.path.exists(path_10q) and (len(os.listdir(path_10q)) >= 6):                                                       # if the path of the 10q exists and there are 6 or more 10q's located within
        filing_list_10q = os.listdir(path_10q)                                                                              # sets the filing_list_10q equal to the directory within the path_10q
        counter = 0                                                                                                         # sets a counter equal to 0
        for filing in filing_list_10q[:3]:                                                                                  # for loop that runs through the filing_list_10q up to index 3                           
            current_filing = path_10q + '\\' + filing                                                                       # sets the current filing equal to the path_10q and adds the looped filing at the end
            if os.path.exists(current_filing + '\\bag_o_words.csv') and os.path.exists(path_10q + '\\' + filing_list_10q[3+counter] + '\\bag_o_words.csv'):
                word_df = pd.read_csv(path_10q + '\\' + filing + '\\bag_o_words.csv', index_col='Unnamed: 0')               # sets word_df equal to the bag of words within
                word_df = pd.merge(
                    left = word_df,
                    right = pd.read_csv(path_10q + '\\' + filing_list_10q[3+counter] + '\\bag_o_words.csv', index_col='Unnamed: 0'),
                    how = 'inner',
                    left_index = True,
                    right_index = True
                    )                                                                                                       # merges the initially read word_df with the new bag of words at counter + 3
                cossim_10q = np.dot(word_df.iloc[:,0], word_df.iloc[:,1]) / (np.linalg.norm(word_df.iloc[:,0]) * np.linalg.norm(word_df.iloc[:,1]))
                cossims.append(cossim_10q)                                                                                  # appends the cossim_10q to the cossims
            else:
                for i in range(3):                                                                                          # for loop that runs for the range of 3
                    cossims.append(0)                                                                                       # appends 0 to cossims
    else:
        for i in range(3):                                                                                                  # for loop that runs for the range of 3
            cossims.append(0)                                                                                               # appends 0 to cossims
    cossims_all_firms.append(dict(zip(cossim_index, cossims)))                                                              # appends the zipped cossim_index and cossims to cossims_all_firms

'''Dataorganization and reporting'''
cosine_df_2010 = pd.DataFrame(cossims_all_firms, index=tickers_2010)                                                        # sets cosine_df_2010 equal to the cossims_all_firms with the index equal to the tickers_2010
```

## 2020 Cosine Similarity Analysis


```python
'''
2020 analyis
NOTE: Same loop as the 2010s
'''
    
'''
For data organization and storage
---
cossim_index - index for returned dictionary
cossims - list for storage of cossine similarities
'''
cossim_index = ['10k','q1','q2','q3']                                                                                   # assigns the cossim_index to 10k, q1, q2, and q3
cossims_all_firms = []                                                                                                  # initializes the cossims_all_firms array
for ticker in tickers_2020:                                                                                             # for loop that runs through the 2020 tickers
    '''Begin by establishing paths and directories'''
    path_10k = '10k_files\\2020\\sec-edgar-filings\\' + ticker + '\\10-K\\'                                             # path of the 2020 ticker 10k's
    path_10q = '10q_files\\2020\\sec-edgar-filings\\' + ticker + '\\10-Q\\'                                             # path of the 2020 ticker 10q's
    '''cosine similarity for each firm'''
    cossims = []                                                                                                        # initialize the cossims array
    '''
    10-K part
    ---
    Loops over filings to get word bags
    stores word bags as 1 df
    computes cosine sim
    '''
    if os.path.exists(path_10k) and (len(os.listdir(path_10k)) >= 2):                                                   # if the 10k path exists and there are 2 or more 10k folders available
        filing_list_10k = os.listdir(path_10k)                                                                          # sets filing_list_10k equal to the directory of the path_10k
        for filing in filing_list_10k:                                                                                  # for loop that runs through the filing_list_10k
            throw_error = False                                                                                         # set throw_error to False
            current_filing = path_10k + filing                                                                          # sets current_filing equal to the path_10k plus the filing
            if os.path.exists(current_filing + '\\bag_o_words.csv'):                                                    # if there is a bag of words csv file
                if filing == filing_list_10k[0]:                                                                        # if the filing is equal to the first item
                    word_df = pd.read_csv(path_10k + '\\' + filing + '\\bag_o_words.csv', index_col='Unnamed: 0')       # sets word_df to the csv of the bag of words
                else:
                    word_df = pd.merge(
                        left = word_df, 
                        right = pd.read_csv(path_10k + '\\' + filing + '\\bag_o_words.csv', index_col='Unnamed: 0'),
                        how = 'inner',
                        left_index=True,
                        right_index=True)                                                                               # merges the current word_df with the next bag of words doing an inner merge
            else:
                throw_error = True                                                                                      # else there is an error and assigns True to the throw_error variable
        if throw_error:                                                                                                 # if the throw_error is True
            cossim_10k = 0                                                                                              # assigns cossim_10k to 0
        else:
            cossim_10k = np.dot(word_df.iloc[:,0], word_df.iloc[:,1]) / (np.linalg.norm(word_df.iloc[:,0]) * np.linalg.norm(word_df.iloc[:,1]))
        cossims.append(cossim_10k)                                                                                      # appends cossim_10k to cossims
    else:
        cossims.append(0)                                                                                               # appends 0 to cossims

    '''
    10-Q part
    ---
    loops over first 3 10-Qs
        * assuming these are the same fiscal year and second 3 are different year
    sotres bag of words as 1 df for each paired statement
    computes cosine sims
    '''
    if os.path.exists(path_10q) and (len(os.listdir(path_10q)) >= 6):                                                   # if the path_10q exists and there are 6 or more folders within the path
        filing_list_10q = os.listdir(path_10q)                                                                          # sets filing_list_10q equal to the directory of within path_10q
        counter = 0                                                                                                     # initializes counter to 0
        for filing in filing_list_10q[:3]:                                                                              # for loop that runs through the first 3 in filing_list_10q                             
            current_filing = path_10q + '\\' + filing                                                                   # sets current_filing to the path_10q and the filing from the for loop
            if os.path.exists(current_filing + '\\bag_o_words.csv') and os.path.exists(path_10q + '\\' + filing_list_10q[3+counter] + '\\bag_o_words.csv'):
                word_df = pd.read_csv(path_10q + '\\' + filing + '\\bag_o_words.csv', index_col='Unnamed: 0')           # sets the word_df to the bag of words
                word_df = pd.merge(
                    left = word_df,
                    right = pd.read_csv(path_10q + '\\' + filing_list_10q[3+counter] + '\\bag_o_words.csv', index_col='Unnamed: 0'),
                    how = 'inner',
                    left_index = True,
                    right_index = True)                                                                                 # does a merge with the current version of word_df and then merges with the next bag of words by doing an inner merge                                                         
                cossim_10q = np.dot(word_df.iloc[:,0], word_df.iloc[:,1]) / (np.linalg.norm(word_df.iloc[:,0]) * np.linalg.norm(word_df.iloc[:,1]))
                cossims.append(cossim_10q)                                                                              # appends cossim_10q to cossims
            else:
                for i in range(3):                                                                                      # for loop that runs for range 3
                    cossims.append(0)                                                                                   # appends 0 to cossim
    else:
        for i in range(3):                                                                                              # for loop that runs for range 3
            cossims.append(0)                                                                                           # appends 0 to cossims
    cossims_all_firms.append(dict(zip(cossim_index, cossims)))                                                          # appends the zip of cossim_index and cossims to cossims_all_firms

'''Dataorganization and reporting'''
cosine_df_2020 = pd.DataFrame(cossims_all_firms, index=tickers_2020)                                                    # sets cosine_df_2020 equal to the dataframe of cossims_all_firms with the index of each ticker from 2020
```

## Returns Data


```python
'''
get returns data
'''
first_df = pdr.DataReader(tickers_2010, data_source = 'yahoo', start = '2008-12-21', end = '2012-01-01')                # first dataframe that stores the returns for each firm from 2009-2011
second_df = pdr.DataReader(tickers_2020, data_source = 'yahoo', start = '2019-06-30', end = '2022-06-30')               # second dataframe that stores the returns for each firm from 2020-2021
```

## Combing Cosine Similartiies and Returns


```python
'''
merge cosine similarities and returns into one data frame
---
Takes returns data for each respective quarter (on a one month lag to match reporting)
merges with cosine similarity calculated previously
'''
first_dates = ['01-29-2010','04-30-2010','07-30-2010','10-29-2010','01-28-2011']                                    # initializes the first dates array         
first_returns = first_df['Adj Close'].loc[pd.to_datetime(pd.Series(first_dates))].pct_change().iloc[1:,:]           # gets the first returns during the first dates above
first_returns = first_returns.transpose()                                                                           # transposes the first returns
first_returns.columns = ['q4_ret','q1_ret','q2_ret','q3_ret']                                                       # creates four different columns for each different returns
cosine_and_returns_df_2010 = pd.merge(
    left=cosine_df_2010[~(cosine_df_2010 == 0).all(axis=1)],
    right=first_returns.dropna(),
    how='inner',
    left_index=True,
    right_index=True)                                                                                               # creates cosine_and_returns_df_2010 to store the merge of the cosine_df_2010 and the new returns for the above dates based on an inner merge

second_dates = ['01-29-2020','04-30-2020','07-30-2020','10-29-2020','01-28-2021']                                   # initializes the second dates array
second_returns = second_df['Adj Close'].loc[pd.to_datetime(pd.Series(second_dates))].pct_change().iloc[1:,:]        # gets the second returns during the second dates
second_returns = second_returns.transpose()                                                                         # transposes the second returns
second_returns.columns = ['q4_ret','q1_ret','q2_ret','q3_ret']                                                      # creates four different columns for each
cosine_and_returns_df_2020 = pd.merge(                      
    left=cosine_df_2020[~(cosine_df_2020 == 0).all(axis=1)],
    right=second_returns.dropna(),
    how='inner',
    left_index=True,
    right_index=True)                                                                                               # creates cosine_and_returns_df_2020 to store the merge of the cosine_df_2020 and the new returns for the above second dates based on an inner merge
```


```python
cosine_and_returns_df_2010.head()                                                                                   # testing of the head of cosine_and_returns_df_2010
```


```python
cosine_and_returns_df_2020.head()                                                                                   # testing of the head of cosine_and_returns_df_2020
```

## Data To CSV Files


```python
cosine_and_returns_df_2010.to_csv('2010_dataset.csv')                                                               # puts the 2010 data in a csv file in the repo
cosine_and_returns_df_2020.to_csv('2020_dataset.csv')                                                               # puts the 2020 data in a csv file in the repo
```
