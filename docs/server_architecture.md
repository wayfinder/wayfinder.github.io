---
title: MC2 High Level Architecture
layout: single
---

### System Overview

User have a client connected to the internet handled by an operator and requests are sent to the distributed and redundant MC2 server. Interface is called and the frontend handles the request and uses the backend to process the data before returning an answer.

![mc2 overview system](/images/mc2_overview_system.png)

### Scalability and redundancy

Since it is possible to have multiple modules, running on multiple computers, the system is both scalable and redundant. When the load increases the system administrator can start new instances of any module, thereby removing any bottlenecks detected.

### Module grouping

In a running Wayfinder Server system each type of module has one of the running modules as leader and the others are availables. The leader is the only one that listens on the leader multicast address that the frontends initially talks to.
The leader gets information from all the availables about their load and loaded resources, which maps it has, and can
direct frontends to a suitable available.

#### Voting

Voting is the way for a multiple running processes, of a specific type of module, 
to come together and select one of the running processes as a leader.

##### Introduction

This section covers the voting mechanism used to make sure that there is
exactly one leader among the modules of a certain type.

##### Situations where voting is initiated

Voting starts if one (or more) of the following statements are true.


*  An *available* module does not receive a `HeartBeatPacket` in a certain amount of time. This usually means that there is no leader on the same net.
*  A *leader* receives a `HeartBeatPacket` from another *leader* and both *leaders* know about more than one *availables*.
*  A *leader* receives a `HeartBeatPacket` from another %leader% and both *leaders* know about zero *availables*.

No voting will occur if a *leader* with more than one
*available* receives a `HeartBeatPacket` from a
*leader* with zero *availables*. The *leader* with zero
*availables* will turn into an *available* when it gets the other
module's `HeartBeatPacket`.

##### The voting procedure

When voting is initated the module initiating the voting will become
*available* and send `VotePacket` packets to the other modules
using the multicast addresses for *leaders* and
*availables*. The modules receiving the packets will turn into
*availables* and compare the `*VotePacket` to its own data,
see *Comparing VotePackets* below. If it finds itself better than
the module sending the first *VotePacket* it replies with its own
`VotePacket` or else it ignores the packet. No replies should
be received by the best module, since the other modules shouldn't
reply to better packets. The voting is finished when the best module
doesn't receive any packets after resending its `VotePacket` a
few times. This module will then become a *leader*.

##### Comparing `VotePacket` packets

The `VotePacket` packets are compared by

1.  Rank
2.  IP-number
3.  *Available* listen port

where `rank` is a value given to the module by the
operator. Better rank always wins. If the ranks are the same the IPs
will be compared and if they are the same too, the ports will be
compared. This way two modules will always have  a different
`VotePacket`.


### Module Load Sharing System

This is a brief description of the load sharing mechanism in the modules. There can be many modules of a specific type running at the same time. This means that a `RequestPacket` can be handled by any one of several modules. To share the work load in an efficient manner, the `RequestPacket` is first delivered to a leader module. The leader module then decides which of the other available modules that should do the job. It is also possible for the leader to process the `RequestPacket` by itself. The `RequestPacket` is first sent via multicast to one IP address that only the leader listens to. If the leader decides that the packet should be sent to one of the available modules, it sends the packet to the IP address of that module. Note that packets for the leader are sent via multicast, but the only module listening to this is the leader, so it works fine.
In order for the modules to be able to have a leader, and for the leader to be able to decide who should take care of an incoming `RequestPacket`, the modules also have a few administrative routines. At regular intervals, the leader multicasts heartbeat packets to the available modules. When an available module receives a heartbeat packet, it immediately multicasts a statistics packet to the leader, who then uses these statistics to decide which modules are available to handle an incoming `RequestPacket`. If the leader module dies, the available modules will detect this, because there will be no heartbeat reaching them for a while. When an available module detects there is no leader module, it starts sending packets saying it wants to be the new leader. The modules now decide by vote who should be the new leader module. The one that become the leader starts sending heartbeats (to the leader-multicast address), and listens to the multicast address for an incoming `RequestPacket`.

### Layer 4 Load Balancer

In addition to the load balancing inside the cluster (between the Modules that carry out the actual tasks) IP load balancing between the interfaces in done by the operating system (Linux Virtual Server). This ensures that requests from the clients are distributed between the Interfaces and that a malfunctioning Interface is disconnected (even if this is very unlikely). There is no dedicated computer for the IP load balancing but rather some of the machines in the clusters handle this.


## Overview Server

![mc2 overview server](/images/mc2_overview_server.png)

### User Request Examples

Suppose the user sends a request for a route from A to B. The server then creates one Request to handle it. A search module (SM) takes the name of a place (mainly street name, company name or point of interest) and returns its ID number. A route module (RM) takes the IDs of two places and returns a route between them. The Request can therefore have its question answered by first sending two `RequestPacket` packets to SMs (there can be more than one SM, so this operation can be performed almost in parallel). Suppose A has ID 17 and B has ID 43. When both IDs have been received by the Request, it asks the RM for a route from 17 to 43. When the Request receives this route, it has all the information it needs. The answer is sent to the user, and the necessary debiting information is send to the module responsible for logging this information and the Request is destroyed. 
A slightly different situation occurs when the user wants a map of a certain area. Let us assume that the user requests a map over an area enclosed by the rectangle with corners in (x1; y1) and (x2; y2). As before, when the server receives a user request, a Request is created. The Request sends a `RequestPacket` with the coordinates to a map module (MM). There is a lot of information in an entire map, so to send many datagrams to the Request, would be inefficient. Instead, the MM returns a port number and its IP address. The server then starts a TCP/IP connection with the MM, and glues it to the TCP/IP connection already established between the server and the user. That way, the MM can send all the map data directly to the user, which is a lot more efficient. When all data has been sent, the TCP/IP line is disconnected, and the Request is deleted.

