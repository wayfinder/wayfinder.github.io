---
title: The WASP database
layout: single
---

The WASP database is the _Wayfinder Administration System for Point of
interests and map correction records_. The POIs and the map corrections are
added to the maps during map generation.

WASP is a mySQL database with tables for POIs, tables for map corrections and
some tables that are shared. Strings are stored in UTF-8 char encoding, and any
coordinates are mc2 coordinates.

## The POI tables overview

The database consists these main tables with data for the POIs

* **POIMain**  
  Main table with administrative info and symbol coordinate.
* **POICategories**  
  Table with the POI search categories for each POI.
* **POIEntryPoints**  
  Table with the entry point coordinates for each POI
* **POIInfo**  
  Table with the static extra information for each POI, such as address, phone number, takeaway availabilities for restaurants
* **POIInfoDynamic**  
  Table with the dynamic extra information for each POI, such as snow depth for ski resorts, price of petrol at petrol stations
* **POINames**  
  Table with the names of each POI
* **POIStatic**  
  Table with the static ID for each POI. This is intended to be static between
  POI releases from one supplier, with the ultimate dream to be static also
  between releases from different suppliers delivering the same POI.
* **POITypes**  
  Table with the POI types of each POI

The rest of the tables for POIs contain data that is more or less static. These
defines types and constants in separate structures. The tables with countries,
version (map release), name languages, name types (and possibly some more),
must be kept in sync with the enums in the mc2-repo. Some of the tables are
shared with the map correction database.

* **EDVersion**  
  Table with the map releases, keep IDs in sync with IDs in the
  `MapGenEnums::mapVersion?  enum. The version string in the table must be the map
  release with the long version of the map supplier string, i.e. the
  `MAP_SUPPLIER` defined in the variable file, e.g. OpenStreetMap_201005
* **POIAttributeTypes**  
  Table with all attributes possible to set for a POI. Used in CPIF file
* **POICategoryTypes**  
  Table with all POI search categories, also links POI types to certain
  categories if they can be mapped uniquely
* **POICategorySearchSynonyms**  
  Table with all POI search synonyms, strings that can be used for searching
  for a certain category.
* **POICountries**  
  Table with all countries in Wayfinder world, keep IDs in sync with IDs of the
  `StringTable::countryCode`
* **POIInfoKeys**  
  Table with all possible POI info
* **POINameLanguages**  
  Table with all defined languages that can be used for POI names, keep IDs in
  sync with IDs in the `LangTypes::language_t` enum.
* **POINameTypes**  
  Table with the defined types of names for the POIs, keep IDs in sync with IDs
  in the `ItemTypes::name_t` enum
* **POIProducts**  
  Table with the known POI products, which group POI sources on supplier and geographical extent
* **POISources**  
  Table with the different POI sources (POI data set)
* **POITypeTypes**  
  Table with the defined POI types, keep IDs in sync with IDs of the
  `ItemTypes::pointOfInterest_t` enum

The `POIProduct` table defines the available POI products in the WASP database. A
product is a POI set that is delivered as one unit, and may be updated on a
regular interval to have e.g. with quarterly releases. One release of a product
is one POI source in WASP. The idea of the POIProduct table is to identify the
previous source so that you could re-use info from previous source when
processing the new source. Hence, the countries that are included in a product
should be constant or increase in numbers. Else, if one country is skipped in
one source and re-appear in the next, info about that country is lost. Info in
this meaning = re-use of geocoded coordinates, static ids etc.

The `POIAttributeTypes.id` for poi info = the POIInfoKeys.id + 1000

The POICategoryTypes table defines the possible POI categories. It links POI
types with POI categories with POI, for the POI types that can be mapped to one
unique category. It also defines the dodonaStringKey that was used for the
translation management of the category string on all different languages.

## The map correction tables overview

The database consists this main tables with data for the map correction records

* **EDMain**  
  Main table with administrative info and the map correction record rest of the tables needed for map corrections contain data that is more or less static. These defines types and constants in separate structures. The tables with countries, and version (map release) must be kept in sync with the enums in the mc2-repo. Some of the tables are shared with the POI database.
* **EDVersion**  
  Table with the map releases, keep IDs in sync with IDs in the
  `MapGenEnums::mapVersion` enum. The version string in the table must be the map
  release with the long version of the map supplier string, i.e. the
  `MAP_SUPPLIER` defined in the variable file, e.g. OpenStreetMap_201005
* **POICountries**  
  Table with all countries in Wayfinder world, keep IDs in sync with IDs of the
  `StringTable::countryCode`
* **EDGroups**  
  Table with defined groups for map correction records, if you want to group a
  bunch of records.
* **EDInsertTypes**  
  Table the with the defined map correction insert types, deciding where in the
  map generation process a certain map correction should be added to the mcm
  map
* **EDTypes**  
  Table with the defined map correction records types, i.e. what kind of map
  corrections are possible to do

Map correction records have once been named extradata records, so that is the
reason for the abbreviation ED (extradata) used in WASP table names, script
names, mc2-repo classes handling map corrections and some documentation.

## The tables definitions, show create table

This section lists the `SHOW CREATE TABLE` statements for the WASP tables, so
you can set up your WASP database.

It also includes documentation about the attributes in some tables, what the
attributes mean and which tables they link to.

Two of the tables have the dodonaStringKey attribute. It links to the string
key in the translation management system that was used for POI category and POI
type strings. For the POI types, most string keys should be found in the mc2
`StringTableUTF8::strings` via the `ItemTypes::getPOIStringCode()?  function. For
the POI categories however, far from all string keys can be found in the mc2
StringTableUTF8.

### POIMain

{% highlight sql %}
CREATE TABLE `POIMain` (
  `ID` int(10) unsigned NOT NULL auto_increment,
  `source` int(10) unsigned NOT NULL default '0',
  `added` datetime default NULL,
  `lastModified` datetime default NULL,
  `lat` int(11) default NULL,
  `lon` int(11) default NULL,
  `inUse` tinyint(1) default '1',
  `deleted` tinyint(1) default '0',
  `sourceReference` varchar(50) default NULL,
  `comment` text,
  `country` int(11) NOT NULL default '0',
  `validFromVersion` int(11) default NULL,
  `validToVersion` int(11) default NULL,
  `rights` varchar(16) NOT NULL default 'FFFFFFFFFFFFFFFF',
  PRIMARY KEY  (`ID`),
  KEY `index_poimain_coord` (`lat`,`lon`),
  KEY `index_poimain_srcref` (`sourceReference`),
  KEY `source` (`source`),
  KEY `index_poimain_country` (`country`),
  KEY `index_poimain_source_coord` (`source`,`lat`,`lon`)
)
{% endhighlight %}

This is the main table, and each POI have one row in this relation.

* **ID** The main ID of the POI. This value is unique to each POI in the database. It is used within the database, as well as stored in the maps to be able to e.g. extract the POI information from the database and add the the m3 maps.
* **source** The source of the data set, which this POI belongs to. Linked to the POISources table.
* **added** The time when the POI was inserted into the database. This must be set by the program / statement that insert the data into the database and is not handled by mySQL. String with pattern "YYYY-MM-DD HH:MM:SS".
* **lastModified** The time when any of the data for the POI was last changed. This must also be set by the program / statement that change the data in the database and is not handled by mySQL. String with pattern "YYYY-MM-DD HH:MM:SS".
* **lat** The latitude part of the coordinate for where the symbol representing the POI will be drawn.
* **lon** The longitude part of the coordinate for where the symbol representing the POI will be drawn.
* **inUse** True (1) if this POI should be inserted into the map, false (0) otherwise. The inUse attribute is not set for individual POIs, but for a complete POI data set (source), per country.
* **deleted** True (1) if this POI should be deleted, i.e. never added to the map, or if it was already added it should be deleted from the map, false (0) otherwise. This attribute is set for individual POIs. Set to deleted if you know that the POI from the supplier does not exist in reality, or if it is a POI type which you do not want to have in your maps.
* **sourceReference** The ID of this POI in the source data, to be able to identify the POI uniquely in the POI source data set.
* **comment** Any kind of comment, text field.
* **country** The country where this POI is located. Linked to the POICountries table. To make sure that the POIs are added to maps of the correct country.
* **validFromVersion** The oldest map version the POI should be added to. If validFromVersion has no value (NULL) there is no lower limit for adding a POI when generating a map from a specific version. Linked to the EDVersion table.
* **validToVersion** The newest map version the POI should be added to. If validToVersion has no value (NULL) there is no upper limit for adding a POI when generating a map from a specific version. Linked to the EDVersion table.
* **rights** The right of this POI. Fill in if intended to have a POI source available only for specific users, else default "FFFFFFFFFFFFFFFF".

### POICategories

{% highlight sql %}
CREATE TABLE `POICategories` (
  `poiID` int(10) unsigned NOT NULL default '0',
  `catID` int(10) unsigned NOT NULL default '0',
  PRIMARY KEY  (`poiID`,`catID`),
  KEY `index_poicategories_poiid` (`poiID`)
)
{% endhighlight %}

Stores the POI search categories for each POI. A POI can have many categories.

* **poiID** The ID of the POI, links to the POIMain table.
* **catID** The ID of the category, links to the POICategoryTypes table.

### POIEntryPoints

{% highlight sql %}
CREATE TABLE `POIEntryPoints` (
  `ID` int(10) unsigned NOT NULL auto_increment,
  `poiID` int(10) unsigned NOT NULL default '0',
  `lat` int(11) NOT NULL default '0',
  `lon` int(11) NOT NULL default '0',
  PRIMARY KEY  (`ID`),
  KEY `index_poientry_poiid` (`poiID`)
)
{% endhighlight %}

Stores the entry point coordinates for each POI. A POI can have many entry points in the database.

* **id** The entry point id
* **poiID** The ID of the POI, links to the POIMain table.
* **lat** The entry point latitude
* **lon** The entry point longitude

### POIInfo

{% highlight sql %}
CREATE TABLE `POIInfo` (
  `ID` int(10) unsigned NOT NULL auto_increment,
  `poiID` int(10) unsigned NOT NULL default '0',
  `keyID` int(10) unsigned NOT NULL default '0',
  `val` text NOT NULL,
  `lang` int(10) unsigned NOT NULL default '0',
  PRIMARY KEY  (`ID`),
  KEY `index_poiinfo_poiid` (`poiID`),
  KEY `index_poiinfo_keyid` (`keyID`)
)
{% endhighlight %}

Stores the static extra informations for each POI. A POI can have many infos. The information for one POI from this table is added to the POI in the m3 map, during map generation.

* **id** The info id
* **poiID** The ID of the POI, links to the POIMain table.
* **keyID** The type of information, links to the POIInfoKeys table
* **val** The information value, e.g. the phone number of a hotel or the street name of where a restaurant is located
* **lang** The language of the information, links to the POINameLanguages table

### POIInfoDynamic

{% highlight sql %}
CREATE TABLE `POIInfoDynamic` (
  `ID` int(10) unsigned NOT NULL auto_increment,
  `poiID` int(10) unsigned NOT NULL default '0',
  `keyID` int(10) unsigned NOT NULL default '0',
  `val` text NOT NULL,
  `lang` int(10) unsigned NOT NULL default '0',
  PRIMARY KEY  (`ID`),
  KEY `index_poiinfodynamic_poiid` (`poiID`),
  KEY `index_poiinfodynamic_keyid` (`keyID`)
)
{% endhighlight %}

Stores the dynamic extra informations for each POI. A POI can have many infos. The information for one POI in this table is of dynamic type, and is updated regularly, e.g. daily, via feeds from the provider that supplied the POIs. The dynamic info is not added to the m3 maps in map generation, but must be added to the POIs real-time by the mc2 back-end server when info about a POI is requested.

* **id** The info id
* **poiID** The ID of the POI, links to the POIMain table.
* **keyID** The type of information, links to the POIInfoKeys table
* **val** The information value, e.g. the snow depth in meters of a ski resort or the price of certain kind of petrol at a petrol station.
* **lang** The language of the information, links to the POINameLanguages table

### POINames

{% highlight sql %}
CREATE TABLE `POINames` (
  `ID` int(10) unsigned NOT NULL auto_increment,
  `poiID` int(10) unsigned NOT NULL default '0',
  `type` int(10) unsigned NOT NULL default '0',
  `lang` int(10) unsigned NOT NULL default '0',
  `name` varchar(255) NOT NULL default '',
  PRIMARY KEY  (`ID`),
  KEY `index_poinames_poiid` (`poiID`),
  KEY `index_poinames_name` (`name`(10))
)
{% endhighlight %}

Stores the names for the POIs. A POI can have many names.

* **id** The name id
* **poiID** The ID of the POI, links to the POIMain table.
* **type** The type of the name, links to the POINameTypes table
* **lang** The language of the information, links to the POINameLanguages table
* **name** The name string

