---
title: Wayfinder maps
layout: single
---

## Organization of map data

The known world is divided into maps. 
 
A map covers one part of, or a whole, country and consists of one or more municipalities. Every
municipality is covered by only one map. Ideally all maps would have the same amount of data, so
the number of municipalities covered by one map would depend on their size. All maps have a unique
mapID-number.

All objects in the maps are referred to as items. An item might be a lake, a street segment,
a building, a point of interest, a built-up area etc. Items are always located in one single map.

All items have a item id that is unique within the map, item type, and names.
Most items have a geographical representation in the item gfxData and groups
defining the item in the search index.

The items are grouped in "zoom levels" in the map. The structure of the levels
is lightly connected to geographical importance, i.e items that should be
visible on maps that cover a large area (e.g. Europe) are set to have a low
zoom level.  
There are 16 zoom levels, 0 - 15. Zoom level 0 contains the largest items and
zoom level 13 the smallest. Zoom level 14 is  reserved for point of interests,
zoom 15 for category items (deprecated item), and zoom 10 for zip code items.
The zoom level is set in the item id, so the itemID depends on which zoom level
in the map it is added to.

	
	  Item ID:
	  MSB                                                        LSB
	  +-------------------------------------------------------------+
	  |N | ZOOM | ID                                                |
	  +-------------------------------------------------------------+
	   31 30  27 26                                                0
	  Bits:
	  0-26 Number within this zoom level (starts at 0)
	  27-30 Indicates the zoom level of this item
	  31 Used to handle nodes, and whether group items should be used as location \\
	     in search hits or not, ignored otherwise


## The internal map formats

The `.mcm` map is the internal map format that is used during map generation.
When all steps of the map generation has been performed the mcm map is
converted into the internal MC2 server .m3 map format. From the m3 map are
further extracted specific information for searching and routing and stored in
the search and route caches.

###  mcm map

The mcm map is of class `OldGenericMap` and has a "header" and a "body". In the header OldGenericMapHeader you find administrative info such as time stamp from creation time (first time the map was saved to disc), time of last save, time of last contact with the POI database WASP; the version of the mcm map format `m_loadedVersion` (currently 9), the id and the name of the map file, the map origin, the driving side (left or right) of the country the mcm map belongs to, and a vector with native languages in this map. The native languages `m_nativeLanguages` are calculated with OldGenericMap::updateNativeLanguages. The map origin string `m_origin` is set by GenerateMapServer from CL_maporigin option when creating the mcm map, and contains the long version of the map supplier and the map release, e.g. OpenStreetMap_201005.

In the mcm map body you find, e.g. 

* Map data items structured in the "zoom levels"
* Geometry of the map file, the so-called map gfxData
* Item name table with all names of all items in the map. The item name
  table is encoded in UTF-8 char encoding
* Map hash table, used to search for items close to a coordinate. All
  objects in the map that have a geographical representation gfxData are sorted
  on their location in this object. This means that it is fast to get the
  closest or all objects within a given distance from a coordinate.

In the end of the body there is a range of tables with information or extra
attribute that does not fit into any of the items' class variables and
structure. In that way we can avoid to break the mcm map format version too
often, instead the OldGenericMap::internalLoad which reads the mcm map from
file continues to read as long as there are more things to read.

The `OldGenericMap` and the "Old*item" classed has a corresponding GMS-version
where GMS stands for GenerateMapServer, the main binary used in the map
generation process. The `GMSMap` and the `GMSStreetSegmentItem` etc classes hold
extra attributes and methods that are temporarily needed during map generation.
Saving a GMSMap to disc results in a OldGenericMap.

###  m3, map module map

