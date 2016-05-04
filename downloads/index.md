---
title: Wayfinder Downloads
layout: single
---

## TeleAtlas data

[Download TeleAtlas test data](teleatlas.html)

## Map downloads

### Raw data

* The OpenStreetMap Geofabrik shape data for Denmark and Sweden downloaded in May 2010  
  [Denmark](/downloads/files/OSMGeofabrik201005shape/denmark.shp.zip) and [Sweden](/downloads/files/OSMGeofabrik201005shape/sweden.shp.zip)
* The OpenStreetMap Geofabrik shape data for Denmark and Sweden converted to midmif  
  [Denmark](/downloads/files/OSMGeofabrik201005midmif/OSM201005midmifDK.zip) and [Sweden](/downloads/files/OSMGeofabrik201005midmif/OSM201005midmifSE.zip)
* Sample shape data for central Paris (directory f2075) and Skåne in the south of Sweden (directory sw212)  
  [The Tele Atlas MultiNet 2010.06 Europe map release](/downloads/teleatlas.html)
* Shapefiles for Paris have been converted to midmif files, stand-alone dbf-files have been converted to txt-files. Only a subset of the shape files have been converted, the ones that are needed for the current conversion to Wayfinder midmif  
  [The Tele Atlas MultiNet shape data converted to midmif](/downloads/teleatlas.html)
* The world borders shape data from Mapping Hacks downloaded in May 2010.  
  [worldborders201005shape](/downloads/files/worldborders201005shape/TM_WORLD_BORDERS-0.2.zip)
* The country polygons of Denmark, France and Sweden as extracted from the world borders shape data.  
* [worldborders201005countrypolygonmifs](/downloads/files/worldborders201005countrypolygonmifs/countrypolygons_dk_fr_se.zip)

### Wayfinder midmif format files

The Wayfinder midmif format files, and CPIF files for POIs

* The resulting files for Denmark from OSM midmif when parsed with the midmif_osm2wayfinder.pl script.  
  [OSM201005DK](/downloads/files/wf_midmifs/OSM201005DK.zip)
* The resulting files for Sweden from OSM midmif when parsed with the midmif_osm2wayfinder.pl script.  
  [OSM201005SE](/downloads/files/wf_midmifs/OSM201005SE.zip)
* The resulting files for France (central Paris) from Tele Atlas midmif when parsed with the midmif_taMNshape2wayfinder.pl script.  
  [TA201006FR](/downloads/teleatlas.html)

### Setting files

The following three files: 

1. [OSMdenmark](/downloads/files/mapGenSettingFiles/OSMdenmark.zip)
2. [OSMsweden](/downloads/files/mapGenSettingFiles/OSMsweden.zip)
3. [TAfrance](/downloads/files/mapGenSettingFiles/TAfrance.zip)

are used for the following steps:

* parse variable file for converting supplier midmif to Wayfinder midmif
* variable file for map generation
* municipal midmif areafiles
* add region (ar) and create overview (co) xml files


[merge_OSM_dkse_TA_fr_settings](/downloads/files/mapGenSettingFiles/merge_OSM_dkse_TA_fr_settings.zip) handles:

* create overview (coo) xml file for super overview map
* copyright bounding box xml file
* street segment stitch file for fixing the gap in the bridge between Denmark and Sweden in OSM maps.

### Results

mcm files containing the results from different map generations:

* The final mcm maps from the first generation of Denmark from OSM, including backups after_mapDataExtr and before_merge:  
  [OSM201005_dk_first](/downloads/files/mcm/OSM201005_dk_first.zip)
* The final mcm maps from the second generation of Denmark from OSM, including the before_merge backup which is used as input for merge map generation:  
  [OSM201005_dk_second](/downloads/files/mcm/OSM201005_dk_second.zip)
* The final mcm maps from the second generation of Sweden from OSM, including the before_merge backup which is used as input for merge map generation:  
  [OSM201005_se_second](/downloads/files/mcm/OSM201005_se_second.zip)
* The final mcm maps from the second generation of France from Tele Atlas, including the before_merge backup which is used as input for merge map generation:  
  [TA201006_fr_second](/downloads/teleatlas.html)
* The final mcm maps, m3 maps and search and route caches for the merge of OSM Denmark and Sweden and TA France:  
  [merge_OSM_dkse_TA_fr](/downloads/teleatlas.html)
