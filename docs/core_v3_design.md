---
title: Core V.3 - High level design
layout: single
---

This document describes the design of the new Core, version 3. A high level
design has been made and the implementation has started but has not been
finished.

### The big picture

The idea behind Core V.3 is to decrease the complexity of the Core and increase
maintainability and to make it more easy to add new functionality. Core V.2 is
today a highly complex software where it can take up to a week for a developer
to successfully add new functionality that includes communication from the API,
through Core, to the Server and then back. With Core V.3 this should be
straight forward and only take less than a day, depending on the extent of the
functionality to add of course. The goal of decreasing complexity would be
reached by decreasing the amount of threads that Core are using, from around
13, down to a few threads. Also to avoid sending serialized messages between
the different modules and instead communicate in a more direct way between the
modules would help to decrease the complexity.

The image below shows the high level blocks of the Core V.3 design.

![cv3overview](/images/cv3overview.jpg)

As you can see in the picture, Core V.3 consists of six parts, the application
UI excluded.

#### PAL

The lowest part of Core V.3 is **PAL**, the _Platform Abstraction Layer_. It
consists of all platform dependent code that is needed for Core V.3 to
function. For example threads and GPS communication will be handled here. The
rest of the Core V.3 will communicate with PAL using defined interfaces in PAL.

#### Server Communication

The block contains all functionality for handling the communication with the
servers, creating the messages to send and parsing replies in a way that Core
V.3 can understand. It also uses the network functionality in PAL to handle the
physical connection.

#### Map

This would be the Map used in Core V.2 but accommodated to fit in Core V.3 of
course, using the new PAL.

#### CV3

The heart of the Core V.3 library, containing the functionality for handling
routing, searching, positioning and so on. The basic functionality would be the
same as in Core V.2, but, as mentioned above, with decreased complexity making
it more easy to add new modules and functionality. The number of threads would
be reduced to one in the normal scenario, but be able to run heavy tasks in
separate threads.

#### MapCV3Gateway

The idea behind this block is that it should work like a gateway between the
map component and CV3, instead of communicating directly into each other. The
gateway can be used to send route data from CV3 to the map and the information
can be used to interact with the map. For example used to draw turn images in
the map using the route data. All information sharing between CV3 and the map
shall go through this gateway. Also this entity will probably have more
knowledge about the map and CV3 than the API has.

#### CV3 API

The API should contain the same functionality as Core V.2 LBS API does, and in
the long run new functionality will be added, of course. The interfaces used by
applications would, as much as possible, be similar to Core V.2 LBS API, but
simplified if possible. The implementation of the API itself would be
simplified and easier to maintain compared to Core V.2 API implementation, this
would be a natural step due to the improvements in Core V.3.

### It is all in the details

This chapter will explain how Core V.3 are supposed to work in a more detailed
way.

#### CV3

As mentioned above, CV3 is the heart of the Core V.3 library, containing the
logic for the core functionality, as navigation, searching and positioning. It
includes the modules shown in the picture below.

![cv3internalcv3](/images/cv3internalcv3.jpg)

* `CV3MainModule` Contains the main functionality for CV3, start up, close down etc.
* `MessageRouter` Handling how messages are sent within CV3, making sure the correct receiver receives the message the sender has sent. Handles messages sent within CV3 as well as messages sent from the API and to and from the server.
* `Authorization` Handles authorization of the user, making sure the user is entitled to use the services at all, and also differs between authorization levels so the user only can do what he/she has permission to.
* `SessionSettings` Handles the settings for the user, this can be both settings set by the user and also settings used by CV3. Also includes persistent storage of the settings.
* `Positioning` Handles the positioning of the user, using the GPS communication implemented in PAL.
* `Routing` Handles the routing functionality in CV3, contains functions for requesting routes and route information etc.
* `Favourites` Handles the user's favourite destinations, syncing, adding and so on.
* `Search` Handles searching.

When implementation of Core V.3 is done it will probably contain more modules.

##### Module communication

The idea is that the modules should work rather independent from each other,
this means that they should not call directly into each others public
functions. This way it will be possible to choose not to include some modules
in a specific build if they are not needed. Still, some modules could have the
need to retrieve information from an other modules. E.g. the routing module
could have the need of retrieving position updates from the
positioning/location module obviously.