The m3 map is of class `GenericMap` and has the same structure as the mcm with a
“header” and a “body”. It is created from the mcm map with the `MapHttpServer´
binary with the `M3Creator` class.

The m3 map is optimized for fast access in the MC2 server. It has the same
content as the mcm map, with addition of POI info for the POIs that are stored
in the map file. The POI info is fetched from the WASP database `POIInfo`,
`POIInfoDynamic` tables with the `GetAdditionalPOIInfo` class and typically
contains the address, city, map supplier, phone number, etc for the POI.

### search map

A map optimized for serving search. Search specific information, such as item
names and search index, is extracted from the m3 map and saved to the search
map with `MapModule --saveSearchMap` option.

### route map

A map optimized for serving routing. Route specific information, such as
connections and traffic rules, is extracted from the m3 map and saved to the
route map with `MapModule --saveRouteMap` option. 

## The map hierarchy

There are 3 different levels of mcm maps, and the country overview map. This
map hierarchy exists for mcm maps, m3 maps and the search and route caches.

First we have the normal mcm maps, called underview maps. This is the
`OldGenericMap` class, or in map generation the `GMSMap` class. These maps contain
all map data items. Normally one country is split into several undreview maps
in order not to get the map files to large. 

Next is the first level overview maps. This is the `OldOverviewMap` class, with
map level 1. The overview map contains data for searching and overview routing
from a set of underview maps. Normally one country has one overview map, so all
underview maps of the country is included in the overview map. Exceptions are
allowed, no problem splitting large countries into several overview maps. But
one underview map must belong to only one overview map. All search areas from
the underview maps are copied to the overview map to serve searching. All ferry
lines and street segment with road class 0-2 are also copied along with any
street segment of road class 3 or 4 connecting to road class 0-2 segments that
are roundabouts. These overview street segments are used for long-distance
routing, typically country level. The special road class 3 and 4 roads are
needed in order to get correct turn descriptions for the the long route passing
roundabouts. Item from the underview masp are added the overview map with
OldOverviewMap::addMap.

Next is the second level overview map, the super overview map. This is also the
`OldOverviewMap` class, but with map level 2. There is typically one super
overview map for the West World, i.e. North and South America and one for East
World, i.e. Europe, Africa, Asia, Oceania. One first level overview map must
belong to only one super overview map. The super overview map contains a copy
of the ferry lines and road class 0-1 street segments and roundabout connected
road class 2-4 from the overview maps, and it is used for really long-distance
routing, typically continent or large country level.

Parallell to the first level overview maps there is the country overview map
(*co-map*). This is the `OldCountryOverviewMap?  class, in map generation the
`GMSCountryOverviewMap` class. The co-map contains map data from the same
underview maps that the overview map does. The purpose of the country overview
map is to have map features for drawing map images on zoomed out levels, i.e.
it contains "large" items that needs to be lifted/copied one level up to be
included in zoomed-out map images. From the underview maps are copied street
segments road class 0-1, built-up areas and municipals. So there is one version
of the street network for display on zoomed-out levels in the country overview
map containing only the most important roads, and one version with the complete
street network (all road classes) in the unverview maps. Large waters and
forest area are not copied, but instead moved from the underview maps to the
country overview map. So the same water and forest features are used in zoomed
out level and zoomed-in level map images. The country overview map also
contains the country polygon for displaying land areas, and country border
items for displaying borders between countries.

## The index.db

The index file index.db is created in all map generations. with
`GenerateMapServer --createNewIndex` option, calling `GenerateMapServer
createNewIndex()` function calling `MapIndex:createFromMaps()` function.

The index.db file contains 

* Information about all maps that are part of the map generation which is
  stored in map notices of class MapModuleNotice
   * MapID, bounding box of the map gfxData, creation time = the date time the
     map was last saved, map name, and country code of the map
* Information about which overview maps belong to which top region
* Names on all languages for all top regions that are part of the map
  generation
* Information about the map hierarchy. Which overview map is the overview of
  which underview etc.
* Available map supplier copyright strings
* Copyright bounding boxes, defining which map supplier copyright string that
  will be used when displaying a map image over a certain area of the world.

The `GenerateMapServer --listIndex` option loads an `index.db` of a map
generation and lists the content of it to standard out.

## The top region

