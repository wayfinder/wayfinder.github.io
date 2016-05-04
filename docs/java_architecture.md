---
title: Java High level architecture
layout: single
---

![java_domain_model](/images/java_domain_model.png)


## Platform Abstraction Layer (PAL)

![java_pal](/images/java_pal.png)

### Description


Since the Core contains the majority of logic and non-UI decision making, it is
inevitable that it will need to send and receive instructions to/from the
platform it's currently running on.  An application only intended to run on a
single platform would do this by accessing external APIs (either made by a
specific manufacturer or based on a global convention, such as JSRs).

Since the Core is intended to be used on several platforms, this package acts
as a bridge between the platform and the Core. This means that the Core will
never access any platform specific API directly and will always use the
interfaces for all tasks. In addition the code within this package is to be
kept separate from the Core source code to prevent any contamination. The Core
itself must be kept in a state where it can be compiled on it's own with ONLY
references to the platform interfaces (but no platform specific
implementation).

In a way this package can be seen as a platform in itself. One important thing
to note is that any interface towards a platform specific thing may not provide
a full implementation of everything the platform actually offers. The reason
for this is that the platform interfaces will only offer functionality that the
Core actually requires. If more functionality is needed in the platform
interfaces it will have to be added accordingly.

When adding or changing the platform interfaces it absolutely crucial that the
abstraction layer is placed on the correct level and not based solely upon how
one platform works. This means that any addition to the platform layer must be
properly reviewed and approved before it is implemented. Failure in this area
means that one or more platforms will fail to properly provide the implemented
service.

### Contents


The PAL package contains two things:
 1.  Interfaces that represent all the actions the core needs to invoke on the platform + any information the core needs to obtain from the platforms.
 2.  Platform specific implementations of the interfaces. These specific implementations must be kept as separate as possible due to the differences between platforms.

### Examples of components

*  Services network requests (high-level requests)
*  Services requests for storage of data (high-level requests)
*  Plays sounds
*  Provides information on current network (low-level) 
*  Reads HardwareKeys (high-level, not possible to do low level due to differences in available keys)
*  Handles GPS interfacing
*  Provides service for logging (replaces System.out/err)

## Core

![java_core](/images/java_core.png)

### Description

This package contains all logic affiliated with core functionality of the Wayfinder classes. It acts as a middle ground between the Platform (which would for example talk to the server) and the UI (which would display all info to the user).

When adding new functionality, the goal should always be to add as much as possible of the logic to the Core while at the same time take care not to corrupt the Core with platform or UI specific code.

The Core itself is divided into several Modules which in turn offer specific functionality. It's the responsibility of the application specific product to assemble those Modules required to obtain the functionality required by the product. This will be done in the Environment.

### Examples of components

*  PowerSearch logic
*  Route and navigation logic
*  Handling of sound syntax and sound batch compiling
*  User data
*  Favorites
*  XML-encoding part of networking (transaction wrapper in network module, handling of requests and replies per module)
*  Handling of settings, per module
*  Handling of persistent data (handling only, actual storage handled by Platform)

### Core vs the platform abstraction layer

The Core must never contain any platform specific code. As such it's also not possible to use any of the standard SDKs available today. Instead a new SDK for usage internally in the core will be created which contains the classes and methods that are common for all current classes. Any class or method that either does not exist in exactly the same shape for all platforms or that poses a problem on a platform will be offloaded to the PAL part.

Any access outside of these packages MUST be done via the platform package since those classes are not available on all platforms.

### Core vs the UI

The Core will never know about, or care about, any UI-specific classes. Furthermore the core may never make any assumptions on how any piece of information will be displayed to the user.

All notifications from the Core towards the UI are solely passed as constants, immutable objects or enumerators with event-specific objects etc attached. As such, the Core will never contain any Strings or images actually displayed to the end user, unless those provided by the MC2 server (POI-names, server customized error messages, POI images). It is the responsibility of the UI package to convert any and all notifications to a means to inform the end user (if the UI so wishes). 

The reason for this is that the core should not contain product-specific switches (also known as branding variables) where one product wishes the user to be informed via dialogs while another product doesn't want to be informed at all.

The Core will however be possible to configure run-time at start-up with certain product-specific information not directly visible to the end-user. For example: client-type (often wrongly known as clientid in older Java clients), initial server list and other network parameters.

## Environment

![java_env](/images/java_env.png)

### Description

This package contains base classes for starting up and exiting an application on a platform. The environment is responsible for the following things:

*  When the user chooses to start the application, the environment should ensure that this is done.
*  Registering any listeners required to receive outside notifications required by the Core or the Platform specific implementations.
*  Taking any required actions to ensure that the Core may operate at peak performance.
*  Obtain any objects from the operating system to ensure that the PAL may be created
*  Creating the correct PAL implementation and passing this to the Core when it's created along with the startup information for the Core (client type etc)
*  Create the correct UI implementation and ensure that it has access to the Core
*  Notifying all other parts when they should pause or restart due to changes in the environment.
*  Notifying all other parts when the application should be shut down.
*  Upon shutdown, cleaning up all changes made and all registered listeners.
*  Ensuring that the process is completely shutdown upon exit.

Other items may be added to the list depending on platform specific APIs and processes.

The classes within this package should be kept in a state where they can be easily reused by other applications.

### Contents

*  Ensures UI is pushed to the screen. UI flow and push/pop procedure is platform dependent and will have to be placed as is appropriate for the platform.
*  Startup / shutdown (needs to direct what / if what parts of the Core will be shut down)
*  External invocations (unknown if we will keep)
*  Handling of device Tier startup procedure (as appropriate per product)
*  Allocates any runtime startup permissions

## UI

![java_ui](/images/java_ui.png)

### Description

This package contains all classes related to the user interaction layer of the
application. Like the platform package, the UI package will be divided into
multiple implementations, and are heavily based on the limitations and
requirements for each platform.

The UI is responsible for everything that is shown to the user. This also
includes Strings and images that are bundled with the application. Strings sent
from the server will be communicated to the UI via the Core.

While it is allowed, extreme care should be taken when attempting to share code
between the platforms. The reason for this is that certain concepts (like the
event-dispatcher) may cause quite undesirable results if the code is not
prepared for this.

When the UI communicates with the Core it may access all exposed methods
freely, but pay careful attention to any clearly stated limitations in the Core
APIs. If the UI sees a requirement to register listeners or callbacks for it's
function, the UI may in no case register a UI class directly into the core, but
must instead use the provided Core interfaces for this action.


### Contents

The UI package contains:

*  Product-specific UI-classes for each platform
*  Platform specific UI-classes that are shared between product-specific classes
*  Platform-independent UI-classes that are shared between platform-specific UI classes
*  Language-dependent Strings
*  Images

## Utility

![java_utility](/images/java_utility.png)

### Description


This package contains convenience classes and method that are intended to be shared between clients but has no connection to LBS or the Wayfinder services and thus should not be located within the Core.

Each platform has things that are harder to do which in turn will influence that contents of this package for each platform. Since the contents of this package will be custom for each platform (both in contents and implementation), it allows for a tighter integration with the UI parts.

Please note however that the utility package itself should not contain any UI classes, only convenience methods to populate them.


### Contents

The Utility package contains platform specific convenience methods:

Examples are:

*  Reading of contact lists
*  Discovery of nearby bluetooth units
*  Listing of all files on the device

