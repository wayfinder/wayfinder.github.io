---
title: Wayfinder midmif specification
layout: single
---

## Requirements for Wayfinder maps

There are some basic requirements that must be met with for maps in the
Wayfinder Server.

The maps must meet with requirements for at least two functions; searching and
routing, but preferably also with the third function - map display. Additional
requirement for navigation is that the street network positional accuracy must
be high enough to match GPS positions.

* **Administrative areas**  
  The maps must have a hierarchy of named administrative areas, which are used
  for searching. At least municipal, in combination with built-up areas or
  other features for the search index are required. The municipals must cover
  the complete mapped area (= part of or an entire country) seamlessly, i.e. no
  area must lack a municipality. The municipals are used for drawing land in
  the map. Built-up areas corresponds to cities and are used for drawing cities
  in the map.
* **The street network**  
  The streets must have names in order to find them when searching. There must
  preferably be a road classification in order to distinguish difference
  importance when drawing the map. If navigation is needed the street network
  must also have connectivity, which means that connecting street segments must
  share a node. Moreover attributes for different kinds of traffic rules must
  be included.
* **Land coverage**  
  Land coverage types are needed to make the map look nice, eg waters, parks,
  railways, industrial areas and public buildings.
* **Points of interests**  
  POIs add value to the map. The POIs are possible to find when searching and
  some POI types are shown as symbols in the map, eg restaurants, hotels,
  train stations, hospitals, petrol stations and airports. POIs of type city
  centres are needed in order to display city names in the map.

## Wayfinder maps from midmif

Maps for the Wayfinder server can be created from midmif files, which is the
MapInfo Interchange Format (MIF). It consists of one file with coordinate
information (file suffix .mif) and one with attribute information (file suffix
.mid). This is not to be confused with the MapInfo Format consisting of files
with suffix .map .dat etc. Regarding the ESRI shape file format, it is easy to
convert from shape to midmif.

Several types of items can be created from the midmif format. Here is a list
with the available types (alphabetical order) and a short description. For a
more detailed description of the types, we refer to eg Tele Atlas MultiNet
documentation.

* **aircraft roads**  
  Airport runways
* **airports**  
  Areas containing the airport ground including terminal buildings and runways.
* **buildings**  
  Areas with special use. Use only if the cartographic item does not fit, eg
  for industrial areas.
* **built-up areas**  
  Populated areas, cities.
* **cartographics**  
  Areas with special use, only used for display in the map. Examples are
  hospital ground, cemetery ground.
* **city parts**  
  Sub areas that divides built-up areas into smaller units well-known to
  residentials, districs. It is an administrative area level beneath municipal
  and built-up area. It is used for further specifying the location of streets
  and POIs when returning search results.
* **ferries**  
  Transportation via ferry lines, for cars or for passengers only.
* **forests**  
  Areas covered with forest.
* **individual buildings**   
  It represents the actual outline of individual houses, not only the ground on
  which the houses are built. An individual building can be any house, eg
  parking garages, airport terminal buldings, sport arenas, churches and other
  buildings that add value to the map.
* **islands**  
  Areas with land in lakes, rivers and archipelagoes.
* **municipals**  
  Administrative areas, equal to Tele Atlas MultiNet order8 areas.
* **parks**  
  Areas covered with parks, either city parks or national parks.
* **railways**  
  The railways used for transportation.
* **street segments**  
  The street network.
* **waters**  
  Areas covered with water.
* **zip code areas**  
  Postal code areas. They do not have a true geographical representation in the
  Wayfinder maps, but are created from attributes of street segments.
    
Points of interest (POIs) are not added to the mcm maps directly from midmif,
but via the internal SQL database called WASP (Wayfinder Administration System
for Points-of-interest). In the database different kinds of information can be
stored about each POI, including name, address, coordinate and business
category. POIs must therefor be imported into the database before added to the
mcm maps. An import format directly from midmif to WASP database has not yet
been developed. Other import formats are used, eg Wayfinder Simple POI Import
Format (SPIF) or Wayfinder Complex POI Import Format (CPIF).

