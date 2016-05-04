---
title: Wayfinder Documentation
layout: single
---

![Vodafone Wayfinder Open Source Software Logo](/images/ps_application_full.png)

Here you will find the technical documentation of the Wayfinder
software. The aim is to cover both operational information such as running the
servers and compile new maps as well as further development of the software.

## The Big Picture

The Wayfinder software is a client-server based system for navigation and
different related location-based-services, including mapping and searching for
places and addresses. The server is designed to be highly scalable and has a
distributed architecture and runs on Linux CentOS operating system. Several
clients, supporting a wide range of mobile phone platforms, are available and
includes navigation clients for Symbian S60, iPhone and Android. Support for
importing Open Street Map data is available.

[Getting Started Guide](getting_started)  
 How to build and run both the server and the clients

## Documentation 

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
 *  [Nav2 Architecture](nav2_architecture)
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

## Access and contributions

The latest version of all the source code is available in a few different repositories at GitHub, as public projects. In order to be able to contribute your changes back to the code base, just get in contact with the administrators.


## Open Source License

The source code is licensed under the BSD license, see below. Even if contributing back your improvements is not required by this license, we encourage this to make the code base evolve and develop. 

	
	Copyright (c) 1999 - 2010, Vodafone Group Services Ltd
	All rights reserved.
	
	Redistribution and use in source and binary forms, with or without modification,
	are permitted provided that the following conditions are met:
	

	    * Redistributions of source code must retain the above copyright notice, 
	      this list of conditions and the following disclaimer.

	    * Redistributions in binary form must reproduce the above copyright notice,
	      this list of conditions and the following disclaimer in the documentation
	      and/or other materials provided with the distribution.

	    * Neither the name of the Vodafone Group Services Ltd nor the names of its 
	      contributors may be used to endorse or promote products derived from this
	      software without specific prior written permission.
	
	THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
	ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED 
	WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE 
	DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR
	ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES 
	(INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; 
	LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON 
	ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT 
	(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS 
	SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.



