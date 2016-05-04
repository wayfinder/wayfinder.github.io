---
title: Generate maps for the MC2 backend
layout: single
---

## Introduction

Maps can be created from any supplier delivering map data in shape or midmif
map format.  Shape data needs to be converted to midmif as a first step. The
supplier midmif data is then converted to Wayfinder midmif format, from which
mcm maps are created.  POI data from the supplier is imported into the POI
database, and added to the mcm maps during map generation.  To fix problems in
the maps, map corrections can be added to the mcm maps.  Finally the mcm maps
are converted to the m3 map format that is used in the MC2 back-end.

![map_generation_process_overview](/images/map_generation_process_overview.png)

The supplier map data need to meet some requirements for location based
services in order to be really good as material for the MC2 back-end server.
There are different requirements for searching, routing, navigation and map
display.

You will need binaries and scripts from the MC2 repo. For dealing with shape
data you need a software which can display the shape files, do some simple
selections and editing, and convert from shape to midmif.

The map generation scripts are generalized to handle map generation from
different suppliers/countries/map releases, so you need to set up a genfiles
structure for everything to work. It is a bit time consuming getting everything
into place in the structure, but once you have done it, it is easy to fill in
with more countries and map releases.

The process for the map conversion is described using OpenStreetMap shape map
data from Geofabrik from May 2010 as an example. Additional examples for Tele
Atlas MultiNet shape data map release 2010.06 are also provided.

## Task list for map conversion

This is what needs to be done to produce maps for the MC2 back-end server (in
this order):

