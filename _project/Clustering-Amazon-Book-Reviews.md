---
layout: project_single
title:  "Clustering Amazon Book Reviews from 2005"
slug: "Clustering-Amazon-Book-Reviews"
---

**This is the fourth project I completed as part of Metis Data Science Bootcamp.**


## Introduction

I wanted to use unsupervised learning to spot trends in book reviews from 2005. Such trends could include detecting low quality reviews. Low quality reviews could inflate/deflate product value. These reviews could focus more on the scene of where the book was read or lack of descriptive content of the book. This could be useful for business applications that thrive on user reviews and need to filter out which ones are useless. 

## Project Overview

This project is broken down into the following

* Where do I get Amazon book review data
* What NLP tools can I use to extract meaningful content/topics?
* What trends can I spot? Can I cluster to find patterns?

## Data Extraction

I uploaded Amazon Customer Book Reviews dataset onto MongoDB, which was hosted on an AWS EC2 instance. there were 3 million book reviews, so I decided to focus on 15,000 from the year 2005. 

# Text Cleaning

I used SpaCy for text cleaning, lemmatization, and removal of stop words. I kept nouns, verbs, adjectives, and adverbs (low quality reviews have more verbs than nouns). I removed pronouns and proper nouns, with the exception of organization, product, and country. I also removed synonyms and components of book (novel, plot, chapter) and unnecessary characters (backslashes, &quote, <br>)

# Topic Modeling

After cleaning all my reviews, I turned to topic modeling to generate the necessary topics. I used NMF to gather my topics. Altogether, I had 30 different topics. Below are some of such topics. 

![image]({{site.url}}/images/projects/clustering-amazon-book-reviews/Topics_0_to_7.png)

Immediately, we can tell the topics in the given order

* Religion
* Love
* Child/Parenting
* Cookbook
* Recommend
* Life
* Description of Content
* American History

From all the 30 topics, we can classify them into two different categories. Content Based Topics and Assessment Based Topics. 

## Content Based Topics

Content based topics follow the description of content of the book. They can include the details of the book (informational, easy to follow, good storytelling). Or they can talk about the genre of the book (American History, Young Adult, Religion). 

## Assessment Based Topics

Assessment based topics talk about how the reviewer felt about the book. Did the reviewer love, hate, or enjoy the book? Additionally, the reviewer could talk abot scene setting/referrals ("My son can't put it down", "My wife loves it"). These add very little to the review and could potentially be indicators of low quality reviews. 

# Dimensionality Reduction

I had 30 topics, and length of each review. I wanted to reduce those components to 2 using PCA. 

# Clustering

After dimensionality reduction, I then used the elbow method to find the optimal clusters for the 2 dimensional space. 

![image]({{site.url}}/images/projects/clustering-amazon-book-reviews/Inertia_Curve.png)

The elbow method shows that 3 is the optimal cluster number. So I ran KMeans with k=3. Below is the plot in 2 dimensional space. 

![image]({{site.url}}/images/projects/clustering-amazon-book-reviews/Book_Clusters.png)

# Analysis

From the K means cluster plot, I can make judgements on which reviews are lengthy (2000 words and above). But they're all in one big blob, and the clusters aren't easily separable. Unfortunately, I couldn't draw any meaningful conclusions from book data. 