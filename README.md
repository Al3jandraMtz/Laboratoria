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

## **5.1 Procesar y preparar la base de datos**

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
  + track_technical_info: Sin duplicados
   
> [!NOTE]
> RESULTADOS
> ![](Imagenes/Duplicados.png)

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

Query basico:
  ~~~
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
    COUNT(*) > 1 
~~~

4.- Eliminar valores fuera del alcance del análisis: Se han manejado datos fuera del alcance a través de comandos SQL.

  + track_in_competition: Se considero que necesitariamos todos los datos para analizar las hipotesis solicitadas.
  + track_in_spotify: Se considero que necesitariamos todos los datos para analizar las hipotesis solicitadas.
  + track_in_technical_info: Se llego a al conclusión de que key y mode no serian utiles para el analisis que estamos realizando.

Query:
  ~~~
SELECT
* EXCEPT (key, mode)
FROM
  `proyecto-1-spotify.Data_set.track_technical_info`
~~~

5.- Identificar y manejar datos discrepantes en variables categóricas: Se han identificado y manejado datos discrepantes (simbolos) en variables numéricas a través de comandos SQL.
  + track_in_competition: Sin datos discrepantes
  + track_in_spotify:  Con datos discrepantes
  + track_in_technical_info: Sin datos discrepantes
    
Nos ayuda a quitar este tipo de simbología ��

Query:
  ~~~
SELECT
track_name,
REGEXP_REPLACE(track_name, r'[^a-zA-Z0-9 ]', ' ') AS track_name_limpio
FROM
  `proyecto-1-spotify.Data_set.track_in_spotify`
~~~

6.- Identificar valores discrepantes en variables numéricas con MAX, MIN y AVG: Se han identificado y manejado datos discrepantes en variables numéricas a través de comandos SQL
  + track_in_competition: Sin datos discrepantes
  + track_in_spotify: Con datos discrepantes en streams
  + track_in_technical_info: Sin datos discrepantes

Utilizamos varias consultas para obtener el resultado deseado:

Query 1:
  ~~~
SELECT
MAX (streams),
MIN (streams),
FROM
  `proyecto-1-spotify.Data_set.track_in_spotify`
~~~

Query 2: (Con Safe cast para convertir STINGS en INTEGER de los streams)
~~~
SELECT
  MAX(in_spotify_playlists) AS max_in_spotify_playlists,
  MIN(in_spotify_playlists) AS min_in_spotify_playlists,
  AVG(in_spotify_playlists) AS avg_in_spotify_playlists,
  MAX(in_spotify_charts) AS max_spotify_charts,
  MIN(in_spotify_charts) AS min_spotify_charts,
  AVG(in_spotify_charts) AS avg_spotify_charts,
  MAX(SAFE_CAST (streams AS INT64)) AS max_streams,
  MIN(streams) AS min_streams,
  AVG(SAFE_CAST (streams AS INT64)) AS avg_streams,
FROM
`proyecto-1-spotify.Data_set.track_in_spotify`
~~~

Query 3: (Completo)
~~~
SELECT
  MAX(`bpm`) AS max_bpm,
  MIN(`bpm`) AS min_bpm,
  AVG(`bpm`) AS avg_bpm,
  MAX(`danceability_%`) AS max_danceability,
  MIN(`danceability_%`) AS min_danceability,
  AVG(`danceability_%`) AS avg_danceability,
  MAX(`valence_%`) AS max_valence,
  MIN(`valence_%`) AS min_valence,
  AVG(`valence_%`) AS avg_valence,
  MAX(`energy_%`) AS max_energy,
  MIN(`energy_%`) AS min_energy,
  AVG(`energy_%`) AS avg_energy,
  MAX(`acousticness_%`) AS max_acousticness,
  MIN(`acousticness_%`) AS min_acousticness,
  AVG(`acousticness_%`) AS avg_acousticness,
  MAX(`instrumentalness_%`) AS max_instrumentalness,
  MIN(`instrumentalness_%`) AS min_instrumentalness,
  AVG(`instrumentalness_%`) AS avg_instrumentalness,
  MAX(`liveness_%`) AS max_liveness,
  MIN(`liveness_%`) AS min_liveness,
  AVG(`liveness_%`) AS avg_liveness,
FROM
 `proyecto-1-spotify.Data_set.track_technical_info`
~~~

7.- Comprobar y cambiar tipo de dato: Se cambiaron el tipo de dato a través de comandos SQL.
  + track_in_competition: No requirio cambios de datos.
  + track_in_spotify:  Con datos tipo STRING en vez de númericos (INTEGER).
  + track_in_technical_info: No requirio cambios de datos.

Query: (Cambiar el tipo de dato)
~~~
SELECT
safe_cast(streams AS INT64) AS streams_limpio
FROM
  `proyecto-1-spotify.Data_set.track_in_spotify`
~~~

Query: (Comando para que no aparezca algúna palabra)
~~~
SELECT
  *
FROM
  `proyecto-1-spotify.Data_set.track_in_spotify`
  WHERE
  streams NOT LIKE '%BPM%'
