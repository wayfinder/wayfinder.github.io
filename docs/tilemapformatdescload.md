---
title: Loading TileMapFormatDesc
layout: single
---

Parts of the code in load may seem strange. For instance the category strings
are updated twice. This is due to compability reasons, 
i.e. the code needs to work both for old (cached) and new TileMapFormatDescs.
This is also the reason for the many checks if the buffer still contains enough
bytes before reading.

`m_serverPrefix` is used when creating tilemap params.

For each layer a bunch of different parameters is loaded:
The parameters that is supplied into the `initTileSizesForLayer` method 
describes the grid of maps and how to request maps covering the screen.
Each layer contains a layer ID. There is a mapping between the ordinal number
and the ID that is stored in `m_layerNbrByID`. 
`m_layerIDsAndDescForComp` is first loaded with the layer ID and text description
of the layer, but at the end of the load method some additional information is also loaded into this data structure.

The `TileImportanceTable` describes which importances that are present on the different scalelevels.

The `m_argsByTileFeatureTypeArray` array contains the transferred `TileFeatureArg`s
indexed by TileFeature type. I.e. these are the args that are transferred
for each feature. For instance the coordinates of a street.

There are four types of TileFeatureArgs: 
    * `simpleArg` (int)
    * `coordArg` (single coordinate)
    * `coordsArg` (many coordinates)
    * `stringArg` (string).
Simple and string args can contain scale specific values.
Example: The color of a road will be different for different scale levels.
The method `TileMapFormatDesc::getScaleIndexFromScale()` converts the actual
scale of the map to which index to use in the TileFeatureArgs.

The `m_defaultArgs` contains the default TileFeatureArgs, i.e. these are the args that are not transferred for each feature, but instead only once in the TMFD. For instance the colors that the streets should be drawn with.

Next up is loading the `m_primitiveDefaultMap`.

Each `TileFeature` is associated with one `TilePrimitiveFeature`. The protocol in TileMapFormatDesc actually allows that a `TileFeature` is associated with more than
one primitive, but it can be assumed that there is always only one primitive for each feature. This should mean that it's possible to make some optimizations in the object hierarchy that is not currently present in the C++ code.

The `TileFeature` contains a feature type which indicates the type of feature in the map that it's representing, for instance a street, park or restaurant.
The associated TilePrimtive feature also has a feature type, but this refers
to how the feature should be drawn, e.g. polyline, polygon, bitmap or circle. Note that circle primitives are deprecated and will never be sent from the server so there is no need to implement any support for this.

For each `TileFeature`, the feature type is read and then comes information about the associated `TilePrimitiveFeature` (primitive type + index to the default arguments).
Prototypes of the `TilePrimitives` including the default (i.e. present in TMFD) arguments are stored for each `TileFeature` type.

The loading of `m_argsPerTypeArray` that comes next does actually not contain any interesting information for a java implementation of the vector maps. However, it's of course necessary to read past this data in the data stream.
It contains information about which default parameters that should be copied to which primitive prototype for each feature type. However, it will not be necessary in java to separate the features and primitives. So therefore there will be no need to use this mapping.

The background color of the map (and all other colors) are loaded as 32 bits, but the most significant byte is unused and then follows red, green, blue.

The first released version of the TMFD ended after the background color. However, since we after a while needed to add more things into TMFD, but still needed the client to work with old cached TMFDs, the code checks that there are enough bytes left in the buffer before reading anything more. The java version can assume that all these `buf.getNbrBytesLeft()` will return true, i.e. that there is more data to read. Note however that once the java vector maps are released, and more data is added into the TMFD buffer, it will be necessary to check that the newly added data is available in the buffer before reading in order to support old cached TMFD:s.

The reserve detail level and number of extra tiles for reserve is used when requesting an overview map of the actual position (i.e. parts of Europe if the map is located in Stockholm). The detail level to use is stored in `m_reserveDetailLevel`, and `m_extraTilesForReserve` indicates the number of tiles that should be used to form the map grid (`m_extraTilesForReserve` x `m_extraTilesForReserve` maps).

The POI categories are read once with latin-1 character encoding and then once again as utf8. The utf8 encoding should replace the latin-1 version.
One or several POI types can be grouped into a POI category.
Each category contains category name, id and whether the POI features of the category should be visible on the map. It should be possible to enable or disable the visibility of these categories in the settings of the application.

At last some more detailed information about each layer is loaded, `TileMapLayerInfo::load`
Layer ID and name are read once again (replacing the old version).
`m_updatePeriod` refers to how many minutes that a map in the layer is valid until it must be re-requested from the server. The update period will be set to 0 if there is no need for the map to be updated from the server. Typically the traffic information layer will require periodic updates from the server, but not the other layers. `m_transient` refers to if maps of the layer is to be cached on disk or not. `m_isOptional` reflects if the layer is optional or not, i.e. if it should be possible for the user to select in the settings to disable the layer.
`m_serverOverride` is a number that is used to know if the new `TileMapLayerInfo` from the server should override the current settings in the client. See the code in `TileMapLayerInfoVector::updateLayer()` for how this works. If the `m_serverOverride` in the `TileMapLayerInfo` from the server differs from the current one, then the version from the server should override certain settings in the client, and update the `m_serverOverride` member in the client version, so that it can only happen on each occasion the number is manually changed on the server.
If `m_presentInTileMapFormatDesc` is false then the layer should be deactivated, i.e. it should not be possible to show the layer in the map. This functionality is used to remove an existing layer. `m_visible` is used to know if the layer should be visible on the map. `m_alwaysFetchStrings` indicates if the string maps should be requested right after geometry maps have been requested, or if they should only be requested if the user has actually clicked on a feature in the map. For instance, it is not necessary to fetch the strings for the POI layer unless the user actually clicks on the POI, since we don't draw the names of the POIs directly on the map. It's an optimization in order to reduce data traffic, e.g. when tracking.
