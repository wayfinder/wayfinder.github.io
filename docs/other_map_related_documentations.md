---
title: Other map related documentations
layout: single
---

## Country polygons and border items

### The country polygon

The country polygon is the geometry for the land mass of a country (mainland
and islands in the ocean). This should be as detailed as possible geometry-wise
since it is used for displaying land areas in map images. It is represented by
the mapGfx data of country overview maps (co-maps) and created from a mif file
with the country geometry. The mif file contains ONE mif feature Region with
many polygons. There is one mif file for each country overview map of each
country, the name of the mif files(s) are the same as the names of the country
overview map(s) followed by the suffix .mif. A very large country may have more
than one country overview map, called e.g. germany_1 and germany 2, must have
country polygon mif files named germany_1.mif and germany_2.mif. Both those
country polygon mif files must have the exact same content, ideally let them be
links to a file called `germany.mif`. The country polygon mif file(s) are stored
in the genfiles country directory structure
`BASEGENFILESPATH/countries/{country}/countrypol`

The country polygon is filtered in coordinate levels to define which geometry
to draw for different zoom levels. To save time in the map generation process,
it is possible to re-use a pre-filtered country polygon from a country overview
map stored on disc. The map id of the co-map is not important, the map
generation process uses the name of the co-map to find which co-map to re-use
filtering from. The co-map(s) with pre-filtered country polygon are also stored
in `BASEGENFILESPATH/countries/{country}/countrypol`

In map merges with several countries, each country polygon is filtered re-using
filtering results from the neighbouring countries to avoid gaps on borders. To
define where a border part should start and end we have a file which defines
these break point coordinates. The break points file is located on disc
`BASEGENFILESPATH/countrypol/countryBorderBreakPoints.txt`. The country polygon
mif files must include the break point coordinates.

The country polygon must not overlap any water in the ocean, since water items
with water type ocean and otherWaterElement are not included when drawing map
images. 

#### Create the country polygon

The country polygon can be created in a generic way by merging the geometry of all municipal items in one country.

* Print municipals from mcm-maps using GenerateMapServer
   * `./GenerateMapServer --printMidMifMapGfx=FILENAME 00*.mcm`
   * FILENAME = eg mun_norway
   * `mun_norway.mid` and `mun_norway.mif` are created
* Use MifTool or ARcView or other GIS software to dissolve all municipals to one region.
* Then you need to remove any ocean/coastal water that covers the land.
  Typically you need to remove all land that is covered by water with type
  `otherWaterElement`. You can print water items to midmif with `GenerateMapServer
  --printMidMif` option

### Country borders and border items

Border items in mcm maps defines the borders between countries on land.

#### Help files

When generating country overview maps, a few files are used and or created for handling country borders and border items.

`countryBorderBreakPoints.txt`:

* File containing coordinates defining start and end of common border parts between countries. Located in BASEGENFILESPATH/countrypol/
* Use: identifying border parts for common country polygon filtering. 
	
	  703748538 136738514  swe-no-ocean
	  823915938 245139346  swe-no-fin
	  785357201 288128788  swe-fin-ocean
	  823841138 345148304  no-fin-russia

`countryBorders.txt`

* key word: BORDERPART countryMapName
* A list with all filtered parts of the polygon(s) from the country polygon
  that are split up by coordinates defined in countryBorderBreakPoints.txt. The
  key word line is followed by 3 identifying coordinates (start, end,
  onTheWay), one row with the number of coordinates in the filtered border
  part, and finally all filtered coordinates.
    * Use: common country polygon filtering and creation of borderItems.txt file. 

	  BORDERPART sweden
	  (703748536,136738512)
	  (823915936,245139344)
	  (767435509,168669286)
	  2644
	   703748539 136738515
	   703741240 136735843
	   ...


`borderItems.txt`:

* key word: BORDERITEM borderPartNumber (from countryBorders.txt)
* A list with all true borders = borders on land that exist between the
  countries included in a merge map generation. It is mapped to the
  countryBorders.txt by borderPartNumber.
