---
title: Core V.2 LBS API - How to get started
layout: single
---

This document describes how you get started with using the Core V.2 LBS API.
The document is based on the test client for Linux. Please also review the test
client after reading this document.

### Wayfinder API test client

The Wayfinder API test client is a simple sample client for using the Core V.2
LBS API that has test implementations of the interfaces provided by the Core
V.2 LBS API. 

##### TestClient

The test client can be run in two different modes, the first one is command
line mode in which there is a text display and input interface and then there
is the Gtk interface with a map window in which keys can be pressed. 

Furthermore, the test client can be started with a parameter stating which
language the user want to run with, affecting both map and sounds. This
parameter is `-l <language number>`.

In the command line mode a help is printed with the available commands to enter
in the main menu, in which there are several sub menus and a quit option
available for selection.

Help menu, following commands exists:

* `d` Image test menu.
* `f` Favourite option. Add, delete, sync and get Favourites.
* `g` Location test menu.
* `h` Displays this help menu.
* `p` Settings test menu.
* `q` Quits.
* `r` Route option, you should specify origin and destination in WGS84 Coordinates.
* `s` Search test menu.

When selecting a sub menu option the help for that menu is printed with the
available options for that menu. In all sub menus there is a back option to go
back to the main menu and a help option to get the options listed again, this
help option is also available in the main menu.

When running the test client there will be a Gtk window opened and a sample
implementation of the map API with overlays and input. The `z` key can be used
to zoom in and the ´x` key to zoom out (m and n zooms in and out in incremental
steps). Note that the map window can only respond to single character commands.
If making a search, switch console window and enter your search string there
and press enter. Then switch back to the map window to continue using the map.

Pressing enter after each character is not needed here, in fact maplib (which
handles the key presses when in graphical mode) does not accept multiple
character input, hence running the test `ss` won't be available and pressing
enter will not do anything.

One typical test case could be to start the client on the device, press `r` for
route menu, then d for calculating a default route. When the route has been
calculated the red line should appear on the map if it's scrolled. Pressing `s`
after that starts the navigation simulation. Pressing 3 activates the 3d map
mode, and 2 would put it back to 2d mode.

##### Tester classes

All tester classes inherits from a common super class named GenricTester that
has the common handling of commands.

The main class for the testers is `WFAPITester` that creates all the other
testers and handles the main menu as well as sending the input commands to the
currently active tester.

There are the following tester classes in the test client:

* `WFAPITester` – starts Nav2API and is the holder of the rest of the testers.
* `FavouriteTester` – tests for favourites.
* `ImageTester` – tests for image retrieval.
* `LocationTester` – tests for receiving location updates
* `RouteTester` – tests for routes and navigation.
* `SearchTester` – tests for searching.
* `SettingsTester` – tests for settings.
* `NetworkTester` - tests the network
* `BillingTester` - tests for the billing interface.
* `ServiceWindowTester` - tests the tunnel interface.

##### Running the testclient

Go to the folder where the test clients binary is located and run the command:

`./WFAPITestClient`

When starting the test client a help menu is presented:

    From here the different test types are started.
    The help menu is always available by inputting “h”

Input to the test client differs depending on which platform it runs on and in
which mode it's run in, but for Linux just enter the commands using the
keyboard.

##### Starting Nav2API

The Nav2API is created in the constructor of WFAPITester, simply by doing

`m_nav2API = new LinuxNav2API(); `

This call creates a new instance of the Nav2API for Linux platform. Below gives
an example of how Nav2API should be started. 

In `WFAPITester::startNav2()`:

{% highlight cpp %}
const char* dataPath = "./nav2data/";
cout << "WFAPITester::startNav2, before StartupData creation." << endl;  
StartupData startupData(dataPath,
                        dataPath,
                        dataPath,
                        ENGLISH,
                        VoiceLanguage::ENGLISH);
// We need gtk for getting the imei even 
// if we run the app without MapLib , therefor it must be called
// before Nav2API is started
gtk_init(&argc, &argv); 
	
// Start the startup of Nav2API, this call will return after
// initiating the startup.
m_startUpStatus =  m_nav2API->start(this, &startupData);
{% endhighlight %}

There are two callback functions that gets called by the Nav2API when startup is done.

{% highlight cpp %}
void 
WFAPITester::startupComplete()
{
   const char* dataPath = "./nav2data/";
   
   cout << "WFAPITester::startNav2, starting m_nav2API done." << endl;  

   // Create the testers.
   initTesters(dataPath);

   // Register interesting listeners.
   if (m_routeTester != NULL) {
      m_routeTester->addNavigationInfoUpateListener(m_uiControl);
   }
   if (m_locationTester != NULL) {
      m_locationTester->addLocationListener(m_uiControl);
   }
   
   // Nav2API calls this when startup has completed. If there was an error
   // during startup the error function is called and not this function.
   cout << "WFAPITester::startupComplete." << endl;  
   m_nav2Started = true;
}

void 
WFAPITester::mapLibStartupComplete()
{
   cout << "WFAPITester::mapLibStartupComplete." << endl;  
}
{% endhighlight %}

When the above is done the Nav2API and MapLib are ready to use. An example of
how to use one of the interfaces in Nav2API is described below.
	
{% highlight cpp %}
RouteInterface& m_routeInterface = m_nav2API->getRouteInterface();
AsynchronousStatus status = m_routeInterface.routeBetweenCoordinates(startCoord, destCoord, transportationType);
{% endhighlight %}

When the route has been calculated the `routeReply()` callback function, which
is inherited from `RouteLister`, will be called by the Nav2API. The UI now
knows that the requested route has been calculated and can start navigation,
fetching the route list etc.

##### Connecting MapLib to Nav2API

After the Nav2API has been successfully started it is possible to create and
connect MapLib to Nav2API. The test application creates and connects MapLib to
Nav2API.

The GtkDevControl is an example class that is using the MapLib API for the most
common use cases, such as zooming, panning the map, adding and removing overlay
items etc.  This constructing and connection is done by the following code.

{% highlight cpp %}
GtkDevControl mapControl("./images/",
                         apiTester->getNav2API().getDBufConnection(),
                         apiTester);
apiTester->getNav2API().connectMapLib(mapControl.getMapLibAPI());
{% endhighlight %}

The first parameter is the path to images, i.e. where to find images for the
overlay items to add. The second parameter is the network connection for MapLib
to use, which are coming from Nav2API. The last parameter is a
`WFAPI::ExternalKeyConsumer*`, which is used to catch key events that
`GtkDevControl` does not handle.

##### UI Controller

For demonstration purposes, there is also a `UIControl` class which shows how
it is possible to update the map (`GtkDevControl`) based on information
received from Nav2. The UI controller is created and connected here in
`WFAPITestClient.cpp`:

{% highlight cpp %}
// Create a UI controller. 
UIControl uiCtrl(&mapControl); 
// Add UI control to the api tester. 
apiTester->setUIControl(&uiCtrl); 
{% endhighlight %}

The `UIControl` class currently implements the listener interfaces from
`NavigationInfoUpdateListener` and `LocationListener`. When receiving location
updates, the me marker on the map is updated to that position and the map is
centered based on the updated position. The map is also rotated according to
the heading.

During navigation, `UIControl` will also update text labels in `GtkDevControl`
with distance to next turn and the name of the current street (just as
examples).  This is a simple and naïve implementation of parts of a navigation
UI, meant to be an example of how a real navigation applications could be
developed using the APIs.  In order to start the navigation which will update
the map as stated above, the easiest is to enter the following commands from
the keyboard:

```'h' To get help text]
'r' To enter route menu]
'd' To create the default route]
.. Wait about 10 seconds until the route has been calculated.
's' To start simulation of the route] ´´´

