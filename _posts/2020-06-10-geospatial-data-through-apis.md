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
I use them extensively in my day to day work especially with geospatial data and are a great way to make your work standout.
I'm going to be covering the City of Regina's Open GIS APIs<sup>[2](#myfootnote2)</sup> using python.

You can find the code at this repo.

First of all when it comes to APIs, we have to understand how requests and responses work. An data API is similar 
to a webpage. When you load a webpage your browser sends a request to a URL and the server return the HTML code for
your browser to render and visualize. The main difference with an API is instead of HTML being returned, you get data
usually in json, csv or xml formats. These formats are managed easily in Python.

Libraries you will need to be familiar with:
- python
- requests
- pandas
- geopandas
- shapely

You'll also need to know how to use your browsers inspect function.

## Data

You will need to see how an API fires for some webpages wher there is limited documentation. There is limited documentation for the City of Regina's
GIS data so we will use the inspect function our browser to see how data is loaded into maps.

First we'll need to navigate to the City of Regina's Open GIS website [https://opengis.regina.ca/arcgis/rest/services](https://opengis.regina.ca/arcgis/rest/services).

![alt text](img/city-of-regina-main-opengis-page.png){:height="100%" width="100%"}

I'm going to select the CGISViewer > TreeWebApp and click on the [ArcGIS Online Map Viewer](http://www.arcgis.com/home/webmap/viewer.html?url=https%3A%2F%2Fopengis.regina.ca%2Farcgis%2Frest%2Fservices%2FCGISViewer%2FTreeWebApp%2FMapServer&source=sd).
This will take you this page and you will need to open the `Inspect` function in your browser then go to the `Network` tab.

![alt text](img/city-of-regina-tree-map.png){:height="100%" width="100%"}

Next we'll need to see how the tree API is used in the map. Under `Contents` you'll need to expand the `TreeWebApp` and highlight the `City Trees` layer.
You'll see a table icon pop-up and click on it. You'll see a list of network requests fire in the `Network` tab of your browser.
Look for one with `query?f=json...` as the name and select it. Copy the URL in `Request URL`, we'll need that for our python code.

## Code

We'll need the `requests` library to get the data from Open GIS. We'll use `pandas` as a convenient way to process the data 
so it can be converted to a `geopandas` object. Shapely is needed to convert json geometries to `shapely` objects because `geopandas` is built
on top of `shapely`.

```python
import requests
import pandas
import geopandas
import shapely
```

Next we'll need to copy the `Request URL` we copied before in the `url` parameter for `requests.get`. Then we'll need to 
change a couple of the url parameters to get WSG48 coordinates and to get each objects geometry. Update the `returnGeometry` parameter
to `true` and change the `outSR` parameter to `4326`. You can read more about ArcGIS REST API parameters 
for Map Service Layers [here](https://opengis.regina.ca/arcgis/sdk/rest/index.html#/Query_Map_Service_Layer/02ss0000000r000000/).

```python
resp=requests.get("https://opengis.regina.ca/arcgis/rest/services/CGISViewer/TreeWebApp/MapServer/0/query?f=json&returnGeometry=true&spatialRel=esriSpatialRelIntersects&outFields=*&orderByFields=OBJECTID%20ASC&outSR=4326&resultOffset=0")
print(resp.json())
```
---
Foot Notes

<a name="myfootnote1">1</a>: [https://en.wikipedia.org/wiki/Representational_state_transfer](https://en.wikipedia.org/wiki/Representational_state_transfer)

<a name="myfootnote2">2</a>: [https://opengis.regina.ca/arcgis/rest/services](https://opengis.regina.ca/arcgis/rest/services)