### POIStatic

{% highlight sql %}
CREATE TABLE `POIStatic` (
  `staticID` int(10) unsigned NOT NULL default '0',
  `poiID` int(10) unsigned NOT NULL default '0',
  PRIMARY KEY  (`staticID`,`poiID`),
  KEY `poiID` (`poiID`),
  KEY `ID` (`staticID`)
)
{% endhighlight %}

Stores the static IDS for the POI. One POI can have only one static ID. One static ID can be re-used for many POIs from different sources.

* **staticID** The static ID.
* **poiID** The ID of the POI, links to the POIMain table.

### POITypes

{% highlight sql %}
CREATE TABLE `POITypes` (
  `poiID` int(10) unsigned NOT NULL default '0',
  `typeID` int(10) unsigned NOT NULL default '0',
  PRIMARY KEY  (`poiID`,`typeID`)
)
{% endhighlight %}

Stores the type for each POI. One POI can have only one POI type.

* **poiID** The ID of the POI, links to the POIMain table.
* **typeID** The POI type ID, links to the POITypeTypes table.

### EDVersion

{% highlight sql %}
CREATE TABLE `EDVersion` (
  `ID` int(10) unsigned NOT NULL default '0',
  `version` varchar(50) default NULL,
  PRIMARY KEY  (`ID`)
)
{% endhighlight %}

Lists the defined map versions. Keep IDs in sync with IDs in the MapGenEnums::mapVersion enum. The version string in the table must be the map release with the long version of the map supplier string, i.e. the MAP_SUPPLIER defined in the variable file, e.g. OpenStreetMap_201005 or TeleAtlas_2010_06

### POIAttributeTypes

{% highlight sql %}
CREATE TABLE `POIAttributeTypes` (
  `ID` int(10) unsigned NOT NULL default '0',
  `attributeName` varchar(50) default NULL,
  PRIMARY KEY  (`ID`)
)
{% endhighlight %}

Lists the available attributes that can be added to a POI in the database. Used in Wayfinder CPIF. The ID of attributes that are POI infos have the keyID + 1000.

* **ID** The Id of the attribute
* **attributeName** Short description of the attribute

### POICategoryTypes

{% highlight sql %}
CREATE TABLE `POICategoryTypes` (
  `catID` int(10) unsigned NOT NULL default '0',
  `dodonaStringKey` varchar(100) default NULL,
  `description` varchar(128) default NULL,
  `poiTypeID` int(10) unsigned default NULL,
  PRIMARY KEY  (`catID`)
)
{% endhighlight %}

Lists the available POI search categories.

### POICategorySearchSynonyms

{% highlight sql %}
CREATE TABLE `POICategorySearchSynonyms` (
  `catID` int(10) unsigned NOT NULL,
  `langID` int(10) unsigned NOT NULL,
  `searchSynonym` varchar(128) NOT NULL,
  UNIQUE KEY `catID` (`catID`,`langID`,`searchSynonym`)
)
{% endhighlight %}

Lists the POI category search synonyms. These are added to the poi_category_tree.xml

* **catID** The ID of the POI category
* **langID** The ID of the language for the category search synonym
* **searchSynonym** The search synonym string


### POICountries

{% highlight sql %}
CREATE TABLE `POICountries` (
  `ID` int(10) unsigned NOT NULL default '0',
  `country` varchar(128) default NULL,
  `iso3166_1_alpha2` char(2) default NULL,
  `detailLevel` tinyint(4) default '0',
  PRIMARY KEY  (`ID`)
)
{% endhighlight %}

Lists the available countries. The ID must be kept in sync with the IDs of the mc2 StringTable::countryCode.

* **ID** The ID of the country, in sync with the mc2 StringTable::countryCode
* **country** The name of the country
* **iso3166_1_alpha2** The ISO 3166-1 alpha-2 code, 2 letter code for the country
* **detailLevel** Information about if the country has detailed map data (1) or not (0). Detailed = has major and minor roads with good geometry precision in most cities and villages in the country.

### POIInfoKeys

{% highlight sql %}
CREATE TABLE `POIInfoKeys` (
  `ID` int(10) unsigned NOT NULL default '0',
  `description` varchar(128) NOT NULL default '',
  `isDynamic` tinyint(4) NOT NULL default '0',
  PRIMARY KEY  (`ID`)
)
{% endhighlight %}

Lists the available POI infos.

* **ID** The info key ID
* **description** Short description about what the information is
* **isDynamic** Whether the information is dynamic (1) or static (0). Static information is stored in the POIInfo table, the dynamic informations are stored in the POIInfoDynamic table.


### POINameLanguages

{% highlight sql %}
CREATE TABLE `POINameLanguages` (
  `ID` int(10) unsigned NOT NULL default '0',
  `langName` varchar(128) default NULL,
  `dodonaLangID` varchar(10) default '',
  PRIMARY KEY  (`ID`)
)
{% endhighlight %}

Lists the available languages that can be used for names, info strings, etc. The ID must be kept in sync with IDs in the LangTypes::language_t enum.

* **ID** The ID of the language
* **langName** The language name in English
* **dodonaLangID** The id of the language in the translation management system.

### POINameTypes

{% highlight sql %}
CREATE TABLE `POINameTypes` (
  `ID` int(10) unsigned NOT NULL default '0',
  `typeName` varchar(128) default NULL,
  PRIMARY KEY  (`ID`)
)
{% endhighlight %}

Lists the available languages that can be used for names, info strings, etc. The ID must be kept in sync with IDs in the ItemTypes::name_t enum.

* **ID** The id of the name type
* **typeName** The type name

### POIProducts

{% highlight sql %}
CREATE TABLE `POIProducts` (
  `ID` int(10) unsigned NOT NULL auto_increment,
  `productName` varchar(100) NOT NULL default '',
  `description` text,
  `defaultMapRight` varchar(16) NOT NULL default 'FFFFFFFFFFFFFFFF',
  `migrationCandidate` tinyint(1) NOT NULL default '0',
  `geocodingCandidate` tinyint(1) NOT NULL default '0',
  `trustedSourceRef` tinyint(1) NOT NULL default '0',
  `supplierName` varchar(100) NOT NULL default '',
  `mapDataPOIs` tinyint(1) NOT NULL default '0',
  PRIMARY KEY  (`ID`)
)
{% endhighlight %}

Lists the POI products that are included in the POI database. Products group POI sources on supplier and geographical extent.

A product is a POI set that is delivered as one unit, and may be updated on a regular interval to have e.g. with quarterly releases. One release of a product is one POI source in WASP. The idea of the POIProduct table is to identify the previous source so that you could re-use info from previous source when processing the new source. Hence, the countries that are included in a product should be constant or increase in numbers. Else, if one country is skipped in one source and re-appear in the next, info about that country is lost. Info in this meaning = re-use of geocoded coordinates, static ids etc. 

* **ID** The ID of the product
* **productName** The name of the product
* **description** A description of the POI sets that are part of this product, e.g. supplier and geographical extent
* **defaultMapRight** The default right for the POIs of this product. Default "FFFFFFFFFFFFFFFF" means available for all users, fill in something else if intended to have the POI sources of this product available only for specific users 
* **migrationCandidate** obsolete
* **geocodingCandidate** obsolete
* **trustedSourceRef** Set to 1 = yes for the products where the sourceRef is stable over POI releases or not
* **supplierName** The supplier name that will be displayed when showing info for the POI in the client
* **mapDataPOIs** Set to 1 = yes if the POIs in sources of this product belongs to a map release, set to 0 = false if the POIs are "external" i.e. not part of any map release.

### POISources

{% highlight sql %}
CREATE TABLE `POISources` (
  `ID` int(10) unsigned NOT NULL default '0',
  `source` varchar(64) NOT NULL default '',
  `productID` int(10) unsigned NOT NULL default '0',
  PRIMARY KEY  (`ID`)
)
{% endhighlight %}

Lists the POI sources that are included in the POI database. A source is a release of a POI set.

* **ID** The ID of the POI source
* **source** The name of the source
* **productID** The ID of the product to which the source belong.

### POITypeTypes

{% highlight sql %}
CREATE TABLE `POITypeTypes` (
  `ID` int(10) unsigned NOT NULL default '0',
  `typeName` varchar(128) default NULL,
  `dodonaStringKey` varchar(100) default NULL,
  PRIMARY KEY  (`ID`)
)
{% endhighlight %}

Lists the available POI types, the IDs must be kept in sync with the IDs of the ItemTypes::pointOfInterest_t enum.

* **ID** The ID of the POI type
* **typeName** The name of the type
* **dodonaStringKey** The string key for managing translations of the POI types in the translation management systems. These string keys should exist in mc2 StringTableUTF8.

### EDMain

{% highlight sql %}
CREATE TABLE `EDMain` (
  `ID` int(10) unsigned NOT NULL auto_increment,
  `edType` int(11) NOT NULL default '0',
  `source` varchar(50) default NULL,
  `sourceReference` varchar(50) default NULL,
  `writer` varchar(50) default NULL,
  `added` datetime default NULL,
  `lastModified` datetime default NULL,
  `lat` int(11) default NULL,
  `lon` int(11) default NULL,
  `active` tinyint(1) default NULL,
  `TAreference` varchar(50) default NULL,
  `country` int(11) NOT NULL default '0',
  `edRecord` text,
  `insertType` int(11) NOT NULL default '0',
  `comment` text,
  `validFromVersion` int(11) default NULL,
  `validToVersion` int(11) default NULL,
  `orgValue` varchar(50) default NULL,
  `groupID` int(11) NOT NULL default '0',
  `mapID` int(11) default NULL,
  PRIMARY KEY  (`ID`)
)
{% endhighlight %}

The main table for map correction records.

* **ID** The main ID of the map correction record. This value is unique to each record in the database.
* **edType** What kind of record (e g updateName, setHouseNumber) the record concerns given as an integer. Links to the EDTypes table.
* **source** The person/company who reported the error (e g customers name or some ticket id).
* **sourceReference** A reference number from an error report list if such is used.
* **writer** The person who created the map correction record. When creating records with the MapEditor tool, the user name of the user running the tool is extracted in MELogCommentInterfaceBox::getLogComments and written in the comment row for the map correction record.
* **added** The date and time the map correction record was inserted in the database. This must be set by the program/statement that insert the data into the database and is not handled by mySQL.
* **lastModified** Date and time for last modification of the record. This must be set by the program/statement that insert the data into the database and is not handled by mySQL.
* **lat** The latitude part of the coordinate that defines the record. The coordinate is chosen from one of the coordinates in the record depending. It is mostly used for finding the area where the record is used.
* **lon** The longitude part of the same coordinate that defines the record.
* **active** Indicates if the correction is in use (1) or not (0).
* **TAreference** An error report reference number, currently from Tele Atlas. When the map supplier has corrected the map error, this reference can be used for identifying the correction to inactivate it.
* **country** The country where the map correction record is used. Links to the POICountries table.
* **edRecord** The map correction record.
* **insertType** Describes the where in the map generation process the record should be added to the maps. The names of street segments should for example be modified before the street is generated. Links to the table EDInsertTypes.
* **comment** Any kind of comment.
* **validFromVersion** The oldest map version the map correction should be added to. Typically the version for which the record was created. Linked to the EDVersion table.
* **validToVersion** The newest map version the map correction should be added to. Default set to NULL when adding the record to teh database. When the error has been fixed in a new map release, the validToVersion can be updated to the new map release -1, since no longer wanted. Links to the EDVersion table.
* **orgValue** The value the correction replaces. E g true replaces false when a streetsegment is set to roundabout.
* **groupID** An ID for a group of map correction records, if you wish to group several records by something (if not source or sourceReference appropriate). Links to the EDGroups table.
* **mapID** The id of the mcm map, in which the correction was created with the MapEditor tool.


### EDGroups

{% highlight sql %}
CREATE TABLE `EDGroups` (
  `ID` int(10) unsigned NOT NULL default '0',
  `groupName` varchar(50) default NULL,
  PRIMARY KEY  (`ID`)
)
{% endhighlight %}

Listing the ED groups that are included in the database.

### EDInsertTypes

{% highlight sql %}
CREATE TABLE `EDInsertTypes` (
  `ID` int(10) unsigned NOT NULL default '0',
  `insertTypeName` varchar(50) default NULL,
  PRIMARY KEY  (`ID`)
)
{% endhighlight %}

Listing the available insert types for map corrections, i.e. where in the map generation process the map correction should be added to the mcm maps.

### EDTypes

{% highlight sql %}
CREATE TABLE `EDTypes` (
  `ID` int(10) unsigned NOT NULL default '0',
  `typeName` varchar(50) default NULL,
  PRIMARY KEY  (`ID`)
)
{% endhighlight %}

Listing the available types of map correction records. The types are extracted from the key word in the map correction record.


## The tables content

