# OpenStreetMap

[OpenStreetMap](https://www.openstreetmap.org) is an OpenSource alternative for GoogleMaps.

It also provides a lot of sub-applications based on those datas as :

- OSRM : A routing service
- Nominatim : A search util

## Copy OSM datas on a server

First we create the directory where to put the data : 
```
mkdir /opt/osm_data
```

Afterwhat we download the OSM datas from [GeoFabrik downloads page](http://download.geofabrik.de/). 
I choosed this provider because it allow to download datas for a Country or a Sub Region only.

So in order to download the data (for France/Auvergne in my example) use : 
```
wget http://download.geofabrik.de/europe/france/auvergne-latest.osm.pbf -O /opt/osm_data/auvergne.osm.pbf
```

Download the data is a good thing but now we need to automatically update it daily for get the latest modifications.
In order to do that, we use the tool **osmupdate** provided by OSM which allow to merge modifications instead of re-download the whole file.

To install this tool, use :
```
apt-get install osmctools
```

And to planify the daily update, we will create a script in **/etc/cron.daily** folder : 
```
#!/bin/bash

osmupdate --base-url=download.geofabrik.de/europe/france/auvergne-updates /opt/osm_data/auvergne.osm.pbf /tmp/new_auvergne.osm.pbf
mv /tmp/new_auvergne.osm.pbf /opt/osm_data/auvergne.osm.pbf
```

Now you have your OSM datas in the **/opt/osm_data** folder and they are daily updated.
