---
layout: project_single
title:  "Analyzing Tech Stocks of the Big 5 Tech Companies"
slug: "Analyzing-Tech-Stocks"
---

**This is the fifth project I completed as part of Metis Data Science Bootcamp.**

**Disclaimer: This is NOT investment advice. Stock markets are volatile and it's hard to predict them.**

## Introduction

Back in 2015, my friends and I worked at Microsoft. We were awarded shares as part of our sign on bonus. I held onto my shares while my friends sold their stock to invest in Apple. Below is a stock price comparison of the two companies and how much the price jumped by in the 4 year span (2015-2019).

![image]({{site.url}}/images/projects/analyzing-stocks/Microsoft_Apple_Comparison.png)

Microsoft jumped by 230% while Apple jumped by 91%. By remaining loyal to the company, I made more of a profit than my friends did. 

But if you told me back in 2015 that Microsoft would be more valuable than Apple, I would have laughed at you. Apple was, and still is, a trendy company for millenials. People still buy the iPhones and many developers prefer Macbooks over PCs. It was interesting to see how a company that was an afterthought still rise up. 

For this project, I sought to do a time series analysis on stocks of the big 5 tech companies: Microsoft, Amazon, Facebook, Apple, and Google. I want to predict which company will be a valuable investment in the long term, while also predicting the stock price in the short term. 

## Project Overview

This project is broken down into the following parts. 

* Where do I get the stock data?
* How do I compare the true value of the companies for long term investment?
* How do I forecast the future based on historical stock data?

## Data Extraction

I used TIINGO to fetch stock data from Jan 2015 to September 2019. However, I want to fetch the latest stock prices. The closing stock prices adjust every day, and I wanted to schedule a job to fetch that data and store it in an Amazon S3 bucket. I used Airflow to do so, and more details on how to instantiate this can be found in this [post]({% link _posts/2019-09-20-airflow.md %}). Below is the ETL diagram for getting stock data.

![image]({{site.url}}/images/projects/analyzing-stocks/ETL.png)


