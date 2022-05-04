# Welcome to our [team project website!](https://jdean53.github.io/px_lazy/)

This is a website to showcase our final project for FIN 377 - Data Science for Finance course at Lehigh University.

## Table of Contents
1. [Introduction](#introduction)
2. [Methodology](#meth)
    1. [Data Grab](#subsec2-1)
    2. [Data Clean](#subsec2-2)
4. [Analysis](#section3)
5. [Summary](#summary)

## Introduction  <a name="introduction"></a>

This project is inspired by the research paper titled [*Lazy Prices*](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=1658471), written by Lauren Cohen, Christopher Malloy, and Quoc Nguyen. The paper analyzes S&P 500 companies' quarterly and annual SEC filings between 1995-2014 to determine significant changes in language and structure over time. They found that changes to the reports predict a pessimistic future for the company in areas like earnings, profitability, and even future firm-level bankruptcies. On the other hand, firms that do not make significant changes to their quarterly and annual reports experience positive returns over time.

The main goal of our project is to replicate the work in *Lazy Prices.* This will lead us to generate sell or buy signals depending on the documents' differences or lack thereof.  The original research focuses on the Management's Discussion and Analysis (MD&A) and Risk Factors sections, which we are also investigating. Our objective is to prove that greater magnitude of changes will lead to significant underperfomance.

## Methodology <a name="meth"></a>

There are 3 main parts of our project: data grab, data clean, and data analyze. 

In data grab, we use sec_edgar_downloader to load the S&P 500 companies' 10k's and 10q's in 2009-2010 and 2020. For each SEC document, we create a csv file that counts the number of times each word is used, excluding the most basic "filler words", which are 'and', 'the', 'of', 'to', 'in', 'a', 'for', 'on', 'or', 'as', 'is', and 'this'. 

In data clean, we import the cosine similarity module using `sklearn.metrics.pairwise` package. For each ticker, in order to create a cosine similarity between 2010 and 2020, we create vectors which are also numpy arrays. We then convert these arrays from multidimensional to two-dimensional in order to plot each vector on a graph. After that, we are able to measure the cosine similarity by passing both vectors, and the result will be a value between [0,1].

In data analyze, all of the data from data clean is pulled and aggregated to an individual dataframe stored in quartiles. The cosine similarities are compared to the companies' respective stock returns from Yahoo Finance to determine whether we should short "changers" or buy "non-changers".


Notice that the output does NOT show! **You have to copy in figures and tables from the notebooks.**

## Section <a name="section2"></a>
Blah blah

### Subsection 1 <a name="subsec2-1"></a>
This is a subsection, formatted in heading 3 style

### Subsection 2 <a name="subsec2-2"></a>
This is a subsection, formatted in heading 3 style

## Analysis <a name="section3"></a>

Here are some graphs that we created in our analysis. We saved them to the `pics/` subfolder and include them via the usual markdown syntax for pictures.

![](pics/plot1.png)
<br><br>
Some analysis here
<br><br>
![](pics/plot2.png)
<br><br>
More analysis here.
<br><br>
![](pics/plot3.png)
<br><br>
More analysis.

## Summary <a name="summary"></a>

Blah blah



## About the Team

### [Jack Dean](https://www.linkedin.com/in/jack-dean-445336150/)

Jack is completing his Financial Engineering Master's degree at Lehigh University in May 2022. Following his program, he will be joining Millenium as a Fixed Income Quantitative Analyst in New York City. 

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
