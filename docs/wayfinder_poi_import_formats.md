---
title: Wayfinder POI import formats
layout: single
---

For import of POIs into the WASP database, there are some import formats defined

* Simple POI import format, SPIF
* Complex POI import format, CPIF

See `poiImport.pl -f readSPIF` and `readCPIF` respectively for details about
how the POI files are parsed and imported into the WASP database. For full
import into WASP including entry points, static IDs etc use the
`poi_autoImport.pl` script.

If you have POIs in SPIF file, it can be converted to CPIF with `poiImport -f
convertSPIFtoCPIF`, so you can add the POIs to WASP with the full routine for
CPIF with the `poi_autoImport.pl` script.

## Simple POI import format

This document describes the file format used for importing points of interest,
POIs, to the Wayfinder server. The format is a single file text format
including columns with a fixed column order. In the text file, each row
represents one POI and the order of the columns defines the type of attribute
value. The columns are separated by semicolon. This format is easy to create
from Excel files. Some attributes are mandatory and some are optional. When an
optional attribute has no value, it is represented by an empty column like
";;". Empty columns at the end of a row are not necessary to include.

Because the column separator is semicolon, it is strictly forbidden to include
semicolon in column value text. Therefore, it is unfortunately impossible to
include semicolon in attribute values. Furthermore, the values should never
include escaped values, such as "&lt;" for "<". The text should be in character
set UTF-8.

Columns marked with "mandatory" below must have values.

	   Columns:
	   1.  sourceReference,      mandatory
	   2.  name,                 mandatory
	   3.  typeID,               mandatory
	   4.  lat,                  mandatory
	   5.  lon,                  mandatory
	   6.  country,
	   7.  zipCode,
	   8.  zipArea,
	   9.  addressStreetName,
	   10. addressHouseNbr,
	   11. fullAddress,
	   12. phoneNbr,
	   13. faxNbr, 
	   14. email, 
	   15. url, 
	   16. openHours,
	   17. shortDescription,
	   18. longDescription,
	   19. image,
	   20. ccDisplayClass,


This is an example of a single row representing a POI. In the example, column 13, 14, 17, 18, 19 and 20 have no values.

	10025;Lauda Air Ligitarsasag;1;47.494921;19.051054;Hungary;1052;Budapest; \\
	Aranykiz utca;6;Aranykiz utca 6B;+36-1-2663169;;;www.luandaair.hu;0 - 24;

### Columns

* **sourceReference**  
  A unique ID to be used for this POI, unique within the POI data set.This column must have a value.
* **name**  
  The name of the POI. Will be used when searching for the POI. This column must have a value.
* **typeID**  
  The ID of the Wayfinder POI type. The POI type is used for defining which icon is used when displaying the POI in the map. The type may also in the WASP POICategoryType table be mapped to a POI category, which is used for making category searches, like searching for all restaurants. The available type IDs can be found in the WASP POITypeTypes table and in the mc2 enum ItemTypes::pointOfInterest_t. This column must have a value.
* **lat**  
  WGS 84 decimal latitude coordinate value. This column must have a value.
* **lon**  
  WGS 84 decimal longitude coordinate value. This column must have a value.
* **country**  
  A text string defining the country of the POI. The country text strings to use can be found in the WASP POICountries table.
* **zipCode**  
  The zip code of the POI.
* **zipArea**  
  The zip area name of the POI (city).
* **addressStreetName**  
  The street name part of the POI address.
* **addressHouseNbr**  
  The house number part of the POI address.
* **fullAddress**  
  Both the street name and house number parts of an address. This column is only used for showing info about a POI to the user.
* **phoneNbr**  
  The phone number of the POI. This number should be formatted so it can be used for calling directly from the phone. The country code of the phone number is mandatory and should be preceded by a "+" sign. All non-digit characters except for the leading "+" sign will be removed before using the number for calling. Example value: "+36-1-2663168".
* **faxNbr**  
  The fax number of the POI. This number should be formatted so it can be used for calling directly from the phone. The country code of the phone number is mandatory and should be preceded by a "+" sign. All non-digit characters except for the leading "+" sign will be removed before using the number for calling. Example value: "+36-1-2663169".
* **email**  
  The email address of the POI.
