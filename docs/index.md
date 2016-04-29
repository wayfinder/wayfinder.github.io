---
title: Wayfinder Documentation
layout: single
---

[Getting Started Guide](getting_started)  
 How to build and run both the server and the clients

### Server 
 
* [Server architecture](server_architecture)  
 High level architecture and design of the server software MC2
* [Server functions](server_functions)  
 The various functions included in the MC2 server software
* [Server Routing Algorithm](server_routing_algorithm)  
 Description of the routing algorithm
* [XML specification (pdf)](/downloads/xml.pdf)
* [Operations manual](operations_manual)  
 Describes how to setup, configure and run MC2 in development, testing and production environments
 

### Java client

* [Java architecture](java_architecture)  
 High level design of the Java clients
* [Java architecture](java_architecture_interaction)  
 Core Interaction Decisions
 
 
### C++ client 

 *  [C++ clients, overview](cpp_clients_overview)
 *  [Nav2 UI Protocol](nav2_ui_protocol)
 *  [C++ architecture, Core v.2 API](core_v2_api)
 *  [C++ architecture, Core v.2 API - Get started](core_v2_api_get_started)
 *  [C++ architecture, Core v.3 - High level design](core_v3_design)


### Tile Map

 *  [Tile Map Format](tilemap_format)
 *  [Loading tile maps](loadingtilemaps)
 *  [Loading TileMapFormatDesc](tilemapformatdescload)

### Map Conversion

* [Generate maps for the MC2 back-end](generate_maps_for_the_mc2_back-end)  
 Step-by-step how to generate maps for the MC2 back-end from midmif format, with examples from OpenStreetMap and Tele Atlas MultiNet shape data
* [Wayfinder maps](wayfinder_maps)  
 Technical documentation of how the maps are structured, the map items, search index, etc including references to mc2 classes and functions. Also includes one section with some requirements on map data which must be met with for good support of location based services.
* [The WASP database](the_wasp_database)  
 The database for administrating Points of interest and map correction records. Lists the table creation statements and content of static tables so you can set up your WASP database. Also includes technical documentation of import and export of POIs and map corrections.
* [Wayfinder midmif format](wayfinder_midmif_format)  
 Describing the Wayfinder midmif format, which is used for generating maps for the MC2 back-end. Includes a small sample data to illustrate the format.
* [Wayfinder midmif specification](wayfinder_midmif_specification)  
 The specification for the Wayfinder midmif format
* [Wayfinder POI import formats](wayfinder_poi_import_formats)  
 Describes the formats that can be used when importing Points of interest data into the WASP database
* [Other map related documentations](other_map_related_documentations)  
 Misc relevant documentation, such as country polygons, country borders, filtering in map generation and the MapEditor tool.

### Tools
 
 *  [Tools](tools_technical)