This section lists the content of more or less static tables, defining enums of countries, name languages, info key ids, categories, etc that must have these enum IDs to sync with the mc2-repo. Fill your database with these enums.


### EDInsertTypes content

Map corrections with type 5 dynamicExtradata is added only in dynamic merge map generation. See the addED.pl script getDefaultInsertType() function for which are the default insert types for certain kinds of map correction records.

	
	+----+--------------------------------+
	| ID | insertTypeName                 |
	+----+--------------------------------+
	|  1 | beforeInternalConnections      |
	|  2 | beforeGenerateStreets          |
	|  3 | beforeGenerateTurndescriptions |
	|  4 | afterGenerateTurndescriptions  |
	|  5 | dynamicExtradata               |
	+----+--------------------------------+


### EDVersion content

The version string in the EDVersion table must be the map release with the long version of the map supplier string, i.e. the MAP_SUPPLIER defined in the variable file, e.g. "OpenStreetMap_201005" or "TeleAtlas_2010_06". The 201005 for OpenStreetMap means that the data is from May 2010, the 2010_06 for Tele Atlas mean that is it their map release 2010.06

	
	+-----+---------------------------+
	| ID  | version                   |
	+-----+---------------------------+
	| 191 | OpenStreetMap_201005      |
	| 192 | TeleAtlas_2010_06         |
	+-----+---------------------------+


### EDTypes content

	
	+----+------------------------+
	| ID | typeName               |
	+----+------------------------+
	|  1 | setRamp                |
	|  2 | setRoundabout          |
	|  3 | setMultiDigitised      |
	|  4 | setEntryRestrictions   |
	|  5 | setSpeedLimit          |
	|  6 | setLevel               |
	|  7 | setHousenumber         |
	|  8 | setTurnDirection       |
	|  9 | setVehicleRestrictions |
	| 10 | addSignpost            |
	| 11 | removeSignpost         |
	| 12 | removeNameFromItem     |
	| 13 | removeItem             |
	| 14 | UpdateName             |
	| 15 | AddNameToItem          |
	| 16 | addlandmark            |
	| 17 | addsinglelandmark      |
	| 20 | setRoundaboutish       |
	| 21 | setControlledAccess    |
	| 22 | setWaterType           |
	| 23 | setTollRoad            |
	+----+------------------------+


### POIAttributeTypes content

	
	+------+---------------------------------------------+
	| ID   | attributeName                               |
	+------+---------------------------------------------+
	|    0 | invalidAttribute                            |
	|    1 | active                                      |
	|    2 | comment                                     |
	|    3 | country                                     |
	|    4 | symbolCoordinate                            |
	|    5 | validFromVersion                            |
	|    6 | validToVersion                              |
	|    7 | poiType                                     |
	|    8 | entryPointCoordinate                        |
	|    9 | name                                        |
	|   10 | source                                      |
	|   11 | sourceReference                             |
	|   12 | poiID                                       |
	|   13 | deleted                                     |
	|   14 | inUse                                       |
	|   15 | poiCategory                                 |
	|   16 | poiExternalIdentifier                       |
	| 1000 | POIInfoKey: Vis. address                    |
	| 1001 | POIInfoKey: Vis. house nbr                  |
	| 1002 | POIInfoKey: Vis. zip code                   |
	| 1003 | POIInfoKey: Vis. complete zip               |
	| 1004 | POIInfoKey: Phone                           |
	| 1005 | POIInfoKey: Vis. zip area                   |
	| 1006 | POIInfoKey: Vis. full address               |
	| 1007 | POIInfoKey: Fax                             |
	| 1008 | POIInfoKey: Email                           |
	| 1009 | POIInfoKey: URL                             |
	| 1010 | POIInfoKey: Brandname                       |
	| 1011 | POIInfoKey: Short description               |
	| 1012 | POIInfoKey: Long description                |
	| 1013 | POIInfoKey: Citypart                        |
	| 1014 | POIInfoKey: State                           |
	| 1015 | POIInfoKey: Neighborhood                    |
	| 1016 | POIInfoKey: Open hours                      |
	| 1017 | POIInfoKey: Nearest train                   |
	| 1018 | POIInfoKey: Start date                      |
	| 1019 | POIInfoKey: End date                        |
	| 1020 | POIInfoKey: Start time                      |
	| 1021 | POIInfoKey: End time                        |
	| 1022 | POIInfoKey: Accommodation type              |
	| 1023 | POIInfoKey: Check in                        |
	| 1024 | POIInfoKey: Check out                       |
	| 1025 | POIInfoKey: Nbr of rooms                    |
	| 1026 | POIInfoKey: Single room from                |
	| 1027 | POIInfoKey: Double room from                |
	| 1028 | POIInfoKey: Triple room from                |
	| 1029 | POIInfoKey: Suite from                      |
	| 1030 | POIInfoKey: Extra bed from                  |
	| 1031 | POIInfoKey: Weekend rate                    |
	| 1032 | POIInfoKey: Nonhotel cost                   |
	| 1033 | POIInfoKey: Breakfast                       |
	| 1034 | POIInfoKey: Hotel services                  |
	| 1035 | POIInfoKey: Credit card                     |
	| 1036 | POIInfoKey: Special feature                 |
	| 1037 | POIInfoKey: Conferences                     |
	| 1038 | POIInfoKey: Average cost                    |
	| 1039 | POIInfoKey: Booking advisable               |
	| 1040 | POIInfoKey: Admission charge                |
	| 1041 | POIInfoKey: Home delivery                   |
	| 1042 | POIInfoKey: Disabled access                 |
	| 1043 | POIInfoKey: Takeaway available              |
	| 1044 | POIInfoKey: Allowed to bring alcohol        |
	| 1045 | POIInfoKey: Type food                       |
	| 1046 | POIInfoKey: Decor                           |
	| 1047 | POIInfoKey: Display class                   |
	| 1048 | POIInfoKey: Image                           |
	| 1049 | POIInfoKey: Supplier                        |
	| 1050 | POIInfoKey: Owner                           |
	| 1051 | POIInfoKey: Price petrol superplus          |
	| 1052 | POIInfoKey: Price petrol super              |
	| 1053 | POIInfoKey: Price petrol normal             |
	| 1054 | POIInfoKey: Price diesel                    |
	| 1055 | POIInfoKey: Price bio diesel                |
	| 1056 | POIInfoKey: Free of charge                  |
	| 1060 | POIInfoKey: Open season                     |
	| 1061 | POIInfoKey: Ski mountain max height         |
	| 1062 | POIInfoKey: Ski mountain min height         |
	| 1063 | POIInfoKey: Snow depth mountain             |
	| 1064 | POIInfoKey: Snow depth valley               |
	| 1065 | POIInfoKey: Snow quality                    |
	| 1066 | POIInfoKey: Lifts total                     |
	| 1067 | POIInfoKey: Lifts open                      |
	| 1068 | POIInfoKey: Slopes total km                 |
	| 1069 | POIInfoKey: Slopes open km                  |
	| 1070 | POIInfoKey: Valley run                      |
	| 1071 | POIInfoKey: Valley run info                 |
	| 1072 | POIInfoKey: Cross country skiing            |
	| 1073 | POIInfoKey: Cross country skiing total km   |
	| 1074 | POIInfoKey: Funpark                         |
	| 1075 | POIInfoKey: Funpark info                    |
	| 1076 | POIInfoKey: Night skiing                    |
	| 1077 | POIInfoKey: Night skiing info               |
	| 1078 | POIInfoKey: Glacier area                    |
	| 1079 | POIInfoKey: Last snowfall                   |
	| 1080 | POIInfoKey: Halfpipe                        |
	| 1081 | POIInfoKey: Halfpipe info                   |
	| 1082 | POIInfoKey: Mountain height (min/max m)     |
	| 1083 | POIInfoKey: Snow depth (valley/mountain cm) |
	| 1084 | POIInfoKey: Lifts (open/total)              |
	| 1085 | POIInfoKey: Slopes (open/total km)          |
	| 1086 | POIInfoKey: Phone Parking TeleP             |
	| 1087 | POIInfoKey: Importance attribute            |
	| 1088 | POIInfoKey: Major road feature              |
	| 1089 | POIInfoKey: Self Service                    |
	| 1090 | POIInfoKey: Has fuel Super95                |
	| 1091 | POIInfoKey: Has fuel Super98                |
	| 1092 | POIInfoKey: Has fuel Normal91               |
	| 1093 | POIInfoKey: Has fuel PKWDiesel              |
	| 1094 | POIInfoKey: Has fuel BioDiesel              |
	| 1095 | POIInfoKey: Has fuel NatGas                 |
	| 1096 | POIInfoKey: Has carwash                     |
	| 1097 | POIInfoKey: Has 24h self service zone       |
	| 1098 | POIInfoKey: Drive In                        |
	| 1099 | POIInfoKey: Unique icon                     |
	| 1100 | POIInfoKey: Sub type icon                   |
	| 1101 | POIInfoKey: Mailbox collection times        |
	| 1103 | POIInfoKey: Booking URL                     |
	| 1104 | POIInfoKey: Booking phone number            |
	| 1105 | POIInfoKey: Star rate                       |
	| 1106 | POIInfoKey: Brand icon base name            |
	| 1107 | POIInfoKey: Smoking allowed                 |
	| 1108 | POIInfoKey: Accessible by car               |
	| 1109 | POIInfoKey: Info on parking by POI          |
	| 1110 | POIInfoKey: Virtual visit URL               |
	| 1111 | POIInfoKey: Free phone number               |
	| 1112 | POIInfoKey: Rating source                   |
	| 1113 | POIInfoKey: Geocoding Accuracy Level        |
	+------+---------------------------------------------+