This is solved by using predefined interfaces. The image below shows the class
diagram over how the communication should be handled:

![cv3modulecommunication](/images/cv3modulecommunication.jpg)

A module that has data to share should implement the `ModuleObserverInterface`.
Through this interface are other modules able to register them self as
observers. This is done by retrieving the `ModuleObserverInterface` for the
module to ask for data from the `ModuleObserverInterfaceProvider` and then
register as an observer. The `ModuleObserverInterfaceProvider` will contain all
module specific interfaces that are available.

The module that has data to share should also provide an interface for
listening modules to implement. This interface are then used by the module to
share the data when needed. E.g. the location module provides an interface
called LocationModuleObserver that the search module implements to listen for
position updates.

![cv3modulecommunicationsequence](/images/cv3modulecommunicationsequence.jpg)
##### Message Router

The message router module is used by the other modules to send and receive data
from the server communication block or the API. It consists of a
`MessageRouter` and the `MessageDispatcher` and also a number of interfaces
used by other modules to receive and send messages. 

The `MessageDispatcher` is used by modules to send messages using three
interfaces, the `NGPMessageSender` (sends NGP messages to the server),
`APIMessageSender` (sends messages to the API) and `InternalMessageSender`
(sends internal CV3 messages).

The `MessageRouter` receives messages addressed to CV3 and make sure the
correct module gets the message. The `MessageRouter` is not known to anyone
outside of CV3, instead are interfaces used for this purpose. For example
`ServerHandler` only knows about the `NGPMessageReceiver` which it uses to
transport incoming messages from the server to the MessageRouter.

![cv3messagedispatcher-servercom-messagerouter-sequence](/images/cv3messagedispatche-servercom-messagerouter-sequence.jpg)

#### Server Communication

Since Core V.3 is part of a client-server solution it is dependent of
retrieving data from the server. A independent block in Core V.3 handles the
server communication. It is independent from CV3 due to the Map block is also
in need for a server connection in order to download map data. By being
independent from CV3 it is not needed that CV3 is running for the Map component
to download data.

The `ServerHandler` is the center of the server communication. It receives
messages from CV3 and the map, through the `NGPMessageSender` interface, and
sends the messages to the server. When the reply is received from the server
the `ServerHandler` make sure the correct recipient receives the reply. The
`ServerHandler` uses `ConnectionManager` for access to a proper network
connection.

##### Connection Manager

The `ConnectionManager` is the entity that handles all network connections, not
just the connections to the server providing map data, route data and search
data. It can also be used for handle connections to third party data providers.
It simply handles the connection to a provided server. Further more the
`ConnectionManager` differentiates between different kind of bearers, e.g WiFi
and PDP, but as standard it gives you the best network bearer available.

![cv3cm_model](/images/cv3cm_model.jpg)

As you can see in the image above `ConnectionManager` uses network
implementations in PAL to handle the connections since that most platform has
its own network implementations.

#### Platform Abstraction Layer, PAL

As mentioned before the PAL consists of unified interfaces for platform
dependent implementations. This way CV3, the map component and the server
communication does not need to bother about different platform limitations and
instead just use the platform independent interfaces. PAL today exists of:

* `PALLocation` Handles GPS communication.
* `PALThreading` Handles threads on different platforms. POSIX is used if possible.
* `PALGraphics` Provides an interface for graphic operations.
* `PALTimeUtils` Handles operations for time functions.
* `PALMachineTypes` Handles machine specific types.
* `PALNetwork`, `PALConnection`,`PALBearer`, `PALSocket` and `PALNetworkInfo` Handles network specific implemenations.

#### Core V.3 LBS API

The basic idea is that Core V.3 shall be implemented in a way that makes it
possible to implement the API in not just C++, but also in C and JNI. For start
the old LBS API used in Core V.2 should be reused and fitted into Core V.3,
that will make it possible for old clients built on Core V.2 LBS API to easily
adapt to Core V.3 instead. By implementing a JNI API the Core V.3 should work
on Android for example.
