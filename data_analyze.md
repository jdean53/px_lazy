# Data Analyze

The purpose of this notebook is to analyze our aggregated data downloaded and found in data_grab and cleaned in data_clean.

## Imports


```python
'''
Necessary packages
'''
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import scipy.stats as stats
import yfinance as yf
import pandas_datareader as pdr
```

## Data Import and Aggregation


```python
'''
Data Import and Aggregation
---
pulls data created from clean file
Aggregates data to individual dataframe stored quarter by quarte (dfs list)
NOTE: this removes cosine values of 0, which implies the two documents absolutely different
    (i.e. one does not exist)
'''
data_2010 = pd.read_csv('2010_dataset.csv',index_col='Unnamed: 0')                                                      # reads 2010 data
data_2020 = pd.read_csv('2020_dataset.csv',index_col='Unnamed: 0')                                                      # reads 2020 data

# data index :       q1        q2         a3       q4
cosine_index = [   '10k',     'q1',     'q2',     'q3']
return_index = ['q1_ret', 'q2_ret', 'q3_ret', 'q4_ret']

dfs = []                                                                                                                # for list of data frames
for df in [data_2010, data_2020]:                                                                                       # appends 2010, then 2020
    for i in range(4):                                                                                                  # over all 3 quarters
        dfs.append(df[[cosine_index[i],return_index[i]]].loc[df[cosine_index[i]]>0])                                    # only pulls nonzero cosine values
```

## Cosine Similarity Sorting, Average Returns, Variance


```python
ports = []                                                                                                              # initialize the ports array
avg_ret = []                                                                                                            # initialize the average return array
var_ret = []                                                                                                            # initialize the variance return array

for i in range(len(dfs)):                                                                                               # for loop that runs through the length of the dfs created in the above cell
    if i <= 3:                                                                                                          # if the incrementing of i is less than or equal to 3
        idx = 0                                                                                                         # sets the variable idx equal to 0
    else:
        idx = 4                                                                                                         # otherwise sets the variable idx equal to 4
    df = dfs[i]                                                                                                         # sets a new df equal to the dfs at index i
    df_bins = pd.qcut(df.iloc[:,0],q=5,retbins=True)[1]                                                                 # sets a new df_bins variable equal to the quintile at the first index position
    df['long_short'] = df[cosine_index[i - idx]].apply(lambda x: 1 if x >= df_bins[-2] else 0)                          # creates a column of long_short that stores a 1 if the ticker is in the top quintile and otherwise stores 0
    port = df[df['long_short'] == 1]                                                                                    # creates a new portfolio spot that includes the tickers with a 1 signaling them being in the top quintile
    df['long_short'] = df[cosine_index[i - idx]].apply(lambda x: -1 if x <= df_bins[1] else 0)                          # edits the column of long_short so that it stores a -1 if the ticker is in the bottom quintile and otherwise stores 0
    port = pd.concat([port, df[df['long_short'] == -1]], ignore_index=False)                                            # concatenates the current portfolio array with the tickers that have a -1
    ports.append(port)                                                                                                  # appends the port variable to the ports array created at the top of the cell
    g_obj = port.groupby('long_short').mean()                                                                           # creates a new variable g_obj that stores the grouped by column of long_short and takes the mean of this
    avg_ret.append(g_obj.iloc[1,1] - g_obj.iloc[0,1])                                                                   # appends the returns for the top quintile and the bottom quintile to the average return array
    var_ret.append(np.var(np.array(port.iloc[:,1]*port.iloc[:,2])))                                                     # appends the variance for the top quintile and the bottom quintile to the variance return array
```

## SPY Returns


```python
first_dates = ['01-29-2010','04-30-2010','07-30-2010','10-29-2010','01-28-2011']                                        # initializes the array of first dates to get the SPY returns during the same dates
second_dates = ['01-29-2020','04-30-2020','07-30-2020','10-29-2020','01-28-2021']                                       # initializes the array of second dates to get the SPY returns during the same dates

rets = []                                                                                                               # initializes the returns array
vars = []                                                                                                               # initializes the variance array
for i in range(len(first_dates)-1):                                                                                     # for loop that runs for the length of the first_dates minus 1
    spy = pdr.DataReader('SPY', data_source = 'yahoo', start = first_dates[i], end = first_dates[i+1])['Adj Close']     # sets the new variable spy equal to the returns of the spy during the first dates
    rets.append(spy[-1]/spy[0]-1)                                                                                       # appends the returns array with the spy at index -1 divided by the spy at index 0 minus 1

for i in range(len(second_dates)-1):                                                                                    # for loop that runs for the length of the second dates minus 1
    spy = pdr.DataReader('SPY', data_source = 'yahoo', start = second_dates[i], end = second_dates[i+1])['Adj Close']   # sets the variable spy equal to the returns of the spy during the second dates
    rets.append(spy[-1]/spy[0]-1)                                                                                       # appends the returns array with the spy at index -1 divided by the spy at index 0 minus 1
```

## Alpha and Dataframe Creation


```python
bnchmk_df = pd.DataFrame([avg_ret, var_ret, rets], index=['port_ret','port_var','mkt']).transpose()                     # sets the new benchmark dataframe equal to the average returns, variance of returns, returns and the indexes as the portfolio return, portfolio variance, and market
bnchmk_df['alpha'] = bnchmk_df['port_ret'] - bnchmk_df['mkt']                                                           # sets a new column in benchmark dataframe equal to alpha and calculates this based off of the portfolio return minus the market
# bnchmk_df['sharpe'] = bnchmk_df['port_ret'] / np.sqrt(bnchmk_df['port_var'])                                          (attempt at Sharpe ratio to try and further describe our findings)
```


```python
bnchmk_df                                                                                                               # prints the benchmark dataframe
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>port_ret</th>
      <th>port_var</th>
      <th>mkt</th>
      <th>alpha</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.040033</td>
      <td>0.016541</td>
      <td>0.110897</td>
      <td>-0.070864</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.024167</td>
      <td>0.022347</td>
      <td>-0.067464</td>
      <td>0.091631</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-0.017025</td>
      <td>0.032635</td>
      <td>0.080297</td>
      <td>-0.097322</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-0.026291</td>
      <td>0.039989</td>
      <td>0.083566</td>
      <td>-0.109857</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.064801</td>
      <td>0.035176</td>
      <td>-0.105419</td>
      <td>0.170220</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.006286</td>
      <td>0.021847</td>
      <td>0.120165</td>
      <td>-0.113879</td>
    </tr>
    <tr>
      <th>6</th>
      <td>-0.044570</td>
      <td>0.080878</td>
      <td>0.022660</td>
      <td>-0.067229</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.049167</td>
      <td>0.066373</td>
      <td>0.149281</td>
      <td>-0.100114</td>
    </tr>
  </tbody>
</table>
</div>




```python
bnchmk_df.mean()                                                                                                        # prints the benchmark dataframe mean
```




    port_ret    0.012071
    port_var    0.039473
    mkt         0.049248
    alpha      -0.037177
    dtype: float64