Mandatory attributes for POIs are: `name* *poi type* and coordinate. Additional
attributes are: address, zip code, city, phone number, web site, opening hours
and more..

### Internal creation introduction

The mcm maps are initialized by adding the municipals, of which each map
constitutes. Here also the mcm map gfxData is created from mif files holding
the outline coordinates of each map. The gfxData forms the geographical extent
of each map, which is needed to decide in which map the rest of the items
should be added (if the check must be performed based on geographical
location). The mcm maps are then successively created by adding the remaining
item types from midmif files, one type at a time in any order. All items of one
type can for this step preferably be included in the same file. When adding
street segments some restrictions and connection attributes can be set using
[turn tables](#turn-tables).

All steps in this creation process are performed with `GenerateMapServer`. To
init each mcm map the `--initMapFromMidMid` option is called with the
municipal file as parameter. To add the remaining items to all of the maps the
`--createItemsFromMidMif` option is called with the item midmif file as
parameter.

The midmif items from files in the call to `createItemsFromMidMif()` will be added to
the mcm map which they fit in. There are some different option for how to
decide if one item fits a mcm map:

* **based on geometry**  
  If the first coordinate of the item is inside the mcm map gfxData it fits the
  map.
* **settlement id**  
  If the item has the settlementID attribute set it is added to the mcm map
  which had the admin area with midID `settlementID` and are of `settlementOrder`
  added to it. The `settlementID` info may be provided from the map supplier, or
  be calculated in the process of creating Wayfinder midmif format.
* **option addAllMidMifItems**  
  An option that makes all items from the midmif file to be added to the mcm
  map. Good to use if you have only one map inte the country and addition on
  geometry will not work, eg street segments outside land (eg bridges) or
  you do not have settlement location available.
* **map ssi coordinate**  
  The item will be added to the mcm map which contains the map ssi coordinate,
  a coordinate which it specified at the end of the mid row for the item. This
  option can not be used in creation of maps from scratch from midmif, but only
  for adding extra itesm to the mcm maps later in the process since it requires
  street segments to be present in the mcm map.

The result of this creation process are raw mcm maps with all type of items
included as well as some restrictions etc. The process must then be followed by
misc post processing to complete locations, turn descriptions, connections
between maps, and overview maps creation.

## The midmif files

For the midmif files to be used when creating Wayfinder maps, there are some requirements.

### File name

For convenience there should be one midmif file per item type. The file name
includes what type of items it contains. If eg one midmif file holds built-up
areas the file name could include the word `builtupareaitem`", if it holds parks
the name could include `parkitem`" etc (case ignored).
The MC2 system really **requires** that the item type is included in the file
name in order to know what type of items to create. See
`ItemTypes::getItemTypeFromString` for the item type strings. Case is ignored.
Likewise, the mif file with the outline coordinates (`gfxData`) of each mcm map
must have the same name as the midmif file with the municipals for that map,
followed by the word `map`. If eg the municipal file is called
`dk_municipalItems.mid` and `dk_municipalItems.mif`, the `gfxData` file must be
called `dk_municipalItemsmap.mif`


### Valid mif features

The mif file must only contain the geographical MIF features `Point`, `Line´, `Pline` or `Region`.

### The mif header

The mif header must state which coordinate system is used for expressing the
coordinates in the mif file. This is done by the word `Coordsys`" followed by a
code for the coordinate system according to:

* `mc2`  
  The internal coordinate system in the Wayfinder mcm maps, with coordinate
  order (latitude, longitude).
* `gs84_latlon_deg`  
  WGS84 coordinates expressed in degrees, with coordinate order (latitude,
  longitude).
* `wgs84_lonlat_deg`  
  WGS84 coordinates expressed in degrees, with coordinate order (longitude,
  latitude).
* `rt90`  
  The Swedish national reference system, with coordinate order (latitude,
  longitude).
* `rt90_lonlat`  
  The Swedish national reference system, with coordinate order (longitude,
  latitude).
* `utm x`  
  UTM coordinate system zone x, with coordinate order (latitude, longitude).
* `utm_lonlat x`  
  UTM coordinate system zone x, with coordinate order (longitude, latitude).
    
