---
title: Wayfinder midmif format
layout: single
---

An *mcm* map can be created from shape files or midmif files. If creating from
shape, the shape files are first converted to midmif, and then the procedure is
the same. The midmif must be the Wayfinder midmif format, so any map supplier
midmif format must be converted to Wayfinder midmif before mcm map generation.

Here you can find the specification of the Wayfinder midmif format and we
provide you a few examples for illustrating the format. You will also find info
about variable files and areafiles for map generation from Wayfinder midmif,
and some rules and tips for file naming etc.

## Wayfinder midmif specification

See also the [Wayfinder midmif specification](../wayfinder_midmif_specification/)

## Creating mcm maps from midmif

The mcm maps are basically created using the `createItemsFromMidMif` function
in the `GMSMidMifHandler` class.

To initialize the mcm maps, the municipals, of which each map constitutes, are
added with the `GenerateMapServer --initMapFromMidMid` option. The municipals
are given in midmif areafiles, one file for every mcm map to create. The
municipal file names are specified in the `AREALIST` and stored in `AREAPATH`,
specified in the variable files (file name without file suffix).

In the initial step also the mcm map gfxData is created from mif files holding
the outline coordinates of each map. The gfxData forms the geographical extent
of each map, which is needed if the decision about in which map the rest of the
items should be added is based on geometry.

The mcm maps are then successively created by adding the remaining item types
from midmif files with the `GenerateMapServer --createItemsFromMidMif` option.
All items of one type can for this step preferably be included in the same
file. The algorithm loops all mcm maps and add the items in the midmif file
that fits the map. Restrictions between street segments and/or ferries are set
using turntables. The items fit the mcm map based on one of these options:

* **Forced**  
  Combine the `--createItemsFromMidMif` option with the `--addAllMidMifItems`
  option. All items in the midmif file will be added to the mcm map. Useful
  when generating mcm maps from midmif, and the country (top region) only has
  ONE mcm map, so all midmif items should be added to one and the same mcm map.
* **Map ssi coordinate**  
  Combine the `--createItemsFromMidMif` option with the
  `--useCoordToFindCorrectMap` option. Used when adding extra items from midmif
  to mcm maps, e.g. in stitching in merge map generation. Not appropriate in
  the raw midmif map generation, since it requires street segments to be
  present in the map already.
* **Settlement order**  
  The mid id of a municipal or city part or built-up area is given, and the mid
  item is added to the mcm map that already has this settlement item added from
  midmif. It reads the midmif_ref file. Hence, the settlement type items must
  be added to the mcm map before the other items referring to the settlement
  items. Useful for raw midmif map generation. The location-group is set.
* **Geometry**  
  If the first coordinate of the item is inside the map gfx data, the item is
  added to the mcm map. Can be used for raw midmif map generation or when
  adding extra items from midmif to mcm maps. It is expensive, especially if
  the map gfx data is complex. This is the slowest method. There is also
  problems with items that has the first coordinate outside or exactly on the
  map gfx data border. Those items will not be added based on geometry. 


When all items have been added to the maps, the map generation continues with
creation of boundry segments, various steps of post processing of the mcm maps,
addition of map corrections and POIs from the WASP database, creation of
external connections, country overview maps, overview maps and regions and map
index file.

Here's a description of the `createItemsFromMidMif()` function:

1. The type of item to create is extracted from the midmif file name.
2. The mif header is read to find out what coordinate systems is used to
   express the coordinates in the mif file.
3. The midmif files are looped and for every line in the mid file:
    1. An empty `gfxData` is created. The `gfxData` creates itself from the mif
       file, given the coordinate system in the mif header.
    2. The `ID` and `name` columns are read from the mid file and saved in a
       temporary name string.
    3. An empty `GMSItem` is created with the given item type. The item creates
       itself from the mid file and is then assigned the `gfxData`.
    4. Then a check is performed to verify that the item fits the map, using
       either a given `settlementId` from the mid file, or a map ssi coordinate at
       the end of the mid row, or based on a geographical test matching the item
       gfxData and map gfxData. If the item does not fit the map: skip to next mif
       feature...
    5. The item is added to the map.
    6. Names and road number are extracted from the name string and added to
       the item.
4. If creating street segments or ferries connections are created. If a
   turntable exists restrictions are extracted and set. If given as parameters
   turn descriptions are generated.
5. If creating street segments that had zip code as attribute in the mid file,
   `zipCode` item are created.

## Variable file

Located in genfiles country directory structure: `genfiles/countries/denmark/script`

Parameters to define in variable files for generating a country from Wayfinder midmif.

* `MAPRELEASE="maprelease"`  
  The name of the country map release directory in the genfiles country
  structure. Example OSM_201005 or TA_2010_06