* **url**  
  The web page URL of the POI. Do not include the "http colon slash slash"
* **openHours**  
  The open hours of the POI. Text field, avoid using words in any specific language if possible.
* **shortDescription**  
  A short description of the POI. Try to keep below 100 characters.
* **longDescription**  
  A long description of the POI.
* **image**  
  The name of the image file with a picture of the POI. The images must be delivered on the side.
* **ccDisplayClass**  
  Only used for POIs with city center POI type, leave blank otherwise. The value should be one of the settlement display classes used in Tele Atlas MultiNet, see Appendix A. 

### Appendix A 

Center of Settlement display class

	
	1 -
	2 capital of country
	3 - 
	4 most important settlement of a municipality >= 1.000.000 inhabitants
	5 most important settlement of a municipality of 500.000-999.999 inhabitants
	6 - 
	7 most important settlement of a municipality of 100.000-499.999 inhabitants
	8 most important settlement of a municipality of 50.000-99.999 inhabitants
	9 -
	10 most important settlement of a municipality of 10.000-49.999 inhabitants
	11 most important settlement of a municipality < 10.000 inhabitants
	12 all other settlements




## Complex POI import format 

The complex format (CPIF) is a text-based format that is very flexible. The attributes for a POI are given one attribute per row. Each row is semicolon separated with the following information:

* POI marker ID to keep the attributes for one POI collected
* POI attribute type id
* POI attribute value

The POI Attribute type is used as defined in the WASP POIAttributeTypes table, and the attribute value as below:

* 2 Comment: String value
* 3 Country: Unsigned integer value as defined in the POICountries table.
* 4 Symbol Coordinate: Coordinate string "lat;lon" with coordinates expressed in mc2.
* 5 Valid From Version: Unsigned integer value as defined in the EDVersion table.
* 6 Valid To Version: Unsigned integer value as defined in the EDVersion table.
* 7 POI Type: Unsigned integer value as defined in the POITypeTypes table.
* 8 Entry Point Coordinate: Coordinate string "lat;lon" with coordinates expressed in mc2.
* 9 Name: Name string "type;lang;name" with type unsigned integer value as defined in the POINameTypes table and lang unsigned integer as defined in the POINameLanguages table. A POI can have many names
* 10 Source: Unsigned integer value as defined in the POISources table.
* 11 Source Reference: String value, e.g. representing POI id at the supplier.
* 12 POI ID: Unsigned integer value representing the ID of the POI in WASP.
* 13 Deleted: Boolean integer value 0 or 1.
* 14 In Use: Boolean integer value 0 or 1.
* 15 POI Category: Unsigned integer value as defined in the POICategoryTypes table.
* 16 POI External Identifier: deprecated
* 1000 + infoKeyID POI Info: String value "lang;val" with lang unsigned integer as defined in the POINameLanguages table. The infoKeyID from the WASP POIInfoKeys table.

All combined attribute value strings are separated by semicolons. The CPIF format handles POI names and POI info values with semicolons since the name-part of these attributes is last in the combined string.

This is one example of a set of rows representing a subway station POI.

	
	poi202500300135368;3;12
	poi202500300135368;4;582718423;28059427
	poi202500300135368;7;99
	poi202500300135368;9;0;7;Place Monge
	poi202500300135368;11;poi202500300135368
	poi202500300135368;1000;7;Rue Monge
	poi202500300135368;1002;31;75005
	poi202500300135368;1004;31;+(33)-(8)-36687714
	poi202500300135368;1005;31;Paris
	poi202500300135368;1009;31;www.ratp.fr
	poi202500300135368;1087;31;2



### Character encoding

   * The encoding in the file should be UTF-8.

### Some rules

* When creating a POI from CPIF, if the deleted value is left out it is by default set to false (0). Likewise the inUse value is set to true (1).
* The POI ID cannot be decided in CPIF when importing POIs into WASP, but is set automatically when adding each POI to WASP.
* If any of country, source, validFromVersion and validToVersion are left out, they can be set by param to the script that is parsing the CPIF.
* A POI can/must/should not be entered into WASP without valid values for at least:
    * Source
    * Source reference
    * Country
    * Symbol coordinate
    * Official name