A top region is a country or state that is used for delimiting searching. Eg
search for a hotel in a city in a top region.  Countries and states are typical
top regions.  
A TopRegion can consist of one or several overview maps of one country. The
`region_ids.xml` file contains the region ids of all available top regions. It is
not allowed to change region ids of existing regions, however it is allowed to
add new regions.

The `country_ar.xml` file in the genfiles country structure contains add_region
xml-tags, and is used in the map generation to define the coverage of the top
regions.

## Map items

All objects in the maps are referred to as items. An item might be a lake, a
street segment, a building, a point of interest, a built-up area etc.
See `ItemTypes::itemType` for a complete list of the different types of item.

The items are objects of the class `OldItem` (class Item in m3 maps) or any of
the item classes that inherits from the generic `OldItem`, eg
`OldStreetSegmentItem` for street segments (class `StreetSegmentItem` in m3 maps).
The `OldItem` has a item id (unique within the map), item type, a name vector
with all names of the item, geometry in item gfxData and the region groups of
the item (for the search index). The special item classes contain item type
specific attributes beyond what's given in the generic `OldItem` class. Eg the
park item `OldParkItem` has a park type attribute and the street segment item
`OldStreetSegmentItem` has attributes for road classification, speed limits and
house numbers, and structures for nodes and connections to other street
segments sharing the same nodes.

Street items (class `OldStreetItem`) is a group item, grouping street segments
that share one name and are located close to each other. The street item
exists, so that when searching for a street name, the user will find one search
hit representing the street, and not one search hit for every street segment
carrying the street name. If seraching for a street address, complete with
street name and house number, the street segment carrying that house number is
returned of course.  
The streets are created with `GMSMap::generateStreetsFromStreetSegments()`. The
max distance between two street segments for considering them to belong to the
same street depends on road class and is defined in
`NationalProperties::getStreetMergeSqDist()`.

Zip code items (class `OldZipCodeItem`) are not created from a Wayfinder midmif
files with zip codes. The Wayfinder street segment midmif file has attributes
for zip code left side + right side for each street segment. The zip code items
are built from these attributes with the
`GMSMap::createZipsFromGMSStreetSegmentItems()` function. If the supplier does
not provide zip codes as attributes of the street segment, the other way of
creating zip codes is by adding them from a zip POI file, with
`GMSMap::addZipsFromPOIFile()` function. It is called by the `GenerateMapServer
--addSectionedZipPOIs` option.

Water items that have waterType otherWaterElement or ocean, will be removed
from the mcm maps in the mixed post processing step of the map generation.
Those waters are waters that represents oceans, thus outside of the country
polygons so we don't draw them in the map image because they will not add any
value.

### Routeable items, nodes and connections

Street segments and ferry lines are routeable items. The `OldRouteableItem` class
has everything that is needed to model a connected network. The routeable items
have nodes from the `OldNode` class, and the nodes can store connections from the
`OldConnection` class connecting the network.

Every routeable item has two nodes, referred to as node 0 and 1. Node 0 is at
the first coordinate of the item gfxData and node 1 is at the last coordinate
of the item gfxData.  The ID of node 0 is the same as for the item and the ID
of node 1 has the most significant bit (MSB) set to one (bit 31 in the example
in "Organization of map data").

Both nodes and connections store navigation attributes.  The nodes have
attributes that describe the maximum allowed speed, the maximum height etc.
They also have a field that describes possible restrictions when entering the
street segment at this node, such as one-way street and no throughfare, node
level for tunnels and bridges in grade separated crossings Every node has a
(possibly empty) list of connections with nodes from where it is possible to
drive to this node. The max number of connections to a node is defined in
`MAX_NBR_ENTRYCONNECTIONS`, currently 64.  The connections contain attributes for
e.g. the vehicles restrictions defining which vehicles that are allowed to
traverse the connection (see `ItemTypes::vehicle_t`), and the turn description
defining how to turn when traversing the connection (see
`ItemTypes::turndirection_t`).  Connections cannot be created between nodes with
different node levels.