### Frontends

A collection of applications that acts like a server from the perspective of clients of the Wayfinder Server. Main task is to parse and translate the incoming request, ask the modules for the answer, format the reply and send that back to the caller.\\
The frontends are named servers in the source tree and are also called interfaces since they connect the clients to the server.

#### Navigation Frontend

Interface to clients with a proprietary protocol that uses little memory, cpu power, and bandwidth.

#### XML Frontend

Handles all different kind of requests in XML-format. It is possible to access most of the MC2 functionality via the generic XML API.

#### Traffic Frontend

Processes incoming traffic information and sends it to *InfoModule* for storage.

### Backends

This class of applications perform the actual calculations and look-ups for each request. Acts like a server from the frontends perspective. The backends are called modules internally in MC2.

#### Communication Module

Sends information to external servers. This information can be location updates to another system. \\
See External connections, PosPush, section in [Server functions](../server_functions).

#### E-mail Module

Handles the sending of e-mails and MMS messages.

#### External Services Module

This module performs searches in remote databases, e.g. white page databases located at the information provider. Designed to make it easy to add new protocols, currently supports XML based and a few proprietary formats. \\
See [Server functions External Search](../server_functions#external_search).

#### Gfx Module

Draws map images (PNG, GIF, WBMP).

##### GfxFeatureMap

A `GfxFeatureMap` is a map over an area with a number of `GfxFeatures` in. `GfxFeature` maps are created in the MapModule and used by the GfxModule and TileModule to make maps.
A `GfxFeature` represents a thing on a map such as street, builtup area, ferry, island etc. A `GfxFeature`
has a number of `GfxPolygons` describing the extent of the `GfxFeature`.

*  `GfxRoadFeature` Represents a part of a route.
*  `GfxPOIFeature´ Represents a Point Of Interest to show on map.
*  `GfxSymbolFeature` Represents a generic symbol to show on map. May be user defined image and name.
*  `GfxTrafficInfoFeature` Represents traffic information (such as an accident, road work etc.).


#### Info Module

Collects and stores information about dynamic changes in traffic and distributes it to the users who need it.

#### Map Module

Stores and handles the map. This includes sending it to the servers and other modules that needs it.

#### Route Module

Performs the route calculations.\\
See [The routing algoritm](../server_routing_algorithm) for details on how routing is made.

#### SMS Module

Sends and receives SMS messages via an SMSC or other SMS service.

#### Search Module

Searches for locations expressed as strings, for example street names, company names and business categories.\\
See section [Search](server_functions#search).

#### Tile Map Module

Creates the proprietary vector map format that is used to stream maps to the clients.\\
The Tile maps starts their creation in `GfxTileFeatureMapProcessor` in the MapModule as a `GfxFeatureMap` that is sent,
by a frontend, to TileModule and converted to a Tile map.

#### User Module

Manages all the (end) users of the system, including their billing and user profiles.

### Entities outside of the Server

#### Maps

This is all logical and geographical data that we have, describing the world. Examples of information are streets, logical connections between streets, built-up areas, Subject to change without notice municipals, point-of-interests and lakes. The maps are stored in a proprietary format to maximize the performance.

#### User DB (RDBM)

A generic relational database is used to store information about the users of the system, including the user profile, transactions and active subscriptions.

### Server to Module communication

The servers communicate with the Modules using packets. The packets consist of binary data
that can be sent between computers and programs and consist of binary data that can be read and
written using the class `Packet` and its subclasses which will be described in this section.

#### The `Packet` class

The data that is to be sent between the different parts of the MC2-server is usually not in the form
of a byte array, which it must be when sending it using network protocols.

The class `Packet` contains helper functions for reading and writing data to a byte array that
is independent of the byte ordering of the sender and recipient computers. `Packet` uses the bigendian
byte ordering which is the same as network order. Network byte ordering was chosen because
most operating systems have (optimized) functions (`ntohl`, `htonl`, etc.) to convert from the native byte
order to network order since these functions are needed when using TCP/IP. The values stored in the
packets are aligned to the same offset as the size of the value, i.e. 16-bit values are stored on even
addresses and 32-bit values are stored on addresses that are evenly dividable by 4.

The `Packet` class has the following data members:

*  `buffer` Byte array containing the actual data.
*  `bufSize` The number of allocated bytes in the buffer.
*  `length` The number of bytes actually used in the buffer.  The buffer of the `Packet` always starts with a header containing information about who sent
the packet, type of packet etc. It is documented in `Packet.h`.

#### Subclasses of Packet

Objects of the class `Packet` are only used when sending and receiving data from the network. 
When information is written to or read from a `Packet` a subclass with more suitable
functions is used. When a `Packet` is read from the network it is cast to the right kind
of packet by looking at the type field in the packet header. For this reason, subclasses to `Packet`
cannot contain any data members of their own. Most of the packets in the MC2 system should inherit
from one of `RequestPacket` and `ReplyPacket`. These contain extra header information for packets sent 
from servers to modules and packets returned from modules to servers respectively.


