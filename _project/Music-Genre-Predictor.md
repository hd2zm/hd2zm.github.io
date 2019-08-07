---
layout: project_single
title:  "Multiclass Classification of Music Primary Genres"
slug: "Music-Genre-Predictor"
---

**This is the third project I completed as part of Metis Data Science Bootcamp.**

## Introduction

I wanted to find some way to tie in machine learning with music/audio files. It began when I was listening to a country station and heard Old Town Road-Lil Nas X for the first time. 

![image]({{site.url}}/images/projects/genre-predictor/Old_Town_Road.png)

My first thought was "How is this even a country rock song? This should be a hip hop single." 

As it turns out, Old Town Road has a hilarious backstory. It was originally released as a hip hop single, but Lil Nas X wanted to hit Billboard's Country charts. So he remixed his single with Billy Ray Cyrus so it can be classified as country/country rock. 

A lot of songs overlap with different genres. Linkin Park's In the End was branded as alternative rock. It had the basis of a rock single with hip-hop elements. David Bowie's Heroes was branded art rock because of its heavy use of synthesizers/electronic sound mixed with rock. 

With so many songs overlapping various genres, I wondered if there is a way to predict the primary genre of a song. Can I predict whether the primary genre is Rock, Hip-Hop, or Electronic? 

This was more of a fun project and has little business applications. But what drove me to this project was the usage of music data from Spotify/Echonest. In addition, I wanted to learn more about creating a multi-classifier model, which could serve as a baseline for predicting non-binary categories in business. 

## Project Overview

This project is broken down into the following

* Where do I get data for audio tracks?
* What trends can I spot? What features can I use for my model?
* What can I do to improve my model (feature engineering, resampling, ensembling, grid search cross validation)? 
* How can I make an interactive visualization of my model?

## Data Extraction

To collect my data, I downloaded around 10,000 songs that use Echonest data. I got these songs from two data sets: non-mainstream and popular/mainstream. Non-mainstream, I used [Free Music Archive](http://freemusicarchive.org/) that already contained Echonest data. For popular, I web scrapped song titles and artists from [Billboard's Year End Charts](https://www.billboard.com/charts/year-end) and used [Spotify](https://www.spotify.com/us/) to fetch their respective Echonest data. 

## Exploratory Data Analysis

I got the average of tempo for each genre and plotted them in a bar chart. 

![image]({{site.url}}/images/projects/genre-predictor/Tempo_Bar_Chart.png)

I got the average of danceability, energy, speechiness, acousticness, instrumentalness, liveness, and valence per genre and plotted them in a radar plot. 

![image]({{site.url}}/images/projects/genre-predictor/Radar_Plot.png)

There are key takeaways from this

* Tempo, liveness, and energy doesn't vary a lot per genre. They may be unimportant features.
* A rock song is distinguished by how high its acoustic value is
* An electronic song is distinguished by how high its instrumentalness value is
* A hip-hop song is distinguished by how high its speechiness, danceability, and valence values are. 
* Hip-hop songs don't overlap a ton with other genres, so models may have an easier time predicting these songs.
* There's a lot of overlap between Electronic and Rock songs, so models may have trouble distinguishing between the two. 

I was surprised to see Electornic and Rock being hard to distinguish. But with synthesizers being more involved in art rock and electronic rock, it makes sense that there is more overlap between those two genres. 

## Features

The final features I used were all from Echonest. 

* danceability
* energy
* speechiness (# of words)
* acousticness
* instrumentalness
* liveness (is track played live)
* valence (positive elements of the song)
* tempo

I did mention that tempo, liveness, and energy may not be important. But I wanted to see how my models performed with those included. 

## Classification Models

I built, trained, tested, and evaluated 9 different models. They included

1. K-Nearest Neighbor Baseline
2. Logistic Multinomial Regression Baseline
3. Logistic Multinomial Regression Optimized with Grid Search
4. One vs Rest Support Vector Classifier Baseline
5. One vs Rest Support Vector Classifier Optimized with Grid Search
6. Naive Bayes Classifier
7. Decision Tree Baseline
8. Random Forest Baseline
9. Random Forest Optimized with Grid Search

**Note:** Logistic Regression and Support Vector Classifier are not inherently multiclass. So I had to add a One vs Rest classifier for SVC and add a multinomial property for Logistic Regression.

There was some class imbalance in my data (Rock songs had 4000 +songs, while Hip-Hop and Electornic songs had 2500+ songs each). Because of this, accuracy wouldn't be a good metric because it will be biased towards Rock. Resampling had a negative impact on my metrics, and I chose to ignore it. I chose weighted F1 as a metric to optimize because of class imbalance, and I also wanted a good balance bewteen precision and recall. 

I also sought to optimize my ROC-AUC score so I can compare which model stood out the most. Because this is multi-classificaiton, I had to create a separate ROC-AUC curve for each genre per model.

Ultimately, **Random Forest Optimized with Grid Search** won in both F1 and ROC-AUC. I got a train F1 score of 0.85 and a test F1 score of 0.76 with threshold of 0.5. I also got  an AUC of 0.84 for Electronic, 0.95 for Hip-Hop, and 0.9 for Rock. So ensembling didn't make sense when this model fit all my criteria. 

Below is the confusion matrix for my model.

![image]({{site.url}}/images/projects/genre-predictor/Random_Forest_Confusion_Matrix.png)

What's interesting is the high value when this model predicted Rock, but was actually Electronic. Going back to the radar plot, I suspected that it would have some difficulty distinguishing the two genres. There are extremely low false positives for Hip-Hop, showing that my model can distinguish this genre extremely well. 

## Interactive Visualization

After creating my model, my next step was deciding how a user can interact with it. I create a Dockerized Flask web application hosted on AWS. Below is the design of the app. 

![image]({{site.url}}/images/projects/genre-predictor/Web_App_Design.png)

The application downloads the model from S3 (if it doesn't have it locally) and prompts the user for a song and artist. It then checks if that track exists in Spotify. If it does, then it fetches the Echonest data from Spotify, runs it through the model, and outputs the prediction. Else, it returns that it can't the track.

So what does my model think of Old Town Road? 

![image]({{site.url}}/images/projects/genre-predictor/Web_App_Old_Town_Road.png)

As I believed, this model says this song is a hip-hop single. 