### POICategoryTypes content

	
	+-------+---------------------------------------+---------------------------------------------------+-----------+
	| catID | dodonaStringKey                       | description                                       | poiTypeID |
	+-------+---------------------------------------+---------------------------------------------------+-----------+
	|     1 | CAT_ON_THE_MOVE                       | On the move                                       |      NULL |
	|     2 | CAT_BY_CAR                            | By car                                            |      NULL |
	|     3 | CAT_FOOD_AND_DRINK                    | Food and drink                                    |      NULL |
	|     4 | SEARCHCAT_LODGING                     | Hotels & Lodging (old: Accommodation)             |      NULL |
	|     5 | SEARCHCAT_TRAVEL                      | Travel & Transport (old: Public transportation)   |      NULL |
	|     6 | SEARCHCAT_NIGHTLIFE                   | Nightlife (old: Entertainment and nightlife)      |        29 |
	|     7 | SEARCHCAT_SPORT                       | Sport & Action (old: Sports activity and leisure) |      NULL |
	|     8 | CAT_TOURISM_AND_CULTURE               | Tourism and culture                               |      NULL |
	|     9 | SEARCHCAT_SHOP                        | Shopping                                          |        56 |
	|    10 | CAT_COMMUNITY_SERVICES                | Community services                                |      NULL |
	|    11 | SEARCHCAT_OTHER                       | Other                                             |      NULL |
	|    12 | CAT_WLAN_HOTSPOTS                     | WLAN Hotspots                                     |        93 |
	|    14 | CAT_TAXI_STAND                        | Taxi stand                                        |       104 |
	|    15 | CAT_RENTACAR                          | Rent-a-car-facility                               |        38 |
	|    18 | CAT_AIRPORT                           | Airport                                           |         1 |
	|    19 | CAT_TOURIST_INFO                      | Tourist info                                      |        48 |
	|    20 | TOURIST_ATTRACTION                    | Tourist attraction                                |        47 |
	|    21 | SCENIC_VIEW                           | Scenic view                                       |        85 |
	|    22 | CAT_MUSEUM                            | Museum                                            |        28 |
	|    23 | MOUNTAIN_PEAK                         | Mountain peak                                     |        77 |
	|    24 | MOUNTAIN_PASS                         | Mountain pass                                     |        76 |
	|    25 | HISTORICAL_MONUMENT                   | Historical monument                               |        21 |
	|    26 | CAT_GUIDED_TOURS                      | Guided tours                                      |      NULL |
	|    27 | CAT_FOREST_AREA                       | Forest area                                       |      NULL |
	|    28 | CULTURAL_CENTRE                       | Cultural centre                                   |        69 |
	|    29 | BEACH                                 | Beach                                             |        64 |
	|    30 | CAT_ART_GALLERIES                     | Art galleries                                     |      NULL |
	|    31 | ZOO                                   | Zoo                                               |        92 |
	|    32 | YACHT_BASIN                           | Yacht basin                                       |        91 |
	|    33 | WATER_SPORTS                          | Water sports                                      |        90 |
	|    34 | TENNIS_COURT                          | Tennis court                                      |        88 |
	|    35 | SWIMMING_POOL                         | Swimming pool                                     |        87 |
	|    36 | STADIUM                               | Stadium                                           |        86 |
	|    37 | SPORTS_CENTRE                         | Sports centre                                     |        45 |
	|    38 | CAT_SPORTS_BEACH                      | Sports beach                                      |      NULL |
	|    39 | SPORTS_ACTIVITY                       | Sports activity                                   |        44 |
	|    40 | CAT_SKI_RESORT                        | Ski resort                                        |        43 |
	|    41 | RECREATION_FACILITY                   | Recreation facility                               |        37 |
	|    42 | PUBLIC_SPORT_AIRPORT                  | Public sport airport                              |        35 |
	|    43 | PARK_AND_RECREATION_AREA              | Park and recreation area                          |        80 |
	|    44 | MARINA                                | Marina                                            |        26 |
	|    45 | ICE_SKATING_RINK                      | Ice skating rink                                  |        24 |
	|    46 | CAT_HIPPODROME                        | Hippodrome                                        |      NULL |
	|    47 | CAT_HIKING_AND_CLIMBING               | Hiking and climbing                               |      NULL |
	|    48 | CAT_GOLF                              | Golf                                              |        19 |
	|    49 | CAT_FISHING                           | Fishing                                           |      NULL |
	|    50 | CAT_EXTREME_OR_ADVENTURE              | Extreme or adventure                              |      NULL |
	|    51 | CAT_EQUESTRIAN                        | Equestrian                                        |      NULL |
	|    52 | CAT_CYCLING_TRACKS                    | Cycling tracks                                    |      NULL |
	|    53 | CAT_BOWLING                           | Bowling                                           |         6 |
	|    54 | AMUSEMENT_PARK                        | Amusement park                                    |         2 |
	|    55 | CAT_TRAVEL_AGENTS                     | Travel agents                                     |      NULL |
	|    56 | CAT_TOYS_AND_GAMES                    | Toys and games                                    |      NULL |
	|    57 | CAT_SUPER_MARKETS_AND_HYPERMARKETS    | Super markets and hyper markets                   |      NULL |
	|    58 | CAT_SPORTS_EQUIPMENT_AND_CLOTHING     | Sports equipment and clothing                     |      NULL |
	|    59 | SHOPPING_CENTRE                       | Shopping centre                                   |        42 |
	|    61 | CAT_OPTICIANS                         | Opticians                                         |      NULL |
	|    62 | CAT_NEWSAGENTS_AND_TOBACCONISTS       | Newsagents and tobacconists                       |      NULL |
	|    63 | CAT_MUSIC_AND_VIDEO                   | Music and video                                   |      NULL |
	|    64 | CAT_JEWELRY_CLOCKS_AND_WATCHES        | Jewelry clocks and watches                        |      NULL |
	|    65 | CAT_HOUSE_AND_GARDEN                  | House and garden                                  |      NULL |
	|    66 | CAT_HAIRDRESSERS_AND_BARBERS          | Hairdressers and barbers                          |      NULL |
	|    67 | GROCERY_STORE                         | Grocery store                                     |        20 |
	|    68 | CAT_GIFTS_AND_SOUVENIRS               | Gifts and souvenirs                               |      NULL |
	|    69 | CAT_FOOD_AND_DRINK_SHOP               | Food and drink shop                               |      NULL |
	|    70 | CAT_FLORISTS                          | Florists                                          |      NULL |
	|    71 | CAT_ESTATE_AGENTS                     | Estate agents                                     |      NULL |
	|    72 | CAT_ELECTRICAL_OFFICE_AND_IT          | Electrical office and IT                          |      NULL |
	|    73 | CAT_DRYCLEANERS                       | Drycleaners                                       |      NULL |
	|    74 | CAT_CONVENIENCE_STORES                | Convenience stores                                |      NULL |
	|    75 | CAT_CLOTHING_AND_ACCESSORIES          | Clothing and accessories                          |      NULL |
	|    76 | CAR_DEALER                            | Car dealer                                        |        66 |
	|    77 | BOOKSHOPS                             | Bookshops                                         |      NULL |
	|    78 | CAT_BEAUTY_SALONS                     | Beauty salons                                     |      NULL |
	|    79 | CAT_ART                               | Art                                               |      NULL |
	|    80 | CAT_ANTIQUES                          | Antiques                                          |      NULL |
	|    81 | CAT_MILITARY_INSTALLATION             | Military installation                             |      NULL |
	|    82 | CAT_BUSINESS                          | Business                                          |      NULL |
	|    83 | CAT_BUILDINGS                         | Buildings                                         |      NULL |
	|    84 | CAT_WINE_AND_BREWERY                  | Wine and brewery                                  |        51 |
	|    85 | RESTAURANTS                           | Restaurants                                       |        40 |
	|    86 | CAFE                                  | Cafe                                              |       100 |
	|    87 | CAT_TICKET_AGENCY                     | Ticket agency                                     |      NULL |
	|    88 | CAT_THEATERS                          | Theaters                                          |        46 |
	|    89 | OPERA                                 | Opera                                             |        79 |
	|    90 | CAT_NIGHT_CLUB                        | Night club                                        |      NULL |
	|    91 | CAT_MUSIC                             | Music                                             |        78 |
	|    93 | CAT_GAY_AND_LESBIAN                   | Gay and lesbian                                   |      NULL |
	|    94 | CAT_DISCOS                            | Discos                                            |      NULL |
	|    95 | CAT_DANCE                             | Dance                                             |      NULL |
	|    96 | CONCERT_HALL                          | Concert hall                                      |        67 |
	|    97 | CAT_COMEDY_AND_CABARET                | Comedy and cabaret                                |      NULL |
	|    98 | CAT_CINEMAS                           | Cinemas                                           |        10 |
	|    99 | CAT_CASINOS                           | Casinos                                           |         9 |
	|   100 | CAT_BARS                              | Bars                                              |      NULL |
	|   101 | CAT_REST_AREAS                        | Rest areas                                        |        39 |
	|   103 | SEARCHCAT_PETROL                      | Petrol Stations (old: Gas station)                |        33 |
	|   104 | CAT_CAR_REPAIR_FACILITY               | Car repair facility                               |        50 |
	|   105 | SEARCHCAT_RELIGION                    | Religion (old: Religious)                         |      NULL |
	|   106 | CAT_PUBLIC_INSTITUTIONS               | Public institutions                               |      NULL |
	|   107 | POST_OFFICE                           | Post office                                       |        52 |
	|   108 | POLICE_STATION                        | Police station                                    |        34 |
	|   109 | CAT_ORGANIZATION                      | Organization                                      |      NULL |
	|   110 | LIBRARY                               | Library                                           |        25 |
	|   111 | SEARCHCAT_HEALTH                      | Health & Medical (old: Health)                    |      NULL |
	|   112 | GOVERNMENT_OFFICE                     | Government office                                 |        75 |
	|   113 | EMBASSY                               | Embassy                                           |        73 |
	|   115 | COURT_HOUSE                           | Court house                                       |        15 |
	|   116 | COMMUNITY_CENTRE                      | Community centre                                  |        13 |
	|   117 | CAT_YOUTH_HOSTEL                      | Youth hostel                                      |      NULL |
	|   118 | CAT_HOTELS                            | Hotels                                            |        23 |
	|   119 | CAT_GUEST_HOUSE_AND_BED_AND_BREAKFAST | Guest house and bed and breakfast                 |      NULL |
	|   120 | CAT_CAMPING                           | Camping                                           |        65 |
	|   121 | CAT_SUBWAY                            | Subway                                            |        99 |
	|   122 | CAT_RAILWAY                           | Railway station                                   |        36 |
	|   123 | COMMUTER_RAIL_STATION                 | Commuter rail station                             |        14 |
	|   124 | CAT_TRAM_STOP                         | Tram stop                                         |        53 |
	|   125 | BUS_STOP_POI_TYPE                     | Bus stop                                          |       103 |
	|   126 | BUS_STATION                           | Bus station                                       |         7 |
	|   127 | CAT_FERRY_TERMINAL_TRAIN              | Ferry terminal train                              |      NULL |
	|   128 | CAT_FERRY_TERMINAL_SHIP               | Ferry terminal ship                               |        17 |
	|   129 | CAT_NATURAL_ATTRACTION                | Natural attraction                                |      NULL |
	|   130 | CAT_MONUMENT                          | Monument                                          |      NULL |
	|   131 | CAT_BUILDING                          | Building                                          |      NULL |
	|   132 | CAT_SCULPTURE                         | Sculpture                                         |      NULL |
	|   133 | CAT_SCIENCE_AND_TECHNOLOGY            | Science and technology                            |      NULL |
	|   134 | CAT_PHOTOGRAPHIC                      | Photographic                                      |      NULL |
	|   136 | CAT_LOCAL_HISTORY_AND_CULTURE         | Local history and culture                         |      NULL |
	|   137 | CAT_GENERAL_HISTORY                   | General history                                   |      NULL |
	|   138 | CAT_FINE_ART                          | Fine art                                          |      NULL |
	|   139 | CAT_CONTEMPORARY_ART                  | Contemporary art                                  |      NULL |
	|   140 | CAT_ARCHITECTURE_AND_DESIGN           | Architecture and design                           |      NULL |
	|   141 | CAT_ANTHROPOLOGY                      | Anthropology                                      |      NULL |
	|   142 | CAT_NAVY                              | Navy                                              |      NULL |
	|   143 | CAT_MARINE_CORPS                      | Marine corps                                      |      NULL |
	|   144 | CAT_COAST_GUARD                       | Coast guard                                       |      NULL |
	|   145 | CAT_ARMY                              | Army                                              |      NULL |
	|   146 | CAT_AIR_FORCE                         | Air force                                         |      NULL |
	|   147 | INDUSTRIAL_COMPLEX                    | Industrial complex                                |        58 |
	|   148 | CAT_CONVENTION_AND_EXHIBITION_CENTRE  | Convention and exhibition centre                  |        16 |
	|   149 | COMPANY                               | Company                                           |         0 |
	|   150 | BUSINESS_FACILITY                     | Business facility                                 |         8 |
	|   151 | BANK                                  | Bank                                              |         5 |
	|   152 | ATM                                   | Atm                                               |         3 |
	|   153 | CAT_PUBLIC_BUILDING                   | Public building                                   |        59 |
	|   154 | CAT_OTHER_BUILDING                    | Other building                                    |        60 |
	|   155 | CAT_VIETNAMESE_KITCHEN                | Vietnamese kitchen                                |      NULL |
	|   156 | CAT_VEGETARIAN_KITCHEN                | Vegetarian kitchen                                |      NULL |
	|   157 | CAT_UNKNOWN_KITCHEN                   | Unknown kitchen                                   |      NULL |
	|   158 | CAT_TURKISH_KITCHEN                   | Turkish kitchen                                   |      NULL |
	|   159 | CAT_THAI_KITCHEN                      | Thai kitchen                                      |      NULL |
	|   160 | CAT_TAVERNAS                          | Tavernas                                          |      NULL |
	|   161 | CAT_TAPAS                             | Tapas                                             |      NULL |
	|   162 | CAT_SWISS_KITCHEN                     | Swiss kitchen                                     |      NULL |
	|   163 | CAT_SURINAMESE_KITCHEN                | Surinamese kitchen                                |      NULL |
	|   164 | CAT_STEAK_HOUSE                       | Steak house                                       |      NULL |
	|   165 | CAT_SPANISH_KITCHEN                   | Spanish kitchen                                   |      NULL |
	|   166 | CAT_SOUTH_AMERICAN_KITCHEN            | South american kitchen                            |      NULL |
	|   167 | CAT_SOUP_BARS                         | Soup bars                                         |      NULL |
	|   168 | CAT_SEA_FOOD                          | Sea food                                          |      NULL |
	|   169 | CAT_SCANDINAVIAN_KITCHEN              | Scandinavian kitchen                              |      NULL |
	|   170 | CAT_SANDWICH                          | Sandwich                                          |      NULL |
	|   171 | CAT_RUSSIAN_KITCHEN                   | Russian kitchen                                   |      NULL |
	|   173 | CAT_PORTUGUESE_KITCHEN                | Portuguese kitchen                                |      NULL |
	|   174 | CAT_POLISH_KITCHEN                    | Polish kitchen                                    |      NULL |
	|   175 | CAT_PIZZERIA                          | Pizzeria                                          |      NULL |
	|   176 | CAT_PACIFIC_RIM_KITCHEN               | Pacific rim kitchen                               |      NULL |
	|   178 | CAT_ORIENTAL_KITCHEN                  | Oriental kitchen                                  |      NULL |
	|   179 | CAT_MIDDLE_EASTERN_KITCHEN            | Middle eastern kitchen                            |      NULL |
	|   180 | CAT_MEXICAN_KITCHEN                   | Mexican kitchen                                   |      NULL |
	|   181 | CAT_MEDITERRANEAN_KITCHEN             | Mediterranean kitchen                             |      NULL |
	|   182 | CAT_MALTESE_KITCHEN                   | Maltese kitchen                                   |      NULL |
	|   183 | CAT_MALAYSIAN_KITCHEN                 | Malaysian kitchen                                 |      NULL |
	|   184 | CAT_LOCAL_KITCHEN                     | Local kitchen                                     |      NULL |
	|   185 | CAT_LATIN_AMERICAN_KITCHEN            | Latin american kitchen                            |      NULL |
	|   186 | CAT_KOSHER_KITCHEN                    | Kosher kitchen                                    |      NULL |
	|   187 | CAT_KOREAN_KITCHEN                    | Korean kitchen                                    |      NULL |
	|   188 | CAT_JEWISH_KITCHEN                    | Jewish kitchen                                    |      NULL |
	|   189 | CAT_JAPANESE_KITCHEN                  | Japanese kitchen                                  |      NULL |
	|   190 | CAT_ITALIAN_KITCHEN                   | Italian kitchen                                   |      NULL |
	|   191 | CAT_INDONESIAN_KITCHEN                | Indonesian kitchen                                |      NULL |
	|   192 | CAT_INDIAN_KITCHEN                    | Indian kitchen                                    |      NULL |
	|   193 | CAT_HUNGARIAN_KITCHEN                 | Hungarian kitchen                                 |      NULL |
	|   194 | CAT_HAWAIIAN_KITCHEN                  | Hawaiian kitchen                                  |      NULL |
	|   195 | CAT_GRILL                             | Grill                                             |      NULL |
	|   196 | CAT_GREEK_KITCHEN                     | Greek kitchen                                     |      NULL |
	|   197 | CAT_GERMAN_KITCHEN                    | German kitchen                                    |      NULL |
	|   198 | CAT_FRENCH_KITCHEN                    | French kitchen                                    |      NULL |
	|   200 | CAT_FISH_AND_SEAFOOD                  | Fish and seafood                                  |      NULL |
	|   201 | CAT_FILIPINO_KITCHEN                  | Filipino kitchen                                  |      NULL |
	|   202 | CAT_FAST_FOOD                         | Fast food                                         |      NULL |
	|   203 | CAT_EAST_EUROPEAN_KITCHEN             | East european kitchen                             |      NULL |
	|   204 | CAT_DUTCH_KITCHEN                     | Dutch kitchen                                     |      NULL |
	|   205 | CAT_DELIS_AND_DINERS                  | Delis and diners                                  |      NULL |
	|   206 | CAT_CREPERIE                          | Creperie                                          |      NULL |
	|   207 | CAT_CONTEMPORARY_KITCHEN              | Contemporary kitchen                              |      NULL |
	|   208 | CAT_CHINESE_KITCHEN                   | Chinese kitchen                                   |      NULL |
	|   209 | CAT_CENTRAL_EUROPEAN_KITCHEN          | Central european kitchen                          |      NULL |
	|   210 | CAT_CARIBBEAN_KITCHEN                 | Caribbean kitchen                                 |      NULL |
	|   211 | CAT_CANADIAN_KITCHEN                  | Canadian kitchen                                  |      NULL |
	|   212 | CAT_CALIFORNIAN_KITCHEN               | Californian kitchen                               |      NULL |
	|   213 | CAT_BUFFET                            | Buffet                                            |      NULL |
	|   214 | CAT_BRITISH_KITCHEN                   | British kitchen                                   |      NULL |
	|   215 | CAT_BREAKFAST_AND_BRUNCH              | Breakfast and brunch                              |      NULL |
	|   216 | CAT_BISTRO                            | Bistro                                            |      NULL |
	|   217 | CAT_BELGIAN_KITCHEN                   | Belgian kitchen                                   |      NULL |
	|   218 | CAT_BARBECUE                          | Barbecue                                          |      NULL |
	|   219 | CAT_AUSTRIAN_KITCHEN                  | Austrian kitchen                                  |      NULL |
	|   220 | CAT_AUSTRALIAN_KITCHEN                | Australian kitchen                                |      NULL |
	|   221 | CAT_ASIAN_KITCHEN                     | Asian kitchen                                     |      NULL |
	|   222 | CAT_AMERICAN_KITCHEN                  | American kitchen                                  |      NULL |
	|   223 | CAT_AFRICAN_KITCHEN                   | African kitchen                                   |      NULL |
	|   224 | CAT_AFGHAN_KITCHEN                    | Afghan kitchen                                    |      NULL |
	|   225 | CAT_TEA                               | Tea                                               |      NULL |
	|   226 | CAT_JUICE_BARS                        | Juice bars                                        |      NULL |
	|   227 | CAT_ICE_CREAM                         | Ice cream                                         |      NULL |
	|   228 | CAT_COFFEE                            | Coffee                                            |      NULL |
	|   229 | CAT_WINE_BARS                         | Wine bars                                         |      NULL |
	|   230 | CAT_THEME_BARS                        | Theme bars                                        |      NULL |
	|   231 | CAT_SPORTS_BARS                       | Sports bars                                       |      NULL |
	|   232 | CAT_PUBS                              | Pubs                                              |      NULL |
	|   233 | CAT_COCKTAIL_BARS                     | Cocktail bars                                     |      NULL |
	|   234 | CAT_BIERKELLER                        | Bierkeller                                        |      NULL |
	|   235 | CAT_BEER_GARDENS                      | Beer gardens                                      |      NULL |
	|   236 | CAT_PARKING_GARAGE                    | Parking garage                                    |        32 |
	|   237 | OPEN_PARKING_AREA                     | Open parking area                                 |        30 |
	|   239 | SYNAGOGUE                             | Synagogue                                         |        98 |
	|   240 | PLACE_OF_WORSHIP                      | Place of worship                                  |        82 |
	|   241 | MOSQUE                                | Mosque                                            |        97 |
	|   242 | CHURCH                                | Church                                            |        96 |
	|   243 | CEMETERY                              | Cemetery                                          |        57 |
	|   244 | VETRINARIAN                           | Vetrinarian                                       |        89 |
	|   245 | PHARMACY                              | Pharmacy                                          |        81 |
	|   246 | HOSPITAL                              | Hospital                                          |        22 |
	|   247 | CAT_HEALTH_CENTRE                     | Health centre                                     |      NULL |
	|   248 | DOCTOR                                | Doctor                                            |        71 |
	|   249 | DENTIST                               | Dentist                                           |        70 |
	|   254 | UNIVERSITY                            | University                                        |        49 |
	|   255 | SCHOOL                                | School                                            |        41 |
	|   256 | HINDU_TEMPLE                          | Hindu temple                                      |       101 |
	|   257 | BUDDHIST_SITE                         | Buddhist site                                     |       102 |
	|   258 | RESTAURANT_AREA                       | Restaurant area                                   |        84 |
	|   259 | PARK_AND_RIDE                         | Park and ride                                     |        31 |
	|   260 | CAT_MARKETS                           | Markets                                           |      NULL |
	|   261 | CAT_BUSINESS_SERVICES                 | Business services                                 |      NULL |
	|   262 | CAT_SPAS_AND_MINERAL_BATHS            | Spas and mineral baths                            |      NULL |
	|   263 | CAT_POST_BUSINESS_CENTRE              | Post business centre                              |      NULL |
	|   264 | CAT_MAIL_BOX                          | Mailbox                                           |      NULL |
	|   265 | CAT_STAMP_SELLER                      | Stamp seller                                      |      NULL |
	|   266 | CAT_CITY_CENTER                       | City centre                                       |        11 |
	|   267 | SEARCHCAT_PARKING                     | Parking                                           |      NULL |
	|   268 | SEARCHCAT_ARTS                        | Arts & Design                                     |      NULL |
	|   269 | SEARCHCAT_BEAUTY                      | Beauty & Wellness                                 |      NULL |
	|   270 | SEARCHCAT_CAR_BIKE                    | Cars & Bikes                                      |      NULL |
	|   271 | SEARCHCAT_ENTERTAIN                   | Entertainment                                     |      NULL |
	|   272 | SEARCHCAT_BANK_ATM                    | Banks & ATM's                                     |      NULL |
	|   273 | SEARCHCAT_FASHION                     | Fashion & Lifestyle                               |      NULL |
	|   274 | SEARCHCAT_FOOD_DRINK                  | Food & Drink                                      |      NULL |
	|   275 | SEARCHCAT_HEALTH                      | Health & Medical                                  |      NULL |
	|   276 | SEARCHCAT_LODGING                     | Hotels & Lodging                                  |      NULL |
	|   277 | SEARCHCAT_NATURE                      | Nature                                            |      NULL |
	|   278 | SEARCHCAT_NIGHTLIFE                   | Nightlife                                         |        29 |
	|   280 | SEARCHCAT_PETROL                      | Petrol Stations                                   |        33 |
	|   281 | SEARCHCAT_PUBLIC                      | Public Services                                   |      NULL |
	|   282 | SEARCHCAT_TRAVEL                      | Travel & Transport                                |      NULL |
	|   283 | SEARCHCAT_RELIGION                    | Religion                                          |      NULL |
	|   284 | SEARCHCAT_SHOP                        | Shopping                                          |        56 |
	|   285 | SEARCHCAT_SPORT                       | Sport & Action                                    |      NULL |
	|   286 | SEARCHCAT_TOURISM                     | Tourism & Sights                                  |      NULL |
	|   287 | SEARCHCAT_ROAD                        | Road Service                                      |      NULL |
	|   288 | SEARCHCAT_OTHER                       | Other                                             |      NULL |
	|   289 | CAT_AIRPORT_TERMINAL                  | Airport terminal                                  |        63 |
	|   290 | CAT_ATTRACTIONS                       | Attractions                                       |      NULL |
	+-------+---------------------------------------+---------------------------------------------------+-----------+


