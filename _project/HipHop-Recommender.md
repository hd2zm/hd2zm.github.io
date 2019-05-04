---
layout: project_single
title:  "Building a Content Filtering Recommender of Billboard's Number One Hip Hop Singles From 3 Different Decades"
slug: "HipHop-Recommender"
---

# Introduction

Many us have used recommendation systems. Netflix and Amazon use collaborative filters to recommend movies/content to users. Collaborative filters make recommendations based on how users similar to us have rated content. To determine whether two users are similar, collaborative filters take into account content used by the two users and the ratings for such content. If two users ranked a group of content similarily, then it is safe to assume that they both enjoy similar content. So if Person A and Person B both enjoy movies from Marvel Cinematic Universe, the filter system will assume both people enjoy action and superhero movies. Now if Person A loved Die Hard (and Person B hasn't seen Die Hard), the collaborative filter will take Person A's recommendation and recommend that movie to Person B due to his/her love for action movies. If Person B didn't enjoy Justice League, the filter system will make sure not to recommend that superhero movie to Person A (who hasn't seen that movie).

This project goes into content-based filtering. These filters are independent of user ratings, and focus more on how similar the content is to each other. Using our example above, Justice League is similar to movies from Marvel Cinematic universe (as they are both action and superhero movies). Die Hard is less similar as there is a lack of an superhero. So if Person A were to use a content-based filtering system, he/she would be recommended Justice League ahead of Die Hard. 

# Project Overview

The goal of this project is to build a content-based music recommendation system. To keep things small, we'll focus on #1 hit hip-hop singles from 3 different eras: 1980s-2000s, 2000s-2010s, and 2010s-present. 

The project is split into two parts: song collection and recommendation analysis. 

## Song Collection 

I've detailed how I collected songs in a different [post]({% link _posts/2019-04-28-rap-genius-hip-hop.md %}). 

## Recommendation System

From my song collection, I've loaded in the data shown below. 

![image]({{site.url}}/images/projects/genius-parser/GeniusParserResults.png)

I'll proceed with 4 steps.

1. Convert lyrics to keywords
2. Clean the data
3. Get Cosine Similarity Matrix
4. Get recommended songs

### 1. Convert Lyrics to Keywords

There are many NLP packages to get keywords. For this example, I used Rake. By default, it uses english stop words and discards all puntuation characters. 

