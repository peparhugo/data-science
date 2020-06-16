---
layout: post
date: 2020-06-10
type: blog
updated: 2020-06-10
title: Getting Geospatial Data through REST APIs in Python
description: Learn how to access geospatial data through the City of Regina's Open GIS endpoints
tags: [python, api, geospatial]
---
## Intro
REST APIs are becoming more and more common data sources with modern applications<sup>[1](#myfootnote1)</sup>. 
I use them extensively in my day to day work, especially with geospatial data and are a great way to make your work stand out.
I'm going to be covering the City of Regina's Open GIS APIs<sup>[2](#myfootnote2)</sup> using Python.

You can find the notebook for the post [here](https://github.com/peparhugo/data-science/blob/gh-pages/notebooks/2020-06-10-geospatial-data-through-apis.ipynb).

First of all, when it comes to APIs, we have to understand how requests and responses work. A data API is similar to a webpage. When you load a webpage, your browser sends a request to a URL, and the server returns the HTML code for
your browser to render and visualize. The main difference with an API is instead of HTML being returned, and you get data
usually in JSON, CSV or XML formats. These formats are managed easily in Python.

Libraries you will need to be familiar with:
- python
- requests
- pandas
- geopandas
- shapely

You'll also need to know how to use your browser's inspect function.

## Data

You will need to see how an API fires for some webpages where there is limited documentation. There is limited documentation for the City of Regina's
GIS data, so we will use the inspect function of our browser to see how the data is loaded into maps.

First, we'll need to navigate to the City of Regina's Open GIS website [https://opengis.regina.ca/arcgis/rest/services](https://opengis.regina.ca/arcgis/rest/services).

![alt text](img/city-of-regina-main-opengis-page.png){:height="100%" width="100%"}

I'm going to select the CGISViewer > TreeWebApp and click on the [ArcGIS Online Map Viewer](http://www.arcgis.com/home/webmap/viewer.html?url=https%3A%2F%2Fopengis.regina.ca%2Farcgis%2Frest%2Fservices%2FCGISViewer%2FTreeWebApp%2FMapServer&source=sd).
The link will take you this page, and you will need to open the `Inspect` function in your browser then go to the `Network` tab.

![alt text](img/city-of-regina-tree-map.png){:height="100%" width="100%"}

Next, we'll need to see how the tree API is called to load the map. Under `Contents` you'll need to expand the `TreeWebApp` and highlight the `City Trees` layer.
You'll see a table icon pop-up and click on it. You'll see a list of network requests fire in the `Network` tab of your browser.
Look for one with `query?f=json...` as the name and select it. Copy the URL in `Request URL`, we'll need that for our python code.

## Code

We'll need the `requests` library to get the data from Open GIS. We'll use `pandas` as a convenient way to process the data 
so it can be converted to a `geopandas` object. Shapely is needed to convert JSON geometries to `shapely` objects because `geopandas` is built
on top of `shapely`.

```python
import requests
import pandas as pd
import geopandas as gpd
import shapely
```

Next, we'll need to copy the `Request URL` we copied before in the `url` parameter for `requests.get`. Then we'll need to 
change a couple of the URL parameters to get WSG48 coordinates and to get each objects geometry. Update the `returnGeometry` parameter
to `true` and change the `outSR` parameter to `4326`. You can read more about ArcGIS REST API parameters 
for Map Service Layers [here](https://opengis.regina.ca/arcgis/sdk/rest/index.html#/Query_Map_Service_Layer/02ss0000000r000000/).

```python
resp=requests.get(url="https://opengis.regina.ca/arcgis/rest/services/CGISViewer/TreeWebApp/MapServer/0/query?outFields=*",
                     params=dict(
                         f='json',
                         returnGeometry='true',
                         spatialRel='esriSpatialRelIntersects',
                         where='(1=1) AND (1=1)',
                         orderByFields='GLOBALID ASC',
                         outSR=4326,
                         resultOffset=0,
                         resultRecordCount=1
                     ))
print(resp.json())
```
Output:
```json
{'displayFieldName': 'STREETNAMEFULL',
 'fieldAliases': {'OBJECTID': 'OBJECTID',
  'YEARINSTALLED': 'YEARINSTALLED',
  'SPECIES': 'SPECIES',
  'NOTES': 'NOTES',
  'DBH': 'DBH',
  'TREEVALUE': 'TreeValue',
  'OWNER': 'OWNER',
  'SIGNIFICANCE_TREE': 'SIGNIFICANCE_TREE',
  'GLOBALID': 'GLOBALID',
  'SOIL_AMENDMENT': 'SOIL_AMENDMENT'},
 'geometryType': 'esriGeometryPoint',
 'spatialReference': {'wkid': 4326, 'latestWkid': 4326},
 'fields': [{'name': 'OBJECTID',
   'type': 'esriFieldTypeOID',
   'alias': 'OBJECTID'},
  {'name': 'YEARINSTALLED',
   'type': 'esriFieldTypeInteger',
   'alias': 'YEARINSTALLED'},
  {'name': 'SPECIES',
   'type': 'esriFieldTypeString',
   'alias': 'SPECIES',
   'length': 50},
  {'name': 'NOTES',
   'type': 'esriFieldTypeString',
   'alias': 'NOTES',
   'length': 1000},
  {'name': 'DBH', 'type': 'esriFieldTypeSmallInteger', 'alias': 'DBH'},
  {'name': 'TREEVALUE', 'type': 'esriFieldTypeDouble', 'alias': 'TreeValue'},
  {'name': 'OWNER',
   'type': 'esriFieldTypeString',
   'alias': 'OWNER',
   'length': 50},
  {'name': 'SIGNIFICANCE_TREE',
   'type': 'esriFieldTypeString',
   'alias': 'SIGNIFICANCE_TREE',
   'length': 20},
  {'name': 'GLOBALID',
   'type': 'esriFieldTypeGlobalID',
   'alias': 'GLOBALID',
   'length': 38},
  {'name': 'SOIL_AMENDMENT',
   'type': 'esriFieldTypeString',
   'alias': 'SOIL_AMENDMENT',
   'length': 50}],
 'features': [{'attributes': {'OBJECTID': 112557,
    'YEARINSTALLED': None,
    'SPECIES': 'American Elm',
    'NOTES': None,
    'DBH': 38,
    'TREEVALUE': 13626.15,
    'OWNER': 'City',
    'SIGNIFICANCE_TREE': 'N',
    'GLOBALID': '{00003627-F4BF-4684-8016-E3FE69BEDDBD}',
    'SOIL_AMENDMENT': 'N'},
   'geometry': {'x': -104.5969337525038, 'y': 50.40507010684367}}],
 'exceededTransferLimit': True}
```

The `features` object is has the data we're looking for and the other fields contain metadata about each field. The example response
only has 1 record but we'll want to pull all 143,375 records. Most APIs limit the number of records per response, and the 
open GIS APIs have a limit of 10,000 records.

Fortunately, there is usually some kind of page or offset function so you can stream or scroll all the results. There are several
ways to get the data, but for now, we'll use a simple while loop to get all the results.
```
results = []
nbr_results = 1
while nbr_results>0:
    resp=requests.get(url="https://opengis.regina.ca/arcgis/rest/services/CGISViewer/TreeWebApp/MapServer/0/query?outFields=*",
                     params=dict(
                         f='json',
                         returnGeometry='true',
                         spatialRel='esriSpatialRelIntersects',
                         where='(1=1) AND (1=1)',
                         orderByFields='GLOBALID ASC',
                         outSR=4326,
                         resultOffset=len(results),
                         resultRecordCount=10000
                     ))
    nbr_results=len(resp.json()['features'])
    results.extend(resp.json()['features'])
    print(len(results))
```
Output:
```
10000
20000
30000
40000
50000
60000
70000
80000
90000
100000
110000
120000
130000
140000
143375
143375
```

We can see the loop function worked to get all 143,375 records. Now the data needs to be loaded into pandas but before I can do that
I'll need to restructure each record because it has an `attributes` object and a `geometry` object. We'll need to
combine those into a single object. 

There are several ways to do this, but I'll simply use a list comprehension and dictionary unpacking to combine them
into a single document

```
data = pd.DataFrame([{**doc['attributes'],
                      "geometry":doc.get('geometry')} for doc in results])
data=data[data.geometry.isna()==False]
print(data.shape)
```
Output:
```
(143373, 11)
```
It ends up there are two trees with no geometry, so the last line drops the records with missing geometries. Next, I'll convert
the pandas dataframe into a geopandas geodataframe.

```
geodata = gpd.GeoDataFrame(data.drop(columns=['geometry']),
              geometry=data.geometry.map(lambda x: shapely.geometry.Point(float(x['x']),
                  float(x['y']))))
```
![alt text](img/regina-trees-dataframe-head.png){:height="100%" width="100%"}

To create the geodataframe, we need to pass in the x and y values of the geometries. Since all of these are point features,
we can use the `shapely.geometry.Point` function.

Next we'll get the boundaries for the different neighborhoods in Regina. The neighborhood data is from the `Neighbourhood_Profile`
map layer. Updating the url to the `Neighbourhood_Profile` and removing the orderByField give the following code:

```
resp=requests.get(url="https://opengis.regina.ca/arcgis/rest/services/CGISViewer/Neighbourhood_Profile/MapServer/0/query?outFields=*",
                     params=dict(
                         f='json',
                         returnGeometry='true',
                         spatialRel='esriSpatialRelIntersects',
                         where='(1=1) AND (1=1)',
                         outSR=4326,
                         resultOffset=0,
                         resultRecordCount=10000
                     ))
resp.json()['features'][0:1]
``` 
Output:
```
[{'attributes': {'OBJECTID': 1,
   'CA': 'WHITMORE PARK',
   'NEIGH_AREA': ' ',
   'Shape_Length': 7504.10496471471,
   'Shape_Area': 2936200.4533456834,
   'PDF_Link': 'https://regina.ca/about-regina/neighbourhood-profiles/.galleries/pdfs/whitmore-park.pdf'},
  'geometry': {'rings': [[[-104.60905929785373, 50.415484024136795],
     [-104.60904849233098, 50.411883493529444],
     [-104.60030508176132, 50.411887900916504],
     [-104.60030517330979, 50.411046775773706],
     [-104.59784252726828, 50.41044205125727],
     [-104.59571523346489, 50.40954850106288],
     [-104.59564430865197, 50.40961735762469],
     [-104.5952625400897, 50.40989199803777],
     [-104.59440614601492, 50.41034860298679],
     [-104.5942343353836, 50.4102063677372],
     [-104.59408080049744, 50.410079260935355],
     [-104.59390699951967, 50.40994766740371],
     [-104.59375398512901, 50.40985189429068],
     [-104.59356404811005, 50.40974281118223],
     [-104.59338334476092, 50.40965782652863],
     [-104.58951311575542, 50.407980774830264],
     [-104.59861523339521, 50.399153428136756],
     [-104.59989918211983, 50.398235051177565],
     [-104.60062218285954, 50.39764052606164],
     [-104.60131574254478, 50.397250696771835],
     [-104.60207627869188, 50.39696260272018],
     [-104.60255323435558, 50.39679452715815],
     [-104.60256362728032, 50.39679456266412],
     [-104.60297313717075, 50.396707668396076],
     [-104.60346963466927, 50.39660932463734],
     [-104.60397677627205, 50.39654005673168],
     [-104.6047315761789, 50.39644730008961],
     [-104.60562932123668, 50.396414310918885],
     [-104.60656264221495, 50.39642011788557],
     [-104.60656377327712, 50.397322578373455],
     [-104.618328978457, 50.39732387208351],
     [-104.6182808166161, 50.400994008660824],
     [-104.61814635588311, 50.404497875457466],
     [-104.61809045972167, 50.415499066284276],
     [-104.60905929785373, 50.415484024136795]]]}}]
```

Now we'll do the same thing we did with the tree data and convert the JSON data to pandas dataframe and the pandas dataframe to a 
geopandas geodataframe. This time the response has a rings field with nested lists. This means these are polygons instead of
points. We'll need to use the `shapely.geometry.Polygon` function. The neighborhoods are single ring polygons, so we'll need to
access the first object in the rings list using `doc['rings'][0]`.

```
data_nbh = pd.DataFrame([{**doc['attributes'],"geometry":doc.get('geometry')} for doc in resp.json()['features']])
geo_nbh = gpd.GeoDataFrame(data_nbh.drop(columns=['geometry']),
                           geometry=data_nbh.geometry.map(lambda doc: shapely.geometry.Polygon(doc['rings'][0])))
```
![alt text](img/regina-neighborhood-dataframe-head.png){:height="100%" width="100%"}

## Conclusion

We can see how easy it is to retrieve geospatial data using APIs and loading the data into GeoDataFrames. This is only
the beginning, though, since all we've done is load the data. In my [next post](visualizing-geo-data-on-maps.html), I'll cover how to visualize 
this data using KeplgerGL, Leaflet using Folium and geoviews. 


---
Foot Notes

<a name="myfootnote1">1</a>: [https://en.wikipedia.org/wiki/Representational_state_transfer](https://en.wikipedia.org/wiki/Representational_state_transfer)

<a name="myfootnote2">2</a>: [https://opengis.regina.ca/arcgis/rest/services](https://opengis.regina.ca/arcgis/rest/services)