This figure illustrates how the connections are created between nodes of street
segments. To node index 1 of street segment id 0 it is possible to drive from
node 3 of street segment 1 and node 5 of street segment 2.

![map_street_segments_nodes_connection](/images/map_street_segments_nodes_connection.png)

Another nice way to examine how the connections are created and stored, is to
load a mcm map into the MapEditor tool and select connections in the item info
widget of nodes of routeable items.

### External connections

Normal connections are created between nodes of routeable items within one mcm
map. For routing between mcm maps we have external connections that connects
border nodes of routeable items in neigbouring mcm maps, i.e. two mcm maps that
are neighbours that share parts of each others geometry outlines.

For modelling external connections the maps contain something called boundry
segments and virtual items. A virtual item is a routeable item (street segment
or ferry line) with 0-length, that is created and connected to the border node
of routeable items in the maps. A border node is the node of a routeable item
that is on the border to a neighbouring mcm map. The property border node can
be given from the map supplier, denoting that a node is on the border between
to countries, and is one attribute in the Wayfinder midmif specification for
street segments and ferry items. If the property is not given from the map
supplier, and for borders between mcm maps within a country there is one step
of the map generation process that identifies border nodes based on the mcm map
geometry, see `GenerateMapServer --createBorderBoundrySegments` option calling
the `GMSUtility::createBorderBoundrySegments()` function.

The virtual item belongs to the border node. The virtual has a item gfxData
with 2 coordinates, both of them with the same coordinate values as the border
node coordinate, giving a 0-length item. The virtual item is connected to the
routeable item of the border node with normal connections. The turn description
for these connections is always set to ItemTypes::FOLLOWROAD.

All virtual item are added to the boundry segments vector
`m_segmentsOnTheBoundry` in `OldGenericMap`, see class `OldBoundrySegmentsVector`.
The vector contains boundry segments of class `OldBoundrySegment`, which
represents the virtual items and has attribute m_closeNode for storing which
node (node 0 or 1) of the virtual item that is the "far node", i.e. the one
that is closest to the boundry and the neighbouring map. The far node does not
have any connections leading TO it from nodes in the mcm map it is located in.
It will only have external connections from not-far-nodes of virtual items in
the neighbouring map. The boundry segment also has attribute vectors
`m_connectionsToNode0` and `m_connectionsToNode1` for storing the external
connections that leads TO the node 0 and node 1 of the boundry segment virtual
item.

So, an external connection is a connection that leads TO the far-node of a
virtual item in one mcm map FROM the not-far-node of a virtual item in the
neighbouring mcm map.

The external connections are created with the `GenerateMapServer
--extconnections` option calling the
`GenerateMapServer::updateExternalConnections()` method. It loops all mcm maps
(underviews, or first level overview, or second level overviews) in the map
generation directory, and collects the boundry segments virtual items of all
maps. For each boundry segment virtual item far node, it tries to find virtual
items far nodes in other maps that are within 5 meters distance and has the
same node level. If one found, the external connection is created from the
not-far-node of the boundry segment virtual in the other map and added to the
far node of the boundry segment virtual in this map.

Areas to improve concerning external connections:

* The `updExtConnsOfMap()` function (called from `updateExternalConnections()`) tries
  to set turn descriptions for the external connections. But this is one area
  that would need improvement.

*  There are no vehicle restrictions set for external connections. This needs
   to be implemented to be able to store restrictions provided from the map
   supplier for traversing between the two routeable items that are connected
   to the virtual items carrying the external connection.

Figure illustrating virtual items and external connections

![map_virtual_items_and_external_connections2](/images/map_virtual_items_and_external_connections2.png)

### Point of Interest Item (POI)

A point of interest (POI) is a point object of class `OldPointOfInterestItem`
(class `PointOfInterestItem` in m3 map) that represents e.g. a restaurant, train
station, airport, city centre or hospital. POIs are added to the mcm maps from
the WASP POI database. Addition directly from midmif could be developed, but to
have the full range of POI info (address, phone numbers etc) available in the
server m3 map, the POIs must currently be added from the WASP database. A POI
is connected to a street segment in the map, to which users should be directed
when navigating to the POI.

