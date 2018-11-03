---
title: News Recommendation using Wed Mining
date: 2018-03-27 15:02:08
type: "categories"
comments: true
categories:
- Portfolios
tags:
- Python
- ReactJS
- Node.js
- MongoDB
- RPC
- RabbitMQ
- TensorFlow
- Shell
---


## I. Introduction
An internet-based news aggregator, providing hot news scraping on popular news sources, with recommendation feature based on users' preference with the help of Machine Learning.

**#Github**: https://github.com/caomingkai/News_Recommendation_System  

**Pull it and run it with Shell script!**

- **Firstly, run `./launcher.sh`:**
    + run redis
    + run mongoDB locally
    + start recommendation service(python)
    + start backend service(phython)
    + start web-server service(Node.js + ReactJS)
- **Secondly, run `./news_pipeline_launcher.sh`:**
    + run redis
    + run mongoDB locally
    + install python requirements
    + start news_topic_modeling_service (python Machine Learning)
    + start news collecting service(data pipeline + web scraping)

## II. Tech stack:
- __Front end:  React, Express, Node.js, OAuth__
    + Built a responsive single-page web application for users to browse news (React, Node.js, RPC, SOA, JWT)


- __Back end: Python RPC, MongoDB, Redis, RabbitMQ__
    + Service Oriented, multiple backends serving via JSON RPC
    + Implemented a data pipeline which monitors, scrapes and deduplicates news


- __News recommendation system: Tensorflow, DNN, NLP__
    + Designed and built an offline training pipeline for news topic modeling
    + Deployed an online classifying service for news topic modeling using trained model


- __News topic classifying system: TF-IDF, NLP, RabbitMQ__
	+ Implemented a click event log processor which collects users' click logs, updated a news model for each user


**Chart1: Login Page with Authentication**
{% asset_img loginPage.png This is an image %}

**Chart2: News feed page**
{% asset_img newsPage.png This is an image %}



## III. System structure:
1. **Front end tier**: React & Node.js
2. **Back end tier**: providing RPC API for communication among different tiers
3. **News recommendation system**: time decay algorithm
4. **News topic classifying system**: Tensorflow with 2-layer CNN model for classification
5. **data pipeline**: get news sources
	- News monitor: gets news from News.API
	- News scraper: web scraper
	- News deduper: news TF-IDF deduplication



**Chart 3: System with Machine Learning module**
![chart](https://github.com/caomingkai/News_Recommendation_System/raw/master/with%20ML%20topic%20classification1.png)

**Chart 4: System with Recommendation module**
![chart](https://github.com/caomingkai/News_Recommendation_System/raw/master/with%20Recommendation%20module.png)

**Chart 5: Service dependency**
![chart](https://github.com/caomingkai/News_Recommendation_System/raw/master/chart.jpg)