* Use: creating border items. 

	
	  BORDERITEM 1
	   from (703748536,136738512)
	   to   (823915936,245139344)
	   via  (767435509,168669286)

#### Country borders

The first country in east world merge list is Sweden. The main polygon in the
Swedish country polygon is split by 3 country border break points; the border
to Norway, the border to Finland, and the coast. After generating the Swedish
country overview map there are 3 BORDERPART keys in the countryBorders.txt
file. The key word line is really made up of "BORDERPART sweden" meaning that
the border part was written to countryBorders.txt when processing the co map
named sweden.

The next country in EU merge list is Norway. The main polygon of the Norwegian
country polygon is split by 5 break points; the border to Sweden, the border to
Finland, the border to Russia, and then the coast is split in 2 pieces for
faster filtering. When processing the Norwegian co map, the existing
countryBorders.txt file is read, looking for any old border parts that fit the
Norwegian parts. And, yes there is one. The Swedish part that is the border
towards Norway exists already. The key word line says that the BORDERPART was
written for co map sweden, and now when processing co map named norway this
means that we have a true border = a border on land. This information is
written into the borderItems.txt file. Key word is "BORDERITEM N" where N is
the border part number. The identification coordinates are also written. When a
border part is re-used like this, no further info is written into the
countryBorders.txt file.

For a large country with many country overview maps, e.g. if Germany has 8 co
maps. After the first co map germany_1 is processed, all German border parts
exist in the countryBorders.txt file with key word BORDERPART germany_1. When
processing the second German co map named germany_2, all border parts match the
ones in the countryBorders.txt file, but nothing is written to the
borderItems.txt file, since we have the same country. The test for "same
country" is done with memMap between the country polygon mif files;
germany_1.mif and germany_1.mif are exactly equal, norway.mif and sweden.mif
are not.

#### Border items

When all country overview maps for all countries in the east world merge have
been created, the border items are created looping all co-maps in the
merge-directory. Again we start at the first co map: Sweden. The country
polygon of the Swedish co map is loaded and border parts are re-identified
using the country border break points file. For each border part, the
borderItems.txt file is looped. When the identifying coordinates match the
current border part, a border item is created with gfxData from the current
border part.


### Create the co-map with pre-filtered country polygon

If you needed to edit the country polygon geometry of a country that already
has a pre-filtered country polygon stored in genfiles/country/countrypol, this
is how to proceed. Follow these instructions to create new co-map(s) with the
pre-filtered country polygon for a certain country. If borders between several
countries were changed, you need to create new pre-filtering for all countries
that share the changed borders.

* Create a working directory and copy the new country polygon mif file(s)
* Copy from the latest east/west world merge
    * The country overview map(s) of the country
    * The countryBorders.txt file
    * The borderItems.txt file (at least if you fix update country polygons directly in a merge)
* Identify the BORDERPARTS of countryBorders.txt that will have a new geometry with the new mif file(s). Remove those border parts from the file. And remove them also from the borderItems.txt file.
* Set a new mapGfxData of the country overview map(s) from the new mif file
     * `MapTool --mapLab2=setNewMapGfx 080000003.mcm|& tee mt_setNewMapGfx_080000003.log`
* Filter the country overview map(s). It re-uses border parts from countryBorders.txt or creates new border parts if no match is found.
      * `GenerateMapServer --filterCountryPolygon --coPolBreakPoints=fullPath/genfiles/countrypol/countryBorderBreakPoints.txt 080000003.mcm|& tee gms_filterCountryPolygon_080000003.log`
* Create the gfx filtering
      * `GenerateMapServer --gfxFilteringOfCoPols 080000003.mcm |& tee gms_gfxFilteringOfCoPols_080000003.log`
