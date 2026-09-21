# Cobertura del Metro de Santiago: dónde falta hoy y qué cambia con la expansión

Análisis geoespacial de la red de Metro del Gran Santiago: cuánta gente vive fuera
del alcance caminable de una estación, dónde se concentra ese déficit, y cuánto lo
corrigen las líneas proyectadas y el Metrotren.

Proyecto del curso IMT2118, Ciencia de Datos Geoespaciales, Ingeniería en Ciencia de
Datos UC. Trabajo en pareja, primer semestre de 2025.

## La pregunta

¿Cómo cambia la cobertura del Metro entre hoy y la red proyectada, y qué zonas
quedan con déficit de acceso al transporte masivo incluso después de la expansión?

Definición operativa de cobertura: una zona está cubierta si queda dentro de un
buffer de 1 km caminable desde una estación. El kilómetro no es arbitrario, es la
distancia que la literatura de transporte usa como umbral razonable de caminata a
una estación.

El análisis se hace sobre zonas censales, no sobre comunas. Una comuna como La
Florida o Maipú es demasiado grande y heterogénea: promediar sobre ella esconde
exactamente el fenómeno que se quiere medir.

## Resultados

**Situación actual**

| | |
|---|---|
| Personas sin acceso cercano al Metro | 2.665.195 |
| De esas, en zonas de alta densidad | 1.520.095 |
| Comunas urbanas sin ninguna estación | 9 (777.211 habitantes) |
| Distancia promedio a la estación más cercana | 1,58 km |
| Distancia promedio en zonas no cubiertas | 2,70 km |

Las nueve comunas sin estación son Cerro Navia, Colina, Huechuraba, La Pintana,
Lampa, Lo Barnechea, Lo Espejo, Renca y Vitacura. La lista mezcla dos realidades
distintas: periferia popular densa y comunas de alto ingreso con baja densidad. El
déficit que importa para política pública es el primero.

**Con las líneas 7, 8 y 9 proyectadas**

| | |
|---|---|
| Personas que ganan cobertura | 971.599 |
| De esas, en zonas de alta densidad | 542.151 |
| Distancia promedio, red nueva | 1,13 km (desde 1,58) |

**Sumando el Metrotren**

| | |
|---|---|
| Personas beneficiadas | 1.217.642 |
| Zonas que ganan cobertura | 335, de ellas 160 de alta densidad |
| Comunas que todavía quedan sin nada | Colina, Huechuraba, Lampa, Lo Barnechea |

La conclusión que importa: la expansión proyectada reduce el déficit de forma
sustantiva, pero no lo cierra. Incluso con líneas nuevas y Metrotren, la distancia
promedio en zonas no cubiertas baja de 2,70 a 2,30 km, y quedan comunas enteras
fuera del sistema. El Metrotren, que suele quedar fuera de la discusión pública
sobre transporte masivo, aporta cobertura a más gente que varias estaciones nuevas
de Metro.

## Método

**Cobertura y déficit.** Buffers de 1 km sobre las estaciones, proyectados en
UTM 19S (EPSG:32719) para medir en metros y no en grados. Intersección con zonas
censales ponderada por población.

**Autocorrelación espacial.** Moran's I local (LISA) sobre la cobertura y sobre la
densidad, con pesos de contigüidad Queen. Esto responde algo que un mapa de colores
no responde: si las zonas sin metro se agrupan geográficamente o están dispersas.
Los clusters Alto-Alto y Bajo-Bajo significativos (p < 0,05) son los que marcan
déficit estructural, no ruido.

**Clustering de zonas críticas.** DBSCAN sobre zonas sin cobertura, cruzado con
cuartiles de densidad poblacional, para separar "sin metro porque no vive nadie" de
"sin metro y con mucha gente".

**Datos satelitales.** Sentinel-2 vía Google Earth Engine con enmascarado de nubes,
para medir cambio de desarrollo urbano entre 2019 y 2025. Luces nocturnas VIIRS como
proxy de actividad, también con Moran local.

**Impacto de líneas ya construidas.** Series del Índice de Calidad de Vida Urbana
(ICVU) 2016-2023 para las comunas afectadas por las líneas 3 y 6, como contraste
empírico de qué pasó donde el Metro efectivamente llegó.

**Trazados proyectados.** Las líneas 7, 8 y 9 no están publicadas como geometría
abierta, así que se reconstruyeron geocodificando las direcciones anunciadas de cada
estación y uniendo los puntos en LineStrings.

## Datos

| Fuente | Uso |
|---|---|
| GTFS de Transantiago y Metro ([Transitland](https://transit.land/feeds/f-66jc-transantiago~metrodesantiago)) | Estaciones y paraderos actuales |
| Zonas censales 2017 | Unidad de análisis y población |
| Límite comunal y polígono urbano del Gran Santiago | Recorte del área de estudio |
| Sentinel-2 y VIIRS (Google Earth Engine) | Desarrollo urbano y actividad nocturna |
| ICVU 2016-2023 | Impacto de líneas existentes |

`Datos.zip` trae las capas vectoriales y los CSV listos para correr. El feed GTFS no
está incluido porque pesa 46 MB descomprimido: se descarga desde Transitland al
directorio `gtfs_transitland/`.

## Cómo correrlo

```bash
unzip Datos.zip
pip install geopandas shapely pyproj libpysal esda scikit-learn geemap earthengine-api geopy folium matplotlib
```

Las secciones de imágenes satelitales necesitan una cuenta de Google Earth Engine
(`ee.Authenticate()`), y la de trazados proyectados una API key de Google Geocoding
en la variable de entorno `GOOGLE_API_KEY`. El resto del análisis corre sin
credenciales.

## Estructura

| Archivo | Qué contiene |
|---|---|
| `Proyecto.ipynb` | Análisis completo: cobertura, clusters, raster, líneas nuevas, Metrotren, ICVU |
| `Futuras_Estaciones.ipynb` | Construcción de los trazados L7, L8 y L9 por geocodificación |
| `Datos.zip` | Capas vectoriales, estaciones y series ICVU |
