---
title: Tile Map Format
layout: single
---

This is a description of how to implement a Tile Map Client, with references to the C++ client's MapLib implementation.

## Overview of protocol

The protocol consists of five types of messages, the *TileMapFormatDesc*, the TileMapFormatDesc CRC, geometry maps, text maps and bitmaps (icons).

   * The TileMapFormatDesc (TMFD) is the style sheet describing almost all general information about the maps. This includes information needed to decode the maps, category information, styles, how to request maps coverings the screen etc. TMFD is language dependent.
   * The TileMapFormatDesc CRC is used to ensure that the latest TileMapFormatDesc is present in the application. Language dependent.
   * Geometry maps contain vector data and are the same for all languages.
   * Text maps contain string information for the vector data in the geometry maps and are language dependent, i.e. if the language is changed in the client, new text maps have to be requested, but not the geometry maps.
   * Bitmaps contain pure png or svg data.

## General information

   * Any code inside `MC2_SYSTEM` defines can be ignored.
   * All text that is sent from the server uses utf8 character encoding.
   * The byteorder used is big endian. Bits are sent highest bit first.
   * The coordinate system of the maps, `MC2Coordinate`, is based on the WGS84 map projection. The longitude and latitude values are stored as 32 bit integers. The longitudes span from `MIN_INT` to `MAX_INT` and the latitude from MIN_INT/2 to MAX_INT/2. (0,0) corresponds to Greenwich at the equator. The factors `GfxConstants::MC2SCALE_TO_METER` and `GfxConstants::METER_TO_MC2SCALE` are used to convert between mc2 units and meters on the equator. 
   * The scale level that is used in MapLib is defined as meters per pixel on a screen with the same resolution as a Nokia 6600 (Symbian Series 60 v2). In case a device uses higher resolution than this, then this must be accounted for so that it the maps are not too cluttered with detail.

##  Cook book 

Fetch CRC for TileMapFormatDesc from server.
See `TileMapFormatDescCRC::createParamString`
 
Fetch TileMapFormatDesc from local map cache (read/write cache or preinstalled cache).
See `TileMapFormatDesc::createParamString`

When TileMapFormatDesc or CRC arrives, compare CRC.
If mismatching CRC, get new TileMapFormatDesc and CRC from server. 
Cache the TileMapFormatDesc but not the CRC.
Use the existing TileMapFormatDesc until the CRC arrives. I.e. if no CRC can be fetched from the server, due to lacking internet, then the cached TileMapFormatDesc should be used.

When language changes, request new CRC and repeat above.

Once TileMapFormatDesc is received:

Load the TileMapFormatDesc.
See `TileMapFormatDesc::load` and [detailed comments of loading TileMapFormatDesc](../tilemapformatdescload)

If failure to load TMFD, request new TMFD from the server with exponential backoff and start over.
Once the TMFD has been properly loaded, then it's time to request the maps that are needed. See section of How to request map data.

Once a TileMap buffer is received from the server the TileMap::load method should be called. 
See the [description of how to load a tilemap](../loadingtilemaps).

Draw the maps. See section of drawing the maps.

## How to request map data 

The code that is used to calculate which tilemap parameters that are needed for the currently viewed map is found in TileMapFormatDesc::innerCreateParams.
It uses the bounding box of the viewed map area to calculate these parameters which leads to some problems and strangeness.
We advise that the center coordinate of the map together with the width and height of the screen are used to calculate what be should be included.

The map data to be requested and drawn is defined by:

   * The center coordinate of the map
   * The desired scale level (defined as meters per pixel on a Nokia 6600 screen (s60v2)) 
   * The size of the canvas in pixels.
   * Which layers to draw (map, poi, route, traffic info). And route id in case of the route layer.
   * The language of the text.

Recommended way to generate the tile parameters:
*Note that the following is psuedo code and does not correspond to the actual methods available*