The POI has attributes for POI type (see `ItemTypes::pointOfInterest_t`, which
are also defined in the WASP `POITypeTypes` table), `waspID` (ID in the WASP
POI database), id of the street segment on which the POI is located and offset
on the street segment (where along the segment counted from node 0 it is
located). It is the entry point in the WASP `POIEntryPoints` table that defines
this street segment and offset for the POI. The POI is not necessarily located
ON the street segment, its symbol coordinate (`POIMain.lat` and `POIMain.lon`) can
be quite far away from the street segment, but the entry point of the POI is
located on the street segment, so it is where the POI is connected to the
street network.

The POI also has zero, one or several POI search categories. These are not
stored in the `OldPointOfInterestItem`, but in a table in the `OldGenericMap`, the
`m_itemCategories` which for each POI has a set of category IDs.

The POI categories are hierarchical, e.g. a super category _Dining and drinking_
contains categories _Restaurants_ and _Pubs_. The _Restaurants_ category then
contains misc food type sub categories such as _chinese_, _pizza_, _cajun_ etc. A POI
gets the most detailed category from the POI category hierarchy, e.g if it is a
pizzeria, it does not store also restaurant since that is a given top category
of pizzeria. But when searching for restaurant also the pizzeria is found,
since hierarchy is expanded in searching.  The POI categories are defined in
the WASP POI database and the category hierarchy is defined in the
`poi_category_tree.xml` file.

The POI type controls which icon is used for the POIs in the map image. The POI
categories are used when searching for a POI by category.

Not all POIs have a `§gfxData` in the mcm maps, after added from the WASP
database. This is to save disc space.  Only those POIs for which the distance
between the POI symbol coordinate (`POIMain.lat` and `POIMain.lon`) and the entry
point coordinate is more than 5 meters are assigned a gfxData. For POIs with
less than 5 meters between the symbol coordinate and the entry point
coordinate, no gfxData is created.  For drawing the POI icon for the POIs
without gfxData, a symbol coordinate will be calculated from the street segment
id and offset on the street segment.

Note that a POI can in reality have more than one entry point, which could also
be delivered from the supplier. All entry points can be stored in the WASP
database, but the model in the mcm maps only supports one entry point, as it
has only one street segment + offset attribute. If there are several entry
points in the database, simply the "first" will be chosen for identifying the
street segment.

#### The poi category tree

The POI category tree `poi_category_tree.xml` `<category_data>` has two parts.
First is the `<category_tree>` with the tree hierarchy of the categories
listing which category ID are top categories and which are sub categories. One
category can be part of more than one top category, e.g. category id 100 Bars
is part of the Food and Drink top category and the Nightlife top category.  
Second is the `<categories>` that lists all categories that are defined in WASP
`POICategoryTypes` table with the translations on all languages for the
categories, and the category search synonyms for the categories that have such
defined in the POICategorySearchSynonyms table. The stringKey found in the WASP
`POICategoryTypes` table refers to the stringKey that was used in the Dodona
translation database. Some stringKeys can be found in the mc2 `StringTableUTF8`,
but unfortunately far from all.

The POI search synonyms are special strings that can be used for searching for
a certain category via string search instead of choosing the category in the
category list.

It is possible to manage some parts of the `poi_category_tree.xml` with the
`maps_xmlCategories.pl` script.

* With the `-xml2file` option you can create a file tree representing the
  category hierarchy. In the file tree it is easy to move one category into
  another top category, or duplicate it to more than one top category.
* With the option `-file2xml -d` the file tree is converted back to the xml
  file. The `<category_tree>` part is completely created. In the `<categories>`
  part the translations are not included, but the category search synonyms are.

## Search index

There are a few parallel indexes that are used for searching. In this section
examples are given for a user searching for an address or a POI in a city. Here
are some terms to consider