##### Handling key events

When the test client is started with MapLib integrated all key events are first
captured by the `GtkDevControl`. The key events that are unhandled by
GtkDevControl, are instead supplied into the Nav2API tester classes using the
`WFAPI::ExternalKeyConsumer`. In this way it is possible to zoom in and out in
the map using `z` and `x`, but also be able access the rest of the Nav2API
tests (making routes etc).

Note that any key events must be supplied using the simulator virtual key pad
when MapLib is enabled.

Initialization of key mappings; The following call in `GtkDevControl` is
responsible for initializing key mappings:

{% highlight cpp %}
// Initialize key mappings. 
initKeyMap(); 
{% endhighlight %}

MapLib contains a class called `MapLibKeyInterface`. This contains a number of
different key names,  listed below. The enum can be found in
`MapLibKeyInterface.h`.

{% highlight cpp %}
/// Move up. 
MOVE_UP_KEY, 
/// Move down. 
MOVE_DOWN_KEY, 
/// Move left. 
MOVE_LEFT_KEY, 
/// Move right. 
MOVE_RIGHT_KEY, 
/// Zoom in. 
ZOOM_IN_KEY, 
/// Zoom out. 
ZOOM_OUT_KEY, 
/// Rotate the map counter clockwise. 
ROTATE_LEFT_KEY, 
/// Rotate the map clockwise. 
ROTATE_RIGHT_KEY, 
/// Set rotation so that north is up. 
RESET_ROTATION_KEY, 
/// Move up and right. 
MOVE_UP_RIGHT_KEY, 
/// Move down and right. 
MOVE_DOWN_RIGHT_KEY, 
/// Move up and left. 
MOVE_UP_LEFT_KEY, 
/// Move down and left. 
MOVE_DOWN_LEFT_KEY, 
/// Send this if nothing should be done. 
NO_KEY, 
{% endhighlight %}