* Fix border items
      * Remove the old border items: `MapTool --removeitems=borderItems 080000003.mcm |& tee mt_rmBorders_080000003.log`
      * Create new ones: `GenerateMapServer --coPolBreakPoints=fullPath/genfiles/countrypol/countryBorderBreakPoints.txt --createBorderItems 080000003.mcm |& tee gms_createBorders_080000003.log`

Done! Check in MapEditor that the mapGfxData has one geometry for every filter
level. If only the coastal parts of the country polygon was updated you can
check that the new mapGfxData is exactly the same as the border items.

Copy the country overview map(s) together with the new country polygon mif file
to the countrypol directory on genfiles
`genfiles/countries/{country}/countrypol`. So they match for next map generation.

If you did this hacking in an existing eastWorld or westWorld merge, you also need to

* Update creation times
* New index.db + m3 maps for the country overview maps
* New search and route caches for the co maps 

## Filtering in map generation

The gfxData of mcm maps and items in the mcm maps need filtering to have a more
simple graphical representation to display when zoomed out. Two different
filterings are available for map generation.

* A simple filtering that only removes unnecessary coordinates from the gfxData (coordinates on a string).
* A level filtering that gives the gfxData a set of graphical representations to use for different zoom levels. 

### Filtering map items

#### `GMSUtility::filterCoordinateLevels` (level filtering)

Currently NOT used in map generation

This filtering in levels is done with `GMSUtility::filterCoordinateLevels`
filtering items in the maps. After filtering the
`OldGenericMapHeader::m_mapFiltered`  is set to true. It can be included in map
generation calling `GenerateMapServer --filterCoordinates --filterLevels
000000000.mcm`

Process:

* The gfxData of the items are first filtered to get rid of coordinates on a string, with filter distances
    * area items 3 meters
    * line items 1 meters, but 50 meters in the country overview map
* The gfxData of the items is updated. The coordinates which to display only on level 0 are removed, and the coords which to display on level 1 (after coords-on-string filtering) are kept.
* The gfxData of the items is filtered again to get the 16 filtering levels.  The filtering distances are fetched with the `MapFilterUtil::getFiltDistanceForLineItem` and
  `MapFilterUtil::getFiltDistanceForAreaItem`. 

Only items of specified item types are filtered. Items of all other types have
the coordinates of their gfxData altered to represent filter level 15. This
means that they are displayed on all levels.

#### GMSUtility::filterCoordinates

Currently used in map generation in the final filtering step in the map generation process

Included in map generation when calling `GenerateMapServer --filterCoordinates 000000000.mcm`   
Used with the purpose of removing coordinates on a string. Not using the filter
levels in coordinates, instead the items are filtered only once.

* First the `GfxData::openPolygonFilter` to get a reference path.
* Then with `GfxFilterUtil::filter` to get the final result. 

This method does not consider shared borders between neighbouring area items.

### Filtering country overview map gfxData (the country polygon)

#### `GMSUtility::filterCountryPolygonLevels` (level filtering)

Currently used in map generation.

This filtering in levels is done with `GMSUtility::filterCountryPolygonLevels`
filtering the map gfx data of country overview maps (country polygon). After
filtering the `OldGenericMapHeader::m_mapGfxDataFiltered` is set to true. It is
included in map generation when calling `GenerateMapServer -X`

   * processXMLData -> createCountryMaps -> createCountryMapBorder -> filterCoPolCoordinateLevels 

If to test separately, called with `GenerateMapServer --filterCountryPolygon 080000001.mcm`

The filtering distances are fetched with the
`MapFilterUtil::getFiltDistanceForMapGfx` method, and defined in the
`MapFilterUtil::m_mapGfxFilterDistances` vector intiated in the
`MapFilterUtil::initMapGfxDistances` method.

Needs to be combined with a file containing break points, which is a set of mc2
coordinates defining the corners of country polygons where they meet the
country polygon of other countries. The country polygon gfxData is split in
border parts based on the break points, end each border part is filtered
individually. The filtering of the first country is then re-used when filtering
the neighbour country, to get a shared border on all levels.