1. [Get supplier map data in midmif format](#get-supplier-map-data-in-midmif-format)
2. [Convert the supplier midmif to Wayfinder midmif format](#convert-the-supplier-midmif-data-to-wayfinder-midmif)
3. [Run individual country first generation](#run-individual-countries-first-map-generation)
4. [Test the first generation](#test-the-individual-countries-map-generation)
5. [Add supplier POI data to the WASP database](#poi-handling)
6. [Migrate map corrections from previous map release so they fit the new release](#map-correction-handling)
7. [Run individual country second generation](#run-individual-countries-second-map-generation)
8. [Test the second generation](#test-the-individual-countries-map-generation)
9. [Run a merge map generation combining maps from several countries second map generations](#run-merge-map-generation)
10. [Test the merge map generation](#test-the-merge-map-generation)

## Prepare genfiles structure, setting files

Create the `BASEGENFILESPATH` genfiles with its sub directories.
`BASEGENFILESPATH` genfiles is the base directory where all setting files for
the map generation are stored. All map generation scripts that has the
`BASEGENFILESPATH` defined need to be updated to point to the full path of where
you create the `BASEGENFILESPATH`.

* All hosts used in map generation need to have access to the `BASEGENFILESPATH`.
* It is a good idea to have the `BASEGENFILESPATH` on storage that is backed up or preferably version controlled separately.

Directort structure of `BASEGENFILESPATH´
	
	genfiles/countries
	genfiles/countrypol
	genfiles/extradata/stitching
	genfiles/mergeWorld
	genfiles/script
	genfiles/xml

With the following content:

* `genfiles/countries`  
  Country directories with country specific setting files for the different map releases that are available for the country. The name of the country directories for the 227 countries in Wayfinder world are defined in the makemaps.mcgen wf_world countrySet list. (Antarctica is a possible country, however not wanted in the maps on Wayfinder server)
* `genfiles/countrypol`  
  Country polygon border break point file.
* `genfiles/extradata/stitching`  
  Files for stitching between countries from different map suppliers to fill gaps in the street network.
* `genfiles/mergeWorld`  
  Setting files for merge map generations.
* `genfiles/script` (with sub dir genfiles/script/perllib)  
  Directory with all sub scripts and Wayfinder Perl modules needed in map
  generation and POI handling, typically scripts that are located in mc2 repo
  `Server/bin/Scripts/MapGen` with addition to some other scripts from `Server/bin`
  and `Server/bin/Scripts`. We have this script dir, so we can run main scripts
  from any directory and have all the subs in a static place.  Recommended is
  to set up some routine that makes sure that the scripts and Perl modules are
  up-to-date in this dir.  
  All map generation scripts from the mc2 repo that refers to the
  `BASEGENFILESPATH/script`, or includes perl modules from
  `BASEGENFILESPATH/script/perllib` via ´use lib`, need to be updated to point
  to the full path of where you created the `BASEGENFILESPATH`.
* `genfiles/xml`  
  Directory with global xml setting files for map generation.

Structure in `genfiles/countries`, example for one country (Denmark) with one specific map release OSM_201005 (OpenStreetMap data from May 2010)

	genfiles/countries/denmark
	genfiles/countries/denmark/countrypol
	genfiles/countries/denmark/mapssicoords
	genfiles/countries/denmark/OSM_201005
	genfiles/countries/denmark/OSM_201005/areafiles
	genfiles/countries/denmark/OSM_201005/xml
	genfiles/countries/denmark/script

With the following content:

* `genfiles/countries/denmark`  
  The countries dir for Denmark, the country name as defined in the
  `makemaps.mcgen wf_world countrySet` list
* `genfiles/countries/denmark/script`  
  Setting files for map generation defining where to find map data, the name of
  the maps to be created for the country, and some other params needed in map
  generation.
* `genfiles/countries/denmark/countrypol`  
  The country polygon mif file, containing geometry for the land areas of the
  country (mainland and islands)
* `genfiles/countries/denmark/mapssicoords`  
  Map release specific files with mapssi coordinates which typically are used
  when stitching in a merge map generation. It is a text file (one for each map
  release) listing for each needed mcm map (decimal id) a mc2 coordinate
  (lat,lon) that is on a large street segment (within 2 meters from the street
  segment).
* `genfiles/countries/denmark/OSM_201005`  
  The country map release directory
* `genfiles/countries/denmark/OSM_201005/areafiles`  
  Areafiles, i.e. map division files for the map release. Defines which mcm
  maps that will be generated and which municipals that are part of each mcm
  map.
* `genfiles/countries/denmark/OSM_201005/xml`  
  Xml files for the map release, with info about the map hierarchy and name of
  the country in different languages

The scripts `adminworld.pl` and `maps_arXMLupdate.pl` can be used to create the
`genfiles/countries` structure for many countries at the same time, and also
create some of the setting files needed in the genfiles country structure.

So, a summary for doing the first setup of the genfiles structure:

1. Create `BASEGENFILESPATH` genfiles somewhere, preferably a git repo or similar
2. Create the sub dirs of `BASEGENFILESPATH`
3. Add the `PerlTools.pm` perl module to `genfiles/script/perllib`
4. Go to the `BASEGENFILESPATH/countries` directory and run the `adminworld.pl` to create all the countries
    1. `adminworld.pl -m createCountryDirs -v OSM_201005 |& tee adminworld_createCountryDirs.log`
    2. The `-v` option specifies the name of the country map release directory
    3. You need to update the `use lib` statement in `adminworld.pl` to point
       to the `BASEGENFILESPATH/script/perllib` so the script can find the
       required perl module.

## Get supplier map data in midmif format

Suggested to store the supplier data in a country folder in `SUPPLIERRELEASEDIR`
`OSM_21005`. If shape data store it in eg `OSM_201005/shape/DK`, if midmif data
store it in `OSM_201005/midmif/DK`.

### Get OpenStreetMap shape data

OpenStreetMap maps in shape can be found eg at Geofabrik and Cloudmade. From
those, thegofabrik style is the better one for creating maps for Wayfinder
server.

Download OpenStreetMap shape files from [Geofabrik](http://www.geofabrik.de/data/download.html)
and store in a shape country folder in `SUPPLIERRELEASEDIR` like `OSM_201005/shape/DK` (where `DK` is Denmark).

These are the shape files that are available from Geofabrik, and the OSM attributes available in each of the files:

* `buildings.shp`  *(osmid, name, type)*
* `natural.shp`    *(osmid, name, type)*
* `places.shp`     *(osmid, name, type, population)*
* `points.shp`     *(osmid, timestamp, name, type)*
* `railways.shp`   *(osmid, name, type)*
* `roads.shp`      *(osmid, name, ref, type, oneway, bridge, maxspeed)*
* `waterways.shp`  *(osmid, name, type, width)*

This Geofabrik OSM map data is not optimal as source for maps in Wayfinder
server for several reasons. There are no administrative areas which means that
we will have a very poor search index in the maps. The street network is not
topologically correct, eg street segments are not split in crossings with
shared nodes. Not all features and attributes from OSM data are included, eg
residential areas, airport grounds and runways, golf courses, cemetery, and
industrial areas are missing. A special note for Sweden is that the the big
lake Vänern is missing.

### Get Tele Atlas MultiNet shape data

Small samples from the Tele Atlas MultiNet Europe 20010.06 map release are
available on the [wayfinder.org](http://wayfinder.org] site. It is samples for
central Paris in France and Skåne in south of Sweden.
Store the data in a shape country folder in `SUPPLIERRELEASEDIR` eg
`TeleAtlas/2010_06_eu_shp/shape/FR` (where `FR` is France). 

If you want other areas, complete countries etc, please contact a Tele Atlas
sales office. Start at [www.tomtom.com](http://www.tomtom.com)

### Get shape data for country polygons

Data for the [country polygon](../other_map_related_documentations#the-country-polygon) for one country
is preferably taken from the same supplier that is providing the map data for
that country. In that way the coastal line fits the maps data, and we don't end
up with street segments etc "in the ocean" which happens if the country polygon
does not match the map data.

#### Country polygons for OSM map data

The OpenStreetMap data from Geofabrik does not contain any area features that
can be used as country polygons, so we need to take other action.

Combine the OSM map data with country polygons from world borders shape data.
It can be found for download at [Thematic Mapping](http://www.thematicmapping.org) or
from [Mapping Hacks](http://www.mappinghacks.com/data).

* Get the `TM_WORLD_BORDERS-0.2.zip` (or 0.3 or whatever version is available). The key things to consider: 
    * Pick the most detailed world borders you can find.
    * In the shape file each country must be represented with only one shape feature, else you need to merge.

This world border map data is very generalized and parts of the map data from
Geofabrik OSM are outside the country polygons.

#### Country polygons for Tele Atlas map data

For complete countries, you follow the instructions in the [country polygon](other_map_related_documentations#the-country-polygon)
section. To create a country polygon before the first map generation, you can
merge any of the orderX areas (eg order8 or order1) and subtract ocean waters.

For partial countries, try to use the world borders shape data. If your Tele
Atlas data is inside the country, like the Paris sample, the rough world
borders shape data country polygon will work. If your Tele Atlas data is a
coastal area, like the Skåne south Sweden sample, you would need to combine the
rough world borders shape data with the Tele Atlas orderX areas to get a nice
country polygon that fits the map data.

### Convert the supplier shape data to midmif

Convert thegofabrik OSM and Tele Atlas shape files to midmif (MapInfo
Interchange Format) and store in a midmif country folder in `SUPPLIERRELEASEDIR`
eg:

* for OSM maps `OSM_201005/midmif/DK` (DK = Denmark `CC_CODE` = the alpha2 ISO country code of the country)
* for Tele Atlas maps `TeleAtlas/2010_06_eu_shp/midmif/FR`
    * Tele Atlas MultiNet shape has many files, convert only the ones you need
      in conversion to Wayfinder midmif and further on in map generation.

Convert the country polygons to midmif files, with one country per file.

## Convert the supplier midmif data to Wayfinder midmif

Supplier midmif is converted to Wayfinder midmif with a set of midmif parse
scripts and a parse variable file with settings. The POIs from the supplier are
written to a Wayfinder CPIF file.

### Set up and fill parts of the genfiles structure

These sub scripts and Perl modules need to be in place in the genfiles structure for the conversion to Wayfinder midmif format:

* `genfiles/script`
    * `mfunctions`
    * `mlog`
* genfiles/script/perllib`
    * `GDFPoiFeatureCodes.pm`
    * `PerlCategoriesObject.pm`
    * `PerlCountryCodes.pm`
    * `POIObject.pm`
    * `PerlTools.pm`
    * `PerlWASPTools.pm`

Edit the `mfunctions` and `mlog`:

* `BASEGENFILESPATH` to point to the full path of the genfiles
* `TEMPLOGFILEPATH`` to point to a dir where temp log files can be written

Edit `PerlWASPTools.pm` and define the POI SQL database settings `dbname`,
`dbhost`, `dbuser` and `dbpw` in the `db_connect()` function for connecting to
the WASP database.

### Define the parse variable file

The parse variable file, named eg 'parseMidMif_OSM_201005.sh' can be put in
the genfiles country
`structuregenfiles/countries/COUNTRY/script/parseMidMif_MAPRELEASE.sh` ie
`genfiles/countries/denmark/script/parseMidMif_OSM_201005.sh`. It defines where
to find the supplier midmif data, which country it is, which files to use in
the conversion and if there are any supplier midmif files that have *Pline
Multiple* mif features which must be resolved (because Wayfinder map generation
cannot handle it).  The `CC_CODE` is the alpha2 ISO country code of the
country.

Example of the parse variable file for OSM Denmark:
	
	# Define OSM data directories and file names
	CC_CODE="DK";
	RELEASEDIR="fullpath/OSM_201005";
	MIDMIFDIR="${RELEASEDIR}/midmif/${CC_CODE}";
	# normal items
	NETWORKMID="${MIDMIFDIR}/roads.mid";
	NETWORKMIF="${MIDMIFDIR}/roads.mif";
	RAILWAYMID="${MIDMIFDIR}/railways.mid";
	RAILWAYMIF="${MIDMIFDIR}/railways.mif";
	NATURALSMID="${MIDMIFDIR}/natural.mid";
	NATURALSMIF="${MIDMIFDIR}/natural.mif";
	WATERWAYSMID="${MIDMIFDIR}/waterways.mid";
	WATERWAYSMIF="${MIDMIFDIR}/waterways.mif";
	BUILDINGSMID="${MIDMIFDIR}/buildings.mid";
	BUILDINGSMIF="${MIDMIFDIR}/buildings.mif";
	# pois
	CITYCENTREMID="${MIDMIFDIR}/places.mid";
	CITYCENTREMIF="${MIDMIFDIR}/places.mif";
	POIMID="${MIDMIFDIR}/points.mid";
	POIMIF="${MIDMIFDIR}/points.mif";
	# grep for Pline Multiple in mif files
	#PLINEMULTIPLES="";


Example of the parse variable file for Tele Atlas MultiNet shape data, the small Paris sample in France:

	# Define directories and file names
	RELEASEDIR="fullpath/TeleAtlas/2010_06_eu_shp";
	CC_CODE="FR";
	TACC_CODE="F2075";
	TACC_CODES="f2075";
	MIDMIFDIR="${RELEASEDIR}/midmif/$CC_CODE";
	
	# normal items
	ADMINALTNAMES="${MIDMIFDIR}/${TACC_CODES}_an.txt";
	MUNICIPALMID="${MIDMIFDIR}/${TACC_CODE}_A8.MID";
	MUNICIPALMIF="${MIDMIFDIR}/${TACC_CODE}_A8.MIF";
	BUAMID="${MIDMIFDIR}/${TACC_CODE}_BU.MID";
	BUAMIF="${MIDMIFDIR}/${TACC_CODE}_BU.MIF";
	CITYPARTMID="${MIDMIFDIR}/${TACC_CODE}_A9.MID";
	CITYPARTMIF="${MIDMIFDIR}/${TACC_CODE}_A9.MIF";
	
	MANOUVERS="${MIDMIFDIR}/${TACC_CODE}_MN.MID";
	MANOUVERSPATH="${MIDMIFDIR}/${TACC_CODES}_mp.txt";
	NETWORKMID="${MIDMIFDIR}/${TACC_CODE}_NW.MID";
	NETWORKMIF="${MIDMIFDIR}/${TACC_CODE}_NW.MIF";
	ELEMINAREA="${MIDMIFDIR}/${TACC_CODES}_ta.txt";
	
	LANDCOVERMID="${MIDMIFDIR}/${TACC_CODE}_LC.MID";
	LANDCOVERMIF="${MIDMIFDIR}/${TACC_CODE}_LC.MIF";
	LANDUSEMID="${MIDMIFDIR}/${TACC_CODE}_LU.MID";
	LANDUSEMIF="${MIDMIFDIR}/${TACC_CODE}_LU.MIF";
	RAILWAYMID="${MIDMIFDIR}/${TACC_CODE}_RR.MID";
	RAILWAYMIF="${MIDMIFDIR}/${TACC_CODE}_RR.MIF";
	WATERAREAMID="${MIDMIFDIR}/${TACC_CODE}_WA.MID";
	WATERAREAMIF="${MIDMIFDIR}/${TACC_CODE}_WA.MIF";
	WATERLINEMID="${MIDMIFDIR}/${TACC_CODE}_WL.MID";
	WATERLINEMIF="${MIDMIFDIR}/${TACC_CODE}_WL.MIF";
	
	# pois
	CITYCENTREMID="${MIDMIFDIR}/${TACC_CODE}_SM.MID";
	CITYCENTREMIF="${MIDMIFDIR}/${TACC_CODE}_SM.MIF";
	POIMID="${MIDMIFDIR}/${TACC_CODE}_PI.MID";
	POIMIF="${MIDMIFDIR}/${TACC_CODE}_PI.MIF";
	PIEA="${MIDMIFDIR}/${TACC_CODES}_piea.txt";
	
	# grep for Pline Multiple in mif files
	#PLINEMULTIPLES="";


### Run the parse script

Create a parse directory where to run the conversion from supplier midmif to Wayfinder midmif, in SUPPLIERRELEASEDIR eg `OSM_201005/parse`. Get the needed parse scripts:

* OSM
    * The autoparse script `midmif_autoParseToWF`, the OSM parse script `midmif_osm2wayfinder.pl` and `midmif_fixPlineMultiple.pl`.
* Tele Atlas shape
    * The autoparse script `midmif_autoParseToWF`, the TA parse script `midmif_taMNshape2wayfinder.pl` and `midmif_fixPlineMultiple.pl`.

Then edit:

* `midmif_autoParseToWF` 
  Update `BASEGENFILESPATH` for pointing to mfunctions.
  If parsing a new release, add the release
* The supplier parse script `midmif_osm2wayfinder.pl` and/or `midmif_taMNshape2wayfinder.pl`:
    * For OSM only, define language `$m_langStr` for the country `$opt_c` in parsing of the 
      `-l` option (missing $opt_l), and the WASP database language id (since OSM does not provide 
      name language for name strings)
    * Change the `use lib` statement to include Perl modules in the `BASEGENFILESPATH/script/perllib`
    * If parsing a new release, add the release and column order of all types
      to the script (else it will die)

Run the conversion in 5 steps, with the `midmif_autoParseToWF` script:
	
#### OSM

	   ./midmif_autoParseToWF -mapRelease OSM_201005 -coordSys wgs84_latlon_deg
	   -parseVarFilegnfiles/countries/denmark/script/parseMidMif_OSM_201005.sh
	   -start 1 -stop 1
	   |& genfiles/script/mlog step1_gf_dk

#### Tele Atlas

	   ./midmif_autoParseToWF -mapRelease TA_2010_06 -coordSys wgs84_lonlat_deg
	   -parseVarFile fullPath/genfiles/countries/france/script/parseMidMif_TA_2010_06.sh
	   -start 1 -stop 1  
	   |& genfiles/script/mlog step1_ta_fr

Continue with step 2, 3 and 4 (change the `-start` and `-stop` options) to
prepare for and to parse the normal items. Then run step 5 to parse the POIs.

If step 4 (normal items) or 5 (POIs) dies, read the logfile of the type that was parsed
`wf_midmif/DK/create/logpath/parse_type.log` and fix the problem in the
supplier parse script (`midmif_osm2wayfinder.pl` or `midmif_taMNshape2wayfinder.pl` respectively). 
Then run the step again, until everything is parsed without exits.

The auto-parse script step 4 creates the Wayfinder WF midmif files in a `wf_midmif`
country folder in `SUPPLIERRELEASEDIR` eg `OSM_201005/wf_midmif/DK/create` 
and `TeleAtlas/2010_06_eu_shp/wf_midmif/FR/create`

The auto-parse script step 5 creates a Wayfinder CPIF file (Wayfinder
complex POI import format) with POIs to be added to the POI database. It is
created in a sub directory to the `wf_midmif` country folder, eg `OSM_201005/wf_midmif/DK/poi` 
and for Tele Atlas `TeleAtlas/2010_06_eu_shp/wf_midmif/FR/poi`

For step 5, parsing POIs, there is room for improvements especially for
parsing OSM POI types to Wayfinder POI types. A lot of OSM POI types are
currently set to non-existing Wayfinder POI type -1 which means that those
POIs will not be included in the resulting POI import file.

These are the step 4 Wayfinder midmif item files to use in map generation. It is ok for
the files to be empty (mif files only containing the mif header). Note the name
of the item files. The midmif items will be added to the mcm maps from the
files in alphabetical order.

	
#### OSM

	  WFferryItems.mid/mif
	  WFforestItems.mid/mif
	  WFindividualBuildingItems.mid/mif
	  WFparkItems.mid/mif
	  WFrailwayItems.mid/mif
	  WFstreetSegmentItems.mid/mif
	  WFstreetSegmentItemsturntable.txt
	  WFwaterItems_all.mid/mif

#### Tele Atlas

	   areaWFbuiltUpAreaItems.mid/mif
	   areaWFcityPartItems.mid/mif
	   WFaircraftRoadItems.mid/mif
	   WFairportItems.mid/mif
	   WFbuildingItems.mid/mif
	   WFcartographicItems.mid/mif
	   WFferryItems.mid/mif
	   WFforestItems.mid/mif
	   WFindividualBuildingItems.mid/mif
	   WFislandItems.mid/mif
	   WFparkItems.mid/mif
	   WFrailwayItems.mid/mif
	   WFstreetSegmentItems.mid/mif
	   WFstreetSegmentItemsturntable.txt
	   WFwaterItems_all.mid/mif



Move them to the `MAPPATH` in map generation (defined in the [variable
file](../wayfinder_midmif_format#variable-file)), which is `wf_midmif/DK` and
`wf_midmif/FR` respectively.  Do clean up according to instructions from the
step 4, eg:

* remove any crap from the end of WF mif files (cosmetic mif features)
* make sure that the `CoordSys` tag is one that map generation can read, eg
  `wgs84_latlon_deg` or `wgs84_lonlat_deg`. See `gSGfxData::readMifHeader()`
  for all available coord sys strings.

### Create the midmif areafiles from Wayfinder municipal midmif file

The so called midmif areafiles are required for map generation from WF midmif. They are
used for initializing the maps, i.e. decides thegographic extent of each
of the mcm map. So the input for the WF municipal midmif files needs to
be covering all of the country land area (mainland and islands).
It is perfectly OK if they also cover coastal ocean areas and/or areas between
islands as long as those areas are part of the country's territory (territory = it must not
overlap areas covered by another country's midmif areafiles. In this way also street segments
that are part of bridges between mainland and/or islands will be *inside* the mcm map, which
is really good in the map generation.

The name of the midmif areafiles is `SOMETHING_municipalItems`.
The `municipalItems` is required for the map generation to detect which type
of item to create from the midmif file, in this case municipals. See
ItemTypes::getItemTypeFromString to find the strings for all item types.
The structure of SOMETHING is COUNTRYCODE_someName. The COUNTRYCODE
is the same as is specified in the variable file, the setting file for
country map generation. The someName part of the name should be descriptive
for the content of the areafile. If having only one map in the country
a good name would be eg `dk_whole_municipalItems`.
The name of the resulting mcm map will be COUNTRYCODE_someName, parsing the
midmif areafile name to define the mcm map name is done in the makemaps script.

Ideally there would be administrative areas data from the map supplier resulting in a
WF municipal midmif file with all municipals in the country. Move or copy the WF
municipal mid and mif file to the muncipal directory in `wf_midmif/DK/municipal`.

#### Create WF municipal files for OSM maps from the world_boundry data

There are no administrative areas from Geofabrik OSM shape files, so the best thing to do
is to use the country from the world_boundry shape file.

From the world_boundry shape file, select the country/countries you are
interested in. Convert each country to midmif and use it as the WF municipal
midmif file. Each country must have a midmif file of its own; `denmark.mid/mif`, 
`sweden.mid/mif`.

Edit the country mid file to have the structure of a WF municipal mid file,
eg for Denmark change:

    "DA","DK","DNK",208,"Denmark",4243,5416945,150,154,9.264,56.058
 1. >
    208,"Denmark","Denmark:officialName:eg

and for Sweden

    "SW","SE","SWE",752,"Sweden",41033,9038049,150,154,15.270,62.011
 1. >
    752,"Sweden","Sweden::officialName:eg

#### Create the midmif areafiles for OSM maps

Create a working directory for the areaUpdate eg `OSM_201005/areaUpdateDK`. 
Copy the script midmif_areaFilesNewRelease and copy or at least locate where 
to find the MifTool binary.

Use the WF municipal midmif file with all municipals from the country. For thegofabrik OSM data, use the edited world_boundry country midmif file.

If possible create a file `mapDivision.txt` in the areaUpdate directory.
The mapDivision.txt is deciding which municipals to include in which
midmif municipal areafile, i.e. which municipals should be in which mcm map
when running individual country map generation for the country. Each line of
the mapDivision.txt holds "midmifAreafileName municipalID", eg for the
municipal with ID 208 to end up in the areafile dk_whole_municipalItems
`dk_whole_municipalItems 208`

Then run the midmif_areaFilesNewRelease script.

In the full run midmif_areaFilesNewRelease can create a mapDivision.txt file from
previous map release to re-use the map division for the new (current) map
release. This can only be used if you have an old release of the current one and
municipal IDs are stable over the release versions.

	
	./midmif_areaFilesNewRelease
	-oldReleaseDir genfiles/countries/denmark/OSM_201002
	-munMidFile OSM_201005/wf_midmif/DK/municipals/denmark.mid
	-munMifFile OSM_201005/wf_midmif/DK/municipals/denmark.mif
	-varFilegnfiles/countries/denmark/script/variables_denmark.OSM_201002.sh
	-mifTool binDir/MifTool
	|& genfiles/script/mlog areaUpdateFull


If you created the mapDivision.txt manually, run with the `-onlyCreateMidMifAreaFiles` option

	
	./midmif_areaFilesNewRelease
	-onlyCreateMidMifAreaFiles
	-oldReleaseDir XXX
	-munMidFile OSM_201005/wf_midmif/DK/municipals/denmark.mid
	-munMifFile OSM_201005/wf_midmif/DK/municipals/denmark.mif
	-mifTool binDir/MifTool
	-varFile XXX
	|& genfiles/script/mlog areaUpdateShort_dk


This results in the midmif areafiles

* `dk_whole_municipalItemsmap.mif`
* `dk_whole_municipalItems.mif`
* `dk_whole_municipalItems.mid`

Move these areafiles to the genfiles country map release areafiles dir
eg `genfiles/countries/denmark/OSM_201005/areafiles`.

#### Create the midmif areafiles for Tele Atlas Paris maps

This was an easy case, since we have only one municipal (from the Tele Atlas order 8 feature) in the Paris sample. And With only one municipal in the midmif areafile, the map.mif file equal to the .mif file. 

* rename the `WFmunicipalItems.mid/mif` to `fr_centralparis_municipalItems.mid/mif`
* copy the `fr_centralparis_municipalItems.mif` to `fr_centralparis_municipalItemsmap.mif`

Move these areafiles to the genfiles country map release areafiles dir
eg `genfiles/countries/france/TA_2010_06/areafiles`

#### Create the midmif areafiles for generic maps

If it is a large country, the country need to be split in more than one map,
else the mcm maps in the map generation will be too large to handle. The limit 
for *large* can be approx more than 500000 street segments or whatever limit you 
find out that your system has. It does not only depend on number of street segments 
but also how many other item types you have in your maps.

If you have several municipals from the map supplier in the WF municipal item
file, simply put the municipals in different midmif areafiles by specifying
different `midmifAreafileName` for the municipal IDs in the `mapDivision.txt`. One
midmif areafile will be one mcm map, it is OK to have many municipals in one
midmif areafile. If splitting into many areafiles, try to keep neighboring
municipals in the same areafile and try to give the areafiles a descriptive
name eg west part of the country in `dk_west_municipalItems` and east part of the
country in dk_east_municipalItems. See the OSM example above for howto run the
`midmif_areaFilesNewRelease` script.

If you only have one municipal from the map supplier for a large country, eg as
for thegofabrik OSM data case, you need to be creative. One option is to split
the country shape file in several parts (as many as you need to havegod mcm map
sizes). Each part must then have unique ID in the WF municipal item mid file.
It is OK if they still all have the same name *Denmark*.

In countries with more than one mcm maps, the midmif items are added to the
correct map based on geometry or via settlement ids. Geometry will not always
work to get eg large bridge street segments and ferry lines into a mcm map. If
the supplier map data does not provide information for settlementID, like
thegofabrik OSM data, then you need to be creative again and find a way to
define settlement ids for street segments pointing at the municipal ids of the
mcm map where you want the street segments to be added.

### Suggestions on things to improve in the supplier parse scripts

For Tele Atlas, the parse script `midmif_taMNshape2wayfinder.pl` can be improved
to add more things that are supported by the Wayfinder midmif format and CPIF
file. These are marked with uppercase fixme in the parse script.

* house numbers to the street segments file in the `handleNetworkFile()` method
* zip codes to the street segment file in the `handleNetworkFile()` method
* use more attributes from the PIEA file for POIs, in
  `thegtInfoStringForPIEA()` method. It concerns eg the BQ bus stop type
  attribute, to decide whether a bus stop POI is really a bus stop or a bus
  terminal (change the POI type)
* improve handling of maneuvers in the `handleRestrictions()` method for maneuvers
  that are marked with handledOK=0. That marks that something needs change to
  handle all entries from the maneuvers file properly.

For OpenStreetMap, the parse script `midmif_osm2wayfinder.pl` can be improved:

* The `initPOITypesHash()` method maps the OSM POI types to Wayfinder POI
  types. A lot of OSM POI types are currently set to non-existing Wayfinder POI
  type -1 which means that those POIs will not be included in the resulting POI
  import file. To include more POIs in the CPIF, define appropriate Wayfinder
  POI type id for them.
* Add more OSM types to the types2wfcat hash if appropriate to set specific
  Wayfinder POI category for them. Needs to be set if the Wayfinder POI type is
  not enough to set a good category using the mapping defined in the WASP
  [POICategoryTypes table](the_wasp_database#poicategorytypes-content).

## Prepare for individual country map generation

An individual country generation creates maps from the supplier map data, for
one country.

The result of an individual country generation should not be added to a MC2
back-end production server. Some necessary info is not added in the country
generation, so you need to do a merge map generation of your countries to get
maps for the MC2 back-end production server.

### Scripts in the genfiles structure

Additional scripts used in individual country map generation in 

* `genfiles/script` (don't forget to update `BASEGENFILESPATH` to point to the full path of the genfiles)
    * `extradata.sh`
    * `maps_countryInfo.pl`
    * `filtermergedMaps.sh`

### Setting files in the genfiles structure

Additional setting files used in individual country map generation in 

*` genfiles/xml`
    * `region_ids.xml` (from `Server/bin/Scripts`)
    * `map_generation-mc2.dtd` (from `Server/bin`)

### Setting files in `the genfiles` country structure

Additional setting files needed in `the genfiles` country structure.

Set up the genfiles country structure, populate the setting files that is needed
for individual country generation.  Remember that the structure and some
setting files can be created for all countries with the scripts `adminworld.pl`
and `maps_arXMLupdate.pl`.

   * `genfiles/countries/denmark`
   * `genfiles/countries/denmark/script`
      * `variables_denmark.OSM_201005.sh`
      * variable file = setting file for map generation eg defining MAPPATH = where to find Wayfinder midmif map data, the name of the maps to be created for the country, and some other params needed in map generation.
      * For Denmark and Sweden from OSM Geofabrik files I specify the `ONLYONEMAP` parameter, which will force all items from the midmif files of the country to be added to the one and only mcm map.
         * positive:   all items added to the one mcm map, else missing roads ets.
         * negative: location will not be set for items outside the admin areas
         * specially need to consider using this attribute for coastal countries
      * Generic variable files can be created for many countries at the same time with the `adminworld.pl -m createVarFile`
   * `genfiles/countries/denmark/countrypol`
      * `denmark.mif`
      * The country polygon mif file, containing geometry for the land of the country (mainland and islands).
      * This should be as detailed as possiblegometry-wise since it is used for displaying land areas in map images. It must contain one mif Region feature only, ok to have as many polygons as is needed in the Region feature.
      * There should be one country polygon mif file for each country overview map in the country, and the mif files should have the country overview map name followed by the suffix .mif. The name of the country overview map is defined in the map release co.xml file. Example in Denmark the country overview map is called denmark so the country polygon mif file is called denmark.mif. A very large country may have more than one country overview map, called eg germany_1 and germany 2, must have country polygon mif files named germany_1.mif and germany_2.mif. Both those country polygon mif files must have the exact same content, ideally let them be links to a file called germany.mif.
      * (08-map)
      * For faster map generation, the country overview map(s) for the country from a "whole-world" merge map generation.
      * The filtering of the country polygon, i.e. the country overview map gfxData is copied from the old country overview map in countrypoldirectory, so map generation can skip the rather time consuming step of filtering of the country polygon. Filtering = remove un-necessary coordinates and pre-store 16 filtering levels in the countrypolygon for fast access of a good filtering level when displaying the country in a map image of any zoom level.
      * Reason for the 08-map must come from a world merge map generation is that neighboring countries must have been filtered together so that the country border parts have the exact samegometry to avoid gaps when displaying the country in a map image. Related: see mfunctions `countryOrder()`, `PerlCountryCodes.pm::countryOrder` vector to see in which order the countries are processed in a merge map generation, i.e. which country border geometry is re-used for the next country.
   * `genfiles/countries/denmark/OSM_201005`
      * the map release directory
   * `genfiles/countries/denmark/OSM_201005/areafiles`
      * Contains the midmif areafiles, i.e. map division files for the map release. Defines which mcm maps that will be generated and which municipals that are part of each mcm map.
      * One midmif areafile consists of .mid .mif and map.mif files. The .mid and .mif file contains the municipal items to add to the mcm map that is created from the areafile. The municipals are stored according to Wayfinder midmif format. The map.mif file contains the outline of the municipals merged in the .mif file, and will be thegxData (thegometry) of the mcm map created from the areafile. The map gfxData is used when adding other midmif items, eg forests, parks, street segments to decide which is the the correct mcm map to add them to. The midmif items are added only if they fall within thegxData of the mcm map.
   * `genfiles/countries/denmark/OSM_201005/xml`
      * Contains xml files with info about the map hierarchy and name of the top region(s) of the country in all languages
      * `map_generation-mc2.dtd`
         * linked to theglobal `genfiles/xml/map_generation-mc2.dtd`
      * `country_co.xml` eg `denmark_co.xml`
         * lists the map hierarchy in the country, according to the create_overviewmap XML spec in map_generation-mc2.dtd. It lists the name of the overview map(s) in overview_ident tag, and for each overview map it lists the name of the underview maps that belong to the overview with the map_ident tag. The map_ident tags may be listed in any order. The underviews are added to the overview in alphabetical order anyway.
         * Example for Denmark the overview_ident (name of the overview map) is "denmark" and the underview for that is map_ident "dk_whole".
         * The `country_co.xml` is map release specific, so this denmark_co.xml is valid only for the OSM_201005 map release. Reson to why it is release specific is that map division is likely to be different if we have maps from another supplier and/or another map release from the same supplier. The co_xml file must contain the name of all underview maps of the specific map release as map_ident tags. See `makemaps` script parsing of `mapName` to find out how the areafile name is parsed into mcm map name. Or, leave the map_ident tags blank until first steps of the individual country generation has been done, when you can use the `whichmaps` script (from `Server/bin/Scripts`) to find out the name of the mcm underview map.
         * The co XML files can be created for many countries at the same time with the `adminworld.pl -m createCoXML`
      * `country_ar.xml` eg `denmark_ar.xml`
         * contains the info needed for setting region identity, according to the add_region XML spec in map_generation-mc2.dtd. The possible regions are defined in the `region_ids.xml` (from `Server/bin/Scripts`) as `region ident`.
         * lists top region region_ident for this country, the available translations to different languages for the top region name (name language and type), and the overview map(s) that belong to this top region content map_ident as defined in co.xml file overview_ident. Example for Denmark the add_region region_ident is "denmark", the translation for language egish officialName is "Denmark", the translation for spanish is "Dinamarca", and the overview maps that are part of this region is content map_ident "denmark" whole_map
         * The translations can be picked from `StringTableUTF8::strings`, using the `StringTable::getCountryStringCode()` to get the stringCode for the specific country. `StringTable::countryCode` is extracted with `MapGenUtil::getCountryCodeFromGmsName()` when generating the map.
         * The add-region XML files can be created for many countries at the same time with the `maps_arXMLupdate.pl -m createArXML`.

### Examples of setting files in `the genfiles` country structure

#### Example of variable file

Example of `the genfiles/countries/denmark/script/variables_denmark.OSM_201005.sh`

	MAPRELEASE="OSM_201005";
	# Long version of the map supplier, needed in variable file
	# to override makemaps default value TeleAtlas_
	MAP_SUPPLIER="OpenStreetMap_"
	
	AREAPATH="${GENFILESPATH}/${MAPRELEASE}/areafiles"
	CO_CODE="dk_"
	
	MAPPATH="fullPath/OSM_201005/wf_midmif/DK"
	
	# Setting ONLYONEMAP means that all items in the item midmifs files will be
	# forced added to the one-and-only mcm map, overriding the other available
	# methods for checking correctMap (within map gfxdata, settlementId,
	# map ssi coord)
	ONLYONEMAP="true"
	
	AREALIST="dk_whole_municipalItems"
	

#### Example of country polygon mif file

Example of the `countries/denmark/countrypol/denmark.mif` as it was created when converting the world_borders shape file to midmif.
	
	VERSION 300
	Charset "WindowsLatin1"
	Delimiter ","
	Coordsys wgs84_latlon_deg
	Columns 11
	  Fips Char (2)
	  Iso2 Char (2)
	  Iso3 Char (3)
	  Un Decimal (3,0)
	  Name Char (50)
	  Area Decimal (7,0)
	  Pop2005 Decimal (10,0)
	  Region Decimal (3,0)
	  Subregion Decimal (3,0)
	  Lon Decimal (8,3)
	  Lat Decimal (7,3)
	Data
	Region 18
	  72
	54.82972000 11.51388700
	54.82193800 11.56444400
	54.87777700 11.64250000
	54.90083300 11.64166600
	54.90388500 11.64194300
	...


#### Example of midmif areafiles

Examples of the midmif areafiles for Denmark OSM_201005 map release as they
were created from the world_borders shape Denmark county, converted to midmif
and then edited to fit Wayfinder municipal midmif files. Finally processed with
the midmif_areaFilesNewRelease script.

`countries/denmark/OSM_201005/areafiles/dk_whole_municipalItems.mid` with the
attributes of all municipal (one) that will be added to the dk_whole mcm map in
map generation.
	
	208,"Denmark","Denmark:officialName:eg

`countries/denmark/OSM_201005/areafiles/dk_whole_municipalItems.mif` with thegometry of all municipal (one) that will be added to the dk_whole mcm map in map generation.

	
	VERSION 300
	Charset "WindowsLatin1"
	DELIMITER ","
	COORDSYS mc2
	COLUMNS 3
	  LOCAL_ID integer(16,0)
	  NAME char(50)
	  ALL_NAMES char(256)
	DATA
	Region 18
	551
	680892285 118997724
	680255126 120779372
	680125967 121057757
	680072900 121183683
	679966862 121445485
	679887298 121747064
	679847522 121906133
	679827682 122028766
	...


countries/denmark/OSM_201005/areafiles/dk_whole_municipalItemsmap.mif with thegometry of the dk_whole mcm map, i.e. the outer line of all municipals in the map. In this case, since we have only one municipal in the dk_whole areafile, the map.mif file equal to the .mif file.

	
	VERSION 300
	Charset "WindowsLatin1"
	DELIMITER ","
	COORDSYS mc2
	COLUMNS 3
	  LOCAL_ID integer(16,0)
	  NAME char(50)
	  ALL_NAMES char(256)
	DATA
	Region 18
	551
	680892285 118997724
	680255126 120779372
	680125967 121057757
	680072900 121183683
	679966862 121445485
	679887298 121747064
	679847522 121906133
	679827682 122028766
	...


#### Example of `co.xml` file

Example of a `countries/denmark/OSM_201005/xml/denmark_co.xml` file:

{% highlight xml %}
	<?xml version="1.0" encoding="ISO-8859-1" ?>
	<!DOCTYPE map_generation-mc2 SYSTEM "map_generation-mc2.dtd">
	
	<map_generation-mc2>
	   <create_overviewmap overview_ident = "denmark">
	      <map map_ident = "dk_whole"/>
	   </create_overviewmap>
	</map_generation-mc2>
{% endhighlight %}

#### Example of `ar.xml` file

Example of a `countries/denmark/OSM_201005/xml/denmark_ar.xml` file:

{% highlight xml %}
	<?xml version="1.0"  encoding="UTF-8"?>
	<!DOCTYPE map_generation-mc2 SYSTEM "map_generation-mc2.dtd">
	
	<map_generation-mc2>
	   <add_region type="country" region_ident="denmark">
	      <name language="czech" type="officialName">Dánsko</name>
	      <name language="danish" type="officialName">Danmark</name>
	      <name language="german" type="officialName">Dänemark</name>
	      <name language="greek" type="officialName">Δανία</name>
	      <name language="egish" type="officialName">Denmark</name>
	      <name language="spanish" type="officialName">Dinamarca</name>
	      <name language="finnish" type="officialName">Tanska</name>
	      <name language="french" type="officialName">Danemark</name>
	      <name language="hungarian" type="officialName">Dánia</name>
	      <name language="italian" type="officialName">Danimarca</name>
	      <name language="dutch" type="officialName">Denemarken</name>
	      <name language="norwegian" type="officialName">Danmark</name>
	      <name language="polish" type="officialName">Dania</name>
	      <name language="portuguese" type="officialName">Dinamarca</name>
	      <name language="russian" type="officialName">Дания</name>
	      <name language="slovenian" type="officialName">Danska</name>
	      <name language="swedish" type="officialName">Danmark</name>
	      <name language="turkish" type="officialName">Danimarka</name>
	      <name language="chinese" type="officialName">丹麦</name>
	      <name language="chinese_traditional" type="officialName">丹麥</name>
	
	      <content map_ident="denmark">
	         <whole_map/>
	      </content>
	   </add_region>
	</map_generation-mc2>
{% endhighlight %}

### Map generation binaries

The following binaries and scripts are needed for individual country map
generation. They are all found in the mc2 repo, the binaries with capital
letter are build artifacts.
Don't forget to update `BASEGENFILESPATH` in misc scripts to point to the full
path of the genfiles directory.

These are the binaries and scripts needed

* `makemaps`        from `Server/bin/Scripts/MapGen`. The main script for individual country map generation.
* `distribute`      from `Server/Tools/distribute´
* `ExtradataExtractor` from `Server/MapGen/ExtradataExtractor`. Needed if to apply map correction records from WASP database.
* `GenerateMapServer`  from `Server/bin`
* `mc2.prop`        from `Server/bin`. Set the MAP_PATH to "./" and define POI SQL database settings.
* `QualTool`        from `Server/MapGen/QualTool? . Needed for analyse/verification of map generation as preparation for test.
* `verify`          from `Server/bin/Scripts/MapGen`
* `Verify`          from `Server/MapGen/Verify`
* `WASPExtractor`   from `Server/MapGen/WASPExtractor`. Needed if to add POIs from WASP database.
* `whichmaps`       from `Server/bin/Scripts`

If generating maps from a new supplier, please add the supplier to the
mc2 repo to the following files and compile new binaries. Example new supplier OpenStreetMap:

* `Server/Shared/include/MapGenEnums.h` `MapGenEnums::mapSupplier()`
* `Server/Shared/src/MC2MapGenUtil.cpp` `MC2MapGenUtil::mapSupCopyrigthStrings()`
* `Server/MapGen/src/MapGenUtil.cpp` `MapGenUtil::initMapSupMapping()`
* `Server/bin/Scripts/map_supplier_names.xml`
    * Add the copyright string for the new supplier, english language. It is
      the copyright string that will be displayed when showing a map image over
      an map area that is provided by the supplier

If generating maps from a new release from a known or new supplier,
please add the map release to the mc2 repo to the following files and compile new binaries.
Example new release OSM_201005 from supplier OpenStreetMap:

* `Server/Shared/include/MapGenEnums.h` `MapGenEnums::mapVersion`  
  If you plan to use the WASP database for POIs and/or map corrections, also
  add the map release to the WASP EDVersion table. The ID in the EDVersion
  table must equal the ID in the `MapGenEnums::mapVersion` enum. The version
  string in the EDVersion table must be the map release with the long version
  of the map supplier string, i.e. the MAP_SUPPLIER defined in the variable
  file, eg `OpenStreetMap_201005` or `TeleAtlas_2010_06`.
* `Server/MapGen/src/MapGenUtil.cpp` `MapGenUtil::getMapVerFromString`
* `Server/bin/Scripts/MapGen/makemaps` parsing of `MAP_RELEASE_ARG`
* `Server/MapGen/src/NationalProperties.cpp`  
  Verify that `NationalProperties::getMapToMC2ChEnc()` gives correct char encoding for the new map supplier/release.

## Run individual countries first map generation

There are two types of individual country generations, a [first
generation](#run-individual-countries-first-map-generation)
and a [second
generation](#run-individual-countries-second-map-generation).
The first generation simply creates maps from the supplier map data (from the
Wayfinder midmif files), the second generation also connects to the WASP
database to add POIs and/or map corrections.

Both the first gen and the second gen can be run in single-mode
country-by-country or in multi-mode with many countries at the same time.

If you have many countries it is recommended to run one country as single
generation to see that you have all setting files etc in place. Then run a
multi generation for the rest of the countries.

All map generations from one map release can be done in the mcm map release
directory eg for OSM mcm/OSM_201005 and for Tele Atlas mcm/TA_2010_06_eu_shp.

### Individual country single first generation

Run a first generation for one country from the map release

   * Create a country generation directory for the map release, eg mcm/OSM_201005/dk_20100520_first
   * Copy the generation binaries to it
   * Run the makemaps script
      * `./makemaps -mapRelease OSM_201005 -noExtFiltOrWASP -dist2 -afterMapDataExtrBackup denmark |& genfiles/script/mlog makemaps_denmark`
      * The country option value is equal to the name of the country directory in genfiles/countries, see the makemaps.mcgen wf_world countrySet list for available countries.
      * The `-dist2` makes some steps in the map generation to be distributed on 2 processors of the localhost
      * The `-afterMapDataExtrBackup` leaves us with a nice backup, which can be used to start the second generation.
      * The `-noExtFiltOrWASP` is the option that defines "first generation", it means no extradata (map corrections) from WASP database or extradata.sh, no POIs from WASP database, and no filtering of the final maps.
   * The mlog creates 2 log files, one simple called `makemaps_denmark.log` and one for debug `makemaps_denmark.dbg.log`. The `dbg.log` file includes all commands and everything that happens and is good to read in case things crash. For all other purposes read the simple makemaps_denmark.log file.

Example for the Tele Atlas France small Paris sample:

     ./makemaps france -mapRelease TA_2010_06 -noExtFiltOrWASP -afterMapDataExtrBackup -dist2 |& genfiles/script/mlog makemaps_france


### Multi first generation

Run first generation for several countries from one map release at the same time. This is done with the makemaps.mcgen script, which sets up and starts generation with makemaps script for all countries.

   * Create a multi directory eg `mcm/OSM_201005/20100603_multiFirst_dk_se`
   * Create the multi countries directory where the countries will be generated eg `mcm/OSM_201005/20100603_multiFirst_dk_se/countries`
   * Create the multi generation directory where the multi generation is done eg `mcm/OSM_201005/20100603_multiFirst_dk_se/1_countries`
      * Copy the `makemaps.mcgen` script here
      * Edit the `makemaps.mcgen` myCountries `COUNTRYLIST` to list all the countries you want to have in the multi generation. The first country is processed first. When one country is finished, the next country in the list is started. List the largest country first and the smallest country last in `COUNTRYLIST`. That way, if you run the multi generation on more than one host (the `-countryGenComputers` option), the distribution will be optimal not having to wait for a large country when all other countries are finished.
   * Create the generation bin directory eg `mcm/OSM_201005/20100603_multiFirst_dk_se/1_countries/bin`
      * Copy the generation binaries to it
   * Goto the multi generation directory and run the makemaps.mcgen script
      * `./makemaps.mcgen -countries myCountries -noMerge -binPath bin -countriesPath ../countries/ -mapRelease OSM_201005  -countryGenComputers "host host" -dist2 -cntrFirstGen |& /home/is/devel/Maps/genfilesPSTC/script/mlog master1_countries`
      * The countries from myCountries COUNTRYLIST will be distributed on 2 processors on the computer named "host". Each country is then run with -dist2, so in total 4 processors on "host" will be used.
      * The `-cntrFirstGen` option tells the `makemaps` script to run with `-noExtFiltOrWASP` and `-afterMapDataExtrBackup` options for each of the countries.


## Run individual countries second map generation

The second generation of a individual country does the same thing as the [first
generation](#run-individual-countries-first-map-generation)
does, but it also connects to the WASP database to add POIs and/or map
corrections. The second gen can be run in single-mode country-by-country or in
multi-mode with many countries at the same time.

### Individual country single second generation

Will contact WASP database for addition of POIs and apply map correction records.

   * Create a country generation directory for the map release, eg `mcm/OSM_201005/dk_20100607_second`
   * Copy the generation binaries to it (don't forget `WASPExtractor` and `ExtradataExtractor`)
   * Run the makemaps script
      * `./makemaps -mapRelease OSM_201005 -noFilt -dist2 denmark |& genfiles/script/mlog makemaps_denmark`
      * Run with `-noFilt` to skip filtering of coordinates in the maps. It is not needed in individual country generation, only necessary in merge map generation.
   * If you want to save time for a large country, it is possible to re-use the `after_mapDataExtr` backup from the first generation. To make this happen add the following to the makemaps command
      * `-fromMapDataExtr -mdeBkpDir mcm/OSM_201005/dk_20100520_first/after_mapDataExtr`

Example for the Tele Atlas France small Paris sample in `mcm/TA_2010_06_eu_shp/fr_20100628_second`:

    ./makemaps france -mapRelease TA_2010_06 -noFilt -dist2 |& genfiles/script/mlog makemaps_france

### Multi second generation

Run second generation for several countries from one map release at the same time.

   * Create the multi directory eg `mcm/OSM_201005/20100608_multiSecond_dk_se`
   * Create the multi countries directory where the countries will be generated eg `mcm/OSM_201005/20100608_multiSecond_dk_se/countries`
   * Create the multi generation directory where the multi generation is done eg `mcm/OSM_201005/20100608_multiSecond_dk_se/1_countries`
      * Copy the `makemaps.mcgen` script here
      * Edit the `makemaps.mcgen` `myCountries` `COUNTRYLIST´ to list all the countries you want to have in the multi generation.
   * Create the multi generation bin directory eg `mcm/OSM_201005/20100608_multiSecond_dk_se/1_countries/bin`
      * Copy the generation binaries to it
   * Goto the multi generation directory and run the makemaps.mcgen script
      * ./makemaps.mcgen -countries myCountries -noMerge -binPath bin -countriesPath ../countries/ -mapRelease OSM_201005  -countryGenComputers "host host" -dist2 |& /home/is/devel/Maps/genfilesPSTC/script/mlog master1_countries
      * The countries from myCountries COUNTRYLIST will be distributed on 2 processors on the computer named "host". Each country is then run with -dist2, so in total 4 processors on "host" will be used.
      * The makemaps script will be started with -noFilt by default.
      * If you want to re-use the `after_mapDataExtr` backup from the first generation of some/all countries, you need to edit the makemaps.mcgen script hardcoding the `mdeBkp` (search for mdeBkp to find the place).



## Test the individual countries map generation

### Verification of mcm maps
Verify the individual country generation with the following actions

   * Check that the log from the makemaps script ends with "Finished country"
   * (Read the qualLog/qualityreport.log). This is an old verification method, and not very informative, it sometimes reports errors that are not errors.
   * Run a `qualDiff` comparing the generation with maps from the previous map release or a previous map generation of the same release.
      * Create and goto a diff-dir in the country directory, where you run the `maps_genQualityCheck` script.
      * Example for comparing the OSM_201005 first generation with the first generation of the previous map release OSM_201002. To see the differences in map data reflecting the updates the supplier has done in the new map release.
         * go to `mcm/OSM_201005/dk_20100520_first/diff_2_OSM_201002_first`
         * `./maps_genQualityCheck -bin .. mcm/OSM_201002/dk_20100215_first .. |& teegnQualCheck.log`
      * Example for comparing the OSM_201005 second generation with the OSM_201005 first generation. To see what differences wegt with POIs and map corrections
         * go to `mcm/OSM_201005/dk_20100607_second/diff_2_first`
         * `./maps_genQualityCheck -bin .. mcm/OSM_201005/dk_20100520_first .. |& teegnQualCheck.log`
      * The script uses the `QualTool` binary to calculate some statistics for the maps so you can see increases and decreases in the maps. Statistics for the `FIRSTMAPS` is written to the `1_[]_MapsQual.log` and the `SECONDMAPS` is written to the `2_[]_MapsQual.log`. These terms `FIRSTMAPS` + `SECONDMAPS` are only a description of the order in which maps are pointed to by the options of the `maps_genQualityCheck` scripts, not a reference to either first or second generation. All differences between first and second maps are written to the `[..]_MapsQualDiff.log` file, the important subset of the diffs is written to the `[]_ShortMapsQualDiff.log` file. If the diff-files are very large, you can use the ``maps_mcmAnalyseQual` script to list only increases/decreases larger than certain number or percentage.
   * Check that all items from the Wayfinder midmif files were added to the mcm maps. Check the makemaps log file "Adding mid/mif items:". Look for the lines saying "Adding from WFforestItems" followed by "added 3234 of 3234". If not all the available items were added to the mcm maps, it is possible to analyze which items were missing. Especially street segments are important that all were added, if you plan on using the maps for navigation. Remember, if the items are added to mcm maps based on geometry, the first coordinate in the item's gfxData must be inside a mcm map gfxData for it to be added.
      * `midmif_tool.pl` can help you list which items were not added to the mcm maps. Run it with option `-w`,  example: `./midmif_tool.pl -w OSM_201005/wf_midmif/DK/WFstreetSegmentItems.mid OSM_201005/wf_midmif/DK/WFstreetSegmentItems.mif mcm/OSM_201005/dk_20100520_first/logpath/create_WFstreetSegmentItems.log |& tee listNotAddedMidItems.log`
      * Result presents the midId, midRow, name (if any) and one coordinate of the not added items.
      * If there are any important items missing, find all coordinates of the item with `CoordConvert -i MIDROW WFstreetSegmentItems.mif`
      * If there are many items missing one approach is to save a file with a list of all first coordinates and view the coordinates in MapEditor with `--highlightCoords` option. The coordinates must be mc2 to load. Example convert a file with wgs84_deg coords to mc2 coords.
         * `CoordConvert --convertTextFile --inFormat=wgs84_deg --outFormat=mc2 --latPos=2 --lonPos=1 --outCoordinateOrder=latlon --fileSep=" " --saveAs=mc2Coords.txt notAdded_water_coords.log`

For a multi first or a multi second generation you can use the
`maps_multiReleaseTasks.pl` script to generate `qualDiff` for all countries at
the same time. For the multi first generation use option `-diffFirstToFirst`,
comparing the map release first generation with the first generation of the
previous map release. For the multi second generation use option
`-diffSecondToFirst`, comparing the second generation with the first generation
of this map release.

   * Goto the mcm map release directory e`g mcm/OSM_201005`
   * Update `maps_multiReleaseTasks.pl` `BASEGENFILESPATH` to point to the full path of the genfiles and the `use lib` statement.
   * Run the `maps_multiReleaseTasks.pl`. It expects to find the `maps_genQualityCheck` and `maps_mcmAnalyseQual` scripts in the `BASEGENFILESPATH/script` directory.
      * `maps_multiReleaseTasks.pl -diffFirstToFirst -prevBaseDir mcm/OSM_201002 -mcmBaseDir mcm/OSM_201005 -binDir mcm/OSM_201005/dk_20100520_first sweden denmark |& tee multiFirstDiff2first_dk_se.log`
      * the country names in tail given as defined in the makemaps.mcgen wf_world countrySet list
      * `-binDir` can point to any directory where there is a `QualTool` binary to use.
   * The script has an automatic routine to find the country generation directories to compare (in this case the `mcm/OSM_201002/dk_20100215_first` and `mcm/OSM_201005/20100603_multiFirst_dk_se/countries/denmark`). It runs a find command in the prevBaseDir and the mcmBaseDir respectively and finds directories that contain the word "first" (not case sensitive) for first generations and either the alpha2 ISO country code of the country or the whole country name (as given in tail to the script). The dk_20100215_first in OSM_201002 is found because the dir name contains "dk_" and "first". The 20100603_multiFirst_dk_se/countries/denmark in OSM_201005 is found because the dir name is "denmark" and it is located in a dir containing "First". If several candidate dirs are found, the script prompts you to decide which of them to use in the comparison.
   * The multi qualDiff creates a directory `analyzeQual_${somename}` in the mcmBaseDir. There you can find some files produced by the `maps_mcmAnalyseQual` script for all countries.
      * Option `-diffFirstToFirst` creates `analyzeQual_diff2other_first`
      * Option `-diffSecondToFirs`t creates `analyzeQual_diff2first`

### Running a test server

To test the individual countries generation on a server, you need to create server m3 maps and search & route caches.

These map generation binaries and scripts are necessary:

* `MapHttpServer   from `Server/bin`
* `MapModule`      from `Server/bin`
* `makemaps.mcgen` from `Server/bin/Scripts/MapGen`
* `mc2.prop.end`   from `Server/bin/Scripts/MapGen`. Set the POI SQL database settings if there are POIs from WASP database in the maps

Goto the country directory, copy the binaries and scripts there and run:

* `./makemaps.mcgen -fromM3 -mapSet 0 -mc2Dir mc2_m17_sr26 |& genfiles/script/mlog cache_0`
* Server m3 maps will be created in the mc2-dir `mc2_m17_sr26`
* Server search and route caches will be created in the cache-dir `mc2_m17_sr26/cache0`
* Nbr of m3 maps equals number of mcm maps, nbr caches twice as many

Put the maps up on a test server. The `mc2.prop` setting file on the server
have `MAP_PATH[_mapset]` pointing at the mc2 dir with the m3 maps and the
`MODULE_CACHE_PATH[_mapset]` pointing at the mc2/cache-dir with the search and
route caches.

For a multi generation you can use the `maps_multiReleaseTasks.pl` script to
create server m3 maps and search & route caches for all countries at the same
time.

   * Goto the mcm map release directory eg `mcm/OSM_201005`
   * Run `maps_multiReleaseTasks.pl`
      * `maps_multiReleaseTasks.pl -m3andCache -mcmBaseDir mcm/OSM_201005 -genType first -mc2dir mc2_m17_sr26 -mapSet 0 -binDir $someDirWhereToFindTheRequiredBinaries denmark sweden |& tee multiM3_first_dk_se.log`
      * Run `-genType first` for first generation and `-genType second` for second generation
      * The required binaries are `MapHttpServer`, `MapModule`,  `makemaps.mcgen`, `mc2.prop.end`
      * In `mc2.prop.end`, set the POI SQL database settings if there are POIs from WASP database in the maps

If you have many countries, generated with multi-gen or not, it might be better to run a small merge of the countries and have all countries on the test server at the same time, instead of testing them one-by-one. That way, you can test also that routing over country borders works before running the final merge generation for the MC2 back-end server. Follow the instructions in the section about merge map generation to do the merge.

## Prepare for merge map generation

A merge map generation creates maps for he MC2 back-end Wayfinder production 
server by combining the maps from several individual country generations to 
a larger map set, covering parts of the world. If only one country available,
no problem only including one country in the merge map generation.

The merge starts with the underview maps from the before_merge directory of the
countries' second map generations. You need to have the before_merge maps,
cannot use the underview maps from the final result of the second map
generation. Reason is eg that there are external connections in the final maps,
and that large waters and forests were moved from the underview to the country
overview map in the final maps.

### Scripts in the genfiles structure

Additional scripts used in merge map generation in 

   * `genfiles/script` (don't forget to update `BASEGENFILESPATH` to point to the full path of the genfiles)
      * `findCopyrightBoxFile.sh´
      * `changemapids´
      * `filterMergedMaps.sh´

### Setting files in the genfiles structure

Additional setting files needed in the genfiles structure

   * `genfiles/xml`
      * `map_supplier_names.xml` from `Server/bin/Scripts`
   * `genfiles/mergeWorld/xml`
      * `map_generation-mc2.dtd`
         * link to the gobal genfiles/xml/map_generation-mc2.dtd
      * `coo.xml` files `east_world_coo.xml` and `west_world_coo.xml`
         * lists the map hierarchy in the world merge, according to the `create_overviewmap` XML spec in `map_generation-mc2.dtd`. It lists the name of the second level overview map (the super overview map) in overview_ident tag, and for each super overview overview map it lists the name of the first level overview maps that belong to the super overview with the map_ident tag. There is one coo.xml file for each mapSet we divide the Wayfinder world in, i.e. one westWorld and one eastWorld.
         * Example for east world the overview_ident (name of the super overview map) is "east_world" and the underview(s) for that (the overview maps of the countries that are included in east_world) are map_ident "denmark" and "sweden" and "norway" and ... as they are defined as overview_ident tags in the country_co.xml file in the genfiles/countries/country/maprelease/xml directory.
         * `oo.xml` is not likely to change alot, so there are no release specific coo.xml files. It is OK if the coo.xml file lists all map_ident tags of the east resp. west world, even if the merge to perform only contains a subset of the countries. Just make sure that the overviews of the countries that are included in the merge are listed as map_ident tags.
   * `genfiles/mergeWorld/xml/crb`
      * `map_generation-mc2.dtd`
         * link to the global `genfiles/xml/map_generation-mc2.dtd`
      * copyright bounding box xml file `east_world_20100528_crb.xml` (plus box and txt file)
         * File defining which map supplier copyright string that will be used when displaying a map image over a certain area of the world. It uses the map_supplier_coverage XML spec in `map_generation-mc2.dtd`.
         * The xml file has two additional files that belongs to it. One box file with simply the bounding boxes in which can be loaded into the BTGPSSimulator to view if the boxes look alright. One txt ID file, which is used as an key for map generation to find out which copyright bounding box xml file to use in the map generation. The ID txt file lists the countries and the supplier map release which are valid for a certain xml file. Example "denmark OSM_201005".
         *  The boxes are hierarchical. You can have a main large box covering "all" of eastWorld for one supplier, then if some countries of eastWorld are from another supplier you add a smaller box inside the main box for the other supplier. No box may overlap thegeenwich 0-meridian or the opposite +180/-180 meridian, so the world must be cut more than one main box. Also no box may overlap another box.
         * See next section for how to create the files.
   * `genfiles/countrypol`
      * `countryBorderBreakPoints.txt`
      * Country polygon border break point file, include break point coordinates, i.e points where the individual countries' country polygons change neigbors. Negbours in this definition may be other countries or ocean. For instance Sweden has 3 break points; one Sweden-Finland-ocean (close to Haparanda), one Sweden-Finland-Norway, one Sweden-Norway-ocean. Example Denmark has 2 break points, where thegrman-danish border reaches the ocean. Include break points for all countries that are to be included in the merge map generation. See list of neighboring countries in mfunctions getNegbours().
      * The break points will be used to split up the country polygons in border parts. With 3 break points to Sweden, it means that the main polygon in Sweden's country polygon (sweden.mif) is split in 3 border parts, one is the border to Norway, one is the border to Finland, and one is the coastal border. The border parts that are on land (one border part that is shared by two countries) will be used for creating border items in the maps. For Sweden one part shared with Norway and one part shared with Finland. The border part for the first country will be saved and re-used for the next country sharing the same pair of break points, so thegometry of the country border in both countries will be the same - i.e. no gaps when displaying the countries in a map image. Related: see mfunctions countryOrder(), PerlCountryCodes.pm::countryOrder vector to see in which order the countries are processed in a merge map generation, i.e. which country border geometry is re-used for the next country.
      * The break point coordinate must be included in the country polygon mif file of all countries sharing the break point. Best would be to have the country polygons for all countries from the same source, thegometry of the shared borders are the same, so it is easy to find the break points. If not from the same source you need to modify the country polygon (mif files) for the countries so that the break point coordinate is included. The rest of the border part shared between 2 countries does not have to be exactly the same, it is ok for them to differ a little, see OldMapFilter::coordCloseToBorderPart for how much they can differ depending on how long the border part is.
     `` * If you have a bunch of country polygon mif files you want to find break points for, use the `MifTool --breakPoints`. 
   * genfiles/extradata/stitching
      * Stitching is needed if there aregps in roads between the countries and you want to connect a road to enable routing between the countries on that road.
      * See next section for how to create the stitch files for fixing thegp in the bridge between Denmark and Sweden in OSM Geofabrik 201005 maps.



#### More details for the copyright bounding box xml file

The copyright bounding box xml file east_world_20100528_crb.xml, has a box file
east_world_20100528_crb.box and a txt file east_world_20100528_crb.box.

##### Create files for few countries

This is howto create the file(s) from scratch for a merge with Sweden and Denmark from OSM data.

The XML file east_world_20100528_crb.xml

   * If running a merge with just a few countries from one supplier, the easy way is to define the bounding box is to use min/max latitude/longitude from the country polygon mif files. With script `midmif_tool.pl` (from `Server/bin/Scripts/MapGen`) you can get the min/max coordinate values in the mif files.
   * `./midmif_tool.pl -m sweden.mif denmark.mif`
   * Gives the min/max of the two files as

	
	     Read 2 mif files
	      tot minLat = 54.56166100
	      tot maxLat = 69.06030300
	      tot minLon = 8.08722100
	      tot maxLon = 24.16861000

   * These coords must be converted to mc2 for the copyright bounding box xml file. Use CoordConvert (Server/Tools/CoordConvert)
   * `./CoordConvert -f wgs84_deg --convertCoord 54.56166100 8.08722100`
   * `./CoordConvert -f wgs84_deg --convertCoord 69.06030300 24.16861000´
   * Gives you info to bounding box, to assign to the map_supplier tag


{% highlight xml‰}
	   <map_supplier_coverage map_supplier="OpenStreetMap">
	      <bounding_box north_lat="823921508" south_lat="650945971"
	                    west_lon="96484305" east_lon="288342749"/>
	   </map_supplier_coverage>
{% endhighlight ‰}

The BOX file `east_world_20100528_crb.box`

   * With the `maps_xmlSupplierCopyright.pl` script you can convert between xml file and box file, so you only need to do one of them manually.
   * The box of the xml file above is :
	
	     [(823921508,96484305),(650945971,288342749)]

The TXT file `east_world_20100528_crb.box`

   * List the country (as defined in the makemaps.mcgen wf_world countrySet list) and the map origin (=supplier map release) of the country that fits with the copyright bounding box xml file. The map origin is the string as can be found in the country generation directory before_merge/mapOrigin.txt file (which is created in the individual country generation). The countries are listed in alphabetical order.

	     denmark OSM_201005
	     sweden OSM_20100

   * If you don't want to create this txt key file manually, run the makemaps.mcgen script as described below for merge map generation, but add option -createMapVerByCountryFile. The script will create the key file for the merge countrySet, and then exit.
   * For the txt file, it is the supplier part of the map origin string that is used. So for OSM_201005 only the OSM part is important and used.
   * Say you have a copyright bounding box xml file for a large part of, or complete, east world or west world defined already.
   * If you then run a new full merge of the exact same countries from a new map release from the same supplier, you don't need to create a new xml, box and txt file since it is the supplier part in the key file that matters. Example a txt key file with "denmark OSM_201002" will work perfectly also for the OSM_201005 map release.
   * If you run a merge generation with only some of the countries from the full the same supplier you also do not need to create a new copyright bounding box xml file for the few countries. It is perfectly possible to re-use an already existsing xml file which contains at least the few countries.
   * However, if one country in the merge, full or partial, is from another supplier, there need to be a new xml file defined beacuse you need another copyright string when displaying a map image of that country.

##### Generic way of creating files

This section describes howto create the boxes and the XML file in a more generic way for a large part of the world. Remember, the key txt file is easy created with makemaps.mcgen script -createMapVerByCountryFile option.

   * Load a MC2 test server with maps with the world coverage you have. Run a BTGPSSimulator against the server. 
   * Zoom to the extent of the area where you want to create a box. 
   * Click the Map Mode "Boxes" circle in the bottom. Create a box in the map by mouse-dragging from upper left to lower right. If you are not happy with the box, press "Erase Box" which will erase the latest created box. 
   * If some countries are from another supplier zoom in to that country's extent in the map and create a smaller box inside the main box. The smaller box may not overlap the larger, but perfectly OK to share upper-left or lower-right corner.
   * When you have created the boxes you want, save the boxes to a file with menue File -> Advanced -> Save Box File
   * If you aimed at re-using upper-left or lower right corner for hierarchical boxes, verify in the Box file that you succeeded. If some corner was not 100%, edit the box file to get the exact upper-left and lower-right corner coordinate. Then re-start the BTGPSSimulator and load the box file with File -> Advanced -> Load Box File to verify the result visually . (Need to re-start, else the old boxes will still be displayed there and disturb.)
   * Iterate create boxes in BTGPSSimulator, save box file, edit the box file, and verify visually with load boxes until you have the boxes you want for you copyright strings. Examples with 2 boxes, one main box and the other smaller box inside the main box, sharing the upper-left coordinate (max lat/min lon).
      * `[(725495615,46795596),(622659810,210394288)]`
      * `[(725495615,46795596),(712659000,50005596)]`
   * If you want, add comment to each box last on each row in the final box file to easier keep track of which supplier each box belongs to. Unfortunately such a commented box file cannot be loaded into BTGPSSimulator.
      * `[(725495615,46795596),(622659810,210394288)] main box supplier A`
      * `[(725495615,46795596),(712659000,50005596)] small box supplier B`
   * Create the copyright bounding box XML file with running the script `maps_xmlSupplierCopyright.pl -box2xml boxFile`. It will create the `map_supplier_coverage` tags to you xml file. Any comment from the box file will be included, the `map_supplier` will be empty.

{% highlight xml %}	
	   <!-- main box supplier A -->
	   <map_supplier_coverage map_supplier="">
	      <bounding_box north_lat="725495615" south_lat="622659810"
	                    west_lon="46795596" east_lon="210394288"/>
	   </map_supplier_coverage>
	
	   <!-- small box supplier B  -->
	   <map_supplier_coverage map_supplier="">
	      <bounding_box north_lat="725495615" south_lat="712659000"
	                    west_lon="46795596" east_lon="50005596"/>
	   </map_supplier_coverage>
{% endhighlight %}	

   * Next you need to edit the xml file
      * Add the header lines (xml version, encoding, and inclusion of the map_generation-mc2.dtd)
      * Add the start `<map_generation-mc2>` and end `</map_generation-mc2>` tags
      * To keep track of the box hierarchy, move the small box into the main box tag and indent it.
      * Add the supplier, see map_supplier_names.xml map_supplier tag for how it is "spelled", eg "TeleAtlas" for Tele Atlas and "OpenStreetMap" for OpenStreetMap.

{% highlight xml %}	
	<?xml version="1.0" encoding="ISO-8859-1" ?>`
	<!DOCTYPE map_generation-mc2 SYSTEM "map_generation-mc2.dtd">`
	
	<map_generation-mc2>`
	
	   <!-- main box supplier A  -->`
	   <map_supplier_coverage map_supplier="SUPPLIER_A">`
	      <bounding_box north_lat="725495615" south_lat="622659810"
	                    west_lon="46795596" east_lon="210394288"/>
	      <!-- small box supplier B  -->
	      <map_supplier_coverage map_supplier="SUPPLIER_B">
	         <bounding_box north_lat="725495615" south_lat="712659000"
	                       west_lon="46795596" east_lon="50005596"/>
	      </map_supplier_coverage>
	   </map_supplier_coverage>
	
	</map_generation-mc2>
{% endhighlight %}	

When you have a copyright xml file with most parts of the world, and only want to add or change some boxes, it is not necessary to everything from scratch again.

   * Open `BTGPSSimulator` and create boxes for the new/changed areas
   * Edit a copy of the latest xml file, adding/changing map_supplier_coverage tags for the new/changed boxes
   * When you think you have edited the XML file to a final result, create a new box file with `maps_xmlSupplierCopyright.pl -xml2box xmlFile`
   * Load the new box file into `BTGPSSimulator` to visually verify the result, eg that the hierarchy is correct, and that the new/changed boxes cover the intended areas.

#### More details for the stitching

Stitching is part of the merge map generation and is done to fix gaps in roads
or ferry lines between countries to enable routing between the countries on
that road/ferry line.

To prepare for stitching, to create stitch-files, is to create WF street
segment items or ferry items midmif files to connect roads or ferry lines that
havegps between countries. It is also possible to create extradata files to
apply before midmif (typically to remove some street segments to facilitate the
stitching) and after midmif (seldom necessary, only needed if need to correct
the turn description that is calculated between the original end-segment in the
map and the stitch-segment that is added connecting to the end-segment).

There is a naming rule for stitch-files, a base name
`alpha2_maprelease-alpha2_maprelease` is the prefix of all stitch files. The
order of the two countries in the file name of a border is always the same. The
one with the alpha2 code first in alphabetical order is always first.  The
suffix of the midmif files is (of course) `.mid` and `.mif`.  The suffix of the
extradata file to apply before midmif is `.A.ext`, and for the one to apply
after `.B.ext`.  A stitching between Sweden and Denmark for the OSM_201005 map
release would have this complete set of stitch-files

	
	dk_OSM_201005-se_OSM_201005.A.ext (before midmif)
	dk_OSM_201005-se_OSM_201005.streetSegmentItems.mid
	dk_OSM_201005-se_OSM_201005.streetSegmentItems.mif
	dk_OSM_201005-se_OSM_201005.ferryItems.mid
	dk_OSM_201005-se_OSM_201005.ferryItems.mif
	dk_OSM_201005-se_OSM_201005.B.ext (after midmif)


For this specific example for fixing thegp in the bridge between Malmö and
Copenhagen we don't need any extradatafiles, only need the midmif files for street segments to add the missing street segment. The stitch file will be named `dk_OSM_201005-se_OSM_201005.streetSegmentItems.mid/mif`
   - Find thegometry of the stitch items. Load the maps on each side of thegp in MapEditor, select the end-segment, and press the Coords-button

      * end coordinate in Sweden north lane: 663054046 152966035 south lane: 663052106 152965347
      * end coordinate in Denmark north: 663082354 152840082 south: 663080702 152839172
   - Create Wayfinder midmif
      - The mif file needs a mif header with a accepted coord sys tag, the "Columns" does not have to reflect the mid file attributes (not read in map generation).
      - Set the midID to the next available ID, check tail on all stitch mid files or run the script `checkStitchFiles.pl -n`. The midID must be unique, comparing with all other stitch mid files, since you may use any stitch file in the merge map generation and cannot have duplicated IDs in the merge stitching.
      - Check attributes on the end-segment to get correct speed, oneWay, node levels etc on the stitch segments. OK to skip the name. Node level is important, there will be no connections created between segments if the nodes sharing the same coordinate have different levels.
      - When setting oneWay or other entry restrictions consider the orientation of thegometry in the mif file. Node 0 is the first coordinate, node 1 is the last coordinate.
      - Decide in which of the maps the stitch segments should be added

         * If one of the maps of the border do not have virtal segment on the end-segment, add the stitch segment to that map.
         * If none of the maps on the border have virtual segment on the end-segment, you need to add two stitch segments, one in each of the maps.
      - Fill in the mapssi coordinate attributes so the stitch segment is added to the map of your choice, else if added based on geometry it might end up in both maps or in none of them. Find a mapssi coordinate that defines the mcm map you want the stitch segment to be added to. If there is a coordinate already defined in the mapssicoords file in genfiles/countries/country/mapssicoord/maprelease.txt re-use that, else define such a coord.
      - Set the borderNode attribute to "N" for the stitch segment node that is close to the end-segment, set the borderNode "Y" on the node that is close to "the other map"
   - Finally, run the `checkStitchFiles.pl -n` to get some verification of that your stitch files are ok.

If you choose to add the stitch segments to the Swedish map.

* map ssi coordinate: not already defined, pick a coordinate on a large road with help of MapEditor and create the mapssicoord file for Sweden OSM_201005 `se_ssicoords_inmap_OSM_201005.txt` with on line
`0,664124744,157016425`

* mif file: the mif header and then thegometry of the two stitch segments(the north gap node 0 fits Sweden's end-segment and node 1 fits Denmark's end-segment, the south gap node 0 fits Denmark node 1 fits Sweden)

	Version 300
	Charset "WindowsLatin1"
	Delimiter ","
	Coordsys mc2
	Columns 3
	  Id Decimal (16,0)
	  Name Char (50)
	  All_names Char (150)
	Data
	Pline 2
	663054046 152966035
	663082354 152840082
	Pline 2
	663080702 152839172
	663052106 152965347

* mid file

	1,"","",0,110,110,0,3,,,,,0,0,0,0,"Y",0,0,"N","N","N","N","N","N","N","","",,,,"N","Y",664124744,157016425
	2,"","",0,110,110,0,3,,,,,0,0,0,0,"Y",0,0,"N","N","N","N","N","N","N","","",,,,"Y","N",664124744,157016425


Check that the country neighbors pair are defined in the neighbors-list in
mfunctions getNeighbors(). Only country pairs that are defined in that list
are considered for stitching.

   * I needed to add `#denmark#sweden#`, since the neighbors-list mostly contains neigbors that share a border on land.


### Examples of setting files in the genfiles structure

#### Example of coo.xml file

Giving you the example for east world
`genfiles/mergeWorld/xml/east_world_coo.xml` listing which overview maps that
belong to the east world super overview map. The `west_world_coo.xml` file of course
has the same structure.

{% highlight xml %}
<?xml version="1.0" encoding="ISO-8859-1" ?>
<!DOCTYPE map_generation-mc2 SYSTEM "map_generation-mc2.dtd">

<map_generation-mc2>
  <create_overviewmap overview_ident = "east_world">

     <map map_ident = "sweden"/>
     <map map_ident = "denmark"/>
     <map map_ident = "germany"/>
     <map map_ident = "france"/>
     <map map_ident = "monaco"/>
     <map map_ident = "andorra"/>
     <map map_ident = "spain"/>
     <map map_ident = "portugal"/>
     <map map_ident = "italy"/>`
     ...

  `</create_overviewmap>
</map_generation-mc2>
{% endhighlight %}

#### Example of copyright bounding box files

Remember there are 3 files. The XML file is used in map generation to define
which box areas should have which copyright text when displaying a map image
over that area. Example of the XML file created with only Denmark and Sweden in
the world `mergeWorld/xml/crb/east_world_20100528_crb.xml`

{% highlight xml %}
<?xml version="1.0" encoding="ISO-8859-1" ?>
<!DOCTYPE map_generation-mc2 SYSTEM "map_generation-mc2.dtd">
<map_generation-mc2>
   <!-- east world -->
   <!-- Sweden and Denmark -->
   <map_supplier_coverage map_supplier="OpenStreetMap">
       <bounding_box north_lat="823921508" south_lat="650945971"
                 west_lon="96484305" east_lon="288342749"/>
   </map_supplier_coverage>
</map_generation-mc2>
{% endhighlight %}


The BOX file is simply a text file with the bounding boxes from the XML file.
It can be loaded into ´BTGPSSimulator` to verify that boxes look OK. Example of
the BOX file created with only Denmark and Sweden in the world
`mergeWorld/xml/crb/east_world_20100528_crb.box`:
	
	[(823921508,96484305),(650945971,288342749)]


The TXT file is a key file which lists the countries and the map supplier and release for the countries that the XML file is valid for. Example of the TXT file created with only Denmark and Sweden in the world `mergeWorld/xml/crb/east_world_20100528_crb.txt`:

	denmark OSM_201005
	sweden OSM_201005

Another example of the XML file when you have more countries in the world, and they are from several different suppliers. This is when you end up with hierarchical boxes; within one larger box for one supplier, there are pieces that are covered by smaller boxes from another supplier.

{% highlight xml %}
<!-- East world main box -->
<map_supplier_coverage map_supplier="SUPPLIER_A">
     <bounding_box north_lat="884476470" south_lat="-1073619829"
                   west_lon="0" east_lon="2147479127"/>

     <!-- europe east part, and russia west part -->
     <map_supplier_coverage map_supplier="SUPPLIER_B">
        <bounding_box north_lat="884476470" south_lat="442006601"
                     west_lon="0" east_lon="523255226"/>

        <!-- Georgia Russia-->
        <map_supplier_coverage map_supplier="SUPPLIER_A">
           <bounding_box north_lat="517455268" south_lat="492831223"
                         west_lon="478475894" east_lon="523255226"/>
        </map_supplier_coverage>
     </map_supplier_coverage>` `<!-- europe east part, and russia west part -->
</map_supplier_coverage>` `<!-- main box east world -->
{% endhighlight %}


#### Example of country border break points file

The `genfiles/countrypol/countryBorderBreakPoints.txt` file defines coordinates to split main country polygons into border parts. Sweden has 3 break points, so Sweden's main polygon of the country polygon has 3 border parts, one is the border to Norway, one is the border to Finland, and one is the coastal border. Example with break points for some part of the world. Not generated from the OSM world_borders shape file, but still illustrative. The map generation only reads the lat+lon, any comment after the coordinates are skipped, so it is good for you to here include some info about which countries are involved in the break point.

	
	703748538 136738514  sweden-norway-ocean
	823915938 245139346  sweden-norway-finland
	785357201 288128788  sweden-finland-ocean
	823841138 345148304  norway-finland-russia
	722335693 331621926  finland-russia-ocean
	832293558 368209720  norway-russia-ocean
	655121652 103026734  denmark-germany-occan west
	654169487 112391133  denmark-germany-ocean east


#### Example of stitch files

Stitch files are for fixing gaps in roads or ferry lines between countries to
enable routing between the countries. Example of stitch files for the bridge
between Sweden and Denmark in OSM_201005 map release.

`genfiles/extradata/stitching/dk_OSM_201005-se_OSM_201005.streetSegmentItems.mif`:

	
	Version 300
	Charset "WindowsLatin1"
	Delimiter ","
	Coordsys mc2
	Columns 3
	  Id Decimal (16,0)
	  Name Char (50)
	  All_names Char (150)
	Data
	Pline 2
	663054046 152966035
	663082354 152840082
	Pline 2
	663080702 152839172
	663052106 152965347

`genfiles/extradata/stitching/dk_OSM_201005-se_OSM_201005.streetSegmentItems.mid`:

	
	1,"","",0,110,110,0,3,,,,,0,0,0,0,"Y",0,0,"N","N","N","N","N","N","N","","",,,,"N","Y",664124744,157016425
	2,"","",0,110,110,0,3,,,,,0,0,0,0,"Y",0,0,"N","N","N","N","N","N","N","","",,,,"Y","N",664124744,157016425



### merge map generation binaries

The following binaries and scripts are needed for the merge map
generation. They are all found in the mc2 repo, the binaries with capital
letter are results of compile.
Don't forget to update `BASEGENFILESPATH` in misc scripts to point to the full
path of the genfiles.

Map generation main script:

   * `makemaps.mcgen`  from `Server/bin/Scripts/MapGen`

Map generation binaries and subs:

   * `distribute`      from `Server/Tools/distribute`
   * `ExtradataExtractor` from `Server/MapGen/ExtradataExtractor`. Needed if running dynamic to apply new map correction records from WASP database.
   * `GenerateMapServer`  from `Server/bin`
   * `makemaps.dynamic` Needed for dynamic part of map generation (map correction records and/or POIs)
   * `mc2.prop`        from `Server/bin`. Set `MAP_PATH to `./` and define POI SQL database settings
   * `mc2.prop.end`    from `Server/bin/Scripts/MapGen`. Set the POI SQL database settings if there are POIs from WASP database in the maps
   * `MapHttpServer`   from `Server/bin`
   * `MapModule`       from `Server/bin`
   * `QualTool`        from `Server/MapGen/QualTool`. Needed for analysis/verification of map generation as preparation for test`
   * `verify`          from `Server/bin/Scripts/MapGen`
   * `Verify`          from `Server/MapGen/Verify`
   * `WASPExtractor`   from `Server/MapGen/WASPExtractor`. Needed if running dynamic to add/remove/modify POIs from WASP database
   * `whichmaps`       from `Server/bin/Scripts`


## Run merge map generation

Set up directories for the merge generation. If merging countries from a
certain map release you can store it in the mcm map release directory where the
individual countries were generated, eg `mcm/OSM_201005/merge_20100610`. If
merging countries from several suppliers and map releases put it
in a general place, one dir for east_world and one dir for west_world. In each
of them, put a base dir named [date]_[mainMapReleases]

   * `mcm/merge_east_world/20100610_OSM_201005`
   * `mcm/merge_west_world/20100610_OSM_201005`

Continue with the structure for this specific merge

   * merge storage directory
     * `mcm/merge_east_world/20100610_OSM_201005/merge1`
     * this is where the merge map generation will store the resulting merge. it is called mergePath
   * merge countries directory
      * `mcm/merge_east_world/20100610_OSM_201005/countries`
      * collect here the countries that will be merged
      * the name of the dirs in countries dir must be the ones defined in the makemaps.mcgen wf_world countrySet list
      * copy the individual country generation country directories, or to save disc space create links to them eg
         * denmark -> `mcm/OSM_201005/dk_20100607_second`
         * sweden  -> `mcm/OSM_201005/20100608_multiSecond_dk_se/countries/sweden`
   * merge generation directory
      * `mcm/merge_east_world/20100610_OSM_201005/1_merge`
      * this is where to run the merge generation from. Copy the makemaps.mcgen script to this dir.
   * merge generation bin directory
      * `mcm/merge_east_world/20100610_OSM_201005/1_merge/bin`
      * includes the binaries used in the merge map generation. Copy all binaries to this directory: the ones that were listed above.

This first merge is done with the `merge1` and `1_merge` directories. The next merge of some (the same or more) countries with OSM_201005 as the main map release will be generated from 2_merge and stored in merge2.

Possibly you need to edit the makemaps.mcgen script

   * Define which countries you want to merge with the -countries option.
   * If noone of the defined countrySets (east_world west_world etc) fits, use the `myCountries` countrySet and edit makemaps.mcgen myCountries to include the countries to merge. Edit the COO_LIST to fit your choice of countries, and the mapSetCount if you want to have another map set than the one pre-defined for myCountries.


Goto the merge generation directory `1_merge` and run the makemaps.mcgen script for a **normal** merge map generation.

	
	./makemaps.mcgen -countries myCountries -onlyMerge -binPath bin
	-countriesPath ../countries/ -mergePath ../merge1/
	-filtComputers "filthost filthost filthost" -mc2Dir mc2_m17_sr26
	|& genfiles/script/mlog master1


If you want to run a **dynamic** merge map generation, and apply latest POI changes and latest map correction records from the WASP database on the merge result, run the merge with option `-dynamicED`. The dynamic changes are added on the merge before filtering is applied.

	
	./makemaps.mcgen -countries myCountries -onlyMerge -binPath bin
	-countriesPath ../countries/ -mergePath ../merge1/ -dynamicED
	-filtComputers "filthost filthost filthost" -mc2Dir mc2_m17_sr26
	|& genfiles/script/mlog master1



The merge generation will be stored in merge1, the m3 maps and search and
route caches will be stored in the mc2 dir merge1/mc2_m17_sr26. The name of
the mc2 dir was chosen to reflect the m3 map version (17) and cache version
(26). Filtering of coordinates in the maps will be distributed on the
computer filthost on 3 processes.

It might happen that the last step of the merge map generation (creating search
and route caches) hang. Check if all cache maps have been created, should be
twice as many as number mcm and m3 maps. Check the log file
`1_merge/logpath/mm_saveSR.log` to see what is going on. If all cache maps have
been created and the log file last line says "Cleaning up" everything is done,
and we have a hang-situation. MapModule is finished, but the makemaps.mcgen
script didn't get the signal, so it cannot continue. Press ctrl-C ONCE to get
the makemaps.mcgen script to run again to complete the very last lines of the
script.



Verify the merge map generation with the following actions

   * Check that nbr of m3 maps is equal to the number of mcm maps, nbr caches twice as many
   * (Read the `qualLog/qualityreport.log`). This is an old verification method, and not very informative, it sometimes reports errors that are not errors.
   * Run a `qualDiff` comparing the merge with a previous merge, eg a merge with the countries from a previous map release or a re-generation of the same map release (for whatever reason)
      * Create and goto a diff-dir in the merge generation directory `merge1/diff_2_20100328_merge2`
      * run the `maps_genQualCheck` script:
      * `./maps_genQualityCheck -bin .. mcm/merge_east_world/20100328_OSM_201002/merge2 .. |& teegnQualCheck.log`
      * The script uses the `QualTool` binary to calculate some statistics for the maps so you can see increases and decreases in the maps. Statistics for the `FIRSTMAPS` is written to the `1_[]_MapsQual.log` and the `SECONDMAPS` is written to the `2_[]_MapsQual.log`. All differences between first and second maps are written to the `[..]_MapsQualDiff.log` file, the important subset of the diffs are written to the `[]_ShortMapsQualDiff.log` file. If the diff files are very large, you can use the `maps_mcmAnalyseQual` script to list only increases/decreases larger than certain number or percentage.


Test the merge map generation by putting them on a test server.  The `mc2.prop?
setting file on the server should have `MAP_PATH[_mapset]` pointing at the
mc2 dir with the m3 maps and the `MODULE_CACHE_PATH[_mapset]` pointing at the
mc2/cache-dir with the search and route caches.

When tested OK the merge map set can be put on a MC2 backend production server.

## POI handling

### Add POI data to the WASP database

#### Define poi source

Define the new source (poi release) for this POI set in the WASP POISources table. If this is a POI set of a POI product never used before, add a new POI product to the WASP table POIProducts.

For the OSM_201005 POI data I defined a POI product called OSM_eu, expecting to contain POIs from OpenStreetMap in European countries. It could have been named OSM_world if you plan on implementing world-wide coverage from the same OSM-source.

   * product name: OSM_eu (productID 51)
   * description: OpenStreetMap data Europe
   * default map right: `FFFFFFFFFFFFFFFF` (no special right)
   * migration candidate: 0=no (obsolete field, not used)
   * geocoding candidate: 0=no (obsolete field, not used)
   * trusted source ref: 1=yes (this is the first map release of Geofabrik OSM data, so we cannot know if the sourceRef (the supplier poi id) is stable over POI releases or not. Change to "no" later if the IDs are not stable)
   * supplier name: "OpenStreetMap" the supplier name will be displayed when showing info for the POI in the client
   * map data poi: 1=yes (the POIs came with a map release, which they fit with)

The POI source was created and tied to the OSM_eu product.

   * source name: OSM_201005 (source id 249)

#### Define ed version

The map release (which the POIs come from), must be defined in the WASP EDVersion
table. Add it, if it was not already done before. The ID in the EDVersion
table must equal the ID in the MapGenEnums::mapVersion enum. The version
string in the table must be the map release with the long version of the
map supplier string, i.e. the `MAP_SUPPLIER` defined in the variable file,
eg OpenStreetMap_201005

#### Scripts in the genfiles structure

Scripts needed in POI import. Don't forget to update `BASEGENFILESPATH` to point to the full path of the genfiles, and edit the "use lib" to include perl modules from `BASEGENFILESPATH/script/perllib` and possibly other scripts from `BASEGENFILESPATH/script`.

   * in the `genfiles/script` directory
      * `poi_doublesCheck.pl`
      * `poi_doublesTypeMapping.pl`
      * `poi_admin.pl`
      * `poiImport.pl`
   * in the `genfiles/xml` directory
      * `poi_category_tree.xml`

#### POI import binaries

The POI import requires 3 binaries:

   * `WASPExtractor`
   * `mc2.prop` with MAP_PATH set to `./` and POI SQL database settings defined to connect to the WASP database.
   * `poi_autoImport.pl` the main import script calling all sub scripts for a complete import.
      * Edit the `poi_autoImport.pl` `BASEGENFILESPATH` to point to the full path of the genfiles directory, to use the POIObject and perl modules from `BASEGENFILESPATH`/script/perllib, and to include sub scripts from `BASEGENFILESPATH`/script.
      * If new product, define import routine for the POI product. Mostly, a matter of adding the productID to the if-statement in the section that handles the runImport. If uncertain where to find the if-statement, run the script and it will die to display on which row to find it.


#### Run POI import of map data POIs

Set up a import directory with :

   * the POI import binaries
   * the tmpEW/tmpWW map countries directories

Use the Wayfinder CPIF file from step 5 of parsing of OSM data to Wayfinder format, the WFcpif_all_DK_utf8.txt. If you have several countries to import POis for, cat the CPIF files into one and run import for all countries in one step.

    cat WFcpif_all_DK_utf8.txt WFcpif_all_SE_utf8.txt > WFcpif_allcountries_utf8.txt

Create the tmpEW/tmpWW map countries directories (`-mapCountriesDirs` option)

   * Copy the countries directories links from latest EW and WW merge map generations to the tmpEW and tmpWW directories. If no previous merge generations, create empty tmpEW and tmpWW dirs.
   * Update links for the countries that are part of the new map release (the POIs to import) to point to the individual countries first generation of the new map release. It is the mapOrigin.txt file in the country/before_merge dir that is interesting, the version that written in the mapOrigin.txt will be used to set the validFromVersion for the POIs, and for mapDataPOIs also the validToVersion. The OSM POIs will have entry points created in the import, so there must be mcm maps in the countries before_merge dirs.
   * The default is that the tmpEW and tmpWW dirs must contain all countries that are defined in WASP POICountries table.
   * If you don't have all countries available, no problem to only link the countries that have POIs to import.

Run the import, default command 

     ./poi_autoImport.pl -verbose -poiSource OSM_201005 -mapCountriesDirs tmpWW,tmpEW -poiData WFcpif_allcountries_utf8.txt |& /home/is/devel/Maps/genfilesPSTC/script/mlog autoImport1

But for this import of OSM_201005 POIs we need some special options.

   * `./poi_autoImport.pl -verbose -poiSource OSM_201005 -mapCountriesDirs tmpWW,tmpEW -okWithMissingCountries -newProduct -poiData WFcpif_allcountries_utf8.txt |& /home/is/devel/Maps/genfilesPSTC/script/mlog autoImport1`
   * option `-okWithMiss`ingCountries` because we don't have all countries in the tmpEW and tmpWW dirs, only links to first generation of Sweden and Denmark in the tmpEW and the tmpWW actually empty.
   * option `-newProduct` because this is the first source of the OSM_eu POI product.


#### Run POI import of not map data POIs

The OSM_201005 POIs came with a map release, and thus are map data POIs, they should be used only with maps from OSM_201005. The WASP database can also have not-map data POIs, so called external POIs. The external POIs are imported with the same routine as the map data POIs, with some additions

   * The validFromVersion will be set from the mapOrigin.txt files in tmpEW and tmpWW
   * The validToVersion will however always be set to NULL, since the POIs should be valid also for next map release and next ... until you import a updated version of the external POI product.
   * After completed the autoImport of external the POIs, run the poi_admin.pl script to set the previous source of the product to not-in-use, and to make sure that the new source of the product is in-use
      * `poi_admin.pl -setSourceInUse newSource`
      * `poi_admin.pl -setSourceNotInUse oldSource`
      * The old (previous) source is stopped, so it will not be valid for the
        map release that the new source belongs to. The old source will have
        the inUse and/or the validToVersion updated.


## Map correction handling

If you have map corrections either in the WASP database or in extradata.sh for
the previous map release, these need to be migrated to fit the new map supplier
release. There are several reasons for this, some of which are:

   * Map errors are corrected in the new release.
   * Geometry has changed making it impossible to identify items/connections using the old map correction record. 

The map correction migration is always done on the first generation of the new map release.

### Migrate map corrections from WASP database for the new map release

Migrating the map corrections from WASP db is done with the
`maps_multiReleaseTasks.pl` script `-edMigration` option. The multi migration
will run the individual ed_migration script for each of the countries specified
in tail. Actions of the `ed_migration` script are:

   * Extracts map correction records (also called extradata records) from the WASP database with `addED.pl` script. All records that are valid for the new map release, typically with validToVersion = NULL, area extracted to files. One file with map correction records with insertType 1 beforeInternalConnections and one file with the insertType 2+3+4. Records with insertType 5 are never migrated.
   * Check if the records are valid by running the **map correction validity check** with `GenerateMapServer -x` in combination with ´--checkextradata`, which will add the map correction records to the mcm maps but never save the maps. I.e. simply check if the records fit or not. The file with insertType 1 is checked on the mcm maps in the first generation after_mapDataExtr backup, and the insertType 2+3+4 records are checked on the mcm maps in the first generation before_merge backup.
   * Result files list the records that 
      * found-use: fit the new map release and are still needed to fix the map error
      * found-nouse: fit the new map release but are no longer needed, since the map error was fixed by the map supplier
      * missing: do not fit the new map release, because thegometry changed or the map error was changed to something else than the map correction record wanted to
      * missing-noprio: do not fit the new map release, because thegometry changed or the map error was changed to something else than the map correction record wanted to. The records are not so important.
      * no-check: records that cannot be checked.
   * If the country has a map ssi coordinate file for the previous map release, the ´ed_migration` script tries to find it and update it for the new map release. It will run a check of the map ssi coordinates on the mcm maps in the first generation `before_merge` backup to see if they are still valid for the new map release with `MapTool --checkmapssicoordFile` option.

#### Run the multi script

This is how to run the `maps_multiReleaseTasks.pl` script

   * Create (consider provider and product) and go to the working directory, example `fullPath/CheckExtradataInNewRelease/migrate_ed_TeleAtlas_2010_06_eu`
   * Run the multi script, example
      * `maps_multiReleaseTasks.pl -edMigration -mapProvider TeleAtlas -mapRelease 2010_06 -prevMapRelease 2010_03 -mapProduct 2010_06_eu -mcmBaseDir fullPath/mcm/TA_2010_06_eu -mapToolBinDir fullPath/binDir -edCheckFileStoreBasePath fullPath/CheckExtradataInNewRelease france spain |& tee multiMigrate_es_fr.logÅ`
      * give option `-yesForSingleDirs` if you do not want to press yes for all country dirs of the first generation that is found unique.
      * The options for `-mapProvider` and `-mapProduct` are used for naming
        the directory where the map corrections from database are extracted,
        and log-files and validity check result files are copied after the
        check is done (will be TeleAtlas_2010_06_eu for the example command
        above). 

The script will try to find the first gen directories and variable files for
each of the countries. It will ask you to verify the result. So monitor the
script and answer y or n when requested to. If a first gen directory cannot be
found unique, you will have to answer which directory is the correct one from a
multi-choice list. When all first gen directories and variable files are
collected, the script runs the individual ed_migration script for all countries
(in alphabetical order) "Start runMultiMigration" 

The multi migration script will not re-run any country. It checks if the
country directories exist or not in the release check directory, eg
`fullPath/CheckExtradataInNewRelease/TeleAtlas_2010_06_eu/france`. If you want to
re-run the migration for a country, you need to do that manually.

After running the multi script,

   * read the multi log file, to check that everything was ok
   * read the individual country migrate log file(s), to check that everything was ok for each of the countries
   * if some records ended up in the missing-file for one or more countries, do the manual check of missing-files with the MapEditor tool
   * finally update the WASP database validToVersions etc to reflect the result of the migration

#### Manual check of missing-files with `MapEditor`

The missing files hold map correction records for which the map correction was
not identified in the automatic validity check. The main reason is
geometry-changes in the new map release. To evaluate if the new release also
has been corrected, the records in the missing file must be manually checked.
If the map error is still present in the new release, new map correction
records must be created to replace the ones in the missing files.

To find out how many records there are for each mcm map (decimal id) use the `ed_countEdRecsInMissingFile.pl` perl script with the missing file as inparameter:

     ed_countEdRecsInMissingFile.pl missing_france_EDCheck_2010_06.txt

The manual check is done using the `MapEditor` ShowExtradata functionality.
   - Goto the dir where the auto validity check was run (firstGen `after_mapDataExtr` or `before_merge` dirs)
   - Open the MapEditor with one mcm map from the `after_mapDataExtr` or
     `before_merge` directory of the first generation from the new release.
   - Click the Show extra data button and load the missing file.
   - Check each missing record, and create new replacement records if needed.
     Reuse comment values from the missing records and add comment text about
     which record (waspId) the new record is replacing. It is important to keep
     any defined validToVersion. If the missing record is valid to 2011_03 the
     new record should also have that version.

The new replace-missing map correction files should be copied to the
validityResult directory (eg
`fullPath/CheckExtradataInNewRelease/TeleAtlas_2010_06_eu/france/validityResult`).
If it is discovered that some records in the found-use file is really not
needed, edit the file (eg move the records from the found_use to the
found_nouse file). All edited result files must be re-copied to the
validityResult directory.

Records in the `found_nouse`  file could also be checked. The automatic
validity check with `GenerateMapServer` is not 100% ok with mix-ups between
noEntry-noWay and for different combinations of vehicle restrictions.

Other tools that might be helpful

   * Load a file with coordinates into MapEditor. The coordinates are marked with purple `X` in the map.
      * `MapEditor --highlightCoords=${fullPath}/MEcoordinates.txt 000000000.mcm.bz2`
   * Create a coordinate file for MapEditor with the coordinates of all map correction records with a certain source.
      * `addED.pl -f createCoordFileFromSource -s "RZM-19490-782"`
      * It creates a file called MEcoordinates.txt 


#### Update status in WASP database

The result of the automatic validity check should be inserted into WASP. It is
stored in eg `fullPath/CheckExtradataInNewRelease/TeleAtlas_2010_06_eu/france/validityResult`

This is how to handle the different result files

   * Records in the *found-use* and *nocheck* files should have their validToVersion extended to comprise the NEW release (no limit or whatever already defined validToVersion that equals the new release or any later version).
   * Records in the *found-nouse* and *missing* files should have the validToVersion set to the OLD release.
   * New records replacing missing records should be added to WASP with validFromVersion set to the NEW release. Do this with the `addED.pl -f addED` as you would when adding any new map correction records to WASP. The validToVersion should be the same as for the missing record that was replaced (no limit or defined).

Use the `ed_migrationUpdateWASP` shell script to update the validToVersion for all use and remove records.

   * Go to the validity result directory
   * Run the ed_migrationUpdateWASP script. Example command when migrating extradata records to the new Tele Atlas 2010.06 map release.
      * ´ed_migrationUpdateWASP -newMapRelease TeleAtlas_2010_06 -oldMapRelease TeleAtlas_2010_03 -validityResultDir fullPath/CheckExtradataInNewRelease/TeleAtlas_2010_06_eu/france/validityResult |& /home/is/devel/Maps/genfilesPSTC/script/mlog update_france`
      * where
         * `-newMapRelease` is the NEW map release
         * `-oldMapRelease` is the OLD map release (prior to `-newMapRelease`)
         * `-validityResultDir` is the directory where the result from the auto validity check is stored 



### Migrate map corrections in `extradata.sh` for the new map release

Any corrections in `extradata.sh` must be checked manually