### POICountries content

Set detailLevel to 1 for the countries where you have detailed maps.

	
	+-----+--------------------------------------+------------------+-------------+
	| ID  | country                              | iso3166_1_alpha2 | detailLevel |
	+-----+--------------------------------------+------------------+-------------+
	|   0 | England                              | GB               |           0 |
	|   1 | Sweden                               | SE               |           0 |
	|   2 | Germany                              | DE               |           0 |
	|   3 | Denmark                              | DK               |           0 |
	|   4 | Finland                              | FI               |           0 |
	|   5 | Norway                               | NO               |           0 |
	|   6 | Belgium                              | BE               |           0 |
	|   7 | Netherlands                          | NL               |           0 |
	|   8 | Luxembourg                           | LU               |           0 |
	|   9 | USA                                  | US               |           0 |
	|  10 | Switzerland                          | CH               |           0 |
	|  11 | Austria                              | AT               |           0 |
	|  12 | France                               | FR               |           0 |
	|  13 | Spain                                | ES               |           0 |
	|  14 | Andorra                              | AD               |           0 |
	|  15 | Liechtenstein                        | LI               |           0 |
	|  16 | Italy                                | IT               |           0 |
	|  17 | Monaco                               | MC               |           0 |
	|  18 | Ireland                              | IE               |           0 |
	|  19 | Portugal                             | PT               |           0 |
	|  20 | Canada                               | CA               |           0 |
	|  21 | Hungary                              | HU               |           0 |
	|  22 | Czech Republic                       | CZ               |           0 |
	|  23 | Poland                               | PL               |           0 |
	|  24 | Greece                               | GR               |           0 |
	|  25 | Israel                               | IL               |           0 |
	|  26 | Brazil                               | BR               |           0 |
	|  27 | Slovakia                             | SK               |           0 |
	|  28 | Russia                               | RU               |           0 |
	|  29 | Turkey                               | TR               |           0 |
	|  30 | Slovenia                             | SI               |           0 |
	|  31 | Bulgaria                             | BG               |           0 |
	|  32 | Romania                              | RO               |           0 |
	|  33 | Ukraine                              | UA               |           0 |
	|  34 | Serbia and Montenegro                | CS               |           0 |
	|  35 | Croatia                              | HR               |           0 |
	|  36 | Bosnia and Herzegovina               | BA               |           0 |
	|  37 | Moldova                              | MD               |           0 |
	|  38 | Macedonia                            | MK               |           0 |
	|  39 | Estonia                              | EE               |           0 |
	|  40 | Latvia                               | LV               |           0 |
	|  41 | Lithuania                            | LT               |           0 |
	|  42 | Belarus                              | BY               |           0 |
	|  43 | Malta                                | MT               |           0 |
	|  44 | Cyprus                               | CY               |           0 |
	|  45 | Iceland                              | IS               |           0 |
	|  46 | Hong Kong                            | HK               |           0 |
	|  47 | Singapore                            | SG               |           0 |
	|  48 | Australia                            | AU               |           0 |
	|  49 | United Arab Emirates                 | AE               |           0 |
	|  50 | Bahrain                              | BH               |           0 |
	|  51 | Afghanistan                          | AF               |           0 |
	|  52 | Albania                              | AL               |           0 |
	|  53 | Algeria                              | DZ               |           0 |
	|  54 | American Samoa                       | AS               |           0 |
	|  55 | Angola                               | AO               |           0 |
	|  56 | Anguilla                             | AI               |           0 |
	|  57 | Antarctica                           | AQ               |           0 |
	|  58 | Antigua and Barbuda                  | AG               |           0 |
	|  59 | Argentina                            | AR               |           0 |
	|  60 | Armenia                              | AM               |           0 |
	|  61 | Aruba                                | AW               |           0 |
	|  62 | Azerbaijan                           | AZ               |           0 |
	|  63 | Bahamas                              | BS               |           0 |
	|  64 | Bangladesh                           | BD               |           0 |
	|  65 | Barbados                             | BB               |           0 |
	|  66 | Belize                               | BZ               |           0 |
	|  67 | Benin                                | BJ               |           0 |
	|  68 | Bermuda                              | BM               |           0 |
	|  69 | Bhutan                               | BT               |           0 |
	|  70 | Bolivia                              | BO               |           0 |
	|  71 | Botswana                             | BW               |           0 |
	|  72 | British Virgin Islands               | VG               |           0 |
	|  73 | Brunei Darussalam                    | BN               |           0 |
	|  74 | Burkina Faso                         | BF               |           0 |
	|  75 | Burundi                              | BI               |           0 |
	|  76 | Cambodia                             | KH               |           0 |
	|  77 | Cameroon                             | CM               |           0 |
	|  78 | Cape Verde                           | CV               |           0 |
	|  79 | Cayman Islands                       | KY               |           0 |
	|  80 | Central African Republic             | CF               |           0 |
	|  81 | Chad                                 | TD               |           0 |
	|  82 | Chile                                | CL               |           0 |
	|  83 | China                                | CN               |           0 |
	|  84 | Colombia                             | CO               |           0 |
	|  85 | Comoros                              | KM               |           0 |
	|  86 | Congo                                | CG               |           0 |
	|  87 | Cook Islands                         | CK               |           0 |
	|  88 | Costa Rica                           | CR               |           0 |
	|  89 | Cuba                                 | CU               |           0 |
	|  90 | Djibouti                             | DJ               |           0 |
	|  91 | Dominica                             | DM               |           0 |
	|  92 | Dominican Republic                   | DO               |           0 |
	|  93 | D.R. Congo                           | CD               |           0 |
	|  94 | Ecuador                              | EC               |           0 |
	|  95 | Egypt                                | EG               |           0 |
	|  96 | El Salvador                          | SV               |           0 |
	|  97 | Equatorial Guinea                    | GQ               |           0 |
	|  98 | Eritrea                              | ER               |           0 |
	|  99 | Ethiopia                             | ET               |           0 |
	| 100 | Faeroe Islands                       | FO               |           0 |
	| 101 | Falkland Islands                     | FK               |           0 |
	| 102 | Fiji                                 | FJ               |           0 |
	| 103 | French Guiana                        | GF               |           0 |
	| 104 | French Polynesia                     | PF               |           0 |
	| 105 | Gabon                                | GA               |           0 |
	| 106 | Gambia                               | GM               |           0 |
	| 107 | Georgia                              | GE               |           0 |
	| 108 | Ghana                                | GH               |           0 |
	| 109 | Greenland                            | GL               |           0 |
	| 110 | Grenada                              | GD               |           0 |
	| 111 | Guadeloupe                           | GP               |           0 |
	| 112 | Guam                                 | GU               |           0 |
	| 113 | Guatemala                            | GT               |           0 |
	| 114 | Guinea                               | GN               |           0 |
	| 115 | Guinea-Bissau                        | GW               |           0 |
	| 116 | Guyana                               | GY               |           0 |
	| 117 | Haiti                                | HT               |           0 |
	| 118 | Honduras                             | HN               |           0 |
	| 119 | India                                | IN               |           0 |
	| 120 | Indonesia                            | ID               |           0 |
	| 121 | Iran                                 | IR               |           0 |
	| 122 | Iraq                                 | IQ               |           0 |
	| 123 | Ivory Coast                          | CI               |           0 |
	| 124 | Jamaica                              | JM               |           0 |
	| 125 | Japan                                | JP               |           0 |
	| 126 | Jordan                               | JO               |           0 |
	| 127 | Kazakhstan                           | KZ               |           0 |
	| 128 | Kenya                                | KE               |           0 |
	| 129 | Kiribati                             | KI               |           0 |
	| 130 | Kuwait                               | KW               |           0 |
	| 131 | Kyrgyzstan                           | KG               |           0 |
	| 132 | Laos                                 | LA               |           0 |
	| 133 | Lebanon                              | LB               |           0 |
	| 134 | Lesotho                              | LS               |           0 |
	| 135 | Liberia                              | LR               |           0 |
	| 136 | Libya                                | LY               |           0 |
	| 137 | Macao                                | MO               |           0 |
	| 138 | Madagascar                           | MG               |           0 |
	| 139 | Malawi                               | MW               |           0 |
	| 140 | Malaysia                             | MY               |           0 |
	| 141 | Maldives                             | MV               |           0 |
	| 142 | Mali                                 | ML               |           0 |
	| 143 | Marshall Islands                     | MH               |           0 |
	| 144 | Martinique                           | MQ               |           0 |
	| 145 | Mauritania                           | MR               |           0 |
	| 146 | Mauritius                            | MU               |           0 |
	| 147 | Mayotte                              | YT               |           0 |
	| 148 | Mexico                               | MX               |           0 |
	| 149 | Micronesia                           | FM               |           0 |
	| 150 | Mongolia                             | MN               |           0 |
	| 151 | Montserrat                           | MS               |           0 |
	| 152 | Morocco                              | MA               |           0 |
	| 153 | Mozambique                           | MZ               |           0 |
	| 154 | Myanmar                              | MM               |           0 |
	| 155 | Namibia                              | NA               |           0 |
	| 156 | Nauru                                | NR               |           0 |
	| 157 | Nepal                                | NP               |           0 |
	| 158 | Netherlands Antilles                 | AN               |           0 |
	| 159 | New Caledonia                        | NC               |           0 |
	| 160 | New Zealand                          | NZ               |           0 |
	| 161 | Nicaragua                            | NI               |           0 |
	| 162 | Niger                                | NE               |           0 |
	| 163 | Nigeria                              | NG               |           0 |
	| 164 | Niue                                 | NU               |           0 |
	| 165 | Northern Mariana Islands             | MP               |           0 |
	| 166 | North Korea                          | KP               |           0 |
	| 167 | Occupied Palestinian Territory       | PS               |           0 |
	| 168 | Oman                                 | OM               |           0 |
	| 169 | Pakistan                             | PK               |           0 |
	| 170 | Palau                                | PW               |           0 |
	| 171 | Panama                               | PA               |           0 |
	| 172 | Papua New Guinea                     | PG               |           0 |
	| 173 | Paraguay                             | PY               |           0 |
	| 174 | Peru                                 | PE               |           0 |
	| 175 | Philippines                          | PH               |           0 |
	| 176 | Pitcairn                             | PN               |           0 |
	| 177 | Qatar                                | QA               |           0 |
	| 178 | Reunion                              | RE               |           0 |
	| 179 | Rwanda                               | RW               |           0 |
	| 180 | Saint Helena                         | SH               |           0 |
	| 181 | Saint Kitts and Nevis                | KN               |           0 |
	| 182 | Saint Lucia                          | LC               |           0 |
	| 183 | Saint Pierre and Miquelon            | PM               |           0 |
	| 184 | Saint Vincent and the Grenadines     | VC               |           0 |
	| 185 | Samoa                                | WS               |           0 |
	| 186 | Sao Tome and Principe                | ST               |           0 |
	| 187 | Saudi Arabia                         | SA               |           0 |
	| 188 | Senegal                              | SN               |           0 |
	| 189 | Seychelles                           | SC               |           0 |
	| 190 | Sierra Leone                         | SL               |           0 |
	| 191 | Solomon Islands                      | SB               |           0 |
	| 192 | Somalia                              | SO               |           0 |
	| 193 | South Africa                         | ZA               |           0 |
	| 194 | South Korea                          | KR               |           0 |
	| 195 | Sri Lanka                            | LK               |           0 |
	| 196 | Sudan                                | SD               |           0 |
	| 197 | Suriname                             | SR               |           0 |
	| 198 | Svalbard and Jan Mayen               | SJ               |           0 |
	| 199 | Swaziland                            | SZ               |           0 |
	| 200 | Syria                                | SY               |           0 |
	| 201 | Taiwan                               | TW               |           0 |
	| 202 | Tajikistan                           | TJ               |           0 |
	| 203 | United Republic of Tanzania          | TZ               |           0 |
	| 204 | Thailand                             | TH               |           0 |
	| 205 | Timor-Leste                          | TL               |           0 |
	| 206 | Togo                                 | TG               |           0 |
	| 207 | Tokelau                              | TK               |           0 |
	| 208 | Tonga                                | TO               |           0 |
	| 209 | Trinidad and Tobago                  | TT               |           0 |
	| 210 | Tunisia                              | TN               |           0 |
	| 211 | Turkmenistan                         | TM               |           0 |
	| 212 | Turks and Caicos Islands             | TC               |           0 |
	| 213 | Tuvalu                               | TV               |           0 |
	| 214 | Uganda                               | UG               |           0 |
	| 215 | United States Minor Outlying Islands | UM               |           0 |
	| 216 | United States Virgin Islands         | VI               |           0 |
	| 217 | Uruguay                              | UY               |           0 |
	| 218 | Uzbekistan                           | UZ               |           0 |
	| 219 | Vanuatu                              | VU               |           0 |
	| 220 | Venezuela                            | VE               |           0 |
	| 221 | Viet Nam                             | VN               |           0 |
	| 222 | Wallis and Futuna Islands            | WF               |           0 |
	| 223 | Western Sahara                       | EH               |           0 |
	| 224 | Yemen                                | YE               |           0 |
	| 225 | Zambia                               | ZM               |           0 |
	| 226 | Zimbabwe                             | ZW               |           0 |
	+-----+--------------------------------------+------------------+-------------+


