---
layout: page
title: Trulia Crime Map
thumbnail: /images/trulia-san-francisco-crime-small.png
thumbnail_description: Trulia San Francisco crime map
description: A heatmap of crime activity for US cities.
---


The [Trulia Crime Map](http://www.trulia.com/local#crimes) is a heatmap that visualises high crime areas for US cities. I worked with [Zain Memon](http://inzain.net/) and [Sha Hwang](http://postarchitectural.com) on this project. I focused on the data import, set up the API, and configured the production servers for this project.

![Crime map for San Francisco](/images/trulia-san-francisco-crime.png)

The screenshot above is a [crime map of San Francisco](http://www.trulia.com/local#crimes/san-francisco), which consists of a Google Maps map, with an image tile overlay for the heatmap, and recent crimes as points clustered on the client side.

### Data
We used crime point data from the [CrimeReports](https://www.crimereports.com) and [SpotCrime](http://www.spotcrime.com) APIs and [Census block polygons](http://www.census.gov/cgi-bin/geo/shapefiles2012/main) to group the crime points visually. Crime feeds are downloaded from both APIs, and crime counts are updated per block on a daily basis. Any blocks with counts &gt; 0 are transferred to a smaller table, since a table with 10 million polygons becomes a little slow to query by the API.

The API generates image tiles dynamically if they are not cached, and a [GeoJSON](http://www.geojson.org) endpoint of recent crime points queryable by [geohash](http://www.geohash.org). This is currently used on [Trulia Local](http://www.trulia.com/local) and the [Trulia iPad App](https://itunes.apple.com/us/app/trulia-real-estate-homes-for/id288487321?mt=8).

### Visualisation
The heatmap overlay consists of coloured blocks. A block's colour is determined by its crime count: the number of crimes that occurred within an 0.1 mile radius of the block's centroid. A radius count allows the data to be smoothed across adjacent blocks, as opposed to counting points in each polygon where the blocks stand out a lot more. The count is then normalized between the minimum and maximum crime count values for all blocks within the county.

The colour is then generated from the ratio with the Hue-Saturation-Lightness (HSL) colour model. The ratio is scaled linearly between green and red HSL tuples. The following Python function calculates a colour given the ratio:

{% highlight py %}
def get_colour(ratio):
    """
    Calculate and return an HSL colour tuple, based on a decimal between 0.0 and 1.0
    """
    green = (90, 75, 40)  # hue = 90°, saturation = 75%, lightness = 40%
    red = (0, 100, 40)    # hue = 0°, saturation = 100%, lightness = 40%
    return [int(start + ratio * (end - start)) for start, end in zip(green, red)]
{% endhighlight %}


HSL was used because it has a nicer traffic light colour range when scaling linearly, unlike RGB:

HSL: ![HSL gradient for crime map](/images/green-red-hsl.png)

RGB: ![RGB gradient for crime map](/images/green-red-rgb.png)

### Technologies
We used [GeoDjango](http://geodjango.org) with [PostgreSQL](http://www.postgresql.org)/[PostGIS](http://postgis.refractions.net), with the excellent [TileStache](http://www.tilestache.org) library for generating image tiles.
