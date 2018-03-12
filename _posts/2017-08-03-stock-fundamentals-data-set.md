---
layout:     post
title:      Stock Fundamentals Training Data
date:       2017-08-03 09:32:19
summary:    There are many sources of stock pricing and fundamental data out on the Internet.  The challenge is to find sources that are free or affordable. 
categories: machine learning
---

### Data Sources

There are many sources of stock pricing and fundamental data out on the Internet.  The challenge is to find sources that are free or affordable.  Here are the best sources I've found:

* [Quandl - Core US Fundamentals Data](https://www.quandl.com/databases/SF1) ($150/quarter for individuals)
* [Alphavantage](https://www.alphavantage.co/) (Free)
* [Interactive Brokers](https://www.interactivebrokers.com) ($1-$10/month for individuals)
* [Google Finance](http://finance.google.com) (Free)
* [Yahoo! Finance](http://finance.yahoo.com) (Free)

### Database Schema

I ended up using combining pricing and volume data from Yahoo! Finance with the Quandl Core US Fundamentals Data.  I added all this data into a MySQL table.  

Here's my MySQL `indicators` table definition:

```sql
CREATE TABLE `indicators` (
  `stock_id` int(11) NOT NULL,
  `dimension` enum('MRQ','MRT','ARQ','ART') NOT NULL,
  `date` datetime NOT NULL COMMENT 'The date of these metrics [if AR.., this is the fiscal end date]',
  `date_fiscal` datetime NOT NULL COMMENT 'The fiscal end date',
  `assets` decimal(19,4) DEFAULT NULL,
  `bvps` decimal(19,4) DEFAULT NULL,
  `cashneq` decimal(19,4) DEFAULT NULL,
  `current_ratio` decimal(10,4) DEFAULT NULL,
  `de_ratio` decimal(10,4) DEFAULT NULL COMMENT 'de => Debt to equity ratio',
  `debt` decimal(19,4) DEFAULT NULL,
  `div_yield` decimal(10,4) DEFAULT NULL,
  `ex_dividend` decimal(10,4) DEFAULT NULL,
  `ebitda` decimal(19,4) DEFAULT NULL,
  `ebitda_margin` decimal(10,4) DEFAULT NULL,
  `eps` decimal(10,4) DEFAULT NULL,
  `equity` decimal(19,4) DEFAULT NULL,
  `ev` decimal(19,4) DEFAULT NULL COMMENT 'daily: subtract market_cap and add in market_cap for day',
  `ev_ebitda` decimal(10,4) DEFAULT NULL,
  `fcf` decimal(19,4) DEFAULT NULL,
  `fcfps` decimal(19,4) DEFAULT NULL,
  `gross_margin` decimal(10,4) DEFAULT NULL,
  `market_cap` decimal(19,4) DEFAULT NULL,
  `net_margin` decimal(10,4) DEFAULT NULL,
  `pb` decimal(10,4) DEFAULT NULL,
  `pe` decimal(10,4) DEFAULT NULL,
  `price` decimal(10,4) DEFAULT NULL,
  `revenue` decimal(19,4) DEFAULT NULL,
  `roa` decimal(10,4) DEFAULT NULL,
  `roic` decimal(10,4) DEFAULT NULL,
  `shares_wa` decimal(20,2) DEFAULT NULL,
  `sps` decimal(10,4) DEFAULT NULL,
  `working_capital` decimal(19,4) DEFAULT NULL,
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  KEY `stock_id` (`stock_id`),
  KEY `date_key` (`date`)
)
```

Note that I didn't end up using all indicators as features in my training set.  The features I did use will be discussed in my next post. 

### Generating the Training and Test Sets

In a separate table, I also collected the sector and industry for each stock:

```sql
CREATE TABLE `stocks` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `symbol` varchar(32) NOT NULL,
  `instrument` varchar(64) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  `sector` varchar(255) DEFAULT NULL,
  `industry` varchar(255) DEFAULT NULL,
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `symbol_key` (`symbol`)
) ENGINE=InnoDB AUTO_INCREMENT=12142 DEFAULT CHARSET=utf8;

```

Note: The sector and industry can be found using the [screener tool on finviz](https://finviz.com/screener.ashx).

The sector and industry information can be useful for normalizing data.  For example, I think it's best to group stocks by sector and industry before applying a sigmoid normalization function.  The grouping is necessary for comparative purposes - for example, a bank stock will probably have a lower P/E ratio than a technology stock due to differing expectations of growth.

The fundamentals data is generated on a quarterly basis.  Note that companies report their results on different days of the year.  To generate a data set on a particular day, adjust the price-based indicators (P/E, EV/EBITDA, EV, Market Cap, etc) based on the price for that day.

After applying a normalization function and creating a label (beating the S&P 500 by 2 percentage points), export the normalized data as a CSV file for use in a training and test set.