### POIInfoKeys content

The Vis. address is the street name part of the address (vis. stands for
visiting), the Vis. house nbr is the house number part of the address. The Vis.
full address is the combined street name and house number. If the POI supplier
provides both the combined and the split up address, it is a good idea to use
both of them.

Similarly the Vis. zip code and Vis. zip area, are the sub parts that are
combined to the the Vis. complete zip. 

Phone and fax must be numbers including the international dialing code, so it
is possible to call the number from any phone from anywhere in the world. Any
non-digit characters within the phone number string (after the leading + sign)
are OK, since they will be removed by the mc2 server.

Short description and long description is the place for storing reviews or any
other kind of text about the POI. Keep the short one short, the long one long.

The info key 49 Supplier, carries the supplier string that is presented to the
user informing about which supplier that has provided the POI. It should be
stored with English language in the POIInfo table.

The info key 100 Sub type icon can be used for specifying that the default POI
icon (derived from POI type) shall not be used when displaying a map image, but
instead a special POI icon. E.g. for metro stations in Paris (POI type 99
subwayStation), give the sub type icon "franceparissubway", then the POI icon
named "subway_station_franceparissubway" will be used instead of the default
"subway_station" (here leaving out the suffix of the file name). The sub type
icons must also be defined in the mc2
`POIImageIdentificationTable::POIImageIdentificationTable` table.

Some of the info keys are boolean, meaning 1 = true, 0 = false. See list in
`GetAdditionalPOIInfo::m_keyID_value_is_bool`