If any false easting or false northing is present on the coordinates this also
must be stated in the mif header. The keywords are `falseEasting` and
`falseNorthing`. Example of mif header for coordinates expressed in the UTM
system:

   Version 300
   Charset "WindowsLatin1"
   Delimiter ","
   Coordsys utm_lonlat 39
   falseNorthing -2000000
   falseEasting 500000
   Columns 25
      id decimal (11,0)
      name char (25)
      ...
   Data
    
The only exception from giving the `Coordsys` tag is when the coordinates are
expressed in the mc2 coordinate system with normal coordinate order (latitude,
longitude). This is considered to be the default system. However, it is
recommended to give the tag anyway in order to always know what coordinate
system is used in the mif file.

### The mid file

The mid file is comma separated and must hold certain attributes for each type
of item. Common attributes for all items are midID and name. Many items do not
have any other attributes than the common ones, eg built-up areas, railways
and individual buildings. Some items have one additional item specific
attribute giving a type for the item, eg waters, parks and buildings. For
street segments several item specific attributes are required giving the speed
limit, house numbering, road class and many more. Below follows a specification
of which attributes are required for each item type. Attributes in italic are
*extra attributes* that can be used if available, but are not mandatory.
    
If adding extra midmif items to maps (not in the creation from midmif), it is
for all item types possible to specify a **map ssi coordinate** at the very end
of the mid row. This coordinate defines in which mcm map the midmif item should
be added. The coordinate is expressed in mc2 units and must be within 2 meters
from any street segment item in the correct mcm map. If using such a map ssi
coordinate the GenerateMapServer `--createItemsFromMidMif` option must be
combined with the `--useCoordToFindCorrectMap` option. The mid row for forest
item will be `midID`, `name`, `allNames`, `mapssilat`, `mapssilon`.

* **AircraftRoadItem**  
  `midID`, `name`, `allNames`
* **AirportItem**  
  `midID`, `name`, `allNames`, *settlementId*, *settlementOrder*
* **BuildingItem**  
  `midID`, `name`, `allNames`, `buildingType`
* **BuiltUpAreaItem**  
  `midID`, `name`, `allNames`, *settlementId*, *settlementOrder*,
  *indexAreaOrder*
* **CartographicItem**  
  `midID`, `name`, `allNames`, `cartographicType`
* **CityPartItem**  
  `midID`, `name`, `allNames`, *settlementId*, *settlementOrder*
* **FerryItem**  
  `midID`, `name`, `allNames`, `roadClass`, `posSpeed`, `negSpeed`,
  `posEntryRstr`, `negEntryRestr`, `levelNode0`, `levelNode1`, `roadToll`,
  *ferryType*, *node0borderNode*, *node1borderNode*
* **ForestItem**  
  `midID`, `name`, `allNames`
* **IndividualBuildingItem**  
  `midID`, `name`, `allNames`, *individualBuildingType*
* **IslandItem**  
  `midID`, `name`, `allNames`
* **MunicipalItem**  
  `midID`, `name`, `allNames`
* **ParkItem**  
  `midID`, `name`, `allNames`, `parkType`
* **RailwayItem**  
  `midID`, `name`, `allNames`, *settlementId*, *settlementOrder*
* **StreetSegmentItem**  
  `midID`, `name`, `allNames`, `roadClass`, `posSpeed`, `negSpeed`,
  `posEntryRestr`, `negEntryRestr`, `nbrLanes`, `width`, `maxHeight`,
  `maxWeight`, `leftStart`, `leftEnd`, `rightStart`, `rightEnd`, `paved`,
  `levelNode0`, `levelNode1`, `roundabout`, `ramp`, `divided`, `multidig`,
  `roadToll`, `controlledAccess`, *roundaboutish*, *leftZipCode*,
  *rightZipCode*, *leftSettlementId*, *rightSettlementId*, *settlementOrder*,
  *node0borderNode*, *node1borderNode*, *roadDisplayClass*
* **WaterItem**  
  `midID`, `name`, `allNames`, `waterType`,  *settlementId*, *settlementOrder*
    

Here follows a definition of all attributes. Common attributes:

