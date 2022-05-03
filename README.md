# Welcome to our [team project website!](https://jdean53.github.io/px_lazy/)

This is a website to showcase our final project for FIN 377 - Data Science for Finance course at Lehigh University.

To see the complete analysis file(s) click [here](https://github.com/julioveracruz/testwebsite/blob/main/notebooks/example.ipynb).

## Table of contents
1. [Introduction](#introduction)
2. [Methodology](#meth)
3. [Data Preparation](#section2)
    1. [Data Grab](#subsec2-1)
    2. [Data Clean](#subsec2-2)
4. [Analysis Section](#section3)
5. [Summary](#summary)

## Introduction  <a name="introduction"></a>

This project is inspired by the research paper titled [*Lazy Prices*] (https://papers.ssrn.com/sol3/papers.cfm?abstract_id=1658471), written by Lauren Cohen, Christopher Malloy, and Quoc Nguyen. The paper analyzes S&P 500 companies' quarterly and annual SEC filings between 1995-2014 to determine significant changes in language and structure over time. They found that changes to the reports predict a pessimistic future for the company in areas like earnings, profitability, and even future firm-level bankruptcies. On the other hand, firms that do not make significant changes to their quarterly and annual reports experience positive returns over time.

The main goal of our project is to replicate the work in *Lazy Prices.* This will lead us to generate sell or buy signals depending on the documents' differences or lack thereof.  The original research focuses on the Management's Discussion and Analysis (MD&A) and Risk Factors sections, which we are also investigating. Our objective is to prove that greater magnitude of changes will lead to significant underperfomance.

## Methodology <a name="meth"></a>

Here is some code that we used to develop our analysis. Blah Blah. [More details are provided in the Appendix](page2).
 
Note that for the purposes of the website, you have to copy this code into the markdown file and  
put the code inside trip backticks with the keyword `python`.

```python
import seaborn as sns 
iris = sns.load_dataset('iris') 

print(iris.head(),  '\n---')
print(iris.tail(),  '\n---')
print(iris.columns, '\n---')
print("The shape is: ",iris.shape, '\n---')
print("Info:",iris.info(), '\n---') # memory usage, name, dtype, and # of non-null obs (--> # of missing obs) per variable
print(iris.describe(), '\n---') # summary stats, and you can customize the list!
print(iris['species'].value_counts()[:10], '\n---')
print(iris['species'].nunique(), '\n---')
```

Notice that the output does NOT show! **You have to copy in figures and tables from the notebooks.**

## Section <a name="section2"></a>
Blah blah

### Subsection 1 <a name="subsec2-1"></a>
This is a subsection, formatted in heading 3 style

### Subsection 2 <a name="subsec2-2"></a>
This is a subsection, formatted in heading 3 style

## Analysis Section <a name="section3"></a>

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



## About the team

<img src="pics/julio.jpg" alt="julio" width="300"/>
<br>
Julio is a senior at Lehigh studying finance.
<br><br><br>
<img src="pics/don2.jpg" alt="don" width="300"/>
<br>
Don is an assistant professor at Lehigh.


## More 

To view the GitHub repo for this website, click [here](https://github.com/donbowen/teamproject).