There are IDs missing in this list, the "empty" IDs are info keys that are deprecated and no longer in use.
	
	+-----+---------------------------------+-----------+
	| ID  | description                     | isDynamic |
	+-----+---------------------------------+-----------+
	|   0 | Vis. address                    |         0 |
	|   1 | Vis. house nbr                  |         0 |
	|   2 | Vis. zip code                   |         0 |
	|   3 | Vis. complete zip               |         0 |
	|   4 | Phone                           |         0 |
	|   5 | Vis. zip area                   |         0 |
	|   6 | Vis. full address               |         0 |
	|   7 | Fax                             |         0 |
	|   8 | Email                           |         0 |
	|   9 | URL                             |         0 |
	|  10 | Brandname                       |         0 |
	|  11 | Short description               |         0 |
	|  12 | Long description                |         0 |
	|  13 | Citypart                        |         0 |
	|  14 | State                           |         0 |
	|  15 | Neighborhood                    |         0 |
	|  16 | Open hours                      |         0 |
	|  17 | Nearest train                   |         0 |
	|  18 | Start date                      |         0 |
	|  19 | End date                        |         0 |
	|  20 | Start time                      |         0 |
	|  21 | End time                        |         0 |
	|  22 | Accommodation type              |         0 |
	|  23 | Check in                        |         0 |
	|  24 | Check out                       |         0 |
	|  25 | Nbr of rooms                    |         0 |
	|  26 | Single room from                |         0 |
	|  27 | Double room from                |         0 |
	|  28 | Triple room from                |         0 |
	|  29 | Suite from                      |         0 |
	|  30 | Extra bed from                  |         0 |
	|  31 | Weekend rate                    |         0 |
	|  32 | Nonhotel cost                   |         0 |
	|  33 | Breakfast                       |         0 |
	|  34 | Hotel services                  |         0 |
	|  35 | Credit card                     |         0 |
	|  36 | Special feature                 |         0 |
	|  37 | Conferences                     |         0 |
	|  38 | Average cost                    |         0 |
	|  39 | Booking advisable               |         0 |
	|  40 | Admission charge                |         0 |
	|  41 | Home delivery                   |         0 |
	|  42 | Disabled access                 |         0 |
	|  43 | Takeaway available              |         0 |
	|  44 | Allowed to bring alcohol        |         0 |
	|  45 | Type food                       |         0 |
	|  46 | Decor                           |         0 |
	|  47 | Display class                   |         0 |
	|  48 | Image                           |         0 |
	|  49 | Supplier                        |         0 |
	|  50 | Owner                           |         0 |
	|  51 | Price petrol superplus          |         1 |
	|  52 | Price petrol super              |         1 |
	|  53 | Price petrol normal             |         1 |
	|  54 | Price diesel                    |         1 |
	|  55 | Price bio diesel                |         1 |
	|  56 | Free of charge                  |         0 |
	|  60 | Open season                     |         1 |
	|  61 | Ski mountain max height         |         1 |
	|  62 | Ski mountain min height         |         1 |
	|  63 | Snow depth mountain             |         1 |
	|  64 | Snow depth valley               |         1 |
	|  65 | Snow quality                    |         1 |
	|  66 | Lifts total                     |         1 |
	|  67 | Lifts open                      |         1 |
	|  68 | Slopes total km                 |         1 |
	|  69 | Slopes open km                  |         1 |
	|  70 | Valley run                      |         1 |
	|  71 | Valley run info                 |         1 |
	|  72 | Cross country skiing            |         1 |
	|  73 | Cross country skiing total km   |         1 |
	|  74 | Funpark                         |         1 |
	|  75 | Funpark info                    |         1 |
	|  76 | Night skiing                    |         1 |
	|  77 | Night skiing info               |         1 |
	|  78 | Glacier area                    |         1 |
	|  79 | Last snowfall                   |         1 |
	|  80 | Halfpipe                        |         1 |
	|  81 | Halfpipe info                   |         1 |
	|  82 | Mountain height (min/max m)     |         1 |
	|  83 | Snow depth (valley/mountain cm) |         1 |
	|  84 | Lifts (open/total)              |         1 |
	|  85 | Slopes (open/total km)          |         1 |
	|  86 | Phone Parking TeleP             |         0 |
	|  87 | Importance                      |         0 |
	|  88 | Major road feature              |         0 |
	|  89 | Has service                     |         0 |
	|  90 | Has fuel Super95                |         0 |
	|  91 | Has fuel Super98                |         0 |
	|  92 | Has fuel Normal91               |         0 |
	|  93 | Has fuel PKWDiesel              |         0 |
	|  94 | Has fuel BioDiesel              |         0 |
	|  95 | Has fuel NatGas                 |         0 |
	|  96 | Has carwash                     |         0 |
	|  97 | Has 24h self service zone       |         0 |
	|  98 | Drive In                        |         0 |
	|  99 | Unique icon                     |         0 |
	| 100 | Sub type icon                   |         0 |
	| 101 | Mailbox collection times        |         0 |
	| 103 | Booking URL                     |         0 |
	| 104 | Booking phone number            |         0 |
	| 105 | Star rate                       |         0 |
	| 107 | Smoking allowed                 |         0 |
	| 108 | Accessible by car               |         0 |
	| 109 | Info on parking by POI          |         0 |
	| 110 | Virtual visit URL               |         0 |
	| 111 | Free phone number               |         0 |
	| 112 | Rating source                   |         0 |
	| 113 | Geocoding Accuracy Level        |         0 |
	+-----+---------------------------------+-----------+


### POINameLanguages content

	
	+----+-------------------------+--------------+
	| ID | langName                | dodonaLangID |
	+----+-------------------------+--------------+
	|  0 | english                 | en           |
	|  1 | swedish                 | sv           |
	|  2 | german                  | de           |
	|  3 | danish                  | da           |
	|  4 | italian                 | it           |
	|  5 | dutch                   | nl           |
	|  6 | spanish                 | es           |
	|  7 | french                  | fr           |
	|  8 | welch                   |              |
	|  9 | finnish                 | fi           |
	| 10 | norwegian               | no           |
	| 11 | portuguese              | pt           |
	| 12 | english (us)            | us           |
	| 13 | czech                   | cs           |
	| 14 | albanian                |              |
	| 15 | basque                  |              |
	| 16 | catalan                 |              |
	| 17 | frisian                 |              |
	| 18 | irish                   |              |
	| 19 | galician                |              |
	| 20 | letzeburgesch           |              |
	| 21 | raeto romance           |              |
	| 22 | serbo croatian          | sr           |
	| 23 | slovenian               | sl           |
	| 24 | valencian               |              |
	| 25 | hungarian               | hu           |
	| 26 | greek                   | el           |
	| 27 | polish                  | pl           |
	| 28 | slovak                  | sk           |
	| 29 | russian                 | ru           |
	| 30 | greek latin syntax      |              |
	| 31 | invalidLanguage         |              |
	| 32 | russian latin syntax    |              |
	| 33 | turkish                 | tr           |
	| 34 | arabic                  | ar           |
	| 35 | chinese                 | zh           |
	| 36 | chinese latin syntax    |              |
	| 37 | estonian                | et           |
	| 38 | latvian                 | lv           |
	| 39 | lithuanian              | lt           |
	| 40 | thai                    |              |
	| 41 | bulgarian               | bg           |
	| 42 | cyrillic transcript     |              |
	| 43 | indonesian              | in           |
	| 44 | malay                   | ms           |
	| 49 | tagalog                 | tl           |
	| 50 | belarusian              |              |
	| 53 | croatian                | hr           |
	| 54 | farsi                   | fa           |
	| 58 | hebrew                  | iw           |
	| 65 | macedonian              |              |
	| 68 | moldavian               |              |
	| 71 | romanian                | ro           |
	| 72 | serbian                 |              |
	| 81 | ukrainian               | uk           |
	| 86 | bulgarian latin syntax  |              |
	| 87 | bosnian                 |              |
	| 88 | slavic                  |              |
	| 89 | belarusian latin syntax |              |
	| 90 | macedonian latin syntax |              |
	| 91 | serbian latin syntax    |              |
	| 92 | ukrainian latin syntax  |              |
	| 93 | maltese                 |              |
	| 94 | chinese traditional     | zht          |
	| 96 | thai latin syntax       |              |
	+----+-------------------------+--------------+


### POINameTypes content

	
	+----+------------------+
	| ID | typeName         |
	+----+------------------+
	|  0 | officialName     |
	|  1 | alternativeName  |
	|  2 | roadNumber       |
	|  3 | invalidName      |
	|  4 | abbreviationName |
	|  5 | uniqueName       |
	|  6 | exitNumber       |
	|  7 | synonymName      |
	+----+------------------+



### POIProducts content

	
	+----+-------------------+---------------------------+------------------+--------------------+--------------------+------------------+---------------+-------------+
	| ID | productName       | description               | defaultMapRight  | migrationCandidate | geocodingCandidate | trustedSourceRef | supplierName  | mapDataPOIs |
	+----+-------------------+---------------------------+------------------+--------------------+--------------------+------------------+---------------+-------------+
	| 51 | OSM_eu            | OpenStreetMap data Europe | FFFFFFFFFFFFFFFF |                  0 |                  0 |                1 | OpenStreetMap |           1 |
	| 52 | TeleAtlas_eu_shp  | Tele Atlas Europe shape   | FFFFFFFFFFFFFFFF |                  0 |                  0 |                1 | Tele Atlas    |           1 |
	+----+-------------------+---------------------------+------------------+--------------------+--------------------+------------------+---------------+-------------+


### POISources content

	
	+-----+--------------------------+-----------+
	| ID  | source                   | productID |
	+-----+--------------------------+-----------+
	| 249 | OSM_201005               |        51 |
	| 250 | TeleAtlas_2010_06_eu_oss |        52 |
	+-----+--------------------------+-----------+


### POITypeTypes content

	
	+-----+------------------------------+---------------------------------+
	| ID  | typeName                     | dodonaStringKey                 |
	+-----+------------------------------+---------------------------------+
	|   0 | company                      | NULL                            |
	|   1 | airport                      | AIRPORT                         |
	|   2 | amusementPark                | AMUSEMENT_PARK                  |
	|   3 | atm                          | ATM                             |
	|   4 | automobileDealership         | AUTOMOBILE_DEALERSHIP           |
	|   5 | bank                         | BANK                            |
	|   6 | bowlingCentre                | BOWLING_CENTRE                  |
	|   7 | busStation                   | BUS_STATION                     |
	|   8 | businessFacility             | BUSINESS_FACILITY               |
	|   9 | casino                       | CASINO                          |
	|  10 | cinema                       | CINEMA                          |
	|  11 | cityCentre                   | CITY_CENTRE                     |
	|  12 | cityHall                     | CITY_HALL                       |
	|  13 | communityCentre              | COMMUNITY_CENTRE                |
	|  14 | commuterRailStation          | COMMUTER_RAIL_STATION           |
	|  15 | courtHouse                   | COURT_HOUSE                     |
	|  16 | exhibitionOrConferenceCentre | EXHIBITION_OR_CONFERENCE_CENTRE |
	|  17 | ferryTerminal                | FERRY_TERMINAL                  |
	|  18 | frontierCrossing             | FRONTIER_CROSSING               |
	|  19 | golfCourse                   | GOLF_COURSE                     |
	|  20 | groceryStore                 | GROCERY_STORE                   |
	|  21 | historicalMonument           | HISTORICAL_MONUMENT             |
	|  22 | hospital                     | HOSPITAL                        |
	|  23 | hotel                        | HOTEL                           |
	|  24 | iceSkatingRink               | ICE_SKATING_RINK                |
	|  25 | library                      | LIBRARY                         |
	|  26 | marina                       | MARINA                          |
	|  27 | motoringOrganisationOffice   | MOTORING_ORGANISATION_OFFICE    |
	|  28 | museum                       | MUSEUM                          |
	|  29 | nightlife                    | NIGHTLIFE                       |
	|  30 | openParkingArea              | OPEN_PARKING_AREA               |
	|  31 | parkAndRide                  | PARK_AND_RIDE                   |
	|  32 | parkingGarage                | PARKING_GARAGE                  |
	|  33 | petrolStation                | PETROL_STATION                  |
	|  34 | policeStation                | POLICE_STATION                  |
	|  35 | publicSportAirport           | PUBLIC_SPORT_AIRPORT            |
	|  36 | railwayStation               | RAILWAY_STATION                 |
	|  37 | recreationFacility           | RECREATION_FACILITY             |
	|  38 | rentACarFacility             | RENT_A_CAR_FACILITY             |
	|  39 | restArea                     | REST_AREA                       |
	|  40 | restaurant                   | RESTAURANT                      |
	|  41 | school                       | SCHOOL                          |
	|  42 | shoppingCentre               | SHOPPING_CENTRE                 |
	|  43 | skiResort                    | SKI_RESORT                      |
	|  44 | sportsActivity               | SPORTS_ACTIVITY                 |
	|  45 | sportsCentre                 | SPORTS_CENTRE                   |
	|  46 | theatre                      | THEATRE                         |
	|  47 | touristAttraction            | TOURIST_ATTRACTION              |
	|  48 | touristOffice                | TOURIST_OFFICE                  |
	|  49 | university                   | UNIVERSITY                      |
	|  50 | vehicleRepairFacility        | VEHICLE_REPAIR_FACILITY         |
	|  51 | winery                       | WINERY                          |
	|  52 | postOffice                   | POST_OFFICE                     |
	|  53 | tramStation                  | TRAM_STATION                    |
	|  54 | multi                        | NULL                            |
	|  56 | shop                         | POI_SHOP                        |
	|  57 | cemetery                     | CEMETERY                        |
	|  58 | industrialComplex            | INDUSTRIAL_COMPLEX              |
	|  59 | publicIndividualBuilding     | NULL                            |
	|  60 | otherIndividualBuilding      | NULL                            |
	|  61 | notCategorised               | NULL                            |
	|  62 | unknownType                  | NULL                            |
	|  63 | airlineAccess                | NULL                            |
	|  64 | beach                        | BEACH                           |
	|  65 | campingGround                | CAMPING_GROUND                  |
	|  66 | carDealer                    | CAR_DEALER                      |
	|  67 | concertHall                  | CONCERT_HALL                    |
	|  68 | tollRoad                     | TOLL_ROAD                       |
	|  69 | culturalCentre               | CULTURAL_CENTRE                 |
	|  70 | dentist                      | DENTIST                         |
	|  71 | doctor                       | DOCTOR                          |
	|  72 | driveThroughBottleShop       | DRIVE_THROUGH_BOTTLE_SHOP       |
	|  73 | embassy                      | EMBASSY                         |
	|  74 | entryPoint                   | NULL                            |
	|  75 | governmentOffice             | GOVERNMENT_OFFICE               |
	|  76 | mountainPass                 | MOUNTAIN_PASS                   |
	|  77 | mountainPeak                 | MOUNTAIN_PEAK                   |
	|  78 | musicCentre                  | MUSIC_CENTRE                    |
	|  79 | opera                        | OPERA                           |
	|  80 | parkAndRecreationArea        | PARK_AND_RECREATION_AREA        |
	|  81 | pharmacy                     | PHARMACY                        |
	|  82 | placeOfWorship               | PLACE_OF_WORSHIP                |
	|  83 | rentACarParking              | RENT_A_CAR_PARKING              |
	|  84 | restaurantArea               | RESTAURANT_AREA                 |
	|  85 | scenicView                   | SCENIC_VIEW                     |
	|  86 | stadium                      | STADIUM                         |
	|  87 | swimmingPool                 | SWIMMING_POOL                   |
	|  88 | tennisCourt                  | TENNIS_COURT                    |
	|  89 | vetrinarian                  | VETRINARIAN                     |
	|  90 | waterSports                  | WATER_SPORTS                    |
	|  91 | yachtBasin                   | YACHT_BASIN                     |
	|  92 | zoo                          | ZOO                             |
	|  93 | wlan                         | WLAN                            |
	|  94 | noType                       | NULL                            |
	|  95 | invalidPOIType               | NULL                            |
	|  96 | church                       | CHURCH                          |
	|  97 | mosque                       | MOSQUE                          |
	|  98 | synagogue                    | SYNAGOGUE                       |
	|  99 | subwayStation                | SUBWAY_STATION                  |
	| 100 | cafe                         | CAFE                            |
	| 101 | hinduTemple                  | HINDU_TEMPLE                    |
	| 102 | buddhistSite                 | BUDDHIST_SITE                   |
	| 103 | busStop                      | BUS_STOP_POI_TYPE               |
	| 104 | taxiStop                     | TAXI_STOP_POI_TYPE              |
	+-----+------------------------------+---------------------------------+