#### `OldCountryOverviewMap::makeSimplifiedCountryGfx`

Used in map generation. Included in map generation when calling `GenerateMapServer -X`

The country overview map has a Stack `m_simplifiedGfxStac´ allocated to be
`[NBR_SIMPLIFIED_COUNTRY_GFX][m_nbrGfxPolygons]? . It is an array of stacks
pointing to coordinate indexes to use for ? NBR_SIMPLIFIED_COUNTRY_GFX? 
pre-decided filterings (currently 7). This array is saved to and read from
disc. Uncertain if this is used for any map images...

The array is created in `OldCountryOverviewMap::makeSimplifiedCountryGfx` based
on some of the coordinate filter levels. Which levels to use in the array is
defined in `OldCountryOverviewMap::mapGfxFilterLevels`.

Previous to the filter levels stored in coordinates, the simplified country gfx
in `makeSimplifiedCountryGfx` was created with the `GfxData::getSimplifiedPolygon`
method.

* The filtering distances are decided in `OldCountryOverviewMap::filtMaxDistance` and `OldCountryOverviewMap::filtMinDistance`, where the min values are always 0.
* This filtering method is fast when the max distance is a large value, slow when the max dist is a smaller value. 

## Walk-through of map generation process

### Country generation with makemaps script

The `makemaps` script is used for generating individual countries. The script is
located in the mc2 repo in `Server/bin/Scripts/MapGen`

