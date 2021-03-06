# SongRecommender Web application
```diff
- DISCLAIMER: THIS IS AN EDUCATIONAL PROJECT. SPOTIFY.COM DATA IS USED FOR EDUCATIONAL PURPOSES ONLY.
```
Recommendation systems play a key role in our everyday lives, and when it comes to music they have a goal to come up with new songs that a user would like to hear. Since the users are collecting huge amounts of digital music on their devices, finding new music is a problem bigger than ever. 
A good recommendation system should try to minimize users' efforts when it comes to choosing music, but at the same time it should fulfill their expectations. 
This Webapp is a Music Recommendation System which compares songs in terms of audio features and tries to find the most similar ones - instead of looking into user's playlist history to determine a recommendation, this app looks for the similarity in tempo, energy, instruments and so on. 

## Overview

How the recommendation works?
1. User enters a name of a song.
2. Application looks it up on Spotify.com
3. Application compares the song with other songs in the local database
4. Application returns "most similar" song from local database along with the preview link.

## The Big Picture 

This app is a central part of the SongRecommender system and it uses few other "supporting" applications which can also be found on GitHub.com:
1. Application that groups songs by audio feature similarity, it uses KMeans algorithm for clustering   https://github.com/Igor-Jovic/SongRecommenderClustering
2. Clojure library that implements machine learning algorithms for clustering, classification and recommendation https://github.com/Igor-Jovic/SongRecommender-Clojure
3. Since the app is not available on public network, there is an application that tweets private dynamic IP address to twitter account https://github.com/Igor-Jovic/SongPickerTwitterUpdater
4. Spotify proxy app for fetching the song info from Spotify.com https://github.com/SlavkoKomarica/SongRecommender-SpotifyProxy

## Starting up your own Song Recommendation server

To start your own Song Recommender server:
1. Clone, build and run Spotify Proxy Project.
2. Clone this project
3. Clone and build Clojure library for machine learning, put it in root directory of this project.
4. Build this project
5. Download the following .csv files: Min.csv, Max.csv, clusters (whole directory), Centroids.csv from https://drive.google.com/open?id=0B3MZlBNxyLNuQzd0bFIwbzN6LXM. 
6. Put the csv files into data folder and move them to the directory where this project's jar is. 
7. Run jar file from command line and go to http://localhost:8080 

```
Before running the jar file, project structure should look like this:
-SOMEDIRECTORY
  -songrecommender.jar
  -data
    -clusters
      -cluster1
      -cluster2
      -...
    -Min.csv
    -Max.csv
    -Centroids.csv
 ```

## Benchmark

The application supports two options for similarity calculation:
1. Euclidean distance - simple Euclidean distance calculated using audio features (described in Clojure lib for machine learning) 
2. Mixed similarity measure - euclidean similarity + "genre similarity". Genre similarity is a measure of genre overlapping. Genres string is splitted into tokens for both songs (eg. "rock blues-rock classic-rock" becomes [rock blues classic]) and similarity is calculated as number of overlapping genres divided by number of genres in union. 

Mixed similarity measure gives more accurate recommendations since it takes into account genres and not only audio features, but because of genre similarity calculation it consumes more time. 

JMH(Java Microbenchmark Harness) was used for benchmarking two methods in the RecommendationService - getRecommendationByEuclideanFor(String songName) and getRecommendationFor(String songName). The first one uses only euclidean similarity while the other uses mixed similarity explained above, benchmarking code can be found in src/test/java/benchmark package. 
The chosen mode for benchmarking is average time needed for completing the recommendation request, and it covers:
1. Getting the song information from Spotify.com by song name.
2. Assigning a cluster to a song.
3. Calculating similarities.
4. Fetching the most similar song's info from Spofify.com.
5. Returning the recommendation.

### Benchmarking results
The table below shows the benchmarking results

Benchmark method                                         | Number of Method calls | Average time | Units
-------------                                            | -------------          | -------------| -------------
recommendationByOnlyEuclideanDistance                    | 200                    | 2705.180     | milliseconds per operation
recommendationByEuclideanDistanceAndGenreSimilarity      | 200                    | 3594.487     | milliseconds per operation

Conclusion: using mixed similarity takes on average 0.889 seconds more than euclidean distance but usually gives more accurate recommendations - for Alborosie - Rastafari Anthem song (modern reggae), euclidean distance method returned rap song I Think They Like Me - feat. Jermaine Dupri, Da Brat & Bow Wow while mixed similarity method returned Damian Marley	-	Educated Fools, which is also modern reggae.  