## POI handling

### WASPing - adding POIs to mcm maps

#### normal WASPing

POIs are added to mcm maps in indivdual countries second map generations with the WASPExtractor --addPOIToMap option. This process is called WASPing, or normal WASPing. All underview mcm maps of the country are looped, and for every mcm map, the WASP database is contacted and POIs that fit the mcm map are add to it.

   * The POIMain attributes inUse+deleted+validFromVersion+validToVersion decide if a POI is candidate to be added to a certain map release
   * The POIMain attribute country decide if the POI belongs to the country of the mcm map
   * The POIMain attributes lat+lon decide if the POI is inside the bounding box the a mcm map
   * Finally the POIEntryPoints lat+lon decides if the POI really fits the mcm map. If the entry point coordinate is within 5 meters from a street segment in the mcm map, it fits. If the POI is added to one mcm map, it is not added to any other of the looped maps, even if also within 5 meters from a street segment in that mcm map.

#### dynamic WASPing

Running a merge map generation it is possible to do a dynamic WASPing with the WASPExtractor --updatePOIInMap option. In the dynamic WASPing all underview mcm maps in the merge storage directory are looped and WASP database contacted to apply changes that happened in the WASP database since the last time the mcm map was WASPed (had POIs added to it, normal or dynamic).

   * New POIs that were imported into the WASP database after the last WASP-time, will be added to the map. New POIs are detected with having the POIMain.added date attribute later than the WASP-time stored in the mcm map header.
   * POIs that have been stopped in the WASP database since the last WASPing, will be removed. Old POIs are detected with that the POIMain.lastModified date attribute is larger (after) the WASP-time stored in the mcm map header, and either of the inUse, deleted or validToVersion attributes indicate that the POI no longer should be used.
   * POIs that had updates, e.g. had a an additional name added, or changed POI type or category, will be modified accordingly in the mcm map where they exist. Modified POIs are detected with that the POIMain.lastModified date attribute is larger (after) the WASP-time stored in the mcm map header, and the inUse, deleted or validToVersion attributes indicates that the POI should be used.


### Import POIs into WASP database

In POI import, these things are happening
   - Import

      * The POI is added to the POI main tables with the import script and the POI Object perl module. The administrative info about the POI, such as validFromVersion, validToVersion, added-date, lastModified-date, inUse, deleted, etc are important to set correctly, for the POIs to be added to the correct mcm maps.
      * The validFromVersion of the POIs is picked from the country/before_merge/mapOrigin.txt file in the tmpEW or tmpWW pointed at with the -mapCountriesDirs option.
      * POIs that are map data POIs (POIProducts.mapDataPOIs) will have the validToVersion set = validFromVersion, since they should be used only with the map data they belong to. POIs that are NOT map data POIs will have the validToVersion set to NULL, since we don't know how long (for which map releases) they should be used yet.
   - Create static IDs

      * The static IDs of the previous source of the product are re-used as much as possible with the routines poi_doublesCheck.pl script. If the product has POIProduct.trustedSourceRef = yes it helps alot, because we can match the POIMain.sourceReference between the old source and the new source. Else the script runs a duplication detection to match on similarities in coordinate, name, and POI type. This detection does not give a 100% result.
      * If no matching POI in the previous source, the POI in the new source is given a new static ID. This is also the case if importing the first source of a product (-newProduct option)
   - Set the POI rights

      * Set the POIMain.rights for each POI in the source, from the POIProducts.defaultMapRight attribute.
   - Create entry points, if not given from the map supplier

      * Loop the mcm maps in all countries/before_merge dirs in tmpEW or tmpWW pointed at with the -mapCountriesDirs option
      * For each mcm map contact WASP database with WASPExtractor --createEntryPoints option.
      * WASPExtractor selects all POIs that fits the mcm map according to the requirements listed above for normal WASPing, i.e. fits the map release and is within the bbox of the mcm map. If the POI has no entry point, one such is calculated by finding the street segment in the mcm map that is
         - closest to the POI symbol coordinate, has the same name as the POI info key 0 visiting address, and is within 200 meters from the symbol coordinate.
         - closest to symbol coord, regardless of name

      * After looping all maps of the country of the POI, the coordinate of the closest street segment is used as the entry point. There is a limit of 10 km, if the POI has no street segment within 10 km it is not given any entry point.
      * The entry point is added to the POIEntryPoints table and POIMain.lastModified is updated.

The order of the steps differs a little for map data POIs and not-map data POIs, see poi_autoImport.pl for details.

A special remark for entry points. For map data POIs, the mcm maps of all countries are looped in creation. But running import and ep-creation for external POIs (not map data POIs), only loops mcm maps of the countries that have POICountries.detailLevel = 1 (true). Reason is, only in detailed countries it is interesting to add external POIs to the maps. If large parts of the street network is missing in one country, the external POIs cannot be added to the correct street segment (entry point) anyway.


## Map correction handling


### Create map correction records with MapEditor

Map correction records (also in some old documentation called extradata records) are created
using the [MapEditor tool](Other map related documentations#MapEditor). When correcting map
errors with the MapEditor a text file with records corresponding to the
corrections is automatically created. To each record a comment is
generated holding such information that is not necessary for making
the correction in the map, but most useful when administrating the
records.


### Add map correction records to the WASP database


The map correction records are parsed with their comments and added to the WASP database.
The import is done with the addED.pl perl script

### Add map correction records to the maps in map generation

When generating mcm-maps appropriate map correction records are extracted
from the database with the ExtraDataExtractor and temporarily
stored in map correction files which then are added to the maps with 
the ExtraDataReader class called with GenerateMapServer -x option.
See ExtraDataReader::parseInto for how the map correction records are parsed and added to the mcm maps.

### Map corrections in extradata.sh

Map errors that cannot be fixed by simple map correction records from MapEditor, need to be fixed in the extradata.sh. It can be e.g. 

   * adding missing street segments, islands or other items from midmif files in Wayfinder midmif format. Appropriate to use map ssi coordinates for defining in which mcm map the items should be added to.
   * fix of incorrect geometry of roads. Which implies removal of the items with the incorrect geometry with map correction records and addition of items with correct geometry from midmif files in Wayfinder midmif format.


### The map correction record format

The map correction record format has a very simple structure, it is a text file that consists of records of different types. Each record has two parts; a comment part with administrative information and the record part with the actual map correction record.

The field separator used in the comment and the record is the special string `<¡>` (inverted exclamation mark within less-than and greater-than signs). In mc2 it is defined by `MC2MapGenUtil::poiWaspFieldSep` for use in the `MapEditor`, `ExtradataExtractor` and `ExtraDataReader` and by param `$m_edFieldSep` in the `addED.pl` script.

Please see `OldExtraDataUtility::record_t` for all available map correction record types, also defined in WASP EDTypes table. See `ExtraDataReader::parseInto` for how each of the record types are parsed.

Here is listed some record types with example records

#### COMMENT

This is not a real record - only a comment. The comment must, however,
contain certain information about the record that follows for
correct parsing into the database. This includes
original value, mapsupplier reference id (if appropriate), extra data insert type, etc.
See the [EDMain table](#edmain) attributes for further explanation of the comment fields.

   * date-and time string
   * writer, the person who created the map correction record)
   * source, who reported the map error
   * comment
   * original value
   * ref id, the map supplier map error report reference id
   * ed record id
   * ed insert type string
   * map release (valid from - valid to)
   * ed group id
   * country
   * map id, id of the mcm map in which the correction was created with the MapEditor tool

Example of comment for record created with MapEditor:

    # Wed Sep 24 15:48:28 2009`<¡>`lotta`<¡>`mc2`<¡>`turn not allowed`<¡>`0xffffffff`<¡>``<!`>``<¡>`beforeGenerateTurndescriptions`<¡>``<¡>`0`<¡>`Sweden`<¡>`0`<¡>`EndOfRecord`<¡>`

A comment extracted from WASP with e.g. ExtradataExtractor has another
format of the date-and-time string. It represents the time when the
extradata record was added to the database. The comment also contains the extradata record database id.
Example:

    # 2010-04-08 18:18:00`<¡>`lotta`<¡>`MSV-60874-685`<¡>``<¡>`0xffffffff`<¡>`W107oqs17`<¡>`657904`<¡>`beforeGenerateTurndescriptions`<¡>``<¡>`0`<¡>`France`<¡>`145`<¡>`EndOfRecord`<¡>`


#### REMOVE_ITEM

Removes one item from the map.  Example:

    removeItem`<¡>`waterItem`<¡>`mc2`<¡>`(664392321,157860254)`<¡>`Triangeldammen`<¡>`(666807565,153555910)`<¡>`EndOfRecord`<¡>`

The item type + coordinate + name identifies the item in the mcm map. The
second coordinate is the map ssi coordinate, for defining from which mcm map
the item should be removed. Really helpful if the item to remove is on or close
to the map boundry, so not an item in the neighbouring mcm map is removed
instead.

#### SET_ROUNDABOUT

Sets the roundabout attribute of a given street segment to a new
(boolean) value.  Example:

    setRoundabout`<¡>`(689139671, 146281933)`<¡>`(689138666, 146282305)`<¡>`true`<¡>`EndOfRecord`<¡>`

The street segment to update roundabout attribute for is identified by two
coordinates; the first one is the coord of one of it's node, the second is a
coordinate along the segment close to the node.

#### UPDATE_NAME

Updates the name of an item.  Example:

    UpdateName`<¡>`streetsegment`<¡>`mc2`<¡>`(782495844,228940632)`<¡>`Kyrkgatan`<¡>`officialName`<¡>`swe`<¡>`Skolgatan`<¡>`EndOfRecord`<¡>`

The item is identified by the item type + coordinate + item name. The name to
update Kyrkgatan is identified by the item name + name type + name language.
The new name Skolgatan will have the same name type and language as the old
one. It is not possible to change type or language with the UpdateName record.
If you want to change name type and/or language, you need to do one
`removeNameFromItem` in combination with `addNameToItem`.