* `MAP_SUPPLIER="mapSupplierLongVersion_"`  
  The long version of the map supplier followed by the under-score sign. Needed
  in variable file to override makemaps default value "TeleAtlas_" for setting
  the mapOrigin in the mcm maps to generate. Example "OpenStreetMap_"
* `CO_CODE="alpha2_"`  
  The ISO 3166-1 alpha-2 code, 2 letter code for the country followed by the
  under-score sign. Used as prefix in the mcm map name. Example for Denmark
  "dk_" for Sweden "se_" and for France "fr_".
* `AREAPATH="areapath"`  
  The directory where to find the midmif areafiles, the municipal item midmif
  and municipal map mif files for this country. It is located in the
  `genfiles/countries/denmark/OSM_201005/areafiles`
* `MAPPATH="mappath"`  
  The directory where to find all other item midmif files. Typically
  `OSM_201005/wf_midmif/DK`
* `AREALIST="munfiles"`  
  A list with the names of all municipal item files that are to be used in this
  country. File names without suffix. Each municipal file will create one mcm
  map, the first will be map id 0x0, the second map id 0x1 etc. One municipal
  file per row in the variable file, quoted using `"`.

Optional parameters to define in variable files for generating a country from Wayfinder midmif format.

* `ONLYONEMAP="true"`  
  The country has only one mcm map = only one municipal areafile in the
  AREALIST. Use this parameter to force the addition of all midmif items to the
  one map, so addition will not use geometry or settlement ids to check if an
  item fits the mcm map.
* `SKIPBOUNDRYSEGMENTS="true"`  
  The street segments and ferry lines which are on the border of the mcm maps
  are already marked with borderNode in the Wayfinder midmif format mid file,
  so all needed virtual items were created. The map generation will not call
  the method which creates virtual items, the candidates for external
  connections.
* `buildings2D="fullFilePathWithoutSuffix"`  
  If you want to add buildings to the mcm maps from a separate file with
  individual building items (ibi) in Wayfinder mdimif format. The buildings
  could for instance be from Tele Atlas 2D city maps. Point to the full path of
  the ibi file, without the mid and mif file suffix, e.g.
  "fullpath/extraIBI/denmark/dk_individualBuildingItems".  
  This step is necessary only if you for some reason do not want to put the ibi
  files with the other item midmif files in the MAPPATH.
* `ADDRPOINTFILE="fullpath/addressPointsUTF8.txt"`  
  If you want to add names to street segments in the mcm maps from an address
  point file from your map supplier. Point to the fill path of the address
  point text file. See GenerateMapServer help text for option
  --addAddrPointNames for the format of the address point text file.
* `SECTIONEDZIPSDIR="fullpathToTheDirWithSectionizedZipFiles"`  
  If you want to add zip codes from a point file from your map supplier. See
  GenerateMapServer help text for option --addSectionedZipPOIs for the format
  of the zip point file.


## Area files

The midmif area files are located in `AREAPATH`. They really are the municipal
item mid, mif and mapmif files. The municipal item midmif file name without
suffix is listed in `AREALIST`. Each file will create one mcm map.

Example of one area file:

* `dk_whole_municipalItems.mid`
* `dk_whole_municipalItems.mif`
* `dk_whole_municipalItemsmap.mif`

Create area files from the one file with municipals in Wayfinder midmif format,
using `midmif_areaFilesNewRelease`. See script documentation for example
commands.

If it is a large country, the country need to be split in more than one map,
else the mcm maps in the map generation will be too large to handle. The limit
for "large" can be approx more than 500000 street segments or whatever limit
you find out that your system has. However, it does not only depend on number
of street segments but also how many other item types you have in your maps. If
it is a rich map data the size of the mcm map will be larger for the same
number of street segments.

## Wayfinder item midmif files

### File names

Some rules for file names

* The item type must be part of the midmif name, to know what item type to
  create. eg `WFferryItems.mid`,  `WFwaterItems_all.mid`,
  `WFparkItems200912.mid`
* The midmif items will be added to the mcm maps from the files in alphabetical
  order. So file name of area items that are used as region groups (built-up
  areas, city parts) must be lower that the names of other item files
  (alphabetically), to make sure that group items are added to the mcm maps
  before any other items. E.g. using `areaWFbuiltUpAreaItems.mid` and
  `WFstreetSegmentItems.mid` will give correct result. This rule exists to make
  sure that other items can be added to a correct map based on eg built-up
  area settlement id = the `midId` of built-up areas. The city parts mid file can
  refer to the built-up area as settlement id, so the built-up areas must be
  added to the mcm maps before the city parts are added. Using the file names
  `areaWFbuiltUpAreaItems.mid` and `areaWFcityPartItems.mid` will work.