{% highlight cpp %}
for each layer l
   detail = getDetail( l, scale );
   mc2UnitsPerTile = getMc2UnitsPerTile( l, detail );
   float centerLatIdx = centerLat / (float)mc2UnitsPerTile;
   float centerLonIdx = centerLon / (float)mc2UnitsPerTile;
   tileSizeHeight = screenHeightInPixels * scale * meter_to_mc2_factor / mc2UnitsPerTile;
   tileSizeWidth = screenWidthInPixels * scale * meter_to_mc2_factor / mc2UnitsPerTile;
   
   startLatIdx = floor(centerLatIdx - tileSizeHeight/2);
   endLatIdx = floor(centerLatIdx + tileSizeHeight/2);
   startLonIdx = floor(centerLonIdx - tileSizeWidth/2);
   endLonIdx = floor(centerLonIdx + tileSizeWidth/2);

   nbrImportances = getNbrImportances( l, scale, detail );

   // Create the geometry tile parameters. They all use swedish as language.
   for each importance imp
      for ( latIdx = startLatIdx; latIdx <= endLatIdx; ++latIdx )
         for ( lonIdx = startLonIdx; lonIdx <= endLonIdx; ++lonIdx )
            Param p = createParam( l, TileMapTypes::tileMapData, imp, LangTypes::swedish, latIdx, lonIdx, detail, routeID );
            paramVector.push_back( p );

   // Create the text tile parameters. Selected langauge is used.
   for each importance imp
      for ( latIdx = startLatIdx; latIdx <= endLatIdx; ++latIdx )
         for ( lonIdx = startLonIdx; lonIdx <= endLonIdx; ++lonIdx )
            Param p = createParam( l, TileMapTypes::tileMapStrings, imp, language, latIdx, lonIdx, detail, routeID );
            paramVector.push_back( p );
{% endhighlight %}

### Some comments on `TileMapFormatDesc::innerCreateParams` and related classes and methods.

   * The `ParamsNotice` class is used to know if the tilemap parameters has been changed since the last time the map was moved. 
   * If the `ParamNotice` is the same, then the resulting params will also be the same and thus the same old params can be used.
   * `getLayerNbrFromID` translates between the layer id that is used in the parameter and the ordinal number of the layer for requesting.
   * Only include the route layer if a route id is present and valid.
   * The detail level to use is calculated by `TileMapFormatDesc::getDetailLevel()`. 
   * The tile indices i.e. the "coordinates" of the tile squares are calculated by `getTileIndex`. There is code that handles the case when the requested map area covers the international date line (180 degrees longitude) which most likely could be handled in a better way using center coordinate and width/height instead of a bounding box.
   * `iTileImportanceTable::getNbrImportanceNbrs()` calculates the number of importances that exists for the current scale and detail level.
   * Reserve maps are requested in order to always having something to show. These maps are located at relatively zoomed out detail level, indicated by `m_reserveDetailLevel` in TMFD. A grid of the reserve maps on importance level 0 is requested after the maps covering the current screen.

## TileImportanceTable

The `TileImportanceTable` contains information about the different importances that are available. There are different importance tables for different layers.

`TileImportanceTable::getNbrImportanceNbrs()` calculates the number of importances for a specific detail level and scale. This is used to know what to request and draw.

The other use of `TileImportanceTable` is to identify importances containing the same type of information for different detail levels. This should be used to be able to replace missing map data for the current detail level with existing map data at another detail level. For example, consider a zoomed out map which displays a road at
a certain detail level. All needed map data is loaded. When zooming in, it is desired to reuse existing map data to avoid a blank screen before the correct data is downloaded. Importances should be exchanged once enough data at the correct detail level has arrived. `TileImportanceTable::getImportanceNbr()` will return a `TileImportanceNotice`, which can be compared between detail levels. TileMap parameters which yield `TileImportanceNotice` with the same type and threshold when put into `TileMapFormatDesc::getImportanceNbr()` will contain the same type of information, and can therefore replace each other at different detail levels.

## How to draw the map

There are limits for the maximum scale level and latitudes which must not be exceeded.

   * `MAX_SCALE 24000`
   * `TOP_LAT 912909609`
   * `BOTTOM_LAT -912909609`