Variables set in the `makemaps` script may be over-rided by options and/or by the
variable file. Some variables controlling the script are only set in the
variable file, see [the Variable file
section](../wayfinder_midmif_format/#variable-file)

For following the actions in the makemaps script, start reading from "Determine
which side to drive on in the country"

### Merge generation with `makemaps.mcgen` script

The `makemaps.mcgen` script is used for merge map generation and generation of
multiple individual countries. The script is located in the mc2 repo in
`Server/bin/Scripts/MapGen`.

Variables set in the `makemaps.mcgen` script may be overriden by options.

For following the actions in the makemaps.mcgen script, start reading from

* "Generates single countries in" for multi generation of individual countries
* "Merge map generation starts" for merge map generation

## MapEditor

The MapEditor is a tool where you can examine the content of mcm maps. It shows
you all items and attributes, how street segments and ferry lines are
connected, etc. The MapEditor also allows you to edit attributes that are not
correct, creating map correction records which can be added to the maps in the
map generation to fix problems. The MapEditor is implemented using gtkmm2.4.

Start MapEditor from a terminal window with the mcm map to load in the tail,
either the hex id of the mcm map (if in current directory) or the full path
to the mcm map file name.
Misc debug info will be printed in the terminal window while working in the MapEditor.

	./MapEditor 1a
	./MapEditor mcm/OSM_201005/dk_20100520_first/00000001a.mcm

The main MapEditorWindow has two parts; the map area to the left and the menue area
to the right. In the top title bar the decimal ID and the name of the loaded mcm map
is displayed, along with the map origin string (the map release with the long
version of the map supplier string).

In the map area you zoom in with left mouse click and zoom out with right click.
Middle mouse button click will select the closest displayed item.
Decide which item types to display in the map in the map area with
the "Drawsetting"-button from the menue area, or short command "d" with
the MapEditorWindow active.

Close the MapEditor program by pressing the "Quit"-button in the menue area, or
simply by short command "q".

The menu area displays some info about the maps: the decimal map ID, the true
creation time (when the map was created in the individual country generation),
and time for latest WASPing (adding POIs from WASP database) and extradata
time (adding map correction records from WASP database).

You can search for mcm item ID, item name (exact spelling, case sensitive) and
POIS with certain WASP POI ID with the "Search for item with ID".
With the "Search for coordinate" you can search for just that, coordinate given in 
either wgs84 degrees or mc2 works.

When searching for an item in the menue area or middle-clicking in the map
area selecting an item, an item info widget will open displaying info about
the item, e.g. location in search index and all attributes. The selected item will also be
highlighted with red color (if part of the displayed item types).
For routeable items (street segments and ferry items) node attributes 
are listed by selecting one node (0 or 1),
connections to each node from other routeable items' nodes are listed when pressing
the little arrow in front of the node (0 or 1). Selecting one connection will
display the turn description and vehicle restrictions for the connection.
A selected node is highlighted with a red square in the end of the routeable item it belongs to.
A selected connection is also highlighted, orange-yellow square and line is the node and routeable item which the connection comes FROM, red square and line is the routeable item which the connection leads TO.

When one item selected, in the item info widget press:

* the "Coords" button to get the coordinates of the selected item geometry printed in the terminal window
* the "MidMif" button and the selected item is printed to item.mid and item.mif. The result is similar to the Wayfinder midmif format, but especially the routeable items will have different attributes.

You can load a text file with either mcm item ids or mc2 coordinates when you start the MapEditor with ''--highlightIDs=ITEMIDFILE'' resp ''--highlightCoords=COORDINATEFILE'' options. If you loose the highlight when clicking around in the map, press the "Re-highlight ids"-button and they are back.
The format of the rows in the highlightIDs file: "itemID". If you want to add special colors to the items the rows are "itemID color" (separated by space). You can choose from the colors defined in the MapEditor ''MEGdkColors::color_t'', e.g. purple1 or green3. Some of the colors are organized in the order of the rainbow, items highlighted with colors closer to purple are drawn before the items with colors closer to red.
The format of the rows in the highlightCoords file: "mc2lat mc2lon" (separated by space).

It is possible to edit most attributes in the MapEditor to create map correction records (extradata records).

* Example of correcting a mis-spelled name: Select the item which has the incorrect name. In the "Names"-part of the item info widget, select the mis-spelled name from the name list and press "Edit". The "Edit Name" dialog will open. Enter the name spelled the correct way and press OK. *NB* that you must enter the names in latin-1 (ISO 8859-1) char encoding, since MapEditor converts all strings from latin-1 to UTF8 before adding it to the map. If you want to add a new name to the selected item, instead select the "--Add name--" entry and press "Edit". Add the name and specify name type and language and press OK.
* The correction records will be written to the file names in the "Extradata log filename" box, default name is "changelog.ext" if you do not change it to something else.
* Pressing the "Delete"-button in the item info widget will remove the selected item.
* Pressing the "OK"-button in the item info widget or name dialog will save the attribute changes and write records to the extradata file. The changes are applied on the mcm map locally loaded in MapEditor, but the mcm map on disc is not affected.
* There is a "log comment box" in the item info widget and name dialog just above the OK-button. Fill the fields there if you want to have something special. Input must be in latin-1 (ISO 8859-1) char encoding.
* Pressing the "Save"-button in the menue area, the mcm map will be saved to disc. The simple rule is AVOID doing this! There is a "are you sure?" dialog opening if you press the "Save"-button by mistake. To see the result of map correction records, it is better run a individual country second map generation, adding map corrections from the WASP database to the maps.

The "Show extradata"-button is used for browsing extradata files, with records extracted from the WASP database with ''addED.pl -f extractED''. It is typically used when migrating map correction records to a new map release.

The "Show nodes"-button is another analyze function that can be used for analyzing the result of changes in the method that calculates turn descriptions. Create input for the "Show nodes" with MapTool --compturns option.

The "Filter level" box and "Update"-button is for inspecting filtering of the country polygon = the map gfx data in the country overview maps. The country polygon is filtered in 16 levels (0-15), level 15 has the least number of coordinates, level 14 has all the coordinates of level 15 and some more, level 13 has all the coordinates of level 14 and some more, etc. Enter a level and press "update" to see how the different filterings look like. This only makes sense for the country polygon of country overview maps, no other items in any maps are filtered in levels like this.


