# Python-Crawler
Hooks for accessing the dGrid database using Python

## Potential Tools

  - [GeoDjango]: Anna, is this what you use?
  - [Psycopg]: Core adapter for Postgres/Python.  Supports both Python 2.7.x and 3.x and has extensive community support. Good command-line support.  Supports server-side PostGIS manipulation from within the query- this is probably a good route.
  - [PPyGIS]: Provides client-side GIS functionality for accessing PostGIS through the Psycopg package.  This seems to have limited support and only runs on Python 2.7, documentation last updated ~2011,  so may not be a good choice. Forums seem to recommend using Psycopg as the main channel, and structuring queries to have 

postgres uses PL/SQL: syntax will be very similar to SQL

Schemas are much more important in Postgres; In SQL, the default schema is .dbo - this is not explicitly defined

POSTgis extension - makes it a geospatially capable database.

##PostGIS:
[University of Washington Video] is a quick 5-minute intro

Uses Spatial SQL: SQL, but with geographic relationships:
SELECT superhero.name FROM city,superhero WHERE ST_contains(city.geom, superhero.geom) AND city.name= 'Gotham';

Other databases:
Spatialite, SQLlite (with Geometry),  GeoPackage

WGS84 - standard map coordinate system used for pretty much everything

Libraries - Shapely, GDAL,

CREATE DATABASE <dbname>;
CREATE EXTENSION postgis;

Relevant tools:
shp2pgsql
ogr2ogr
pgsql2shp

For R:
```
library(RPostgreSQL)
drv <- dbDriver("PostgreSQL") # driver
con <- dbConnect(drv, host='localhost', user='myuser', password = password, dbname = 'myGISDB')
thisQuery <- sprintf("SELECT COUNT(*) FROM %s;",i)  # use this structure to append in the desired data into our structure
myData <- dbGetQuiery(con, thisQuery)
```

## Python
Good [zetcode tutorial] on getting started with psycopg to connect to a database from python and add or select data.

Using OGR vector data through Python: Fiona
Using Raster Data: Rasterio

Video from [SciPy2013 Geospatial Data Tutorial]: Does not go into database details
Video from [ScyPy2014 Geospatial Data Tutorial]: the last of the three videos deals with databases

sample:
```
import psychopg2 as db
conn = db.connect("host=myserver dbname=mydbname user=myusername")
cursor = conn.cursor()

sql = 'SELECT st_astext(geom) AS wkt, name, address, type FROM wifi'

cursor.execute(sql)
results = cursor.fetchall()
conn.commit()
conn.close()
```

in Pandas, showing options using both sqlalchemy and psycopg2:

```
import pandas as pd
# Two options 

# Using SQL Alchemy:
from sqlalchemy import create_engine
conn = create_engine("postgresql://myusername@myserver/mydbname")

# Or using psycopg as before:
import psycopg2 as db
conn = db.connect("host=myserver dbname=mydbname user=myusername")

df = pd.read_sql(sql, conn)
df.head() # print the first five entries
```
Working to import a csv file with (x, y) coordinates as WGS84

assuming the above:
```
sql = 'SELECT st_x(geom) as x, st_y(geom) as y, name, address, type from wifi'
df_pds = pd.read_sql(sql, conn)
df_pds.head()
# df_pds now looks like a csv file with (x,y) coordinates
# We want to give these geometry attributes that can be used for geospatial work

from shapely.geometry import Point

# Create a list of Shapely point geometries:
geoms = df_pds[["x","y"]].apply(lambda x: Point(*x) axis=1)
df_pds["geom"] = geoms

# Can coerce the pandas DataFrame to a GeoDataFrame (need to specify the CRS)
df_geo = df_pds.set_geometry("geom",crs={"init":"epsg:2263"}) # EPSG code 2263 is for the state of NY; ignore if different

df_wgs84 = df_geo.to_crs(epsg=4326)  # Convert to WGS84, which has the EPSG code 4326 - WGS is normal lat-lon

# Plotting:
ax = df_wgs84.plot()  # This will do the plotting; the following will color the points:
free = df_wgs84["type"]=="Free"
col = {True: "green", False:"blue"}
for i, l in enumerate(ax.lines):
    l.set_markersize(6)
    l.set_color(col[free[i]])
ax.axis('off')
plt.show()

# Show a map on a web-map
import mplleaflet
mplleaflet.display(fig=ax.figure, crs=df_wgs84.crs)

# As an offline html page:
mplleaflet.show(fig=ax.figure, crs=df_wgs84.crs)

# Online:
import geojsonio
res = geojsonio.display(boros.to_crs(epsg=4326).tojson(), force_gist=True)
print(res) # 
```

# QGIS and PyQGIS
PyQGIS provides full API control of QGIS through Python
  - Commands in python console within QGIS
  - Use plugins in python
  - Create custom applications running on the API

```from qgis.core import * ``` to control processing/geometry library
```from qgis.gui import *``` to control gui
```import qgis.utils```

## GeoPandas:Pandas::PostGis:PostGres

Builds on: Pandas, Shapely, Fiona, PyProj, Matplotlib, Descartes, PySAL, geopy
Has built-in attributes for geography objects which provide useful operations.

Using GeoPandas:
```
import psycopg2 as db
import geopandas as gpd
sql = sql = 'SELECT geom, name, address, type FROM wifi'
conn = db.connect("host=myserver dbname=mydbname user=myusername")
df = gpd.read_postgis(sql, conn)
df.head()

```

--
[zetcode tutorial]: http://zetcode.com/db/postgresqlpythontutorial/
[SciPy2014 Geospatial Data Tutorial]: https://www.youtube.com/watch?v=ctdjAir4TUg&list=PL16QizNnMFGd64BIRhbe6t86sgv4MvOdI
[SciPy2013 Geospatial Data Tutorial]: https://www.youtube.com/watch?v=1fzQKMp_tdE
[University of Washington Video]: https://www.youtube.com/watch?v=ZgyT3tY4hoI 
[GeoDjango]: https://docs.djangoproject.com/en/1.7/ref/contrib/gis/install/
[Psycopg]: http://initd.org/psycopg/
[PPyGIS]: http://www.fabianowski.eu/projects/ppygis/