* If the muncipal midmif file is called `dk_whole_municipalItem.mid/mif` the
  municipal map mif file must be called `dk_whole_municipalItemmap.mif`
* The municipal midmif file name is used for deciding the name of the mcm map.
  The `makemaps` script removes everything from the last `_` and adds `CO_CODE`
  from the variable file (if `CO_CODE` already is included in the municipal file
  name it is first removed). Example: a municipal file name
  `fr_whole_municipalItems.midmif` gives mcm map name `fr_whole`.
* The turn table with vehicle restrictions and closed roads must have the same
  name as the street segment item midmif file but ending with `turntable.txt`. If
  the ssi file is called `WFstreetSegmentItems.mid/mif` the turn table file must
  be called `WFstreetSegmentItemsturntable.txt`.

### File char encoding

Names etc from item midmif files need to be converted to UTF-8 before added to
the mcm maps. The easy way is to ave the Wayfinder midmif item files encoded in
UTF-8 before map generation.

In reality the midmif files can have any char encoding. Because the
`NationalProperties::getMapToMC2ChEnc()` is used for building a `CharEncoding` object
that does the convert job. If having char encoding other than UTF-8, update the
`NationalProperties::getMapToMC2ChEnc()` to get the correct encoding for your
specific combination of country and mapOrigin (supplier+release).

### Mid ids

The ID in the Wayfinder mid file must be an uint32 (unsigned integer). It is
used for linking the mid item to mcm map item, writing a midmif_ref file.

### Mif files checklist

Some things to think about for the mif files

* Coord sys tag must be valid. All other stuff from the mif file header is ignored.
    * Available coordinate systems: `mc2`, `mc2_lonlat`, `wgs84_latlon_deg`, `wgs84_lonlat_deg`, `rt90`, `rt90_lonlat`, `utm`, `utm_lonlat`
    * See more details in Wayfinder midmif specification and `GMSGfxData::readMifHeader()`