~~~

8.- Crear nuevas variables: Se crearon nuevas variables a través de comandos SQL.
  + track_in_competition: Sin nueva variable.
  + track_in_spotify:  Se crea la fecha (aaaa-mm-dd)
  + track_in_technical_info: Sin nueva variable.

Query: (Concatenar)
~~~
SELECT
CONCAT(released_year,"-", released_month,"-", released_day)
FROM
  `proyecto-1-spotify.Data_set.track_in_spotify`
LIMIT
  1000
~~~

Query: (Fecha día/mes a dos cifras)
~~~
SELECT
  DATE( SAFE_CAST(released_year AS INT64), SAFE_CAST(released_month AS INT64), SAFE_CAST(released_day AS INT64) ) AS release_date 
 FROM
    `proyecto-1-spotify.Data_set.track_in_spotify`
~~~

9.- Unir tablas
  + track_in_competition: La tabla no requirio limpieza de datos.
  + track_in_spotify:  Se crea tabla en views con los datos limpios.

Query:  (Crear tabla nueva limpia)
~~~
SELECT
*,
REGEXP_REPLACE(track_name, r'[^a-zA-Z0-9 ]', ' ') AS track_name_limpio,
REGEXP_REPLACE(artists_name, r'[^a-zA-Z0-9 ]', ' ') AS artist_name_limpio,
safe_cast(streams AS INT64) AS streams_limpio,
DATE( SAFE_CAST(released_year AS INT64), SAFE_CAST(released_month AS INT64), SAFE_CAST(released_day AS INT64) ) AS release_date 
FROM
  `proyecto-1-spotify.Data_set.track_in_spotify`

WHERE
streams NOT LIKE '%BPM%'
~~~

 + track_in_technical_info:  Se crea tabla en views con los datos limpios.

Query:  (Crear tabla nueva limpia)
~~~
SELECT
*
EXCEPT (key, mode)
FROM
  `proyecto-1-spotify.Data_set.track_technical_info'
~~~

Para la unión de las  3 tablas usamos:

Query:
~~~
SELECT
  *
FROM
  `proyecto-1-spotify.Data_set.Track_in_Spotify_limpio` AS TS
LEFT JOIN
  `proyecto-1-spotify.Data_set.Track_in_competition_limpio` AS TC
ON
  TS.track_id = TC.track_id
LEFT JOIN
  `proyecto-1-spotify.Data_set.track_technical_info_limpio` AS TI
ON
  TS.track_id = TI.track_id
~~~

Creación del DATA SET por orden de categorias:

Query:
~~~
SELECT
TS.track_id,
TS.track_name_limpio,
TS.artist_name_limpio,
TS.artist_count,
TS.released_year,
TS.released_month,
TS.released_day,
TS.release_date,
TS.in_spotify_playlists,
TS.in_spotify_charts,
TS.streams_limpio,
TC.in_apple_playlists,
TC.in_apple_charts,
TC.in_deezer_playlists,
TC.in_deezer_charts,
TC.in_shazam_charts,
TI.bpm,
TI.`danceability_%`,
TI.`valence_%`,
TI.`energy_%`,
TI.`acousticness_%`,
TI.`instrumentalness_%`,
TI.`liveness_%`,
TI.`speechiness_%`

FROM
  `proyecto-1-spotify.Data_set.Track_in_Spotify_limpio` AS TS
LEFT JOIN
  `proyecto-1-spotify.Data_set.Track_in_competition_limpio` AS TC
ON
  TS.track_id = TC.track_id
LEFT JOIN
  `proyecto-1-spotify.Data_set.track_technical_info_limpio` AS TI
ON
  TS.track_id = TI.track_id
~~~

10.- Construir tablas auxiliares: Se ha utilizado la estructura de tablas temporales WITH para crear una tabla temporal para total de canciones por artista y veces que una canción se repite en un playlist.

Query:  (Canciones por artista)
~~~
with canciones_por_artista AS (
  SELECT 
  artist_name_limpio,
  count(track_id) as total_canciones
   FROM `proyecto-1-spotify.Data_set.Data_set_completo2`
   group by
   artist_name_limpio
)
select
*
from
canciones_por_artista
~~~

Query: (Veces que aparece una cancion en playlist)
~~~
WITH
  participacion_en_playlists AS (
  SELECT
    track_name_limpio,
    SUM(in_spotify_playlists) AS total_playlists
  FROM
    `proyectospotify-426316.dataset.track_in_spotify_limpio`
  GROUP BY
    track_name_limpio )
SELECT
  *
FROM
  participacion_en_playlists
  ORDER BY total_playlists DESC
~~~

> [!NOTE]
> ![](Imagenes/Cancion-Playlist.png)

## **5.2 Análisis exploratorio**

### Agrupar variables categóricas a través de tablas en Power BI

Se conectó la información desde BigQuery a PowerBI para poder comenzar a trabajar.
Se realizó una matriz con artista y número de tracks, así como una de tracks por año.
