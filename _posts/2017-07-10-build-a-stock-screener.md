---
layout:     post
title:      Building a Stock Picker
date:       2017-07-10 09:32:19
summary:    I've taken several online courses in machine learning, and I was interested in seeing if I could build a stock picker to choose stocks that could outperform the S&P 500.
categories: 
---

I've taken several online courses in machine learning, including:
* [Machine Learning](https://www.coursera.org/learn/machine-learning) on Coursera 
* [Big Data Analysis with Apache Spark](https://www.edx.org/course/big-data-analysis-apache-spark-uc-berkeleyx-cs110x) on edX 
* [Distributed Machine Learning with Apache Spark](https://www.edx.org/course/distributed-machine-learning-apache-uc-berkeleyx-cs120x) on edX 

The courses were fantastic and helped me really understand the power of machine learning for pattern recognition and prediction.

At the same time I was taking the courses, I was reading the book [_Inside the Investments of Warren Buffett: Twenty Cases_](https://www.amazon.com/Inside-Investments-Warren-Buffett-Publishing/dp/0231164629/).  In the book, the author discusses many company fundamental indicators that Buffet likely evaluated when deciding to make an investment in a company.  Some of these fundamentals include:

* Enterprise value to EBITDA
* Return on invested capital
* Price to earnings
* Free cash flow
* Price to book value
* Book value per share
* Revenue per share

Given the availability of a large amount of stock fundamental and pricing data, I thought it'd be interesting to see if a machine learning program could be trained to pick stocks that would outperform the S&P 500 over a certain time period (3 months, 6 months, 12 months).  This project would also be a great opportunity to reinforce some of the things I learned in machine learning courses I took.