I've also done some manipulation of keywords. I've included the artist name in lowercase and joined their first and last names into one word. Why? Take Adam Lavine and Adam Lambart. Both are two different artists, but they share a similar first name. In later steps, I'm applying cosine similarity to create the rankings of the songs and I don't want to have two songs be similar just because the artist share the same first name. Hip Hop artists rarely have the same stage name (there isn't another Jay-Z or Nas), but I'm doing this just in case. 

I credit this approach to Emma Grimaldi. Check out her movie recommendation system [here](https://towardsdatascience.com/how-to-build-from-scratch-a-content-based-movie-recommender-with-natural-language-processing-25ad400eb243)

```python
def convert_lyrics_to_keywords(lyrics_df):
     # assigning the key words to the new column
    lyrics_df['Key_Words'] = ""
      
    for index, row in lyrics_df.iterrows():
        lyric = row['Lyrics']

        r = Rake()

        # extracting the words by passing the text
        r.extract_keywords_from_text(lyric)

        # getting the dictionary whith key words and their scores
        key_words_dict_scores = r.get_word_degrees()
    
        # assigning the key words to the new column
        row['Key_Words'] = ' '.join(list(key_words_dict_scores.keys()))
        lyrics_df.at[index, 'Key_Words'] = row['Key_Words']


    lyrics_df.drop(columns = ['Lyrics'], inplace = True)
    
    # merging together hip hop artist names to treat as unique values
    lyrics_df['Artists_Lower'] = lyrics_df['Artists'].map(lambda x: x.split(' '))
    for index, row in lyrics_df.iterrows():
        row['Artists_Lower'] = ''.join(row['Artists_Lower']).lower()
        lyrics_df.at[index, 'Artists_Lower'] = row['Artists_Lower']

    lyrics_df['Key_Words'] = lyrics_df['Artists_Lower'] + ' ' + lyrics_df['Key_Words']
    lyrics_df.drop(columns = ['Artists_Lower'], inplace = True)
    
    return lyrics_df
```

### 2. Clean the data

```python
def clean_data_frame(lyrics_df):
    #Remove duplicate values and where songs = artists (errors from parsing)
    lyrics_df = lyrics_df.drop_duplicates(subset=['Songs', 'Artists'], keep='first')
    lyrics_df = lyrics_df.drop(lyrics_df[lyrics_df['Songs'] == lyrics_df['Artists']].index)
    
    #Reset index to songs
    lyrics_df.set_index('Songs', inplace = True)
    lyrics_df.drop(columns = ['Unnamed: 0'], inplace = True) 
    
    return lyrics_df
```

![image]({{site.url}}/images/projects/genius-parser/GeniusParserResultsCleaned.png)

### 3. Get Cosine Similarity Matrix

After getting all the keywords and cleaning the data frame, I want to compare the song keywords with each other and see how similar the songs are. I need to calculate the frequency of each of these keywords for each song. `TfIdfVectorizer` is a common approach to calculate frequency, but I want a simple frequency counter `CountVectorizer`. Reason being is that `TfIdfVectorizer` gives less importance to words that are heavily present in keywords for all songs (corpus). Hip hop songs have frequent collaborations and shoutouts (Drake and Jay-Z appear frequently in other songs), and `TfIdfVectorizer` will accord less weight to those artists who have released a lot of number #1 hit singles. 

I then apply cosine similarity to this new matrix. Cosine similarity is a measure of similarity between two non-zero vectors. The cosine angle between two vectors is closer to 1 if both vectors are similar. In this scenario, a keyword is assigned a different dimension, and a lyric is characterized by a vector with values in each dimension corresponding to number of times the keyword appears in the lyric. The drawback is that it's computationally expensive.

```python
def get_cosine_sim_matrix(lyrics_df):
    # instantiating and generating the count matrix
    count = CountVectorizer()
    count_matrix = count.fit_transform(lyrics_df['Key_Words'])
    
    # generating the cosine similarity matrix
    cosine_sim = cosine_similarity(count_matrix, count_matrix) 
    
    return cosine_sim
```

### 4. Get recommended songs

**Note:** The dataset I used includes #1 hit singles in hip hop. If you're inputing a hip hop song that isn't a #1 hit single, you won't get any results. 

```python
def get_recommended_songs(lyrics_df, cosine_sim, song):  
    recommended_songs = []
    
    # creating a Series for the song titles so they are associated to an ordered numerical
    indices = pd.DataFrame(lyrics_df.index, lyrics_df['Artists'])
    indices = indices.reset_index()
    
    # gettin the index of the song that matches the title
    idx = indices[indices['Songs'] == song].index[0]
    
    # creating a Series with the similarity scores in descending order
    score_series = pd.Series(cosine_sim[idx]).sort_values(ascending = False)

    # getting the indexes of the 10 most similar songs
    top_10_indexes = list(score_series.iloc[1:11].index)
    
    # populating the list with the titles of the best 10 matching songs
    for i in top_10_indexes:
        recommended_songs.append(list(lyrics_df['Artists'])[i] + ': ' + list(lyrics_df.index)[i] + " " + list(lyrics_df['Era'])[i])
    
    return recommended_songs
```

If I input `Empire State of Mind`, I get these following songs

```
['Puff Daddy: Satisfy You 1980-2000', 
'Puff Daddy: Come with Me 1980-2000', 
'Jay-Z: Holy Grail 2010-2020', 
'D-Nice: Call Me D-Nice 1980-2000', 
'Luke: Raise the Roof 1980-2000', 
'Drake: Forever 2000-2010', 
"DJ Khaled: I'm the One 2010-2020", 
"DJ Khaled: I'm on One 2010-2020", 
'T.I.: Dead and Gone 2000-2010', 
'Fat Joe: Flow Joe 1980-2000']
```

If I input `Hotline Bling`, I get these following songs

```
['Drake: Forever 2000-2010', 
'Bow Wow: Like You 2000-2010', 
'Kanye West: Heartless 2000-2010', 
'Young Jeezy: Soul Survivor 2000-2010', 
'Young Jeezy: Put On 2000-2010', 
'Puff Daddy & the Family: Been Around the World 1980-2000', 
'Nelly: Dilemma 2000-2010', 
'Drake: In My Feelings 2010-2020', 
'Solé: 4, 5, 6 1980-2000', 
'Plies: Shawty 2000-2010']
```

By using cosine similarity, I have created a music recommendation system of #1 hit hip hop singles. 

# Github Code

[Scraping Songs Code](https://github.com/hd2zm/Data-Science-Projects/blob/master/Music/Genius/GeniusParser.py)

[Song Recommender Code](https://github.com/hd2zm/Data-Science-Projects/blob/master/Music/Genius/SongRecommender.py)

# Resources

[Scraping Songs](https://dev.to/willamesoares/how-to-integrate-spotify-and-genius-api-to-easily-crawl-song-lyrics-with-python-4o62)

[Content Recommender System](https://towardsdatascience.com/how-to-build-from-scratch-a-content-based-movie-recommender-with-natural-language-processing-25ad400eb243)