* **Search area**  
  The search area is what you can find an address or a POI in or
  close to. Eg you can search for a hotel in city, or search for an address
  in a zip code.

* **Search§ location**  
  The location is where the search hit is located, as is presented in the
  search hit list. The location can  for an address typically be city part,
  city, municipal. For a POI the location is typically street name, city part,
  city, municipal. If the name of the municipal and the city is the same, they
  are not repeated. E.g. the search hit for a hotel in Paris can be Best
  Western Hotel De Weha, 205 Avenue De Choisy, Paris. Here the location
  contains street name with house number and city. The name of the municipal
  and the city for this search hit is the same Paris, which is not repeated.

Item types that can be part of the search index and used as search areas and/or
locations are: city part items, built-up area items, municipal items, zip code
items and city centre POIs.

The search index is stored in the maps modeled as region groups. I.e. the
London built-up area is the region group of the city parts in London and of all
street segments that are located in London. The `OldItem` class has a `m_groups`
arrray with itemIDs of the items that are region group of the item. Get region
groups for one item with the `OldGenericMap` functions `getRegionID()`,
`getRegion()`, etc. Add and remove region groups to an item with the
`OldGenericMap` function `addRegionToItem()`, `removeRegionFromItem()` etc. When
adding a group to an item, it is possible to decide if the group should be used
only as group and search area for the item, but not for location. It is defined
with the `noLocationRegion` param of the `addRegionToItem()` function.

The region groups can be provided from the map supplier and is supported in the
Wayfinder midmif specification for some item types with the settlementID
attribute. If not provided from the supplier and/or not supported from
Wayfinder midmif, call the `GMSMap::setAllItemLocation()` function which calculates
location based on geometry. A street segment located inside the gfxData of
built-up area will have the built-up area as one of it's region groups.

Point of interest items added to the mcm map from WASP do not have any groups.
The POI is tied to a street segment via its entry point, and placed in the
search index from the groups of the street segment.

The groups of one item can easily be browsed in the MapEditor tool. Open a mcm
map in the MapEditor and select the item you want to examine groups for. The
item info widget shows the groups of the item. Any group of the item can be
selected to further examine the groups of that.

###  Search index with municipal, built-up area and city part

The number one search index is the index with municipal item + built-up area
item + city part item. Here the municipals and built-up areas are the search
areas, while all three items can be used for giving the location of the search
hit. If the built-up area is completely within the municipal geometrically, the
municipal is the region group of the built-up area. If it is a big built-up
area such as Paris and the built-up area covers several municipals
geometrically, then built-up area is the region group for the municipals. The
city part always has the built-up area as region group.

###  Search index with city centres

City centres are used as search areas when they don't have a group (via it's
street segment) with the same name. For instance the "Lund" city center is
located in the "Lund" built-up area. Therefore it is not used as a search area.
The reason for not having the city centers as search areas when they are
located in an area with the same name is that if they had been used, two hits
for Lund would have been returned from the server. However, the city part city
centre Linero in Lund, is used as a city centre search area. Such a city centre
search is done by using a radius from the city centre coordinate. The radius is
defined in `TemporarySearchMapItem m_radiusMeters`, in `TemporarySearchMapItem
::TemporarySearchMapItem()` currently set to 10 km. Additionally, the mc2 server
expands to search in the search area where the city centre is located. For
Linero city centre in Lund, the search also includes the Lund municipal of the
Lund built-up area.

The location for search hits found in city centre searches, is taken from the
groups of the search hit item, i.e. the normal city part, built-up area and
municipal, or in case of using index areas the index areas of the search hit
item. The city centre itself is never part of the location.

###  Search index with zip codes

