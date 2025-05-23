# Spotify Advanced SQL Project and Query Optimization P-6

![Spotify Logo](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_logo.jpg)

## Overview
This project involves analyzing a Spotify dataset with various attributes about tracks, albums, and artists using **SQL**. It covers an end-to-end process of normalizing a denormalized dataset, performing SQL queries of varying complexity (easy, medium, and advanced), and optimizing query performance. The primary goals of the project are to practice advanced SQL skills and generate valuable insights from the dataset.

```sql
-- create table
DROP TABLE IF EXISTS spotify;
CREATE TABLE spotify (
    artist VARCHAR(255),
    track VARCHAR(255),
    album VARCHAR(255),
    album_type VARCHAR(50),
    danceability FLOAT,
    energy FLOAT,
    loudness FLOAT,
    speechiness FLOAT,
    acousticness FLOAT,
    instrumentalness FLOAT,
    liveness FLOAT,
    valence FLOAT,
    tempo FLOAT,
    duration_min FLOAT,
    title VARCHAR(255),
    channel VARCHAR(255),
    views FLOAT,
    likes BIGINT,
    comments BIGINT,
    licensed BOOLEAN,
    official_video BOOLEAN,
    stream BIGINT,
    energy_liveness FLOAT,
    most_played_on VARCHAR(50)
);
```
## Project Steps

### 1. Data Exploration
Before diving into SQL, it’s important to understand the dataset thoroughly. The dataset contains attributes such as:
- `Artist`: The performer of the track.
- `Track`: The name of the song.
- `Album`: The album to which the track belongs.
- `Album_type`: The type of album (e.g., single or album).
- Various metrics such as `danceability`, `energy`, `loudness`, `tempo`, and more.

### 4. Querying the Data
After the data is inserted, various SQL queries can be written to explore and analyze the data.

#### Easy Queries
- Simple data retrieval, filtering, and basic aggregations.
  
#### Medium Queries
- More complex queries involving grouping, aggregation functions, and joins.
  
#### Advanced Queries
- Nested subqueries, window functions, CTEs, and performance optimization.

### 5. Query Optimization
In advanced stages, the focus shifts to improving query performance. Some optimization strategies include:
- **Indexing**: Adding indexes on frequently queried columns.
- **Query Execution Plan**: Using `EXPLAIN ANALYZE` to review and refine query performance.
  
---

1.**Retrieve the names of all tracks that have more than 1 billion streams**.
```sql
EXPLAIN ANALYZE --Execution Time: 8.534 ms|||After indexing: Execution Time: 0.310 ms
SELECT track FROM spotify 
WHERE stream>1000000000;

CREATE INDEX IX_spotify_stream ON spotify(stream);
```
2.**List all albums along with their respective artists**.
```sql
SELECT DISTINCT album AS Album_1, artist AS Spotify_Artist FROM spotify 
ORDER BY 1;
```
3. **Get the total number of comments for tracks where `licensed = TRUE`**.
```SQL
Explain analyze --Execution Time: 11.213 ms changed to Execution Time: 9.622 ms after indexing
SELECT SUM(comments) FROM spotify
WHERE licensed = 'True';

CREATE INDEX ix_spotify_licensed ON SPOTIFY(licensed);

```
4. **Find all tracks that belong to the album type `single`**.
```SQL
SELECT DISTINCT track FROM spotify
WHERE album_type ='single';
```
5. **Count the total number of tracks by each artist**.
```SQL
SELECT COUNT(track) as songs,artist FROM spotify
GROUP BY artist
ORDER BY songs;
```
6. **Calculate the average danceability of tracks in each album**.
```SQL
SELECT album, AVG(danceability) FROM spotify
GROUP BY album
ORDER BY 1 DESC;
```
7. **Find the top 5 tracks with the highest energy values**.
```SQL
EXPLAIN ANALYZE --Execution Time: 12.721 ms ||| Execution Time: 0.114 ms after index
SELECT * FROM spotify
WHERE energy = ANY (SELECT energy FROM spotify ORDER BY energy DESC LIMIT 5);
;

CREATE INDEX ix_spotify_energy ON spotify (energy);

![7 BEFORE INDEXING](https://github.com/user-attachments/assets/e95edb94-bcd3-4358-910b-af096222bb7c)

SELECT track,AVG(energy) FROM spotify
GROUP BY track
ORDER BY AVG(energy) DESC
LIMIT 5;
```
8. **List all tracks along with their views and likes where `official_video = TRUE`**.
```SQL
SELECT track, SUM(views) AS total_views, SUM(likes) AS total_likes FROM spotify
WHERE official_video='True'
GROUP BY track
ORDER BY total_likes DESC;
```
9. **For each album, calculate the total views of all associated tracks**.
```SQL
SELECT album,track, SUM(views) FROM spotify
group by album,track
HAVING SUM(views)>10000000000
order by track
;
```
10. **Retrieve the track names that have been streamed on Spotify more than YouTube**.
```SQL
SELECT * FROM (SELECT track, COALESCE(SUM(CASE WHEN most_played_on = 'Spotify' THEN stream END),0) AS spotify_played, 
COALESCE(SUM(CASE WHEN most_played_on = 'Youtube' THEN stream END),0) AS youtube_played FROM spotify
GROUP BY track) AS t1
WHERE t1.spotify_played > t1.youtube_played AND t1.youtube_played > 0;
```

