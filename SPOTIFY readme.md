# Spotify SQL Analysis 🎵

*SQL practice project where I explore a Spotify dataset, clean it up, and run queries from basic to advanced to pull out insights.*

---

## 📊 Dataset Schema

The database is built around a single table called **`spotify`**.  
Here are the main columns:

| Column Name        | Data Type    | Description |
|--------------------|--------------|-------------|
| `artist`           | VARCHAR(255) | Name of the artist |
| `track`            | VARCHAR(255) | Name of the track |
| `album`            | VARCHAR(255) | Album the track belongs to |
| `album_type`       | VARCHAR(50)  | Type of album (e.g., *single*, *album*) |
| `danceability`     | FLOAT        | Measure of how suitable a track is for dancing |
| `energy`           | FLOAT        | Energy/intensity of a track |
| `loudness`         | FLOAT        | Overall loudness in dB |
| `speechiness`      | FLOAT        | Presence of spoken words in a track |
| `acousticness`     | FLOAT        | Confidence measure of acoustic sound |
| `instrumentalness` | FLOAT        | Predicts whether a track is instrumental |
| `liveness`         | FLOAT        | Likelihood of live performance |
| `valence`          | FLOAT        | Musical positiveness (happy/sad vibe) |
| `tempo`            | FLOAT        | Estimated beats per minute (BPM) |
| `duration_min`     | FLOAT        | Track duration in minutes |
| `title`            | VARCHAR(255) | Video/track title |
| `channel`          | VARCHAR(255) | Channel/artist channel name |
| `views`            | FLOAT        | Total video views |
| `likes`            | BIGINT       | Total likes |
| `comments`         | BIGINT       | Number of comments |
| `licensed`         | BOOLEAN      | Whether the track is licensed |
| `official_video`   | BOOLEAN      | If it’s the official video |
| `stream`           | BIGINT       | Number of streams |
| `energy_liveness`  | FLOAT        | Combined metric of energy & liveness |
| `most_played_on`   | VARCHAR(50)  | Platform with most streams (Spotify or YouTube) |

📌 Visual Schema Representation:

![Spotify Table Schema](spotify_schema.png)

---

## 📂 Project Workflow

### 1. Database Setup
I created the main `spotify` table with attributes for artists, tracks, albums, audio features, and engagement metrics.

```sql
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
2. Exploratory Data Analysis (EDA)
I ran some quick checks to understand the dataset and clean invalid values.

sql
Copy code
-- Total rows
SELECT COUNT(*) FROM spotify;

-- Unique albums
SELECT COUNT(DISTINCT album) FROM spotify;

-- Unique album types
SELECT COUNT(DISTINCT album_type) FROM spotify;

-- Unique artists
SELECT COUNT(DISTINCT artist) FROM spotify;

-- Max & min track duration
SELECT MAX(duration_min) FROM spotify;
SELECT MIN(duration_min) FROM spotify;

-- Tracks with duration = 0
SELECT * FROM spotify
WHERE duration_min = 0;

-- Delete invalid tracks
DELETE FROM spotify
WHERE duration_min = 0;
3. Data Analysis Questions
🔹 Easy Queries
sql
Copy code
-- Q1. Retrieve the names of all tracks that have more than 1 billion streams
SELECT * FROM spotify
WHERE stream > 1000000000;

-- Q2. List all albums along with their respective artists
SELECT DISTINCT album, artist
FROM spotify
ORDER BY 1;

-- Q3. Get the total number of comments for tracks where licensed = TRUE
SELECT SUM(comments) AS total_comments
FROM spotify
WHERE licensed = TRUE;

-- Q4. Find all tracks that belong to the album type 'single'
SELECT * FROM spotify
WHERE album_type = 'single';

-- Q5. Count the total number of tracks by each artist
SELECT artist, COUNT(*) AS total_no_songs
FROM spotify
GROUP BY artist
ORDER BY 2 DESC;
🔹 Medium Queries
sql
Copy code
-- Q1. Calculate the average danceability of tracks in each album
SELECT album, AVG(danceability) AS avg_danceability
FROM spotify
GROUP BY 1
ORDER BY 2 DESC;

-- Q2. Find the top 5 tracks with the highest energy values
SELECT track, AVG(energy) AS avg_energy
FROM spotify
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;

-- Q3. List all tracks along with their views and likes where official_video = TRUE
SELECT track,
       SUM(views) AS total_views,
       SUM(likes) AS total_likes
FROM spotify
WHERE official_video = TRUE
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;

-- Q4. For each album, calculate the total views of all associated tracks
SELECT album, track, SUM(views) AS total_views
FROM spotify
GROUP BY 1, 2
ORDER BY 3 DESC;

-- Q5. Retrieve the track names that have been streamed on Spotify more than YouTube
SELECT * FROM (
    SELECT track,
           COALESCE(SUM(CASE WHEN most_played_on = 'Youtube' THEN stream END), 0) AS streamed_on_youtube,
           COALESCE(SUM(CASE WHEN most_played_on = 'Spotify' THEN stream END), 0) AS streamed_on_spotify
    FROM spotify
    GROUP BY 1
) AS t1
WHERE streamed_on_spotify > streamed_on_youtube
  AND streamed_on_youtube <> 0;
🔹 Advanced Queries
sql
Copy code
-- Q1. Find the top 3 most-viewed tracks for each artist using window functions
WITH ranking_artist AS (
    SELECT artist,
           track,
           SUM(views) AS total_view,
           DENSE_RANK() OVER (PARTITION BY artist ORDER BY SUM(views) DESC) AS rank
    FROM spotify
    GROUP BY 1, 2
    ORDER BY 1, 3 DESC
)
SELECT * FROM ranking_artist
WHERE rank <= 3;

-- Q2. Find tracks where the liveness score is above the average
SELECT track, artist, liveness
FROM spotify
WHERE liveness > (SELECT AVG(liveness) FROM spotify);

-- Q3. Calculate the difference between the highest and lowest energy values for tracks in each album
WITH cte AS (
    SELECT album,
           MAX(energy) AS highest_energy,
           MIN(energy) AS lowest_energy
    FROM spotify
    GROUP BY 1
)
SELECT album,
       highest_energy - lowest_energy AS energy_diff
FROM cte
ORDER BY 2 DESC;
🛠️ Skills Practiced
SQL DDL & DML (table creation, cleaning, querying)

Aggregations, filtering, ordering

Window functions and ranking

CTEs for advanced analysis

Data cleaning and EDA

🚀 How to Use
Clone this repository.

Save these queries into a queries.sql file (or run them in your SQL client).

Run the queries step by step (organized as: Table Creation → EDA → Easy → Medium → Advanced).

📊 Example Insights
Some tracks cross 1 billion streams easily.

Danceability varies a lot by album.

Many artists’ most popular tracks aren’t always official videos.

Energy difference between songs in the same album can be huge.

🧑‍💻 Author
Made with curiosity and SQL practice 🙂