* **midID**  
  Mid id number of the item to create, positive integer value.
* **name**  
  A string with the official name or any other name of the item, surrounded by
  `"` characters, eg `"Lund"`. If the item has no name give an empty string,
  `""`.
* **allNames**  
  A string holding all names of the item, eg officialName, alternativeName and
  roadNumber in any language (short version), eg swe, eng and ger. See section
  [Name types](#name-types) and section [Name languages](#name-languages) for
  valid name types and languages. For `roadNumber` give `invalidLanguage` as
  language since a number has no language. Every name is constructed according
  to `name:nameType:nameLanguage`. The separator is the `:` character (colon).
  This implies that Wayfinder currently can not handle names that includes the
  `:` character. If the item has more than one name the names are separated
  with a space character. The whole string is surrounded by `"` characters, eg
  `"North Avenue:officialName:eng A10:roadNumber:invalidLanguage"`. If the item
  has no name give an empty string, `""`.
* **settlementId**  
  The mid settlement id for the item, integer value.
* **settlementOrder**  
  The order of the settlementId, integer value. 1 meaning order1, 8 meaning
  order8, etc. The orders are tied to Wayfinder item types in this way:
    * **8** Municipal item.  **9** City part item.  **99** Built-up area item.
    * **mapssilat**  
  The map ssi coordinate, decides correct map for an item if there is a street
  segment within 2 meter from this coord. Signed integer value, expressed in
  mc2 units.
* **mapssilon**  
  The map ssi coordinate, decides correct map for an item if
  there is a street segment within 2 meter from this coord. Signed integer
  value, expressed in mc2 units.
 
Item sub type attributes. For more detailed description of the attributes, we refer to Tele Atlas MultiNet documentation:

* **waterType** 
  A string with the water type for the waterItem (ocean, lake, river, canal,
  harbour). The string is surrounded by "-characters, eg "lake", "ocean".
* **parkType**  
  A string with the park type for parkItems (cityPark, regionOrNationalPark).
  The string is surrounded by `"` characters, i.e. "cityPark" or
  "regionOrNationalPark".
* **buildingType**  
  A string with the building type for BuildingItems. Not used anymore, no
  effect regardless of what is given in this string, simply use unknownType.
  The string is surrounded by "-characters, "unknownType".
* **ferryType**  
  The type of ferry where 0=operatedByShip (default) and 1=operatedByTrain.
* **cartographicType**  
  A string with the cartographic type for CartographicItems. The string is
  surrounded by "-characters, eg "golfGround", "hospitalGround". For a list of
  possible string values see section [Cartographic
  types](wayfinder_midmif_specification#cartographic_types) or the mc2
  OldCartographicItem::stringToCartographicType() method.
* **individualBuildingType**  
  A string with the type for individualBuildingItems (publicIndividualBuilding,
  otherIndividualBuilding, airportTerminal, parkingGarage). The string is
  surrounded by "-characters, i.e. "publicIndividualBuilding" or
  "parkingGarage". Please, see the mc2 enum
  `OldIndividualBuildingItem::stringToBuildingType` for all possible types.
    
Built-up areas if they are index areas. For more detailed description of the
attributes, we refer to eg Tele Atlas MultiNet documentation.

* **indexAreaOrder**  
  The order of index area if the built-up area item should be used as an index
  area. Integer value. Index area order 7 represents some kind of large area,
  eg county or so. Order 8 represents large city, order 9 represents city part,
  and order 10 represents sub city part.
    
Street segment (and ferry) attributes. For more detailed description of the
attributes, we refer to eg Tele Atlas MultiNet documentation:

* **roadClass** 
  The road class of the street segment, enum integer 0-4 (see section [Road
  class](#road-class) or mc2 enum `ItemTypes::roadClass`).
* **posSpeed**  
  The speed limit (km/h) for travelling the street segment in positive
  direction (from node0 to node1), any integer value.
* **negSpeed**  
  The speed limit (km/h) for travelling the street segment in negative
  direction (from node1 to node0), any integer value.
* **posEntryRestr**  
  The entry restriction for node0, enum integer 0-3 (see section [Entry
  restrictions](#entry-restrictions) or mc2 enum `ItemTypes::entryrestriction_t`).
* **negEntryRestr**  
  The entry restriction for node1, enum integer 0-3 (see section [Entry
  restrictions](#entry-restrictions) or mc2 enum `ItemTypes::entryrestriction_t`).
* **nbrLanes**  
  Number of lanes in each direction of the segment, integer value. If this
  information is not present, leave the field empty.
* **width**  
  The width (meters) of the segment, integer value. If this information is not
  present, leave the field empty.
* **maxHeight**  
  The maximum free height (meters) when traveling the segment, integer value.
  If this information is not present, leave the field empty.
* **maxWeight**  
   The maximum allowed weight on the segment, integer value. If this
   information is not present, leave the field empty.
* **leftStart**  
  Left side starting house number, integer value. If no house numbers, use 0.
* **leftEnd**  
  Left side end house number, integer value. If no house numbers, use 0.
* **rightStart**  
  Right side starting house number, integer value. If no house numbers, use 0.
* **rightEnd**  
  Right side end house number, integer value. If no house numbers, use 0.
* **paved** 
  If the segment is paved or unpaved, boolean string "Y" or "N". "True" and
  "False" also accepted.
* **levelNode0**  
  Level in node 0, integer value [ -1, 0, 1 ]. If this information is not
  present, leave the field empty.
* **levelNode1**  
  Level in node 1, integer value [ -1, 0. 1 ]. If this information is not
  present, leave the field empty.
* **roundabout**  
  If the segment is part of a roundabout, boolean string "Y" or "N". "True" and
  "False" also accepted.
* **ramp**  
  If the segment is a ramp, boolean string "Y" or "N". "True" and "False" also
  accepted.
* **divided**  
  If the segment is divided, boolean string "Y" or "N". "True" and "False" also
  accepted.
* **multidig**  
  If the segment is multiply digitalized, boolean string "Y" or "N". "True" and
  "False" also accepted.
* **roadToll**  
  If the segment has a road toll, boolean string "Y" or "N". "True" and "False"
  also accepted.
* **controlledAccess**  
  If the segment is a controlled access road, ie the only way to access the
  road is via a ramp, boolean string "Y" or "N". "True" and "False" also
  accepted.
* **roundaboutish**  
  If the segment is part of a roundabout-like traffic figure, boolean string
  "Y" or "N". "True" and "False" also accepted.
* **leftZipCode**  
  The zip code for the left side of the street segment (counted from node0),
  string value eg "22280". Use the empty string "" if the street segment has no
  left zipCode.
* **rightZipCode**   
  The zip code for the right side of the street segment (counted from node0),
  string value eg "22280". Use the empty string "" if the street segment has no
  right zipCode.
* **leftSettlementId**  
  The mid settlement id for the left side of the street segment (counted from
  node0), integer value.
* **rightSettlementId**  
  The mid settlement id for the right side of the street segment (counted from
  node0), integer value.
* **settlementOrder**  
  The order of the settlementIds, integer value. 1 meaning order1, 8 meaning order8, etc. The orders are tied to Wayfinder item types in this way:
      * **8** Municipal item.
      * **9** City part item.
      * **99** Built-up area item.
* **node0borderNode**   
  If node 0 is on the border to another country or dataset, boolean string "Y"
  for true and "N" or empty string "" for false. "True" and "False" also
  accepted.
* **node1borderNode**  
  If node 1 is on the border to another country or dataset, boolean string "Y"
  for true and "N" or empty string "" for false. "True" and "False" also
  accepted.
* **roadDisplayClass**  
  The road display class of the street segment. For distinguishing different
  types of streets when drawing map images. Defined values are (see also
  `ItemTypes::roadDisplayClass_t`):
      * `-1` invalid road display class
      * `0` partOfWalkway
      * `1` partOfPedestrianZone
      * `2` roadForAuthorities
      * `3` entranceExitCarPark
      * `4` etaParkingGarage
      * `5` etaParkingPlace
      * `6` etaUnstructTrafficSquare
      * `7` partOfServiceRoad
      * `8` roadUnderContruction
    
Any left and/or right zipCode provided for a street segment will be used for
creating zip code items in the Wayfinder map. The left and/or right
settlementId is used for deciding in which map a street segment fits. The
settlementOrder gives the order of the left and right settlementId, order
meaning order0, order1, order2, etc, where order8 is Wayfinder municipal item
and order9 is Wayfinder city part item. The special order99 is Wayfinder
built-up area item. Example: A street segment has left settlementId 67 and
order 9. The segment should be added to the map that has a city part item
created from midmif which has midId 67. If no city part items exist, try
finding a order8 with midId 67 instead.

    
Example of mid file for street segments

    1,"A10","Pampas Highway}officialName}eng A10}roadNumber}invalidLanguage" \
       ,0,110,-1,0,3,,,,,0,0,0,0,"Y",,,"N","N","N","N","N","Y"
    2,"A10","Pampas Highway}officialName}eng A10}roadNumber}invalidLanguage" \
       ,0,110,-1,0,3,,,,,0,0,0,0,"Y",,,"N","N","N","N","N","Y"
    4,"North Ramp","North Ramp}officialName}eng" \
       ,1,90,-1,0,3,,,,,0,0,0,0,"Y",,,"N","Y","N","N","N","N"
    20,"Pine Street","Pine Street}officialName}eng" \
       ,3,50,50,0,0,,,,,5,13,6,18,"Y",,,"N","N","N","N","N","N"
    606969,"Árok utca","Árok utca}officialName}hun",3,50,50,0,0,,,,,0,0,0,0 \
       ,"Y",0,0,"N","N","N","N","N","N","N","2500","2500",25131,25131,9
    
Example of mid file for built-up areas

    25,"Copenhagen","Copenhagen:officialName:eng Köpenhamn:officialName:swe \
       Købenavn:officialName:den"
    
Example of mid file for building items

    12,"","","unknownType"
    
Example of mid file for cartographic items

    13,"University of London","Univeristy of London:officialName:eng" \
       ,"universityOrCollegeGround"
    14,"Old Sanctuary","Old Sanctuary:officialName:eng","cemetaryGround"
    15,"King Georges Hospital","King Georges Hospital:officialName:eng" \
      ,"hospitalGround"
    16,"Abbey Field","Abbey Field:officialName:eng","golfGround"


## Turn tables

The midmif files only holds attributes for each item individually. Any
restrictions between street segments, such as vehicle restrictions, cannot be
stored in midmif files. However, by using turn tables restrictions and other
connection attributes can be expressed.  
The turn table is a tab separated text file. It holds columns with information
about relations between the segments in the street segments file. The choice of
keys for the columns in the turn table is inspired by ESRI turn tables. The
segments are identified by the midIDs given in the street segment mid-file, the
from-segment as ARC1_ and the to-segment as ARC2_. The restriction or other
connection attribute is is given by the IMPEDANCE. Currently the following
values are valid:

* **0**  
  No impedance, means that there are no restriction or other connection
  attribute.
* **-1** 
  The turn from from-id to to-id is restricted. The vehicle restrictions of the
  connection from from-id to to-id is set to 0x80, meaning that all vehicles,
  pedestrians excepted, are prohibited to make the turn. If the from-id is -1
  all connections to the to-id are given vehicle restriction 0x80.
* **-2** 
  The turn from from-id to to-id is a bifurcation. The toNode of the connection
  is marked with junction type bifurcation.
    
One example of a turn table:

    KEY     NODE_   ARC1_   ARC2_    IMPEDANCE
    1       1       2       2        -1
    2       2       1       1        -1
    3       3       20      20       -1
    4       3       20      4         0
    4348    16      1080    1070     -2
    1386511 -1      -1      602947   -1
    
To enable handling in the MC2-system when adding midmif items to mcm maps, it must have the same name as the street segment file followed by the word "turntable". If eg the street segment file is named streetSegmentItems.mid/mif the turn table must be called streetSegmentItemsturntable.txt.
    
Please see `GMSMidMifHandler::extractRestrictionsFromTurnTable` for more details.


## Definitions

Here follows definitions of some attributes, as well as enumerations of available Wayfinder name types and name languages.


### Road class

The street network is classified in a hierarchical structure of road classes, see also the mc2 enum `ItemTypes::roadClass`.

* **0** Main Road.  
  These roads allow for high volume, maximum speed traffic movement between and
  through major metropolitan areas. There are very few, if any, speed changes.
  Access to the road is usually controlled.
* **1** First Class Road.  
  These roads are used to channel traffic to Main Roads for travel between and
  through cities in the shortest amount of time. There are very few, if any,
  speed changes.
* **2** Second Class Road.  
  These roads interconnect First Class Roads and provide a high volume of
  traffic movement at a lower level of mobility than First Class Roads.
* **3** Third Class Road.  
  These roads provide for a high volume of traffic movement at moderate speeds
  between neighborhoods. These roads connect with higher Class Roads to collect
  and distribute traffic between neighborhoods.
* **4** Fourth Class Road.  
  These roads' volume and traffic movements are below the level of any other
  Class Roads.

### Entry restrictions

The available types of entry restrictions, see also the mc2 enum `ItemTypes::entryrestriction_t´

* **0 - noRestrictions**  
  There are no restrictions when entering the street segment or ferry in this node.
* **1 - noThroughfare**  
  Throughfare is not allowed when entering the street segment or ferry in this node.
* **2 - noEntry**  
  It is not allowed to enter the steet segment or ferry in this node.
* **3 - noWay**  
  The street segment or ferry is one-way, not allowed to travel in direction from this node.

### Name types

The available name types, see also the mc2 enum `ItemTypes::name_t`

* **officialName**  
  The name is the official name in one name language.
* **alternativeName**  
  The name is an alternative name.
* **roadNumber**  
  The name is a roadnumber (has no name language).
* **abbreviationName**  
  The name is an abbreviation of the true name (probably the official or
  alternative name).
* **exitNumber**  
  The name is an exit number on a road sign that can be seen on certain
  highways.
* **synonymName**  
  The name is a synonym. Synonym names will be used in seraching, but will not
  be displayed unless there is no other name with any of the other name types.

### Name languages

A snapshot of the available name languages, short version and long version. See
the mc2 enum `LangTypes::language_t` and the `LangTypes::languageAsString`
table for a list of all available languages.  
The name language invalidLanguage shall be used for names with name type
`roadNumber`, since `roadNumbers` by definition have no language.

* alb - albanian
* baq - basque
* cat - catalan
* cze - czech
* dan - danish
* dut - dutch
* eng - english
* ame - english (us)
* fin - finnish
* fre - french
* fry - frisian
* glg - galician
* ger - german
* gre - greek
* grl - greek latin syntax
* hun - hungarian
* gle - irish
* ita - italian
* ltz - letzeburgesch
* nor - norwegian
* pol - polish
* por - portuguese
* roh - raeto romance
* rus - russian
* scr - serbo croatian
* slo - slovak
* slv - slovenian
* spa - spanish
* swe - swedish
* val - valencian
* wel - welch
* invalidLanguage - invalidLanguage
* rul - russian latin syntax
* tur - turkish
* ara - arabic


### Cartographic types

The available string values for cartographic type of cartographic items. See also mc2 enum class `ItemSubTypes::cartographicType_t`

* amusementParkGroun`d
* campingGround
* toll
* freeport
* abbeyGround
* artsCentreGround
* castleNotToVisitGround
* castleToVisitGround
* churchGround
* cityHallGround
* courthouseGround
* fireStationGround
* fortressGround
* golfGround
* governmentBuildingGround
* hospitalGround
* libraryGround
* lightHouseGround
* monasteryGround
* museumGround
* parkingAreaGround
* placeOfInterestBuilding
* policeOfficeGround
* prisonGround
* railwayStationGround
* recreationalAreaGround
* restAreaGround
* sportsHallGround
* stadiumGround
* statePoliceOffice
* theatreGround
* universityOrCollegeGround
* waterMillGround
* zooGround
* postOfficeGround
* windmillGround
* institution
* otherLandUse
* cemetaryGround
* militaryServiceBranch
* shoppingCenterGround