11. **Find the top 3 most-viewed tracks for each artist using window functions**.
```SQL
WITH ranking_Artist AS(SELECT artist,track, SUM(views) AS total_views,
DENSE_RANK() OVER(PARTITION BY artist ORDER BY SUM(views) DESC) AS rank
FROM spotify
GROUP BY artist,track
ORDER BY 1,SUM(views) desc)
SELECT * FROM ranking_Artist
WHERE rank<=3;
```
12. **Write a query to find tracks where the liveness score is above the average**.
```SQL
SELECT track,liveness FROM spotify
WHERE liveness> (SELECT AVG(liveness) FROM spotify);
```
    
13. **Use a `WITH` clause to calculate the difference between the highest and lowest energy values for tracks in each album.**
```sql
WITH cte
AS
(SELECT 
	album,
	MAX(energy) as highest_energy,
	MIN(energy) as lowest_energery
FROM spotify
GROUP BY 1
)
SELECT 
	album,
	highest_energy - lowest_energery as energy_diff
FROM cte
ORDER BY 2 DESC
```
   
14. **Find tracks where the energy-to-liveness ratio is greater than 1.2**.
```SQL
SELECT track, CAST(energy/liveness AS DECIMAL(10,2)) AS energy_to_liveness FROM spotify
WHERE CAST(energy/liveness AS DECIMAL(10,2))> 1.2 AND liveness !=0 
ORDER BY energy_to_liveness;
```
15. **Calculate the cumulative sum of likes for tracks ordered by the number of views, using window functions**.
```SQL
WITH aggregated_data AS
(SELECT track,likes, SUM(likes) AS total_likes, views FROM spotify
GROUP BY track,views,likes)
SELECT 
    track,	
	likes,
    views,
    SUM(likes) OVER (ORDER BY views DESC) AS cumulative_likes
FROM
    aggregated_data
	ORDER BY views DESC;
```

Here’s an updated section for your **Spotify Advanced SQL Project and Query Optimization** README, focusing on the query optimization task you performed. You can include the specific screenshots and graphs as described.

---

## Query Optimization Technique 

To improve query performance, we carried out the following optimization process:

- **Initial Query Performance Analysis Using `EXPLAIN`**
    - We began by analyzing the performance of a query using the `EXPLAIN` function.
    - The query retrieved tracks based on the `artist` column, and the performance metrics were as follows:
        - Execution time (E.T.): **7 ms**
        - Planning time (P.T.): **0.17 ms**
    - Below is the **screenshot** of the `EXPLAIN` result before optimization:
      ![EXPLAIN Before Index](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_explain_before_index.png)

- **Index Creation on the `artist` Column**
    - To optimize the query performance, we created an index on the `artist` column. This ensures faster retrieval of rows where the artist is queried.
    - **SQL command** for creating the index:
      ```sql
      CREATE INDEX idx_artist ON spotify_tracks(artist);
      ```

- **Performance Analysis After Index Creation**
    - After creating the index, we ran the same query again and observed significant improvements in performance:
        - Execution time (E.T.): **0.153 ms**
        - Planning time (P.T.): **0.152 ms**
    - Below is the **screenshot** of the `EXPLAIN` result after index creation:
      ![EXPLAIN After Index](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_explain_after_index.png)

- **Graphical Performance Comparison**
    - A graph illustrating the comparison between the initial query execution time and the optimized query execution time after index creation.
    - **Graph view** shows the significant drop in both execution and planning times:
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%203.png)
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%202.png)
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%201.png)

This optimization shows how indexing can drastically reduce query time, improving the overall performance of our database operations in the Spotify project. Though indexing reduces the execution time, over indexing should be avoided as it could influence the Data modification function time.
---

## Technology Stack
- **Database**: PostgreSQL
- **SQL Queries**: DDL, DML, Aggregations, Joins, Subqueries, Window Functions
- **Tools**: pgAdmin 4 (or any SQL editor), PostgreSQL (via Homebrew, Docker, or direct installation)
---