{% highlight cpp %}
startPass = 0;
if ( skipOutlines )
    startPass = 1;

for ( level = tmfd->getMinLevel(); level `< tmfd->`getMaxLevel(); ++level )
   for ( pass = startPass; pass < 2; ++pass )
      for each tilemap (same order as they are requested in)
         for each feature in tilemap where feature level == level
            // Pass 0 draws the outlines of the roads.
            // Pass 1 draws the inner part of the roads and everything else.
            plotFeature( feature, pass );
{% endhighlight %}


## TileFeature argument types

### `TileArgNames::coords`

Contains multiple MC2 coordinates for the feature. Only makes sense for polylines and polygons.

### `TileArgNames::coord`

Contains a single MC2 coordinate for a feature. Only makes sense for bitmap features.

### `TileArgNames::image_name´

Contains the image name for a bitmap feature. Note that the image should be fetched separately. See section of bitmaps.

### `TileArgNames::color`

Indicates the color (24 bits RGB big endian, top byte empty) of the feature.
This is the fill color for polygons and ordinary color for polylines.

### `TileArgNames::width`

Indicates the width of a polyline feature in pixels.

### `TileArgNames::width_meters`

Indicates the width of a polyline feature in meters.

### `TileArgNames::border_color`

Indicates the border color and the presence of a border of a feature. The width of the border is the calculated width in pixels plus 2 pixels. 
The border color should only be drawn if the following is true:

	#define VALID_TILE_COLOR( a ) (((a & 0x1ffffff) != 0x1ffffff) ? true : false)

### `TileArgNames::max_scale`

Indicates the maximum scale that this feature should be visible at. Currently only present for for bitmap features.
If not present, always draw.

### `TileArgNames::level`

Indicates the level (i.e. z-axis) of the feature.
If no level argument is available, then level 13 is assumed for all features (bitmaps, polylines and polygons).
If level_1 argument also is available, then level indicates the level of the first coordinate of the feature (polyline).

### `TileArgNames::level_1`

Indicates the level (i.e. z-axis) of the last coordinate of the feature (polyline).
If not present, all coordinates are on the level indicated by TileArgNames::level.
This argument is currently ignored by the c++ implementation.

### `TileArgNames::name_type`

Indicates how text should be placed for the feature when drawing the map,
however is currently ignored by the c++ implementation.

### `TileArgNames::radius`

This argument is deprecated.

### `TileArgNames::border_width`

This argument is deprecated.

### `TileArgNames::font_type`

This argument is deprecated.

### `TileArgNames::font_size`

This argument is deprecated.

### `TileArgNames::min_scale`

This argument is deprecated.

## Text placement

It is unfortunately hardcoded in the clients how to place text for the different features. 
The following types of text placement is currently used:

   * Horizontal - Horizontal text below the feature.
   * On line - Text following a polyline.
   * On round rect - Text inside a blue round rect with white border (supposed to look like a swedish road number sign).
   * Inside polygon - Text placed inside a polygon. Does not currently work in the c++ implementation.

The following code indicates which type of text placement that should be done for which types of features:

{% highlight sql %}
switch ( type ) {
   case ( TileFeature::city_centre_2 ):
   case ( TileFeature::city_centre_4 ):
   case ( TileFeature::city_centre_5 ):
   case ( TileFeature::city_centre_7 ):
   case ( TileFeature::city_centre_8 ):
   case ( TileFeature::city_centre_10 ):
   case ( TileFeature::city_centre_11 ):
   case ( TileFeature::city_centre_12 ):
      retval = placeHorizontal(
         nameOfFeature,
         pointsInFeature, type );
   break;
   case ( TileFeature::street_class_0 ):
   case ( TileFeature::street_class_0_level_0 ): {
      bool roundRect = false;
      // Check that at least one char is a digit.
      for ( uint32 i = 0; i `< theString->`length(); ++i ) {
         if ( theString->c_str()[ i ] >= '0' &&
              theString->c_str()[ i ] <= '9' ) {
            roundRect = true;
            break;
         }
      }
      // Cannot be too long string.
      // Otherwise for instance "Vei Merket Mot E18" 
      // gets a roundrect.
      if ( theString->length() > 6 ) {
         roundRect = false;
      }
      if ( roundRect ) {   
         // XXX: Skip E-roads if the scalelevel is too high.
         if ( getPixelScale() > 50.0 ) {
            break;
         } 
         // Place roundrect.
         retval = placeOn_RoundRect( 
            nameOfFeature,
            pointsInFeature, type );
      } else {
         // Place on line.
         retval = placeOn_Line( 
            nameOfFeature,
            pointsInFeature, type,
            m_mapHandler.getPolylineOuterPixelWidth( *prim ) );
      }
   } break;
   case ( TileFeature::street_class_1 ):
   case ( TileFeature::street_class_2 ):
   case ( TileFeature::street_class_3 ):
   case ( TileFeature::street_class_4 ):
   case ( TileFeature::street_class_1_level_0 ):
   case ( TileFeature::street_class_2_level_0 ):
   case ( TileFeature::street_class_3_level_0 ):
   case ( TileFeature::street_class_4_level_0 ): {
      retval = placeOn_Line( 
         nameOfFeature,
         pointsInFeature, type,
         m_mapHandler.getPolylineOuterPixelWidth( *prim ) );
      break;
   } 
   case ( TileFeature::water ):
   case ( TileFeature::park ):
      retval = placeInsidePolygon(
         nameOfFeature,
         pointsInFeature, type );
      break;
   default:
      break;
}
{% endhighlight %}


## TileMapFormatDescCRC

The TileMapFormatDesc itself contains a CRC. However, to avoid unnecessary downloads of TileMapFormatDesc from the server, it's also
possible to directly download the TileMapFormatDescCRC. Then the CRCs can be compared to see if a newer version of the TileMapFormatDesc should be downloaded from the server.


## Geometry and text data

The geometry and text data is divided into squares, henceforth called tiles. Each tile is divided into geometry maps and text maps with different importance numbers. Each geometry map corresponds to a text map with the same parameters except for the language, which is always Swedish in the geometry maps. The number of squares in the world depends on the detail level and layer.

### gzip

Geometry and text maps may be compressed using gzip. If they are, they start with 0x1f followed by 0x8b. The gunzipped data contains the original data.

###  CRC for maps

The geometry map and the text maps contain a CRC. In case the CRC between a text map and the geometry map (for the same tile) differs, then both these maps must be discarded from the cache and requested again.

### Map Parameters

#### Lat and lon indices

Latitude and longitude indices are calculated using the center of the screen, the screen size, the scale factor and information in !TileMapFormatDesc.

#### Layer