These key names should be mapped against real key/pointer events coming from
the framework. The way we do it in the `GtkDevControl.cpp` is to map a Gtk
specific key against a key name from the above list, e.g. `GDK_KP_Subtract`
against `MapLibKeyInterface::ZOOM_OUT_KEY`. By doing this you can easily
translate a key event from the framework to an action that MapLib can execute. 

Below is an example of how we do when mapping key types against map operations
in the `GtkDevControl`.

{% highlight cpp %}
m_keyMap[GDK_x] = MapLibKeyInterface::ZOOM_OUT_KEY;
m_keyMap[GDK_z] = MapLibKeyInterface::ZOOM_IN_KEY;
{% endhighlight %}

When getting a callback by the framework in the listening function for key
events it is quite simple to retrieve the correct MapLib key. Below is an
example of how this is done in the `GtkDevControl.cpp`:

{% highlight cpp %}
// Get the key interface from MapLib.
MapLibKeyInterface* keyInterface = m_mapLib->getKeyInterface();

// Look up in the key map which action on the map that should be performed
// based on the supplied key code (refer to initKeyMap method).
KeyMap::iterator itr = m_keyMap.find( keyEvent->keyval );

if( itr != m_keyMap.end() ) {
   // Key was found in key map and desired action is defined 
   // by the key object. 
   MapLibKeyInterface::keyType key = itr->second; 

   MapLibKeyInterface::kindOfPressType type = 
      translateKeyType( keyEvent->type ); 
   return keyInterface->handleKeyEvent( key, type ); 
}
{% endhighlight %}

As you can see above, the mapped MapLib key is fetched by using the key from
the key map, in this application the GdkEventKey. This key is then sent to the
MapLibKeyInterface which will perform the correct map operation, such as
zooming in, zooming out, panning the map etc.

##### Initializing window components

The following code in GtkDevControl is responsible for creating and
initializing windows, registering key event callbacks and other Gtk specific
code. This part does not contain any MapLib specific code.

{% highlight cpp %}
// Initialize key mappings. 
initKeyMap(); 
{% endhighlight %}

##### Initializing MapLib

This part is where you add the code that contains the actual map and it's
components.  The initial method call in `GtkDevControl` looks like this
(supplying the connection retrieved from Nav2API):

{% highlight cpp %}
// Initialize MapLib. 
initMapLib(connection); 
{% endhighlight %}

The basic steps performed by this section is to create the MapLibAPI using the
supplied factory methods. In addition to this, listeners are registered to
MapLib to get notifications about when a map object has been selected, etc.

##### Initialization of overlays and dialogs

The following section in GtkDevControl is responsible for setting up some
sample overlay items:

{% highlight cpp %}
// Initialize shared overlay properties 
initOverlay(); 
{% endhighlight %}

A seperate class, `OverlayZoomSetup`, is added as a sample implementation of
how to generally configure the visual representation of overlay items. This
includes, which background images to use for the different zoom levels (S,M,L)
and modes (highlighted or stacked). It also contains information about where
the focus point of these images should be (i.e. the position of the image that
should be set to the coordinate of the overlay item), etc. The keys `I` and `l`
are configured to toggle visibility for the friends and location layer
respectively.

The following method call sets up the visuals for how the stacked dialogs
should look, i.e. the dialog that appears when selecting one of multiple
overlay items.

{% highlight cpp %}
// Create the stacked dialogs. 
createStackedDialog(); 
{% endhighlight %}

##### Wayfinder API utility functions

Included in the `WFAPITestClient` source code directory, there is also one
Utility class available which can be used to determine distance between two
coordinates.

{% highlight cpp %}
/** 

 *    This static class contains utility-methods concerning 
 *    geographical data. 
 */ 
class WFGfxUtil 
{ 
public: 
   /** 

    *    Calculate the distance between two coordinates. 
    * 
    *    @param coord1 The first coordinate. 
    *    @param coord2 The second coordinate. 
    *    @return The squared distance in meters between the two 
    *            coordinates. 
    */ 
   static WFAPI::wf_float64 squareDistBetweenCoords( 
         const WFAPI::WGS84Coordinate& coord1, 
         const WFAPI::WGS84Coordinate& coord2); 

}; 
{% endhighlight %}


The file `WFAPITestClient.cpp` contains commented code which uses the utility
function to calculate the distance between two coordinates:

{% highlight cpp %}
#if 0 
   // Test the distance calculation between two coordinates. 
   
   WFAPI::WGS84Coordinate c1(55.59, 13.0078); 
   WFAPI::WGS84Coordinate c2(55.565, 12.9763); 

   cout << "Distance between " << c1 << " and " << c2 << " is " 
        << sqrt(WFGfxUtil::squareDistBetweenCoords(c1,c2)) 
        << " meters." 
        << endl; 
#endif 
{% endhighlight %}

If the calculated distance should be compared with another distance threshold,
then we suggest that instead of taking the square root of the calculated
distance, instead square the distance threshold (to increase performance)
before doing the comparison.