The zip code items are also part of the search index. They are used as search
areas, but never as location. If created from the zip code attribute of street
segments, the zip code will be the group of all street segments that had the
zip code attribute. If created from zip POI file, the street segments closest
to the zip point will have the zip code as group. A street segment can in this
zip POI file case have max `maxNbrZipGroups` (currently hardcoded to 10) zip
codes as groups, i.e. the 10 closest zip points. See `GMSMap::setZipForSSIs` for
details about setting zip groups for street segments from zip POI files. Please
note that the zip codes are not related to the municipal, built-up area, city
part in any way, it is a separate search index.

###  Search index with index areas

If the search index with municipal, built-up area and city part combined with
city centres is not good enough, it is possible to replace it with index areas.
The model for index areas offers a more flexible search index, where you can
have any number of levels in the index. All levels of the index areas are used
as search areas and location for search hits. It is a bit more complex than the
municipal-built-up area model, but good to use if e.g. the administrative areas
that are used for municipal items is something that the users are not familiar
with or has no meaning in addressing. The UK is one such case, where e.g. the
order8 from Tele Atlas, which are excellent to use as municipals in the mcm
maps, are not known to UK users and is not part of the UK postal addressing
index.

Index areas are in the mcm maps represented as built-up areas, specially marked
as index areas not to be displayed as cities in any map image. If index areas
are used, the normal built-up areas are used only for map display and are
referred to as display buas in the code and documentation, while the index
areas are referred to as index area buas. The normal built-up areas and
municipal items are lifted out of the search index by
`OldGenericMap::setItemNotInSearchIndex()`, only keeping the index area in the
search index. The zip codes, however are still kept in the search index,
parallel to the index areas. And city centre search is also still appropriate
should there be a city centre located in index areas that does not have the
same name as the city centre.

One example of the levels, called index area orders is to have index area order
7, 8, 9 and 10. Index area order 7 represents some kind of large area, e.g.
county or so. Order 8 represents large city, order 9 represents city part, and
order 10 represents sub city part. In this example typically order 7 is
covering all parts of the country (for all items to be located in something).
To make addresses unique within one order 7, it is sub divided into index areas
of order 8. Not all parts of the order 7 must be covered by order 8s. If needed
to make addresses unique within the order 8, it is sub divided into index areas
of order 9. Not all parts of the order 8 must be covered by order 9s. And so
on. As for region groups: the order 7 is the group of its order 8s, the order 8
is the group of its order 9s, and the order 9 is the group of its order 10s.
The items in the mcm map, e.g. street segments, parks etc, have all index areas
(order7+order8+order9+order10) as region group.

The location of a search hit in index areas will be "order10, order9, order8,
order7". In case some of the index areas have the same name, the most detailed
of the same-name index areas is skipped in the location, e.g. resulting in
"order10, order8, order7" if the order 8 and order 9 has the same name.