The layer, e.g. map, pois, route or traffic information. The map, poi and route layers will always be present in the
´TileMapFormatDesc`. The route layer is special in the sense that a route id will have to be supplied when requesting maps
in order to get the correct route. `TileMapFormatDesc` contains additional information about each layer, e.g. if the client is allowed to cache it forever or only for a while.

#### Detail level

The detail level corresponds to a zoom level. Information about the detail level thresholds are layer dependent and stored
in the `TileMapFormatDesc`.

#### Importance

The importance is used to ensure that the most important features are received before the less important features. If e.g. the
current screen is covered by four tiles, the client fetches the first importance for all of the tiles before the second etc.
The data for the first importance generally contains at least land and oceans. It also contains information about which of the
other importances (up to the maximum defined in `TileMapFormatDesc`) that actually contain data. The number of importances for a tile is layer and detail level dependent.

#### Route ID

The route id must be present for all route tiles, but not any other.

### Bitmaps

The image name of the bitmap will be found in the StringArg `TileArgNames::image_name` for the TileFeature. It should be possible to draw the map also
before all bitmap images has been fetched and loaded.
Bitmaps should be placed with the center of the bitmap at the coordinate sent with the corresponding feature. When collision detection is used only the non-transparent part of the bitmap should be considered. Typically three quarters of the POI images consists of transparency so it's necessary to calculate the
non-transparent part of the image.

## How to create the tilemap parameter strings.

### TileMapFormatDesc
See `TileMapFormatDesc::createParamString()` for how to create the parameter string.
It is possible to supply random characters in the `TileMapFormatDesc::createParamString()` method. This is used to make sure that the TMFD will not be fetched from the local cache. In case this is solved in another way, then these random characters are not necessary. Currently three random characters are used in the C++ version.

In case precached maps are available, these will contain a TileMapFormatDesc with parameter string DYYY. 

### TileMapFormatDescCRC

See TileMapFormatDescCRC::createParamString for how to create the parameter string
The format for the parameter string for TileMapFormatDescCRC is the same as for TileMapFormatDesc except that the initial character is 'C'.
Note that it is not necessary to use the same random string for the TileMapFormatDescCRC as for the TileMapFormatDesc to compare CRC:s with.
The language of the TileMapFormatDescCRC must be the same as the one for the TileMapFormatDesc to compare with.

### TileMaps

Refer to the code in `TileMapParams`.
A `TileMapParams` object is constructed by using the constructor supplying the different parameters to identify the requested tile map.

   * The serverPrefix should be set to the value that is present in TileMapFormatDesc.
   * useGzip should be set to true.
   * The route id must be specified in case it's the route layer.
Once the TileMapParams object is constructed, then the actual parameter string (that's used when requesting the map) will be returned in the getAsString() method, which in turn will call updateParamString(). The updateParamString() method contains two old, obsolete chunks of code that is currently not active by ifdefs. The active code is last in the method (starts by #elif 1). First the different parameters are written into a binary buffer, and then this buffer is converted into a string.
Geometry data starts with the character `G` and string data starts with the character `T`. 
6 bits is read from the binary buffer at a time, and the value (spanning between 0 - 63) is used to index in the c_sortedCodeChars character array. The resulting character is appended to the parameter string. Once the entire binary buffer is handled, any trailing `+` in the constructed parameter string are removed.

### Bitmaps

The bitmap parameter string is constructed by using an initial `b` character and then appending the image name, including the desired image type as suffix. For instance `btat_restaurant.png`.

## General behaviour 

   * Geometry maps should be drawn even if the text maps haven't arrived.
   * The screen should be updated when new maps arrive.
   * If a map request fails, exponential backoff should be used until the map has been moved.
   * Point of interest icons should not collide with each other.
   * Information (name) should be shown only for features actually drawn on the screen.
   * Additional information from the server can only be fetched when the name of a drawn feature has arrived.
   * Memory cache.
   * Write / Read file cache, where buffers received from the server can be written.
   * Read only file cache, which is pregenerated and installed on the phone.
   * It should be possible to use the file cache even if no internet connection exists. This means that a previously saved format description must be used until the checksum can be verifed using the server.
   * Zooming in or out of the map will lead to that the desired detail level of the map changes. Until the requested maps arrive with the correct detaillevel, use map data from an already downloaded detail level with better coverage.
   * Should be possible to download overview maps to be used when zooming out quickly when zoomed in (referred to as reserve maps in the code).

## How to implement getServerString in the API

The server string is used to identify a feature so that it is possible for an outside user to fetch more detailed data about it from the server.
The map component only needs to implement the getServerString method. Lat and lon are in MC2Coordinates. The string is in utf8 as always.

{% highlight cpp %}
	   const bool closed =
	      mapAndFeature.second->getType() == TilePrimitiveFeature::polygon;
	
	   static const int latLonLength = 11*2 + 2;
	   static const int extraLength = 4 + 100; // Colons and more.
	   char* tempString = new char[strlen(name) + latLonLength + extraLength];
	   sprintf(tempString, "C:%ld:%ld:%ld:%s",
	           (long int)coord.lat, (long int)coord.lon, 
	           (long int)closed, name);  
	   // Return and delete the string   
	   return MC2SimpleStringNoCopy( tempString );
{% endhighlight %}
