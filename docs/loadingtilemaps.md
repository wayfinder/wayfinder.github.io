---
title: Loading tilemaps
layout: single
---

The code for loading tilemaps begins at `TileMap::load`.

In case there is any problem with loading a TileMap, the following must be done:

   * Remove the tilemap buffer from the memory cache and any file cache (i.e. all caches).
   * Request a new `TileMapFormatDesc`.
   * And then continue as usual (i.e. request the maps that are needed).
In the C++ code, failure to load (any part of) the tilemap is indicated by returning false in `TileMap::load`.

It is necessary to keep track of the arrival time of the buffers, so that it's possible to know when to fetch updated versions (read traffic info layer).
In the C++ clients, this has been solved by creating a new buffer where the first 2 bytes are used to store the reception time, when receing the buffer from the server. This can be seen in `TileMap::writeReceiveTime()` and `TileMap::readReceiveTime()`. Note that the actual buffer that is received from the server does not contain these 2 extra bytes for the time.

The contents of the buffer may be gzipped, and the intial magic bytes of the buffer are used to determine this (0x1f followed by 0x8b means gzipped data).

The scale and reference coordinate for the TileMap is calculated using `TileMapFormatDesc`.
Depending on the tile map type, the map will either contain geometry or string data.
A description of loading either the geometry or string data is described below.

TileMaps that are not empty (i.e. the geometry map does not contain any features, or the string map corresponds to a geometry map without features) should read the crc from the buffer.
The crc is used to make sure that the pairs of geometry and string maps match.

After this, a 16 bit value called `m_emptyImportances` is read. It contains a bitfield indicating which of the importances that are empty and therefore do not have to be fetched. Example, if the maps with importance nbr 1 and 4 are empty, and the rest contain data, then the bitfield would look like this: `0000000000010010b` (set bits indicate an empty importance). Note that importance number 0 must always be fetched in order to know which importances that are empty. This first importance will also contain data since it will contain either water or land features. *Correction, this is not always true!* In case a builtup area totally covers the land area, then importance 0 will not contain any data, since the builtup area will come in a later importance and be drawn on top of the land (if included) anyway.

### Loading geometry maps

The number of features are read, and then each feature is created using `TileFeature::createFromStream()`. See `TilePrimitiveFeature::createFromStream()`:
The first bit indicates if the feature type has differed from the previous one.
If it differs, the actual type is read.

`TileMapFormatDesc::getArgsForFeatureType()` is used to fetch prototypes for the transferred `TileFeatureArgs`, using the `m_argsByTileFeatureTypeArray` as described in `TileMapFormatDesc` section.
Once the `TileFeature` has been set up with the transferred `TileFeatureArgs`, then `TilePrimitiveFeature::internalLoad()` is called, which loads each of these arguments using `TileFeatureArg::load()`.

In case the previous `TileFeature` in the buffer was of the same type as the one currently loading, the previous feature's argument will be supplied in `TileFeatureArg::load()` (see the c++ code for `TilePrimitiveFeature::internalLoad()` for how this is done).

The argument to be loaded can either be a `SimpleArg`, `StringArg`, `CoordArg` or `CoordsArg`.

#### `SimpleArg::load()`

In case the buffer indicates that the argument is the same as the previous one, then the values from the previous argument are used.
The argument can either contain a single value or a number of different values, different for different scale levels. The argument is loaded from the buffer according to the code in `SimpleArg::load()`.


#### `StringArg::load()`

Here the previous argument is not used. In the same way as for `SimpleArg`, either a single value or a number of scale dependent values are loaded from the buffer.

#### `CoordArg::load()`

The previous argument is not used here either. The latitude and longitude are read as relative values to the reference coordinate of the map and the real coordinate is calculated as according to the code in `CoordArg::load()`.

#### `CoordsArg::load()`

The loading of the `CoordsArg` is more complicated than for the rest.
In case no previous argument is supplied, then the TileMap's reference coord is used as reference coordinate for the argument.
Otherwise the last coordinate of the previous argument is used as reference coord.
The buffer contains relative coordinates which must be converted into real coordinates according to the code. The C++ code keeps all coordinates for a TileMap in a big vector (`m_allCoords`), but another way of storing the coordinates can be used in the java implementation. Note that a bounding box is stored for each `CoordsArg` in the C++ implementation, but this requires quite a lot of memory and is most likely not a good idea for the java implementation. (Actually it's not a good idea for the c++ implementation either.)


The features that have been loaded into the TileMap at this stage only contains the transferred `TileFeatureArgs`. However, it's also needed to add the default arguments
from `TileMapFormatDesc` for each feature. In the C++ code, the features are divided into features and primitives to be able to keep track of what to delete. There are also indeces referring back and forth between the `TileFeaturePrimitive` and its `TileFeature`. This is not necessary for java, and therefore all the default arguments should be added to the features as well. If the `TileFeature` and `TilePrimitiveFeatures` are merged, then both their feature types must be stored.

The code that creates the primitive features in c++ is present in `TileMap::createPrimitives()`.
It creates all the primitive features and sorts them in level order. The level corresponds
to the drawing order (z-level) of the features. Lowest level means that the feature should be drawn first. In case no explicit level argument is present for a feature, then the default level 13 is used instead.

The method `TileMapFormatDesc::getFeaturePrimitivesDefault()` extracts the primitive feature for the specified `TileFeature` type using the `m_primitiveDefaultMap` mapping. The arguments of this primitive feature should be combined with the arguments of the already loaded `TileFeature`.

### Loading string maps

Comments to the code in `TileMap::load()` (string map loading part)

The feature index and string index is read for all features containing strings.
The feature index is referring to the index of the feature in the geometry map.
These pairs of indeces are sorted in text placement order, i.e. in the order that the text placement algorithm should try to place text.

Then each and one of the strings are read from the buffer. The string index mentioned above is referring to position of the strings when they are read from the buffer.
`m_strIdxByFeatureIdx` is used to map between feature index to string index.
`m_featureIdxInTextOrder` keeps the feature indeces in text placement order.
`m_strings` keeps the strings in string index order.

