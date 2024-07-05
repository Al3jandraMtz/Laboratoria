# **Spotify**

## Validación de hipótesis para el éxito de una canción.

### **Objetivo:**

Aumentar las posibilidades de éxito de la discográfica y del nuevo artista, mediante el análisis de una base de datos de Spotify, esto con la finalidad de refutar o confirmar las siguientes hipótesis:
  
    1.-Las canciones con un mayor BPM (Beats Por Minuto) tienen más éxito en términos de cantidad de streams en Spotify.
  
    2.-Las canciones más populares en el ranking de Spotify también tienen un comportamiento similar en otras plataformas como Deezer.
  
    3.-La presencia de una canción en un mayor número de playlists se relaciona con un mayor número de streams.
  
    4.- Los artistas con un mayor número de canciones en Spotify tienen más streams.
  
    5.-Las características de la canción influyen en el éxito en términos de cantidad de streams en Spotify.

Y por último, proporcionar recomendaciones estratégicas basadas en los hallazgos. 

### **Herramientas**
  + SQL
  + BigQuery
  + PowerBI
  + Python

## **Procesar y preparar la base de datos**

1.- Importación de 3 data set a BigQuery: Se creó el proyecto en BigQuery con nombre Proyecto_Spotify, una vez creado el proyecto se realiza la importación de las tablas:

+ track_in_competition
+ track_in_spotify. Se modifico el archivo en artist(s), se quitan los parentesis para poderlo subir.
+ track_technical_info
  
2.- Identificar y manejar valores nulos: Se identificaron nulos a través de comandos SQL

+ track_in_competition
  + Track_id : 0 null
  + in_apple_charts : 0 null
  + in_apple_playlists: 0 nulls
  + in_deezer_playlists: 0 nulls
  + in_deezer_charts: 0 nulls
  + in_shazam_charts: 50 nulls
+ track_in_spotify
  + track_in_spotify: 0 nulls
  + track_name: 0 nulls
  + artists_name: 0 nulls
  + artist_count: 0 nulls
  + released_year: 0 nulls
  + released_month: 0 nulls
  + released_day : 0 nulls
  + in_spotify_playlists: 0 nulls
  + in_spotify_playlists: 0 nulls
  + streams: 0 nulls

Query:
~~~
 SELECT
COUNT (*)
FROM
  `proyecto-1-spotify.Data_set.track_in_competition`
WHERE in_shazam_charts IS NULL
 
~~~

+ track_technical_info
  + track_id: 0 nulls
  + bpm: 0 null
  + key: 95 nulls
  + mode: 0 nulls
  + danceability_%: o nulls
  + valence_%: 0 nulls
  + energy_%: 0 null
  + acousticness_%: 0 null
  + instrumentalness_%: 0 null
  + liveness_%: 0 null
  + speechiness_%: 0 null

Query (Buscar en todas las columnas)
~~~
SELECT
COUNT (*)
FROM
  `proyecto-1-spotify.Data_set.track_technical_info`
WHERE
track_id IS NULL
OR bpm IS NULL
OR key IS NULL
OR mode IS NULL
OR 'danceability_%' IS NULL
OR "valence_%" IS NULL
or "energy_%" IS NULL
OR "acousticness_%" IS NULL
OR "instrumentalness_%" IS NULL
OR "liveness_%" IS NULL
OR "speechiness_%" IS NULL
 
~~~

Query (Obtención de nulos en la nueva columna)
~~~
SELECT
  COUNTIF(track_id IS NULL) AS track_id_nulos,
  COUNTIF(track_name IS NULL) AS track_name_nulos,
  COUNTIF(artists_name IS NULL) AS artists_name_nulos,
  COUNTIF(artist_count IS NULL) AS artist_count_nulos,
  COUNTIF(released_month IS NULL) AS released_month_nulos,
  COUNTIF(released_day IS NULL) AS released_day_nulos,
  COUNTIF(in_spotify_playlists IS NULL) AS in_spotify_playlists_nulos,
  COUNTIF(streams IS NULL) streams_nulos,
FROM
 
~~~

3.- Identificar Duplicados: Se han identificado duplicados a través de comandos SQL
  + track_in_competition: Sin duplicados
  + tranck_in_spotiffy: Presenta Duplicados
  + + track_technical_info: Sin duplicados
   
> [!NOTE]
> RESULTADOS


Query:
  ~~~
WITH
  duplicados AS (
  SELECT
    track_name,
    artists_name,
    COUNT(*) AS duplicado
  FROM
    `proyecto-1-spotify.Data_set.track_in_spotify`
  GROUP BY
    track_name,
    artists_name
  HAVING
    COUNT(*) > 1 )
SELECT
  track_name,
  artists_name,
  duplicado
FROM
  duplicados

~~~