* Supported mif features
    * Supported: `Point`, `Line`, `Pline`, `Region`
    * It does not support `Pline Multiple`. Split `Pline Multiple` features
      into several `Pline` with the `midmif_fixPlineMultiple.pl` script. (this
      is part of the `midmif_autoParseToWF` script).
    * Cosmetic mif features such as `Pen`, `Brush, `Center`, ... are ignored
      (skipped). If there is a cosmetic mif feature on the last row of a mif
      files, this must be removed manually. Else the map generation will exit
      with error message.

## Sample data

Here are a few examples to illustrate the Wayfinder midmif data format.

Examples of item mid files

	##### > hu_municipalItem.mid<==
	197,"Budapest","Budapest:officialName:hun"
	215,"Érd","Érd:officialName:hun"
	219,"Salgótarján","Salgótarján:officialName:hun"
	
	##### > areaWFbuiltUpAreaItems.mid<==
	197,"Budapest","Budapest:officialName:hun"
	201,"Pécs","Pécs:officialName:hun"
	205,"Székesfehérvár","Székesfehérvár:officialName:hun"
	
	##### > areaWFcityPartItems.mid<==
	3342,"I. kerület","I. kerület:officialName:hun 1. kerület:alternativeName:hun"
	3343,"II. kerület","II. kerület:officialName:hun 2. kerület:alternativeName:hun"
	3344,"III. kerület","III. kerület:officialName:hun 3. kerület:alternativeName:hun"
	3345,"IV. kerület","IV. kerület:officialName:hun 4. kerület:alternativeName:hun"
	
	##### > WFaircraftRoadItems.mid<==
	1,"",""
	2,"",""
	
	##### > WFairportItems.mid<==
	1,"Ferihegyi repülõtér","Ferihegyi repülõtér:officialName:hun"
	2,"",""
	
	##### > WFallparkItems.mid<==
	1001,"","","cityPark"
	1002,"","","cityPark"
	1003,"","","cityPark"
	1004,"","","regionOrNationalPark"
	1005,"","","cityPark"
	
	##### > WFferryItems.mid<==
	38912,"","7102:roadNumber:invalidLanguage",3,5,5,0,0,0,0,"Y"
	46502,"","5107:roadNumber:invalidLanguage",3,90,90,0,0,0,0,"Y"
	61387,"","4522:roadNumber:invalidLanguage",3,90,90,0,0,0,0,"Y"
	61673,"","4412:roadNumber:invalidLanguage",3,50,50,0,0,0,0,"Y"
	
	##### > WFindividualBuildingItems.mid<==
	2,"","","airportTerminal"
	3,"","",""
	4,"","","parkingGarage"
	5,"","",""
	
	##### > WFrailwayItems.mid<==
	1,"",""
	2,"",""
	3,"",""
	4,"",""
	5,"",""
	
	##### > WFstreetSegmentItems.mid<==
	1,"","M1:roadNumber:invalidLanguage E60:roadNumber:invalidLanguage",0,130,130,3,0,,,,,0,0,0,0,"Y",0,0,"N","N","N","N","N","Y","N","","",197,197,8,"","Y",-1
	2,"","M1:roadNumber:invalidLanguage E60:roadNumber:invalidLanguage",0,130,130,0,3,,,,,0,0,0,0,"Y",0,0,"N","N","N","N","N","Y","N","","",197,197,8,"","Y",-1
	3,"","M1:roadNumber:invalidLanguage E60:roadNumber:invalidLanguage",0,130,130,0,3,,,,,0,0,0,0,"Y",0,0,"N","N","N","N","N","Y","N","","",197,197,8,"","",-1
	4,"","M1:roadNumber:invalidLanguage E60:roadNumber:invalidLanguage",0,130,130,3,0,,,,,0,0,0,0,"Y",0,0,"N","N","N","N","N","Y","N","","",197,197,8,"","",-1
	2677,"Hungaria körut","Hungaria körut:officialName:hun",1,70,70,3,0,,,,,44,58,0,0,"Y",0,0,"N","N","N","Y","N","N","N","1146","1146",197,197,8,"","",-1
	3447,"","",1,70,70,0,3,,,,,0,0,33,41,"Y",0,0,"N","N","N","Y","N","N","N","1143","1143",197,197,8,"","",-1
	58729,"Emma Utca","Emma Utca:officialName:hun",4,50,50,0,0,,,,,0,0,0,0,"Y",0,0,"N","N","N","N","N","N","N","2030","2030",215,215,8,"","",0
	
	##### > WFwaterItems_all.mid<==
	1,"Balaton","Balaton:officialName:hun","lake"
	2,"Tisza","Tisza:officialName:hun","river"
	3,"Duna","Duna:officialName:hun","river"
	
	##### > WFstreetSegmentItemsturntable.txt<==
	KEY     NODE_   ARC1_   ARC2_   IMPEDANCE LAT   LON
	27651   -1  -1  3937    -1      0       0
	284461  -1  -1  45868   -1      0       0
	382068  -1  -1  62889   -1      0       0
	484349  -1  -1  79976   -1      0       0
	
Examples of street segment item mif file
	
	Version 300
	Charset "WindowsLatin2"
	Delimiter ","
	CoordSys wgs84_lonlat_deg
	Columns 33
	  id Decimal (11,0)
	  Name Char (25)
	  Allnames Char (254)
	  Roadclass Decimal (11,0)
	  Posspeed Decimal (11,0)
	  Negspeed Decimal (11,0)
	  PosEntryR Decimal (11,0)
	  NegEntryR Decimal (11,0)
	  Nbrlanes Decimal (11,0)
	  Width Decimal (11,0)
	  Height Decimal (11,0)
	  Weight Decimal (11,0)
	  Lstart Decimal (11,0)
	  Lend Decimal (11,0)
	  Rstart Decimal (11,0)
	  Rend Decimal (11,0)
	  Paved Char(2)
	  Level0 Decimal (11,0)
	  Level1 Decimal (11,0)
	  Roundabt Char (2)
	  Ramp Char (2)
	  Divided Char (2)
	  Multi Char (2)
	  roadToll Char (2)
	  contrAcc Char (2)
	  roundaboutish Char(2)
	  zipLeft Char(254)
	  zipRight Char (254)
	  adminAreaIdLeft Decimal (11,0)
	  adminAreaIdRight Decimal (11,0)
	  adminAreaOrder Decimal (11,0)
	  borderNode0 Char (2)
	  borderNode1 Char (2)
	  roadDisplayClass Decimal (11,0)
	Data
	Pline 5
	17.123847 47.916151
	17.120732 47.918038
	17.111679 47.924296
	17.111272 47.924545
	17.110997 47.924705
	Pline 6
	17.123957 47.916303
	17.122986 47.916887
	17.120899 47.918155
	17.111618 47.924464
	17.111219 47.924713
	17.111056 47.924801
	Pline 2
	17.124212 47.916149
	17.123957 47.916303
	Pline 2
	17.124665 47.915656
	17.123847 47.916151
	Pline 3
	17.127317 47.914269
	17.125747 47.915218
	17.124212 47.916149
	Pline 2
	17.127394 47.914004
	17.124665 47.915656
	Pline 10
	17.142915 47.906592
	17.141807 47.906957
	17.139402 47.907863
	17.137203 47.908763
	17.135303 47.909621
	17.133693 47.910396
	17.132609 47.910956
	17.130348 47.912264
	17.128476 47.913349
	17.127394 47.914004

