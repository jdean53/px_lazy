# Welcome to our [team project website!](https://jdean53.github.io/px_lazy/)

This is a website to showcase our final project for FIN 377 - Data Science for Finance course at Lehigh University.

This is not intended as investment advice.

<br>

## Table of Contents
1. [Introduction](#introduction)
2. [Methodology](#meth)
    1. [Data Grab](#subsec2-1)
    2. [Data Clean](#subsec2-2)
    3. [Data Analyze](#subsec2-3)
3. [Conclusion](#conclusion)
4. [About the Team](#about)

<br>

## Introduction  <a name="introduction"></a>

This [project](notebooks/introduction.md) is inspired by the research paper titled [*Lazy Prices*](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=1658471), written by Lauren Cohen, Christopher Malloy, and Quoc Nguyen. The paper analyzes S&P 500 companies' quarterly and annual SEC filings between 1995-2014 to determine significant changes in language and structure over time. They found that changes to the reports predict a pessimistic future for the company in areas like earnings, profitability, and even future firm-level bankruptcies. On the other hand, firms that do not make significant changes to their quarterly and annual reports experience positive returns over time.

The main goal of our project is to replicate the work in *Lazy Prices.* This will lead us to generate sell or buy signals depending on the documents' differences or lack thereof.  The original research focuses on the Management's Discussion and Analysis (MD&A) and Risk Factors sections, which we are also investigating. Our objective is to prove that greater magnitude of changes will lead to significant underperfomance.

<br>

## Methodology <a name="meth"></a>

There are 3 main parts of our project: data grab, data clean, and data analyze.

### [Data Grab](notebooks/data_grab.md) <a name="subsec2-1"></a>
In data grab, we use sec_edgar_downloader to load the S&P 500 companies' 10k's and 10q's in 2009-2010 and 2020. For each SEC document, we create a csv file that counts the number of times each word is used, excluding the most basic "filler words", which are 'and', 'the', 'of', 'to', 'in', 'a', 'for', 'on', 'or', 'as', 'is', and 'this'. 

### [Data Clean](notebooks/data_clean.md) <a name="subsec2-2"></a>
In data clean, in order to create a [cosine similarity](notebooks/cosine_sim.pdf) between 2010 and 2020 for each ticker, we create vectors which are also numpy arrays. We then convert these arrays from multidimensional to two-dimensional in order to plot each vector on a graph. After that, we are able to measure the cosine similarity by passing both vectors, and the result will be a value between [0,1].

### [Data Analyze](notebooks/data_analyze.md) <a name="subsec2-3"></a>
In data analyze, all of the data from data clean is pulled and aggregated to an individual dataframe stored in quintiles. The cosine similarities are compared to the companies' respective stock returns from Yahoo Finance to determine whether we should short "changers" or buy "non-changers".

<br>

## Conclusion  <a name="conclusion"></a>
We found that a long-short portfolio constructed using this strategy did generally generate returns of 4.92% per annum. That said, we did not observe significant market outperformance for the periods we analyzed. We list the returns we generated and market returns for the period in the table below  
*Note*: These are listed as quarterly returns and are not an annualized figure  

| Quarter | Return | Market Return |  
| -- | -- | -- |  
| Q1 2010 |  4.00% |  11.09% |  
| Q2 2010 |  2.42% |  -6.75% |  
| Q3 2010 | -1.70% |   8.03% |  
| Q4 2010 | -2.63% |   8.36% |  
| Q1 2020 |  6.48% | -10.54% |  
| Q2 2020 |  0.63$ |  12.02% |  
| Q3 2020 | -4.46% |   2.27% |  
| Q4 2020 |  4.92% |  14.93% |  

To conclude, this is a valid strategy, but back end data leakage is likely leading to weak results in our analysis. Given more time, we would implement more functionality to read filing meta data, which would allow specific names to be tied to returns starting on the day after the filing date. Additionally, there would be more put into cleaning through all the filings downloaded in the downloader file.

<br>

## About the Team <a name="about"></a>

### [Jack Dean](https://www.linkedin.com/in/jack-dean-445336150/)

Jack is completing his Financial Engineering Master's degree at Lehigh University in May 2022 and will be joining Millennium as a Fixed Income Quantitative Analyst in New York City in June. 

<img src="pics/Jack.jpg" alt="Jack Dean" width="300"/>
<br><br><br>

### [Noah Sutherland](https://www.linkedin.com/in/noahsutherland/)

Noah is in his senior year of undergrad at Lehigh University. Following his graduation in May 2022, he will be attending UCLA's Master's of Financial Engineering program.

<img src="pics/Noah.jpg" alt="Noah Sutherland" width="300"/>
<br><br><br>

### [Matt Morana](https://www.linkedin.com/in/matthewmorana/_)

Matt is finishing up his Master's of Business Administration degree with a concentration in Business Analytics at Lehigh University in August 2022. Following his program, he will be joining PwC as a mergers and acquisitions consultant in New York City.

<img src="pics/Matt.jpg" alt="Matthew Morana" width="300"/>
<br><br><br>

### [Sherzod Esanov](https://www.linkedin.com/in/sherzodesanov/)

Sherzod is completing his Master's of Business Administration degree with a concentration in Business Analytics at Lehigh University in August 2022. He has experience in Uzbekistan as a Financial Analyst at VEON.

<img src="pics/Sher1.jpg" alt="Noah Sutherland" width="300"/>
<br>


## More 

To view the GitHub repo for this website, click [here](https://github.com/jdean53/px_lazy).