The index area buas are created from a Wayfinder midmif built-up area file,
with the attribute index area order attribute set. The
NationalProperties::useIndexAreas must be set to define if to use index areas
for a certain combination of country and map supplier, else the index area
order attribute in the Wayfinder midmif bua file has no effect. The
`NationalProperties::indexAreaToUse?  must also be set, it defines which orders of
index areas to use for the country and map release combination.

Note. Index areas are stored as built-up area items in the mcm maps, specially
marked as part of search index. In the m3 maps, index area buas are converted
to be municipal items.

## GfxData geometry and the MC2 coordinate system

The gfxData is of class `GfxData` or any of its sub/super classes and holds
geometry for map items or the map itself. One gfxData has one or more polygons
and each polygon has one or more coordinates. The gfxData attribute
`m_closedFigure` is closed=yes for areas, and closed=no for open polygons, such
as lines and points.

The coordinate system used in the MC2 server and the internal map formats is
called mc2 coordinates. It is constructed to minimize data storage. It has a
strict relation to the WGS84 coordinate system, a full lap around the earth is
`MAX_UINT32 = 2^32 = 4294967296?` units so we can store integers instead of
floats.
At the equator one meter is 107 mc2 units (Circumference of the globe / 2^32),
so we can say that 1 mc2-unit is approximately 1 centimeter at the equator. For
latitude value this is true all over the earth. But the longitude value
changes, the further you are from the equator the shorter 1 mc2-unit becomes.
For example, in Malmö in south of Sweden 1 meter equals to approx 194
mc2-units.

Constants for converting between distances in mc2 units and meters can be found
in the class `GfxConstants`. The `GfxConstants` class also holds defines for
converting between WGS84 degrees, radians and mc2 coordinates. Functions for
transforming between other different coordinate systems can be found in the
class `CoordinateTransformer`. It handles mc2 coordinates, WGS84, Swedish RT90,
and some others.

## Landmarks support

The `OldGenericMap` has a table `m_landmarkTable` for storing landmark. The
landmarks in the table are tied to a connection (fromNode, toNode) and stores
info about a landmark that is to be used when traversing the connection on a
route. The information is stored using the the `ItemTypes::lmdescription_t`
struct.

Landmarks are miscellaneous items along a route that are used to clarify the
route description. The landmarks are a complement to the distance- and
turndirections in the route description and they provide information that can
be related to the surroundings. They state e.g. which built-up areas, streets
and other significant objects to pass along the route.

Some landmarks are "static", does not change depending on from where/to where
you are routing.  One example of this is a church, which obviously always is
located on the same side of a crossing.  These static landmarks can be saved in
the landmark table of the mcm map. The typical route description for a landmark
in the table can be

* _Turn right after the church on your left side_
* _at the school on your right side_
* _pass through the university area_

The landmark table can currently be filled with landmarks from extradata (map
correction) records containing landmark candidates during map generation. The
extradata records types for landmarks are `addLandmark` and
`addSingleLandmark`. See class `ExtraDataReader` and `OldExtraDataUtility` for
how the landmarks are extracted and calculated for the landmarks table.

Other landmarks, like pass a built-up area, or a certain street on the right
side depend on from where/to where you are routing and cannot be pre-calculated
for certain connections. They can instead be added to the route description in
the `ExpandRouteProcessor` with the `addLandmarks()` method, if requested to the
route expander. Typical route descriptions for these landmarks are

* _After 3.2 km go into Lund_
*  _Pass Main Street on your left side_

Other types of landmarks are traffic related, like disturbances, and are also
added to the route in the `ExpandRouteProcessor`.

## Map data requirements

The map data need to meet some requirements for location based services in
order to be really good as material for the MC2 back-end server. There are
different requirements for searching, routing, navigation and map display. 

The more requirements that are met, the better service.

### Routing services

Routing services are the services used when routing and navigating.

#### Route calculation

* Road network connectivity.
* Functional road class used for deterring the importance of a road.
* Speed road attribute. A speed value attribute or derived from for instance road class.
* One way street road attribute.
* Blocked passage road attribute.
* Forbidden turns road relationships.
* No thoroughfare / residential vehicle road attribute.

#### Driving instructions

* Street names.
* Roundabout road attribute.
* Ramp attribute on freeway enter and exit ramps.
* Multi-digitalized road attribute on streets with dual carriageways.
* Bifurcation attribute used or differentiating between keep left and turn left instructions.
* Good road coverage, since we want to tell the driver how many roads to pass before making a turn.

#### Route following, navigation

   * Road network geometric accuracy of at least about 25 meters.

### Destination search service

For finding address, POIs or other things.

* Street names.
* House numbers.
* Administrative areas with names, such as municipal.
* Built up areas with names. Used for identifying what is located in a city or village.
* City centers. Used for identifying what is located in a city or village, and also to locate city parts.
* Zip codes, preferably as road attribute but polygon areas are also possible.
* Points of interest including petrol stations, restaurants, train stations, shops etc.

### Map display

For having a nice looking map display, we need different kind of items.

* Roads with display class in order to show it on the appropriate zoom level. Functional class can also be used for this.
* City centers with display class and names
* Built up areas with names
* Waters; lakes and rivers
* Forest
* City parks
* Industrial areas
* Airport areas
* Railroad tracks
* Buildings
* etc

