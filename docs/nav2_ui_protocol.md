---
title: Nav2 UI protocol
layout: single
---

## General 

This document describes how the communication is done between Nav2 and the UI.
The communication between Nav2 and UI is done by sending messages between the
two components. In this document you can see what kind of messages and message
types that can be sent and received to and from Nav2 from the UI. Typically a
request is sent from the UI, Nav2 parses the request and replies with another
message that can be mapped to the request by saving the the request id when
sending the request and compare that request id against the message id
(`GuiProtMess::getMessageID()`).

The Nav2 - UI protocol is used inside a navigation client program. The Nav2
part of the program is responsible for communication with the MC2 server,
keeping track of the navigation, store larger amounts of data etc. The UI is
responsible for interaction with the user, both graphical and with sound. 

The protocol is in a binary format, using different messages identified by message type. 

### Used Syntax in The Following Specification 

* **typeOrClass[ ] ** Indicates that a vector with elements of the type typeOrClass is included in the structure. Usually the protocol includes a variable before this structure telling how many elements there are in the vector. If typeOrClass is a class, there is a separate table describing how the data of the class is serialized in the protocol. 
* **typeOrClass[aNumber]** Indicates that a vector with elements of the type typeOrClass is included in the structure. The vector has the fixed length aNumber. 
* **string ** A null terminated, UTF-8 encoded, string of characters. 
* **Nav2 -> UI** Indicates that the message can be sent from Nav2 to the UI. Used in the `Direction`-row in the table describing a message. 
* **Nav2 <- UI** Indicates that the message can be sent from the UI to Nav2. Used in the `Direction`-row in the table describing a message. 
* **Nav2 `<->` UI** Indicates that the message can be sent both from Nav2 to the UI and from the UI to Nav2. Used in the `Direction`-row in the table describing a message. 
* **0xff** The marked `0x` indicates that the following number is given in the hexadecimal base. 
* **99 ** A number with no preceding `0x` is given in the decimal base. 

#### Data Types and Constants Used in The Protocol 

##### Types 

All multi byte numbers use network byte order, i.e. the most significant byte first. All signed integers are sent as twos complement. 

* `bool` unsigned eight bit integer, otherwise as a C++ bool 
* `uint8` unsigned eight bit integer 
* `int8` eight bit integer 
* `uint16` unsigned 16 bit integer 
* `int16` 16 bit integer 
* `uint32` unsigned 32 bit integer 
* `int32` 32 bit integer 
   
##### Constants 
*
* `MAX_UINT32` 0xffffffff 
* `MAX_INT32` 0x7fffffff 
* `MAX_UINT16` 0xffff 
* `MAX_INT16` 0x7fff 

### Message Header

The following table show data included in the header. 

 | Header data      | Size[bytes] | 
 | -----------      | ----------- | 
 | Protocol version | 1           | 
 | Data length      | 3           | 
 | Data type        | 1           | 
 | Message type     | 1           | 
 | Message ID       | 2           | 

* **Data length** The length of the message, including the header. 
* **Protocol version ** The version of the protocol used between the UI and Nav2. This specification is version 4 of the protocol. 
* **Data type ** The type of data contained in the message. See subsection . 
* **Message type ** Tells what kind message this is, i.e. what data, if any, this message contains. Each message type corresponds to a message described below, see section . 
* **Message ID ** An ID uniquely identifying this message in a series of messages. There are two such series. One in Nav2 and one in the UI. The Nav2 message id series start at 0, and the UI series start at 1. Both series are incremented by two for each message sent. I.e. Nav2 increments the number in its series by 2 when it sends a message to the UI and the UI increments the number in its series by 2 when it sends a message to Nav2. Using 16 bit unsigned integers guarantee that when they wrap around, the pattern will continue. We assume that it is safe to reuse the ids by then. \\ All this means that messages from Nav2 to the UI will have the least significant bit cleared, and messages from UI to Nav2 will have the least significant bit set. \\ Messages that are replies to previously received messages should have the same id as the request. This is an exception to the ID series described above. Note that no request-reply-sequence contains more than 2 messages. 

#### DataType 

{% highlight cpp %}
   enum DataType {
      only_type = 0x00,
      type_and_uint8 = 0x01,
      type_and_uint16 = 0x02,
      type_and_uint32 = 0x03,
      type_and_bool = 0x04,
      type_and_string = 0x05,
      type_and_int64 = 0x06,
      type_and_two_uint8 = 0x08,
      type_and_two_uint16 = 0x09,
      type_and_two_uint32 = 0x0a,
      type_and_two_bool = 0x0b,
      type_and_two_strings = 0x0c,
      type_and_three_bool = 0x13,
      type_and_data = 0x20,
      type_and_length_and_data = 0x21,
   };
{% endhighlight %}

All the names of the data types contain the word "type". Here, "type" refers to the message type, which is contained in the header and therefore naturally included in all messages. The rest of the name tells what type the data contained in the message has. 

All data items are, when not otherwise noted, sent unaligned, i.e. there are never any padding between data items. 

* **only_type ** No data is contained in the message. 
* **type_and_uint8 ** The message contains one uint8. 
* **type_and_uint16 ** The message contains one uint16. 
* **type_and_uint32 ** The message contains one uint32. 
* **type_and_bool ** The message contains one bool. 
* **type_and_string ** The message contains one string. 
* **type_and_int64 ** The message contains one int64. 
* **type_and_two_uint8 ** The message contains two uint8. 
* **type_and_two_uint16 ** The message contains two uint16 
* **type_and_two_uint32 ** The message contains two uint32 
* **type_and_two_bool ** The message contains two bools. 
* **type_and_two_strings ** The message contains two strings. 
* **type_and_three_bool ** The message contains three bools. 
* **type_and_data ** The message contains binary data. The formatting of this data depends on the message type of the message. 
* **type_and_length_and_data ** The message contains a 32 bit length field followed by a binary data block. 


#### Message Types 

Each of these values is associated with a unique message from XXX

{% highlight cpp %}
   ERROR                        = 0x01,
   REQUEST_FAILED               = 0x02,
   PANIC_ABORT                  = 0x03,
   GET_TOP_REGION_LIST          = 0x10,
   GET_GENERAL_PARAMETER        = 0x13,
   SET_GENERAL_PARAMETER        = 0x1a,
   GET_TOP_REGION_LIST_REPLY    = (GET_TOP_REGION_LIST     | 0x80),
   SEND_MESSAGE                 = 0x1e,
   SEND_MESSAGE_REPLY           = (SEND_MESSAGE            | 0x80),
   PARAMETER_CHANGED            = 0x1f,
   PARAMETERS_SYNC              = 0x20,
   PARAMETERS_SYNC_REPLY        = (PARAMETERS_SYNC         | 0x80),
   FILEOP_GUI_MESSAGE           = 0x21,
   GET_FAVORITES                = 0x30,
   GET_FAVORITES_REPLY          = (GET_FAVORITES           | 0x80),
   GET_FAVORITES_ALL_DATA       = 0x31,
   GET_FAVORITES_ALL_DATA_REPLY = (GET_FAVORITES_ALL_DATA  | 0x80),
   SORT_FAVORITES               = 0x32,
   SYNC_FAVORITES               = 0x33,
   GET_FAVORITE_INFO            = 0x34,
   GET_FAVORITE_INFO_REPLY      = (GET_FAVORITE_INFO       | 0x80),
   ADD_FAVORITE                 = 0x35,
   ADD_FAVORITE_FROM_SEARCH     = 0x36,
   REMOVE_FAVORITE              = 0x37,
   CHANGE_FAVORITE              = 0x38,
   ROUTE_TO_FAVORITE            = 0x39,
   ROUTE_TO_HOT_DEST            = 0x3a,
   ROUTE_MESSAGE                = 0x3b,
   FAVORITES_CHANGED            = 0x40,
   CONNECT_GPS                  = 0x50,
   DISCONNECT_GPS               = 0x51,
   PROGRESS_INDICATOR           = 0x52,
   PREPARE_SOUNDS               = 0x53,
   PLAY_SOUNDS                  = 0x54,
   PREPARE_SOUNDS_REPLY         = (PREPARE_SOUNDS          | 0x80),
   PLAY_SOUNDS_REPLY            = (PLAY_SOUNDS             | 0x80),
   UPDATE_POSITION_INFO         = 0x58,
   REQUEST_CROSSING_SOUND       = 0x5a,
   REQUEST_LICENSE_UPGRADE      = 0x5b,
   LICENSE_UPGRADE_REPLY        = (REQUEST_LICENSE_UPGRADE | 0x80),
   CELL_INFO_TO_SERVER          = 0x5c,
   CELL_INFO_FROM_SERVER        = (CELL_INFO_TO_SERVER     | 0x80),
   TUNNEL_DATA                  = 0x5d,
   TUNNEL_DATA_REPLY            = (TUNNEL_DATA | 0x80),
   GET_FILTERED_ROUTE_LIST      = 0x59,
   ROUTE_LIST                   = (GET_FILTERED_ROUTE_LIST | 0x80),
   ROUTE_TO_POSITION            = 0x60,
   ROUTE_TO_SEARCH_ITEM         = 0x61,
   STARTED_NEW_ROUTE            = 0x62,
   REROUTE                      = 0x63,
   INVALIDATE_ROUTE             = 0x64,
   UPDATE_ROUTE_INFO            = 0x6a,
   SATELLITE_INFO               = 0x6b,
   SEARCH                       = 0x70,
   SEARCH_RESULT_CHANGED        = 0x71,
   GET_SEARCH_AREAS             = 0x72,
   GET_SEARCH_AREAS_REPLY       = (GET_SEARCH_AREAS        | 0x80),
   GET_SEARCH_ITEMS             = 0x73,
   GET_SEARCH_ITEMS_REPLY       = (GET_SEARCH_ITEMS        | 0x80),
   GET_FULL_SEARCH_DATA         = 0x74,
   GET_FULL_SEARCH_DATA_REPLY   = (GET_FULL_SEARCH_DATA    | 0x80),
   GET_MAP                      = 0x75,
   GET_MAP_REPLY                = (GET_MAP                 | 0x80),
   GET_MORE_SEARCH_DATA         = 0x76,
   GET_VECTOR_MAP               = 0x77,
   GET_VECTOR_MAP_REPLY         = (GET_VECTOR_MAP          | 0x80),
   GET_MULTI_VECTOR_MAP         = 0x78,
   GET_MULTI_VECTOR_MAP_REPLY   = (GET_MULTI_VECTOR_MAP    | 0x80),
   GET_FULL_SEARCH_DATA_FROM_ITEMID       
                                = 0x79,
   GET_FULL_SEARCH_DATA_FROM_ITEMID_REPLY 
                                = (GET_FULL_SEARCH_DATA_FROM_ITEMID | 0x80),
   FORCEFEED_MULTI_VECTOR_MAP   = 0x7a,
   FORCEFEED_MULTI_VECTOR_MAP_REPLY       
                                = (FORCEFEED_MULTI_VECTOR_MAP | 0x80),
{% endhighlight %}

These messages are removed in this version and should not be used. 

{% highlight cpp %}
   GET_CALL_CENTER_NUMBERS      = 0x11,
   GET_SIMPLE_PARAMETER         = 0x12,
   SET_SIMPLE_PARAMETER         = 0x18,
   SET_CALL_CENTER_NUMBERS      = 0x19,
{% endhighlight %}

### Message Help Classes 

Class diagram for the message help classes shown in the following image: 

![nav2_ui_prot_help_classes3](/images/nav2_ui_prot_help_classes3.png)

The figure shows the classes that are used for creating, serializing, de-serializing and accessing the data of a message. 
All messages except those with data type type_and_data can be represented with the class GenericGuiMess. For these messages the data type of the message depends on which constructor is used when creating the message object. E.g. if using the constructor 

`uint8 firstUint8, uint8 secondUint8);`

the message's data type will be set to `type_and_two_uint8`. It is recommended to cast the variables used as parameters to the `GenericGuiMess` constructor so they make an exact match to the constructor parameters. This should be done to prevent the compiler from using the wrong constructor. The messages that have the data type `type_and_data` have one help class each. 

The members of the message help classes are not deleted in the constructor. Instead, the members are deleted in the `deleteMembers` method. This is because it should be possible to create some of the messages with pointers to objects as parameters, without copying the objects used as parameters. 

### Enums Used in message data 

Where no integer value is specified, normal C++ enum rules apply. 


#### AdditionalInfoType 

{% highlight cpp %}
   enum AdditionalInfoType {
      dont_show = 0x00,
      dontShow = 0x00,
      text = 0x01,
      url = 0x02,
      wap_url = 0x03,
      email = 0x04,
      phone_number = 0x05,
      mobile_phone = 0x06,
      fax_number = 0x07,
      contact_info = 0x08,
      short_info = 0x09,
      vis_address = 0x0a,
      vis_house_nbr = 0x0b,
      vis_zip_code = 0x0c,
      vis_complete_zip = 0x0d,
      Vis_zip_area = 0x0e,
      vis_full_address = 0x0f,
      brandname = 0x10,
      short_description = 0x11,
      long_description = 0x12,
      citypart = 0x13,
      ad_info_state = 0x14,
      neighborhood = 0x15,
      open_hours = 0x16,
      nearest_train = 0x17,
      start_date = 0x18,
      end_date = 0x19,
      start_time = 0x1a,
      end_time = 0x1b,
      accommodation_type = 0x1c,
      check_in = 0x1d,
      check_out = 0x1e,
      nbr_of_rooms = 0x1f,
      single_room_from = 0x20,
      double_room_from = 0x21,
      triple_room_from = 0x22,
      suite_from = 0x23,
      extra_bed_from = 0x24,
      weekend_rate = 0x25,
      nonhotel_cost = 0x26,
      breakfast = 0x27,
      hotel_services = 0x28,
      credit_card = 0x29,
      special_feature = 0x2a,
      conferences = 0x2b,
      average_cost = 0x2c,
      booking_advisable = 0x2d,
      admission_charge = 0x2e,
      home_delivery = 0x2f,
      disabled_access = 0x30,
      takeaway_available = 0x31,
      allowed_to_bring_alcohol = 0x32,
      type_food = 0x33,
      decor = 0x34,
      image_url = 0x35,
      supplier = 0x36,
      owner = 0x37,
      price_petrol_superplus = 0x38,
      price_petrol_super = 0x39,
      price_petrol_normal = 0x3a,
      price_diesel = 0x3b,
      price_biodiesel = 0x3c,
      free_of_charge = 0x3d,
      tracking_data = 0x3e,
      post_address = 0x3f,
      post_zip_area = 0x40,
      post_zip_code = 0x41,
      open_for_season = 0x42,
      ski_mountain_min_max_height = 0x43,
      snow_depth_valley_mountain = 0x44,
      snow_quality = 0x45,
      lifts_open_total = 0x46,
      ski_slopes_open_total = 0x47,
      cross_country_skiing_km = 0x48,
      glacier_area = 0x49,
      last_snowfall = 0x4a,
      special_flag = 0x4b,
      more = 0xff
   };
{% endhighlight %}

### BoxType

{% highlight cpp %}
   enum BoxType {
      invalidBox = 0,
      boxBox = 1,
      vectorBox = 2,
      diameterBox = 3,
      routeBox = 4,
   };
{% endhighlight %}

### ErrorNbr

{% highlight cpp %}
   enum ErrorNbr {
      NO_ERRORS = 0x00000000,
      MAX_USER_ERROR = 0x00f0,
      PANIC_ABORT = 0x00f1,
      DUMMY_ERROR = 0x00f2,
      DEST_SYNC_ALREADY_IN_PROGRESS = 0x0100,
      DEST_SYNC_FAILED = 0x0101,
      DEST_REMOVE_DEST_MISSING_DEST_ID = 0x0102,
      DEST_TO_LONG_STRING_IN_DEST = 0x0103,
      DEST_DEST_INFO_MISSING_DEST_ID = 0x0104,
      DEST_INVALID_FAVORTE_ID_IN_LIST = 0x0105,
      NSC_NO_GPS = 0x0200,
      NSC_OUT_OF_SERVERS = 0x0201,
      NSC_EXPIRED_USER = 0x0202,
      NSC_AUTHORIZATION_FAILED = 0x0203,
      NSC_CANCELED_BY_REQUEST = 0x0204,
      NSC_OUT_OF_THIS_WORLD = 0x0205,
      NSC_OPERATION_NOT_SUPPORTED = 0x0206,
      NSC_BEYOND_SALVATION = 0x0207,
      NSC_CANCELED_BY_SHUTDOWN = 0x0208,
      NSC_NO_USERNAME = 0x0209,
      NSC_NO_TRANSACTIONS = 0x0210,
      NSC_NO_LICENSE_ID = 0x0211,
      NSC_TCP_INTERNAL_ERROR = 0x0212,
      NSC_PARAM_REQ_NOT_FIRST = 0x0213,
      NSC_REQUEST_ALLOC_FAILED = 0x0214,
      NSC_NO_ROUTE_RIGHTS = 0x0215,
      NSC_NO_NETWORK_AVAILABLE = 0x0216,
      NSC_FLIGHT_MODE = 0x0217,
      NSC_NO_GPS_WARN = 0x0218,
      NSC_NO_GPS_ERR = 0x0219,
      NSC_TCP_INTERNAL_ERROR2 = 0x021a,
      NSC_UPGRADE_MUST_CHOOSE_REGION = 0x021b,
      NSC_SERVER_COMM_TIMEOUT_CONNECTED = 0x0300,
      NSC_SERVER_NOT_OK = 0x0301,
      NSC_SERVER_REQUEST_TIMEOUT = 0x0302,
      NSC_SERVER_OUTSIDE_MAP = 0x0303,
      NSC_SERVER_UNAUTHORIZED_MAP = 0x0304,
      NSC_SERVER_NOT_FOUND = 0x0305,
      NSC_SERVER_UNREACHABLE = 0x0306,
      NSC_SERVER_NOT_RESPONDING = 0x0307,
      NSC_SERVER_PROTOCOL_ERROR = 0x0308,
      NSC_SERVER_CONNECTION_BROKEN = 0x0309,
      NSC_SERVER_NO_ROUTE_FOUND = 0x030a,
      NSC_SERVER_ROUTE_TOO_LONG = 0x030b,
      NSC_SERVER_BAD_ORIGIN = 0x030c,
      NSC_SERVER_BAD_DESTINATION = 0x030d,
      NSC_SERVER_NO_HOTDEST = 0x030e,
      NSC_SERVER_NEW_VERSION = 0x030f,
      NSC_TRANSPORT_FAILED = 0x0310,
      NSC_UNAUTH_OTHER_HAS_LICENSE = 0x0311,
      NSC_OLD_LICENSE_NOT_IN_ACCOUNT = 0x0312,
      NSC_OLD_LICENSE_IN_MANY_ACCOUNTS = 0x0313,
      NSC_NEW_LICENSE_IN_MANY_ACCOUNTS = 0x0314,
      NSC_OLD_LICENSE_IN_OTHER_ACCOUNT = 0x0315,
      NSC_SERVER_COMM_TIMEOUT_CONNECTING = 0x0316,
      NSC_SERVER_COMM_TIMEOUT_DISCONNECTING = 0x0317,
      NSC_SERVER_COMM_TIMEOUT_CLEAR = 0x0318,
      NSC_SERVER_COMM_TIMEOUT_WAITING_FOR_USER = 0x0319,
      NSC_FAKE_CONNECT_TIMEOUT = 0x031a,
      NAVTASK_ROUTE_INVALID = 0x0400,
      NAVTASK_NSC_OUT_OF_SYNC = 0x0401,
      NAVTASK_INTERNAL_ERROR = 0x0402,
      NAVTASK_FAR_AWAY = 0x0403,
      NAVTASK_CONFUSED = 0x0404,
      NAVTASK_ALREADY_DOWNLOADING_ROUTE = 0x0405,
      NAVTASK_NO_ROUTE = 0x0406,
      UC_REQUEST_TIMED_OUT = 0x0500,
      UC_CONFUSED = 0x0501,
      UC_ASKED_ROUTE_FROM_WRONG_MODULE = 0x0502,
      UC_INVALID_GUI_REQUEST = 0x0503,
      UC_NO_ROUTE = 0x0504,
      UC_INVALID_PARAM = 0x0505,
      UC_UNKNOWN_PARAM = 0x0506,
      UC_UNKNOWN_SEARCH_ID = 0x0507,
      GUIPROT_FAILED_GET_TOP_REGION_LIST = 0x0600,
      GUIPROT_FAILED_GET_CALL_CENTER_NUMBERS = 0x0601,
      GUIPROT_FAILED_GET_SIMPLE_PARAMETER = 0x0602,
      GUIPROT_FAILED_SET_CALL_CENTER_NUMBERS = 0x0603,
      GUIPROT_FAILED_SET_SIMPLE_PARAMETER = 0x0604,
      GUIPROT_FAILED_GET_FAVORITES = 0x0605,
      GUIPROT_FAILED_GET_FAVORITES_ALL_DATA = 0x0606,
      GUIPROT_FAILED_SORT_FAVORITES = 0x0607,
      GUIPROT_FAILED_SYNC_FAVORITES = 0x0608,
      GUIPROT_FAILED_GET_FAVORITE_INFO = 0x0609,
      GUIPROT_FAILED_ADD_FAVORITE = 0x060a,
      GUIPROT_FAILED_ADD_FAVORITE_FROM_SEARCH = 0x060b,
      GUIPROT_FAILED_REMOVE_FAVORITE = 0x060c,
      GUIPROT_FAILED_CHANGE_FAVORITE = 0x060d,
      GUIPROT_FAILED_ROUTE_TO_FAVORITE = 0x060e,
      GUIPROT_FAILED_ROUTE_TO_HOT_DEST = 0x060f,
      GUIPROT_FAILED_DISCONNECT_GPS = 0x0610,
      GUIPROT_FAILED_CONNECT_GPS = 0x0611,
      PARAM_NO_OPEN_FILE = 0x0700,
      PARAM_NO_READ_LINE = 0x0701,
      PARAM_CORRUPT_BINARYBLOCK = 0x0702,
      PARAM_CORRUPT_STRING = 0x0703,
      PARAM_CORRUPT_SYNC = 0x0704,
      PARAM_CORRUPT_TYPE = 0x0705,
      PARAM_TRYING_BACKUP = 0x0706,
      PARAM_TRYING_ORIGINAL = 0x0707,
      PARAM_NO_VALID_PARAM_FILE = 0x0708,
      PARAM_NO_SPACE_ON_DEVICE = 0x0709,
      PARAM_NO_SPACE_ON_DEVICE_2 = 0x0710,
      INVALID_ERROR_NBR = 0xffff
   };
{% endhighlight %}

#### ImageFormat 

{% highlight cpp %}
   enum ImageFormat {
      PNG_OLD_IS_GIF_NOW = 0,
      WBMP = 1,
      JPEG = 2,
      GIF = 3,
      PNG = 4,
      WIN_CE_BMP = 5,
      NBR_OF_IMAGE_TYPES,
      INVALID_IMAGE_TYPE
   };
{% endhighlight %}

#### MapInfoType

{% highlight cpp %}
   enum MapInfoType {
      InvalidMapInfoType = 0,
      Category = 1,
      TrafficInformation = 2,
      Ruler = 3,
      Topographic = 4,
      MapFormat = 5,
      Rotate = 6,
   };
{% endhighlight %}

####  MapItemType

{% highlight cpp %}
   enum MapItemType {
      InvalidMapItemType = 0,
      MapRouteItem = 1,
      MapSearchItem = 2,
      MapDestinationItem = 3,
      MapUserPositionItem = 4,
   };
{% endhighlight %}

#### ObjectType

{% highlight cpp %}
   enum ObjectType {
      invalidObjectType = 0,
      DestinationMessage = 1,
      SearchItemMessage = 2,
      FullSearchItemMessage = 3,
      RouteMessage = 4,
      ItineraryMessage = 5,
      MapMessage = 6,
      PositionMessage = 7,
   };
{% endhighlight %}

#### OnTrackEnum 

{% highlight cpp %}
   enum OnTrackEnum {
      OnTrack,
      OffTrack,
      WrongWay,
      Goal
   };
{% endhighlight %}

#### ParameterType

{% highlight cpp %}
   enum ParameterType {
      paramTopRegionList = 0x0,
      paramServerNameAndPort = 0x1,
      paramSoundVolume = 0x2,
      paramUseSpeaker = 0x3,
      paramAutoReroute = 0x4,
      paramTransportationType = 0x5,
      paramDistanceMode = 0x6,
      paramTollRoads = 0x7,
      paramHighways = 0x8,
      paramTimeDist = 0x9,
      paramVectorMapSettings = 0x27,
      paramFavoriteShow = 0x29,
      paramGPSAutoConnect = 0x2a,
      paramPoiCategories = 0x2b,
      paramDemoMode = 0x2c,
      paramVectorMapCoordinates = 0x2d,
      paramAutoTracking = 0x2f,
      paramPositionSymbol = 0x30,
      paramTrackingLevel = 0x31,
      paramTrackingPIN = 0x32,
      paramBtGpsAddressAndName = 0x33,
      paramMapLayerSettings = 0x34,
      userRights = 0x35,
      paramLinkLayerKeepAlive = 0x36,
      paramUserTermsAccepted = 0x37,
      paramTurnSoundsLevel = 0x39,
      userTrafficUpdatePeriod = 0x40,
      paramUserAndPassword = 0x41,
      paramHttpServerNameAndPort = 0x42,
      paramShowNewsServerString = 0x43,
      paramShownNewsChecksum = 0x44,
      paramNeverShowUSDisclaimer = 0x45,
      paramRegistrationSmsSent = 0x46,
      paramMapACPSetting = 0x47,
      paramCheckForUpdates = 0x48,
      paramNewVersion = 0x49,
      paramNewVersionUrl = 0x50,
      paramInvalid = 0xffff,
   };
{% endhighlight %}

#### PositionType 

{% highlight cpp %}
   enum PositionType {
      PositionTypeInvalid = 0,
      PositionTypeSearch = 1,
      PositionTypeFavorite = 2,
      PositionTypePosition = 3,
      PositionTypeHotDest = 4,
      PositionTypeCurrentPos = 5,
   };
{% endhighlight %}   

#### Quality

{% highlight cpp %}
   enum Quality {
      QualityMissing = 0,
      QualitySearching,
      QualityUseless,
      QualityPoor,
      QualityDecent,
      QualityExcellent,
      QualityDemohx,
      QualityDemo1x,
      QualityDemo2x,
      QualityDemo4x
   };
{% endhighlight %}

####  RegionType

{% highlight cpp %}
   enum RegionType {
      invalid = 0x0,
      streetNumber = 0x1,
      address = 0x2,
      cityPart = 0x3,
      city = 0x4,
      municipal = 0x5,
      county = 0x6,
      state = 0x7,
      country = 0x8,
      zipcode = 0x9,
      zipArea = 0xa,
   };
{% endhighlight %}

#### RouteAction 

{% highlight cpp %}
   enum RouteAction {
      InvalidAction = 0xffff,
      End = 0x0000,
      Start = 0x0001,
      Ahead = 0x0002,
      Left = 0x0003,
      Right = 0x0004,
      UTurn = 0x0005,
      StartAt = 0x0006,
      Finally = 0x0007,
      EnterRdbt = 0x0008,
      ExitRdbt = 0x0009,
      AheadRdbt = 0x000a,
      LeftRdbt = 0x000b,
      RightRdbt = 0x000c,
      ExitAt = 0x000d,
      On = 0x000e,
      ParkCar = 0x000f,
      KeepLeft = 0x0010,
      KeepRight = 0x0011,
      StartWithUTurn = 0x0012,
      UTurnRdbt = 0x0013,
      FollowRoad = 0x0014,
      EnterFerry = 0x0015,
      ExitFerry = 0x0016,
      ChangeFerry = 0x0017,
      EndOfRoadLeft = 0x0018,
      EndOfRoadRight = 0x0019,
      OffRampLeft = 0x001a,
      OffRampRight = 0x001b,
      Delta = 0x03fe,
      RouteActionMax = 0x03ff,
   };
{% endhighlight %}

####  RouteCrossing

{% highlight cpp %}
   enum RouteCrossing {
      NoCrossing,
      Crossing3Ways,
      Crossing4Ways,
      CrossingMultiway,
   };
{% endhighlight %}

#### SortingType

{% highlight cpp %}
   enum SortingType {
      alphabeticalOnName = 0x00,
      distance = 0x01,
      newSort = 0x02,
      invalidSortingType = 0xffff,
   };
{% endhighlight %}

####  UserMessageType

{% highlight cpp %}
   enum UserMessageType {
      invalidMessageType = 0,
      HTML_email = 1,
      non_HTML_email = 2,
      SMS = 3,
      MMS = 4,
      FAX = 6,
      InstantMessage = 5,
   };
{% endhighlight %}

####  VehicleType

{% highlight cpp %}
   enum VehicleType {
      passengerCar = 0x01,
      pedestrian = 0x02,
      emergencyVehicle = 0x03,
      taxi = 0x04,
      publicBus = 0x05,
      deliveryTruck = 0x06,
      transportTruck = 0x07,
      highOccupancyVehicle = 0x08,
      bicycle = 0x09,
      publicTransportation = 0x0a,
      invalidVehicleType = 0xff
   };
{% endhighlight %}

####  WayfinderType

{% highlight cpp %}
   enumWayfinderType {
      InvalidWayfinderType = -1,
      Trial = 0,
      Silver = 1,
      Gold = 2,
      Iron = 3,
   };
{% endhighlight %}

#### YesNoAsk 

{% highlight cpp %}
   enum YesNoAsk {
      yes = 0x00,
      no = 0x01,
      ask = 0x02,
      invalidYesNoAsk = 0x03,
   };
{% endhighlight %}

### Messages 

#### General Messages 

##### Error 

Error messages are sent from Nav2 to the UI when an unsolicited error the UI should know about is trapped in Nav2. This means that the error that occurred cannot be associated with any request sent from the UI. 

 | Data type    | type_and_data | 
 | ---------    | ------------- | 
 | Message type | ERROR         | 
 | Direction    | Nav2 -> UI    | 
 | Reply        | NONE          | 
 | Data:        |               | 
 | error number | uint32        | 
 | error string | string        | 

* `error number` A number in the enum ErrorNbr (see ), uniquely identifying the error.  
* `error string ` A text describing the error. 

##### Request Failed 

Request failed messages are sent when an error that can be associated with a specific request from the UI has occurred. The message makes it possible to inform the user that some action she took did not have any effect. 

 | Data type                   | type_and_data  | 
 | ---------                   | -------------  | 
 | Message type                | REQUEST_FAILED | 
 | Direction                   | Nav2 -> UI     | 
 | Reply                       | NONE           | 
 | Data:                       |                | 
 | error number                | uint32         | 
 | error string                | string         | 
 | failed request message ID   | uint16         | 
 | failed request message type | uint16         | 
 | failed request string       | string         | 

* ` error number` A number in the enum ErrorNbr (see ), uniquely identifying the error causing the request to fail. 
* `error string` A text describing the error causing the request to fail. 
* `failed request message ID` The ID of the message used for the request that failed. This is one of the numbers in the UI message number series, see . 
* `failed request message type ` The message type of the message used for the request that failed. This is one of the message types, see . 

##### Panic Abort 

Panic abort messages are sent from Nav2 when something has gone so wrong that it is no use to continue running. The UI should try to shut down the program as soon as possible. 

 | Data type    | only_type   | 
 | ---------    | ---------   | 
 | Message type | PANIC_ABORT | 
 | Direction    | Nav2 -> UI  | 
 | Reply        | NONE        | 
 | Data:        |             | 


##### Position Info Update 

Position info update messages are sent from Nav2 when a position update is received from the positioning service. 

 | Data type        | type_and_data        | 
 | ---------        | -------------        | 
 | Message type     | POSITION_INFO_UPDATE | 
 | Direction        | Nav2 -> UI           | 
 | Reply            | NONE                 | 
 | Data:            |                      | 
 | latitude         | uint32               | 
 | longitude        | uint32               | 
 | position quality | uint8                | 
 | heading          | uint8                | 
 | heading quality  | uint8                | 
 | speed            | uint16               | 
 | speed quality    | uint8                | 
 | altitude         | uint32               | 


* `latitude` The latitude of the latest position update. 
* `longitude` The longitude of the latest position update. 
* `position quality` The quality of the latest position update. This is one of the values in the enum Quality, see  
* `heading` The current heading. XXX 
* `heading quality` The quality of the heading. This is one of the values in the enum Quality, see  
* `speed` The current speed. XXX 
* `speed quality` The quality of the speed. This is one of the values in the enum Quality, see  
* `altitude` The current altitude in decimeters above calculated earth elipsoid. 

##### Send Message 

This message is used to send a Wayfinder object such as search results or itineraries to someone. The messages can be sent by email, SMS, MMS, and other formats. 

 | Data type         | type_and_data      | 
 | ---------         | -------------      | 
 | Message type      | SEND_MESSAGE       | 
 | Direction         | Nav2 `<- <->` UI     | 
 | Reply             | SEND_MESSAGE_REPLY | 
 | Data:             |                    | 
 | Message Media     | uint16             | 
 | Object Type       | uint16             | 
 | Object identifier | String             | 
 | From              | String             | 
 | To                | String             | 
 | Signature         | String             | 

* `Message Media ` One of the values from enum UserMessageType (see ) specifying what kind of message to send. 
* `Object Type ` One of the values from enum UserMessageType (see ) specifying what kind of object to send. 
* `Object identifier` A string identifier of th object to send. If the object only has a numeric identifier, this string should contain the string representation of that number. The usual C/C++ notation for octal and hexadecimal numbers are honored. 
* `From ` A string containing the appropriate address of the sender. This should be used if the receiver wants to reply to the message. 
* `To ` A string containing the address of the receiver. If there are more than one receiver, a comma separated list is acceptable. 
* `Signature` A string containing a signature to be placed last in the message. If this string is empty, the server will decide on a signature to use. 

##### Message Sent 

This is the reply for the 'Send Message' Message. 

 | Data type    | type_and_data      | 
 | ---------    | -------------      | 
 | Message type | SEND_MESSAGE_REPLY | 
 | Direction    | Nav2 `<- <->` UI     | 
 | Reply        | none               | 
 | Data:        |                    | 
 | Data length  | uint32             | 
 | Message data | data length        | 

If the server wasn't willing to send the message, the message content is included as payload in this message. The requesting party will then have to use whatever means available to send the message. 

#### Parameter Messages 

Parameter messages are messages used for setting and getting the values of parameters. When the UI wants a parameter value, it uses a get parameter message. Get parameter messages are replied to with either a set parameter message or a get parameter reply message. Get parameter reply messages are only used this way while set parameter messages are used also when the UI wants Nav2 to change the value of a parameter. 

When the value of a parameter has changed in Nav2, it tells the UI by sending a parameter changed message. 


##### Get General Parameter 

Get a parameter from Nav2. 

 | Data type      | type_and_uint16       | 
 | ---------      | ---------------       | 
 | Message type   | GET_GENERAL_PARAMETER | 
 | Direction      | Nav2 `<- <->` UI        | 
 | Reply          | GET_GENERAL_PARAMETER | 
 | Data:          |                       | 
 | parameter type | uint16                | 

An unset parameter is indicated by Nav2 by sending a GET_GENERAL_PARAMETER to the UI. 

##### Set General Parameter 

Set a parameter. Sent both from UI and Nav2. 


 | Data type      | type_and_data         | 
 | ---------      | -------------         | 
 | Message type   | SET_GENERAL_PARAMETER | 
 | Direction      | Nav2 `<->` UI           | 
 | Reply          |                       | 
 | Data:          |                       | 
 | parameter type | binary block          | 

The data format for each parameter is specified in a separate document. 

There is no reply to this message, but when sent from UI to Nav2, there will probably be a SET_GENERAL_PARAMETER sent from Nav2 when the parameter has been set by in the parameter module. However, this should not be relied upon. 


##### Get Top Region List -- OBSOLETE, DO NOT USE 

 | Data type    | only_type                 | 
 | ---------    | ---------                 | 
 | Message type | GET_TOP_REGION_LIST       | 
 | Direction    | Nav2 `<- <->` UI            | 
 | Reply        | GET_TOP_REGION_LIST_REPLY | 
 | Data:        |                           | 


##### Get Top Region List Reply -- OBSOLETE, DO NOT USE

Get top region list reply messages are sent from Nav2 to the UI when the UI has requested a top region list from Nav2. 

 | Data type                 | type_and_data             | 
 | ---------                 | -------------             | 
 | Message type              | GET_TOP_REGION_LIST_REPLY | 
 | Direction                 | Nav2 `<- <->` UI            | 
 | Reply                     | NONE                      | 
 | Data:                     |                           | 
 | top region list check sum | uint32                    | 
 | number of top regions     | uint32                    | 
 | top regions               | TopRegion[ ]              | 

* `top region list check sum` XXX 
* `number of top regions ` Number of top regions in the top regions vector. 
* `top regions ` Vector of top regions. 

 | TopRegion       |        | 
 | ---------       |        | 
 | top region ID   | uint32 | 
 | top region type | uint32 | 
 | top region name | string | 


* `top region ID ` The ID of the top region. This is the ID that is used in the search messages. 
* `top region type ` XXX. Some enum signifying different kinds of top regions, such as country or state. 
* `top region name ` The name of the top region. 

##### Get Call Center Numbers -- OBSOLETE, DO NOT USE

Get call center numbers messages are sent from the UI when it wants the value of the call center number parameter. 

 | Data type    | only_type               | 
 | ---------    | ---------               | 
 | Message type | GET_CALL_CENTER_NUMBERS | 
 | Direction    | Nav2 `<- <->` UI          | 
 | Reply        | SET_CALL_CENTER_NUMBERS | 
 | Data:        |                         | 

##### Set Call Center Numbers -- OBSOLETE, DO NOT USE 

Set call center numbers messages are used both as replies to get call center numbers messages and when the UI wants Nav2 to change the value of the call center number parameter. 

 | Data type           | type_and_data           | 
 | ---------           | -------------           | 
 | Message type        | SET_CALL_CENTER_NUMBERS | 
 | Direction           | Nav2 `<->` UI             | 
 | Reply               | NONE                    | 
 | Data:               |                         | 
 | call center numbers | string[8]               | 

* `call center numbers ` One string for each call center number. If not all eight string are used, the ones not used contains the empty string. 

   
##### Get Simple Parameter -- OBSOLETE, DO NOT USE

Get simple parameter messages are sent when the UI wants the value of a simple parameter. A simple parameter is a parameter, of which the value can be described using the message data types see subsection . 

 | Data type      | type_and_uint16      | 
 | ---------      | ---------------      | 
 | Message type   | GET_SIMPLE_PARAMETER | 
 | Direction      | Nav2 `<- <->` UI       | 
 | Reply          | SET_SIMPLE_PARAMETER | 
 | Data:          |                      | 
 | parameter type | uint16               | 

* `parameter type ` The type of parameter wanted. The number is one of the values of the enum ParameterType, see subsection . 


##### Set Simple Parameter -- OBSOLETE, DO NOT USE

Set simple parameter messages are used both as replies to get simple parameter messages and when the UI wants Nav2 to change the value of a simple parameter. A simple parameter is a parameter, of which the value can be described using the message data types see subsection . 

 | Data type       | only_type                      | 
 | ---------       | ---------                      | 
 | Message type    | SET_SET_SIMPLE_PARAMETER       | 
 | Direction       | Nav2 `<->` UI                    | 
 | Reply           | NONE                           | 
 | Data:           |                                | 
 | parameter value | Depends on the parameter type. | 
 | parameter type  | uint16                         | 

* `parameter value ` The formatting of this value depends on the parameter type. The different parameters use different data types for the values. (XXX describe what data type is connected to what parameter type in a table somewhere). These are the data types that the messages use, see . 
* `parameter type ` The parameter of which the value is contained in this message. 


##### Parameter Changed -- OBSOLETE, DO NOT USE

Parameter changed messages are sent from Nav2 to the UI when the value of a parameter has changed in Nav2. 

 | Data type      | type_and_uint16   | 
 | ---------      | ---------------   | 
 | Message type   | PARAMETER_CHANGED | 
 | Direction      | Nav2 -> UI        | 
 | Reply          | NONE              | 
 | Data:          |                   | 
 | parameter type | uint16            | 

* `parameter type` The parameter of which the value has changed. 


##### Synchronize Parameters 

Sent to Nav2 when the GUI wants to synchronize parameters such as WayfinderType, TopRegionList, Categories, and LatestNews. 

 | Data type    | only_type             | 
 | ---------    | ---------             | 
 | Message type | PARAMETERS_SYNC       | 
 | Direction    | Nav2 `<->` UI           | 
 | Reply        | PARAMETERS_SYNC_REPLY | 
 | Data:        |                       | 


##### Synchronize Parameters Reply 

Reply to a Synchronize Parameters message. Holds the post-sync value of WayfinderType. 

 | Data type     | type_and_uint32       | 
 | ---------     | ---------------       | 
 | Message type  | PARAMETERS_SYNC_REPLY | 
 | Direction     | Nav2 `<->` UI           | 
 | Reply         | NONE                  | 
 | Data:         |                       | 
 | WayfinderType | uint32                | 

WayfinderType should be one of the values from the WayfinderType enum, see . This value should be whatever the server thinks this client should have. 


##### FILEOP_GUI_MESSAGE

 | Data type    | type_and_data      | 
 | ---------    | -------------      | 
 | Message type | FILEOP_GUI_MESSAGE | 
 | Direction    | Nav2 -> UI         | 
 | Reply        | NONE               | 
 | Data:        |                    | 
 | Data         | binary block       | 

* `data ` Data interpreted by the file operation receiver. 


#### Favorite Messages 

Favorites are the items stored in My Destinations. Nav2 stores favorites locally in the navigation client and it is possible to synchronize these with favorites stored on the MC2 server. 
When the favorites in Nav2 have changed in any way the UI is notified with a favorites changed message. Favorite changed message are sent when for instance the sorting of the list or a favorite included in the list has changed. 


##### Get Favorites 

Get favorites messages are sent from the UI when it wants a list of favorites where each element keeps only a subset of the favorite's data, intended for showing the favorites in a simple list view in the UI. 

 | Data type    | type_and_two_uint16 | 
 | ---------    | ------------------- | 
 | Message type | GET_FAVORITES       | 
 | Direction    | Nav2 `<- <->` UI      | 
 | Reply        | GET_FAVORITES_REPLY | 
 | Data:        |                     | 
 | start        | index uint16        | 
 | end          | index uint16        | 

* `start index ` The index in the list of favorites in Nav2, using the current sort order, from where to start the list of favorites that will be contained in the reply. The value 0 indicates that the list of favorites contained in the reply should start in the beginning of the list in Nav2. 
* `end index ` The index in the list of favorites in Nav2, using the current sort order, where to end the list of favorites that will be contained in the reply. The value MAX_UINT16 indicates that the list of favorites contained in the reply should end at the end of the list in Nav2. 


##### Get Favorites Reply 

Get favorites reply messages are sent from Nav2 to the UI when the UI has requested a list of favorites with a get favorites message. 

 | Data type               | type_and_data       | 
 | ---------               | -------------       | 
 | Message type            | GET_FAVORITES_REPLY | 
 | Direction               | Nav2 -> UI          | 
 | Reply                   | NONE                | 
 | Data:                   |                     | 
 | number of GUI favorites | uint16              | 
 | GUI favorites           | GuiFavorite[ ]      | 

* `number of GUI favorites ` The number of GUI favorites contained in the GUI favorites vector. 
* `GUI favorites ` Vector (XXX perhaps vector is not a good word here. Used on many places, search for vector) of GUI favorites. 

 | GuiFavorite   |        | 
 | -----------   |        | 
 | favorite ID   | uint32 | 
 | favorite name | string | 

* `favorite ID ` The ID of the favorite. 
* `favorite name ` The name of the favorite. 


##### Get Favorites All Data 

Get favorites all data messages are sent from the UI to Nav2 when it wants a list of favorites including all data of the favorites. 

 | Data type    | type_and_two_uint16          | 
 | ---------    | -------------------          | 
 | Message type | GET_FAVORITES_ALL_DATA       | 
 | Direction    | Nav2 `<- <->` UI               | 
 | Reply        | GET_FAVORITES_ALL_DATA_REPLY | 
 | Data:        |                              | 
 | start index  | uint16                       | 
 | end index    | uint16                       | 

* `start index ` The index in the list of favorites in Nav2, using the current sort order, from where to start the list of favorites that will be contained in the reply. The value 0 indicates that the list of favorites contained in the reply should start in the beginning of the list in Nav2. 
* `end index ` The index in the list of favorites in Nav2, using the current sort order, where to end the list of favorites that will be contained in the reply. The value MAX_UINT16 indicates that the list of favorites contained in the reply should end at the end of the list in Nav2. 

##### Get Favorites All Data Reply 

Get favorites all data reply messages are sent from Nav2 to the UI when the UI has requested a list of favorites with a get favorites all data message. 

 | Data type           | type_and_data                | 
 | ---------           | -------------                | 
 | Message type        | GET_FAVORITES_ALL_DATA_REPLY | 
 | Direction           | Nav2 -> UI                   | 
 | Reply               | NONE                         | 
 | Data:               |                              | 
 | number of favorites | uint16                       | 
 | favorites           | Favorite[ ]                  | 

* `number of favorites ` The number of favorites in the favorites vector. 
* `favorites ` A vector of favorites. 

 | Favorite      |        | 
 | --------      |        | 
 | synced        | bool   | 
 | favorite ID   | uint32 | 
 | latitude      | uint32 | 
 | longitude     | uint32 | 
 | name          | string | 
 | short name    | string | 
 | description   | string | 
 | category      | string | 
 | map icon name | string | 

* `synced ` This bool tells whether the favorite has been synced with the server or not. 
* `favorite ID ` The ID of the favorite. 
* `latitude ` The latitude part of the favorite's position. 
* `longitude` The longitude part of the favorite's position. 
* `name ` The name of the favorite. 
* `short name ` The short name of the favorite. 
* `description` The favorite description. 
* `category ` The category of the favorite. 
* `map icon name ` The name of the favorite's map icon. 

##### Sort Favorites 

Sort favorites messages are sent from the UI to Nav2 when the UI wants Nav2 to change its favorites sorting order. 

 | Data type    | type_and_uint16 | 
 | ---------    | --------------- | 
 | Message type | SORT_FAVORITES  | 
 | Direction    | Nav2 `<- <->` UI  | 
 | Reply        | NONE            | 
 | Data:        |                 | 
 | sorting type | uint16          | 

* `sorting type ` The method used for sorting the favorites. This value is one of the values in the values in the enum SortingType, see subsection . 


##### Sync Favorites 

Sync favorites messages are sent from the UI to Nav2 when the UI wants Nav2 to synchronize the locally stored favorites with the favorites stored on the MC2 server. 

 | Data type    | only_type      | 
 | ---------    | ---------      | 
 | Message type | SYNC_FAVORITES | 
 | Direction    | Nav2 `<- <->` UI | 
 | Reply        | NONE           | 
 | Data:        |                | 


##### Get Favorite Info 

Get favorite info messages are sent from the UI to Nav2 when it wants all data of a favorite. The ID of the favorite which to get the data of typically comes from a get favorites reply. 

 | Data type    | type_and_uint32         | 
 | ---------    | ---------------         | 
 | Message type | GET_FAVORITE_INFO       | 
 | Direction    | Nav2 `<- <->` UI          | 
 | Reply        | GET_FAVORITE_INFO_REPLY | 
 | Data:        |                         | 
 | favorite ID  | uint32                  | 

* `favorite ID ` The ID of the favorite of which all the data is wanted. 


##### Get Favorite Info Reply

Get favorite info reply messages are sent from Nav2 to the UI when the UI has requested to get all data of a specific favorite using the get favorite info reply message. 

 | Data type    | type_and_data            | 
 | ---------    | -------------            | 
 | Message type | GET_FAVORITES_INFO_REPLY | 
 | Direction    | Nav2 -> UI               | 
 | Reply        | NONE                     | 
 | Data:        |                          | 
 | favorite     | Favorite                 | 

See subsection  for the definition of the Favorite structure. 

* `favorite ` This favorite structure contains all data Nav2 has on the favorite asked for. 


##### Add Favorite 

Add favorite messages are sent from the UI to Nav2 when the UI wants to add a favorite, setting the data of the favorite itself. This favorite will also be added on the MC2 server when the favorites are synchronized. 

 | Data type    | type_and_data  | 
 | ---------    | -------------  | 
 | Message type | ADD_FAVORITE   | 
 | Direction    | Nav2 `<- <->` UI | 
 | Reply        | NONE           | 
 | Data:        |                | 
 | favorite     | Favorite       | 

See subsection  for the definition of the Favorite structure. 

* `favorite ` This structure contains the data used when creating the favorite to add. 


##### Add Favorite from Search 

Add favorite from search messages are sent from the UI to Nav2 when the UI wants to add a favorite, using the data from a search match. This favorite will also be added on the MC2 server when the favorites are synchronized. 

 | Data type      | type_and_string          | 
 | ---------      | ---------------          | 
 | Message type   | ADD_FAVORITE_FROM_SEARCH | 
 | Direction      | Nav2 `<- <->` UI           | 
 | Reply          | NONE                     | 
 | Data:          |                          | 
 | search item ID | string                   | 

* `search item ID ` The ID of the search match of which the data is used when creating the favorite to add. This ID must be one of the IDs from a get search items reply message. 


##### Remove Favorite 

Remove favorite messages are sent from the UI to Nav2 when the UI wants Nav2 to remove a favorite. This favorite will also be removed from the MC2 server when the favorites are synchronized. 

 | Data type    | type_and_uint32 | 
 | ---------    | --------------- | 
 | Message type | REMOVE_FAVORITE | 
 | Direction    | Nav2 `<- <->` UI  | 
 | Reply        | NONE            | 
 | Data:        |                 | 
 | favorite ID  | uint32          | 

* `favorite ID ` The ID of the favorite to remove. 


##### Change Favorite 

Change favorite messages are sent from the UI to Nav2 when the UI wants Nav2 to change the data of a favorite. This favorite will also be changed on the MC2 server when the favorites are synchronized 

 | Data type    | type_and_data   | 
 | ---------    | -------------   | 
 | Message type | CHANGE_FAVORITE | 
 | Direction    | Nav2 `<- <->` UI  | 
 | Reply        | NONE            | 
 | Data:        |                 | 
 | favorite     | Favorite        | 

See subsection  for the definition of the Favorite structure. 

* `favorite ` The ID in this structure tells what favorite to change. The other data of the structure tells what data the changed favorite should have. 


##### Favorites Changed 

Favorite changed messages are sent from Nav2 to the UI every time any locally stored favorite, or the favorite list itself, is changed. Favorite changed message are sent for instance when the sorting of the list or the data of a favorite included in the list has changed. 

 | Data type    | only_type         | 
 | ---------    | ---------         | 
 | Message type | FAVORITES_CHANGED | 
 | Direction    | Nav2 `<- <->` UI    | 
 | Reply        | NONE              | 
 | Data:        |                   | 



#### Special Messages 

Special messages are messages that may not be have a meaning for all UI implementations. 

##### Connect GPS 

Connect GPS messages are sent from the UI to Nav2 when the UI wants Nav2 to connect to a physical GPS unit. 

 | Data type    | only_type      | 
 | ---------    | ---------      | 
 | Message type | CONNECT_GPS    | 
 | Direction    | Nav2 `<- <->` UI | 
 | Reply        | NONE           | 
 | Data:        |                | 

* `Disconnect GPS ` Disconnect GPS messages are sent from the UI to Nav2 when the UI wants Nav2 to disconnect a physical GPS unit. 

 | Data type    | only_type      | 
 | ---------    | ---------      | 
 | Message type | DISCONNECT_GPS | 
 | Direction    | Nav2 `<- <->` UI | 
 | Reply        | NONE           | 
 | Data:        |                | 


##### Progress Indicator 

Progress indicator messages are sent from Nav2 to the UI when the UI should turn on or turn off the progress indicator. The progress indicator is something in the UI telling the user that server communication or some other time consuming task is performed. 

 | Data type    | type_and_two_uint8 | 
 | ---------    | ------------------ | 
 | Message type | PROGRESS_INDICATOR | 
 | Direction    | Nav2 -> UI         | 
 | Reply        | NONE               | 
 | Data:        |                    | 
 | active       | uint8              | 
 | action       | uint8              | 

* `active` This uint8 tells whether the progress indicator should be active or not. If the value is non-zero it should be active, otherwise it should not be active. 
* `action ` This uint8 should hold a value from the ServerActionType enum, that describes the reason for the activity. 


##### License Upgrade Request 

Message sent to Nav2 when the user want to upgrade his application. 

 | Data type    | type_and_data           | 
 | ---------    | -------------           | 
 | Message type | REQUEST_LICENSE_UPGRADE | 
 | Direction    | Nav2 `<- <->` UI          | 
 | Reply        | LICENSE_UPGRADE_REPLY   | 
 | Data:        |                         | 
 | Region       | uint32                  | 
 | Key          | string                  | 
 | Phonenumber  | string                  | 
 | Name         | string                  | 
 | E-mail       | string                  | 

* `Region` A 32 bit id from the TopRegion list. 
* `Key ` A 20 character license key. 
* `Phonenumber ` The users mobile phone number. 
* `Name ` The users real name. 
* `E-mail ` The users email address. 


##### License Upgrade Reply 

Reply from the server whether the license upgrade was accepted or not. If all fields in the reply are true, the license was accepted. If not, one or more fields needs to be corrected before the license will be accepted. 

 | Data type     | type_and_data         | 
 | ---------     | -------------         | 
 | Message type  | LICENSE_UPGRADE_REPLY | 
 | Direction     | Nav2 `<- <->` UI        | 
 | Reply         | none                  | 
 | Data:         |                       | 
 | KeyOk         | bool                  | 
 | PhonenumberOk | bool                  | 
 | RegionOk      | bool                  | 
 | NameOk        | bool                  | 
 | EmailOk       | bool                  | 
 | Type          | uint8                 | 

* `KeyOk ` If false, the license key was either wrong or already in use with another phone. 
* `PhonenumberOk` If false, the server didn't accept the mobile phone number as a valid, international phone number. 
* `RegionOk ` If false, the user wasn't allowed to register his Wayfinder application with this region. It may be that the region is to small and therefore must is bundled with a neighboring region. 
* `NameOk ` If false, the server didn't accept the name of the user. 
* `EmailOk` 
* `Type ` The Wayfinder Type the user should have now according to the server. Should be one of the values from the WayfinderType enum (see ). 


##### Cell Info to Server 

The Cell Info Report message is used to send collected cell network information to the Navigator Server for processing. 

 | Data type    | type_and_length_and_data | 
 | ---------    | ------------------------ | 
 | Message type | CELL_INFO_TO_SERVER      | 
 | Direction    | Nav2 `<- <->` UI           | 
 | Reply        | CELL_INFO_FROM_SERVER    | 
 | Data:        |                          | 
 | Size         | uint32                   | 
 | Data         | binary block             | 


##### Cell Info from Server 

The Cell Info Confirm message is used as a reply from the Navigator Server when it has received a Cell Info Report. The Cell Info Confirm message may contain new data collection orders. 

 | Data type    | type_and_length_and_data | 
 | ---------    | ------------------------ | 
 | Message type | CELL_INFO_FROM_SERVER    | 
 | Direction    | Nav2 `<- <->` UI           | 
 | Reply        | none                     | 
 | Data:        |                          | 
 | Size         | uint32                   | 
 | Data         | binary block             | 


##### Tunnel Data 

Send tunnel data to the server. 

 | Data type    | type_and_length_and_data | 
 | ---------    | ------------------------ | 
 | Message type | TUNNEL_DATA              | 
 | Direction    | Nav2 `<- <->` UI           | 
 | Reply        | TUNNEL_DATA_REPLY        | 
 | Data:        |                          | 
 | Binary data  | data                     | 

##### Tunnel Data Reply 

Returns tunnel data from server. 

 | Data type       | type_and_length_and_data | 
 | ---------       | ------------------------ | 
 | Message type    | TUNNEL_DATA_REPLY        | 
 | Direction       | Nav2 -> UI               | 
 | Reply           | NONE                     | 
 | Data:           |                          | 
 | Vector map data | data                     | 


#### Route Messages 

Route messages are messages that is used for the route downloads and route following. 

##### Route Message 

Route Message is used as a replacement for the below messages and to add the possibility of setting the origin. 

 | Data type             | type_and_data     | 
 | ---------             | -------------     | 
 | Message type          | ROUTE_TO_FAVORITE | 
 | Direction             | Nav2 `<- <->` UI    | 
 | Reply                 | NONE              | 
 | Data:                 |                   | 
 | origin type           | uint8             | 
 | origin ID             | string            | 
 | origin latitude       | int32             | 
 | origin longitude      | int32             | 
 | destination type      | uint8             | 
 | destination ID        | string            | 
 | destination latitude  | int32             | 
 | destination longitude | int32             | 
 | destination name      | string            | 

* `origin type ` Type of the origin. One of the position type enum values from . 
* `origin ID ` Id of the origin, parsed according to type. 
* `origin latitude ` Latitude of the origin. 
* `origin longitude ` Longitude of the origin. 
* `destination type ` Type of the destination. One of the position type enum values from . 
* `destination ID ` Id of the destination, parsed according to type. 
* `destination latitude ` Latitude of the destination. 
* `destination longitude` Longitude of the destination. 
* `destination name ` Name of the destination. Only needed in cases where Nav2 does not have any information on the destination (such as destination via SMS). 

 | Position default values |             |          |           |           | 
 | ----------------------- |             |          |           |           | 
 | Position type           | ID          | Lat      | Long      | Dest name | 
 | PositionTypeInvalid     | MAXUINT32   | MAXINT32 | MAXINT32  | ""        | 
 | PositionTypeSearch      | Search ID   | MAXINT32 | MAXINT32  | ""        | 
 | PositionTypeFavorite    | Favorite ID | MAXINT32 | MAXINT32  | ""        | 
 |                         | (as string) |          |           |           | 
 | PositionTypePosition    | MAXUINT32   | Latitude | Longitude | String    | 
 | PositionTypeHotDest     | MAXUINT32   | MAXINT32 | MAXINT32  | ""        | 
 | PositionTypeCurrentPos  | MAXUINT32   | MAXINT32 | MAXINT32  | ""        | 

* `PositionTypeInvalid ` This value is not used. 
* `PositionTypeSearch ` Use position from the indicated search item. 
* `PositionTypeFavorite ` Use position from the indicated favorite. 
* `PositionTypePosition ` Use the specified position. 
* `PositionTypeHotDest ` Contact server for position value. 
* `PositionTypeCurrentPos` Use the latest position from the GPS. 

##### Route to Favorite 

Route to favorite messages are sent from the UI to Nav2 when the UI wants Nav2 to download a route to the destination represented by the favorite. 

 | Data type    | type_and_uint32   | 
 | ---------    | ---------------   | 
 | Message type | ROUTE_TO_FAVORITE | 
 | Direction    | Nav2 `<- <->` UI    | 
 | Reply        | NONE              | 
 | Data:        |                   | 
 | favorite ID  | uint32            | 

* `favorite ID ` The ID of the favorite to route to. 


##### Route to Hot Dest 

Route to hot dest messages are sent from the UI to Nav2 when the UI wants Nav2 to download a route to the destination represented by the favorite stored on the MC2 server that is marked as a hot dest. The hot dest mark can only be set on one favorite at a time and it is intended for making it easy to prepare navigation from a desktop computer. 

 | Data type    | only_type         | 
 | ---------    | ---------         | 
 | Message type | ROUTE_TO_HOT_DEST | 
 | Direction    | Nav2 `<- <->` UI    | 
 | Reply        | NONE              | 
 | Data:        |                   | 

##### Route to Search Item 

Route to search item messages are sent from the UI to Nav2 when the UI wants to start a new route to a search item from the previous search. 

 | Data type      | type_and_string      | 
 | ---------      | ---------------      | 
 | Message type   | ROUTE_TO_SEARCH_ITEM | 
 | Direction      | Nav2 `<- <->` UI       | 
 | Reply          | NONE                 | 
 | Data:          |                      | 
 | search item ID | string               | 

* `search item ID ` The ID of the search item to route to. 


##### Route to Position 

Route to position messages are sent from the UI to Nav2 when the UI wants to start a new route to a position given by latitude and longitude. 

 | Data type             | type_and_data     | 
 | ---------             | -------------     | 
 | Message type          | ROUTE_TO_POSITION | 
 | Direction             | Nav2 `<- <->` UI    | 
 | Reply                 | NONE              | 
 | Data:                 |                   | 
 | destination name      | string            | 
 | destination latitude  | uint32            | 
 | destination longitude | uint32            | 

* `destination name ` Name of the destination. This name will be use for telling the user what destination she is routing to. 
* `destination latitude ` The latitude of the position routed to. 
* `destination longitude ` The longitude of the position routed to. 


##### Started New Route 

Started new route messages are sent from Nav2 to the UI when Nav2 has started to follow a new route, i.e. if Nav2 followed a route before this message was sent, it indicates that the followed route has changed. It is also sent when Nav2 has reached the end of a route and has stopped following the route. In this case, the route id is 0 (zero), which is an invalid value for route ids. 

 | Data type               | type_and_data     | 
 | ---------               | -------------     | 
 | Message type            | STARTED_NEW_ROUTE | 
 | Direction               | Nav2 -> UI        | 
 | Reply                   | NONE              | 
 | Data:                   |                   | 
 | Route Id                | int64             | 
 | Bounding Box Top Lat    | int32             | 
 | Bounding Box Left Lon   | int32             | 
 | Bounding Box Bottom Lat | int32             | 
 | Bounding Box Right Lon  | int32             | 
 | Origin Latitude         | int32             | 
 | Origin Longitude        | int32             | 
 | Destination Latitude    | int32             | 
 | Destination Longitude   | int32             | 
 | Destination             | string            | 

* `Route Id ` The 64 bit route id for the route. 
* `Bounding Box Top Lat ` The latitude of the northern limit of the smallest possible bounding box that contains the entire route. 
* `Bounding Box Left Lon ` The latitude of the western limit of the smallest possible bounding box that contains the entire route. 
* `Bounding Box Bottom Lat ` The latitude of the southern limit of the smallest possible bounding box that contains the entire route. 
* `Bounding Box Right Lon ` The latitude of the eastern limit of the smallest possible bounding box that contains the entire route. 
* `Origin Latitude ` The latitude of the routes starting point. 
* `Origin Longitude ` The longitude of the routes starting point. 
* `Destination Latitude` The latitude of the routes end point. 
* `Destination Longitude ` The longitude of the routes end point. 
* `Destination ` A string description of the destination. Can be the name of a favorite or a search item description. 

#####  Reroute

Reroute messages are sent from the GUI to Nav2 whenever the GUI desires a new route to the same target. This is usually because the user has left the route and Nav2 has signaled with OffTrack status. 

 | Data type    | only_type  | 
 | ---------    | ---------  | 
 | Message type | REROUTE    | 
 | Direction    | Nav2 -> UI | 
 | Reply        | NONE       | 
 | Data:        |            | 


##### Invalidate route 

Used by the GUI to tell Nav2 to stop route following. 

 | Data type    | only_type        | 
 | ---------    | ---------        | 
 | Message type | INVALIDATE_ROUTE | 
 | Direction    | Nav2 -> UI       | 
 | Reply        | NONE             | 
 | Data:        |                  | 


##### Route Info Update 

Route info update messages are sent from Nav2 to the UI while a route is being followed. The messages are sent aperiodically, the frequency depends on what GPS data is received by Nav2. The route info update messages is meant to contain all information needed to inform the user about the route up to the next two turns, as well as the distance to the end of the route. 

 | Data type    | _type_and_data    | 
 | ---------    | --------------    | 
 | Message type | ROUTE_INFO_UPDATE | 
 | Direction    | Nav2 -> UI        | 
 | Reply        | NONE              | 
 | Data:        |                   | 
 | route info   | RouteInfo         | 
 | destination  | string            | 

* `route info ` An instance of the RouteInfo struct. 
* `destination ` A string informing the user where the route leads. Usually matches the destination string in the Route To Position() message, if that is how the route was requested. Routes to favorites has the favorite name as destination string and routes to search items get the destination name from the search item. 

 | RouteInfo                                 |          | 
 | ---------                                 |          | 
 | on track status                           | uint8    | 
 | distance to waypoint                      | int32    | 
 | distance to track                         | int32    | 
 | distance to goal                          | int32    | 
 | to on track turn                          | uint8    | 
 | to track direction                        | uint8    | 
 | time to waypoint                          | uint16   | 
 | latency                                   | uint16   | 
 | speed                                     | uint16   | 
 | overSpeed                                 | bool     | 
 | current segment                           | Segment  | 
 | distance to alternative attribute segment | int32    | 
 | alternative attribute segment             | Segment  | 
 | current crossing                          | Crossing | 
 | next segment 1                            | Segment  | 
 | next crossing                             | Crossing | 
 | next segment 2                            | Segment  | 

* `on track status ` A value from the OnTrackState enum specified in . 
* `distance to waypoint` The distance in meters to the next turn or action. 
* `distance to goal ` The distance to the end of the route. 
* `to on track turn ` The compass heading to turn once the route is reached again. Note that this parameter only matters when on track status is OffTrack. Also note that this compass only goes from 0 to 255 where 0 is north and 64 is east. 
* `to track direction` The compass heading to the closest point of the route. Note that this parameter only matters when on track status is OffTrack. Also note that this compass only goes from 0 to 255 where 0 is north and 64 is east. 
* `time to waypoint` The estimated time until the next turn or action is reached. Measured in tenth of a second. 
* `latency ` Estimated latency of the information in this struct. Measured in tenth of a second. 
* `speed ` The current speed. Measured i m/s times 32. 
* `overSpeed ` Indicates if the user is traveling faster than the current speed limit. 
* `current segment ` The road segment the user is currently traveling on. Detailed in the Segment struct below. 
* `distance to alternative attribute segment` The distance until the current segment ends and the alternative attribute segment start. If negative, the alternative attribute segment is filled with garbage. 
* `alternative attribute segment ` Used when the road from the users current position and the next turn or action changes in some fundamental way. Changes may be any information contained in a Segment struct. 
* `current crossing ` The next crossing or action on the route. Detailed in the Crossing struct below. 
* `next segment 1 ` Describes the road between the next crossing and the crossing after that. 
* `next crossing ` Describes the crossing after the next one. 
* `next segment 2 ` Describes the road segment two crossings from now. 

 | Segment     |        | 
 | -------     |        | 
 | street name | string | 
 | speed limit | uint16 | 
 | flags       | uint8  | 
 | valid 1     | byte   | 

* `street name ` The street name of the road segment. 
* `speed limit ` The speed limit of the road segment. Measured in 32 times m/s. 
* `flags ` Information about the road type. The least significant bit signifies highway. No other bits are defined yet. 
* `valid ` Indicates that the struct is filled with valid information and not garbage. 

 | Crossing                  |        | 
 | --------                  |        | 
 | action                    | uint32 | 
 | distance to next crossing | uint32 | 
 | crossing type             | uint8  | 
 | exit count                | uint8  | 
 | valid                     | 1 byte | 

* `action ` The action to take in this crossing. One of the RouteAction enum values from . 
* `distance to next crossing ` Distance in meters to the next crossing of the route. 
* `crossing type ` The action to take in this crossing. One of the RouteAction enum values from . 
* `exit count ` Number of exits from a crossing. Only valid if crossing type is CrossingMultiway. 
* `valid ` Indicates that the struct is filled with valid information and not garbage. 

#### Search Messages 

Search messages are used for handling searches for destinations in the MC2 server. Searching is done in two steps. The first step is to identify the area to search in. This is done by using a query string that matches the search area, or by calculating which search area the user is located in. The search area the user is located in is calculated using the current position data. The second step is to search for the search item, that represent actual destinations. This step can only be performed when the search area has been uniquely identified because the search items are searched for using the search area to identify where in the map to search. If the search area query results in a unique search area directly, both these steps are performed by a single search message. Otherwise, it may be needed to ask the user, which of the matched search areas she intended to search in and send another search message with the search area ID. 

When the result from a search is received by Nav2, a search result changed message is sent to the UI. Depending on what search matches that was received, the UI can use either a get search areas message or a get search items message to fetch the result from Nav2. 


##### Search 

Search messages are sent from the UI to Nav2 when the UI wants to search for destinations in the MC2 server. The message contains data telling the server what to how to search. 

 | Data type         | type_and_data  | 
 | ---------         | -------------  | 
 | Message type      | SEARCH         | 
 | Direction         | Nav2 `<- <->` UI | 
 | Reply             | NONE           | 
 | Data:             |                | 
 | country ID        | uint32         | 
 | search area ID    | string         | 
 | search area query | string         | 
 | search item query | string         | 

* `country ID ` Zero based index. The invalid value MAX_UINT32 means that the current location should be used for determining the country. The valid values are stored in the top region parameter. 
* `search area ID ` A string that uniquely identifies the search area. This value is always used unless it is set to the empty string, ```. 
* `search area query` A string used for finding search areas, which match the query string. This value is only used when the search area ID is set to the empty string. Setting this string to the empty string, ```, means that the search area should be found using the current location, i.e. when the search area should be found using the current location, both the search area ID and the search area query should be set to the empty string. 
* `search item query ` A string used for finding search items that match the query string. This string is only used if the search area ID has been set to a valid ID or the search area query results in a unique search area match. 


##### Search Result Changed 

Search result changed messages are sent from Nav2 to the UI when Nav2 has received new results from a search. 

 | Data type            | type_and_two_bool     | 
 | ---------            | -----------------     | 
 | Message type         | SEARCH_RESULT_CHANGED | 
 | Direction            | Nav2 -> UI            | 
 | Reply                | NONE                  | 
 | Data:                |                       | 
 | search areas changed | bool                  | 
 | search items changed | bool                  | 

* `search areas changed ` If this values is set to true, new search areas has been received. 
* `search items changed ` If this values is set to true, new search items has been received. 

This table explains when areas and items are marked as changed. 

 | Areas / Items                         |    | Received |          |    | 
 | -------------                         |    | -------- |          |    | 
 | :::                                   |    | 0        | 1        | >1 | 
 | Cached            |0  |F |T      |T |
 | :::                                   | 1  | F        | not T    | T  | 
 | :::                                   | >1 | F        | not in T | T  | 

Almost all cases in the table are self explanatory. In the case of one received and one cached, items are always marked as changed. Areas are only marked as changed if the one received area is not equal to the one cached area. 
In the case of one received and multiple cached, items are always marked as changed. Areas are marked as changed only if the one received area is not in the set of cached areas. 


##### Get Search Areas 

Get search areas messages are sent from the UI to Nav2 when the UI wants the search areas of the search result most recently received by Nav2. 

 | Data type    | only_type              | 
 | ---------    | ---------              | 
 | Message type | GET_SEARCH_AREAS       | 
 | Direction    | Nav2 `<- <->` UI         | 
 | Reply        | GET_SEARCH_AREAS_REPLY | 
 | Data:        |                        | 


##### Get Search Areas Reply 

Get search areas reply messages are sent from Nav2 to the UI when the UI has requested the search areas with a get search area message. These search areas are invalid when a search result changed message indicating that search areas have changed is sent from Nav2. 

 | Data type              | type_and_data               | 
 | ---------              | -------------               | 
 | Message type           | GET_SEARCH_AREAS_REPLY      | 
 | Direction              | Nav2 rightleftrightright UI | 
 | Reply                  | NONE                        | 
 | Data:                  |                             | 
 | number of search areas | uint32                      | 
 | search areas           | SearchArea[ ]               | 

* `Number of search areas ` Number of search areas in the search area vector. 
* `search areas ` Vector of search areas. 

 | SearchArea       |        | 
 | ----------       |        | 
 | search area name | string | 
 | search area ID   | string | 

* `search area name ` Name of the search area. 
* `search area ID ` Unique ID of the search area. Use this ID in the search messages. 


##### Get Search Items 

Get search items messages are sent from the UI to Nav2 when the UI wants the search items of the search result most recently received by Nav2. 

   Data type               type_and_uint16 
   Message type            GET_SEARCH_ITEMS 
   Direction               Nav2 `<- <->` UI 
   Reply                   GET_SEARCH_ITEMS_REPLY 
   Data: 
   Index of first item     uint16 

The index variable is an offset into the grand master vector of search items maintained by the server. On the first request of Search Items after a search it should always be zero. 

Be aware that every request with an index outside the currently held subvector will result in a server request. 

Indexes larger than the total available number of items will result in an error. 

##### Get Search Items Reply 

Get search items reply messages are sent from Nav2 to the UI when the UI has requested the search items with a get search items message. These search items are invalid when a search result changed message indicating that search items have changed is sent from Nav2. 

 | Data type                | type_and_data          | 
 | ---------                | -------------          | 
 | Message type             | GET_SEARCH_ITEMS_REPLY | 
 | Direction                | Nav2 -> UI             | 
 | Reply                    | NONE                   | 
 | Data:                    |                        | 
 | Total hits               | uint16                 | 
 | Index from total         | uint16                 | 
 | number of search matches | uint32                 | 
 | search matches           | SearchMatch[ ]         | 

* `Total hits ` The number of items actually found by the server. The items in this reply are a subset of those. 
* `Index from total ` The index the first item in this message would have in the full set of items found by the server. 
* `Number of search items ` Number of search items in the search items vector. 
* `Search items ` Vector of search items. 
    
 | SearchMatch                  |           | 
 | -----------                  |           | 
 | search match name            | string    | 
 | search match ID              | string    | 
 | search match type            | uint16    | 
 | search match sub type        | uint16    | 
 | distance to current position | uint32    | 
 | number of search regions     | uint8     | 
 | search region list           | Region[ ] | 
 | position latitude            | int32     | 
 | position longitude           | int32     | 

search item name 

* `search item ID ` Unique ID of the search item. Use this ID in the route to search item message. 
* `search match type ` The type of this search match. The value can be used to select an appropriate icon for the search match. 
* `search match sub type ` The sub type of the search match. The value can be used to select an appropriate icon for the search match. 
* `distance to current position ` The distance from the search hit in meters from the current (or last known position). If no position is available, this value is set to 0xffffffff and should not be shown in the search result list. 
* `number of search regions ` The number of regions that the search hit is associated with. 
* `search region list ` A list of structures which contains the data of the regions that the search hit is associated with. This list is sorted with the smallest region first and contains only the regions which the search hit uniquely matches (i.e. if the search hit is associated with multiple regions of the same level, then these regions will not be in the region list). It is the responsibility of the server to select the appropriate regions. 
* `position latitude ` The latitude of the search match. 
* `position longitude ` The longitude of the search match. 

 | Region      |        | 
 | ------      |        | 
 | region type | uint32 | 
 | region id   | string | 
 | region name | string | 

* `region type ` The type of the region. A value from the RegionType enum, see . 
* `region id ` An id relevant to the region. 
* `region name ` The name string for this region. 

Example: 

         streetnumber         "1"
         address              "Baravage"
         city part            "Mollevagen"
         city                 "Lund"
         municipal            "LUND"
         county               "Skane"
         state                ""
         country              "Sweden"
         zipcode              "222 40"


##### Get Full Search Data 

Get full search data messages are sent from the UI to Nav2 when it wants a list of search matches including all data for each search match. 

 | Data type    | type_and_two_uint16       | 
 | ---------    | -------------------       | 
 | Message type | GET_SEARCH_ALL_DATA       | 
 | Direction    | Nav2 `<- <->` UI            | 
 | Reply        | GET_SEARCH_ALL_DATA_REPLY | 
 | Data:        |                           | 
 | start index  | uint16                    | 
 | end index    | uint16                    | 

* `start index ` The index in the list for the latest search matches in Nav2, using the current sort order, from where to start the list of search matches that will be contained in the reply.  The value 0 indicates that the list of search matches contained in the reply should start in the beginning of the list in Nav2. 
* `end index ` The index in the list for the latest search matches in Nav2, using the current sort order, where to end the list of search matches that will be contained in the reply. The value MAX_UINT16 indicates that the list of search matches contained in the reply should end at the end of the list in Nav2. 


##### Get Full Search Data Reply 

Get search all data reply messages are sent from Nav2 to the UI when the UI has requested a list of search matches with a get search all data message. 

 | Data type                | type_and_data               | 
 | ---------                | -------------               | 
 | Message type             | GET_FULL_SEARCH_DATA_REPLY  | 
 | Direction                | Nav2 rightleftrightright UI | 
 | Reply NONE               |                             | 
 | Data:                    |                             | 
 | number of search matches | uint16                      | 
 | search matches           | FullSearchMatch[ ]          | 

* `number of search matches ` The number of search matches in the search matches vector. 
* `search matches ` A vector of search matches. 

 | FullSearchMatch            |                   | 
 | ---------------            |                   | 
 | Ordinary match data        | SearchMatch       | 
 | number of additional info  | uint8             | 
 | additional info            | AdditionalInfo[ ] | 
 | position altitude          | uint32            | 
 | search module status flags | uint32            | 

* `Ordinary match data ` All the data from the associated SearchMatch (see ) is replicated here. 
* `number of additional info` The number of additional info structures included. If this value is zero, no additional info is included. 
* `additional info ` Additional info structure. This structure is used to define additional information that should be presented in a full view. Each item is a key-value pair. 
* `position altitude ` The altitude value of the search match, in meters above sea-level. If unknown, the value is set to zero. 
* `search module status flags` Flags which indicate status from the search module. 

 | AdditionalInfo |        | 
 | -------------- |        | 
 | key            | string | 
 | value          | string | 
 | type           | uint32 | 


* `key ` String which to be presented as first value or key. 
* `value ` String which to be presented as second value. 
* `type ` This value defines the type of the data. Most data are in the "text" format, but the "phonenumber" and "URL" types are also available. See below for list. Use values from the AdditionalInfoType, see . 

The client can use the type information to perform an appropriate action, such as offering a new menu option that allows the user to call the phone number. If a client receives a type other than the ones it understands, it will show it as text. 


##### Get More Search Data 

In some cases the AdditionalInfo items in a FullSearchItem have the type more. The GET_MORE_SEARCH_DATA message is used to request even more data from the server about one such FullSearchItem. 

 | Data type    | type_and_data              | 
 | ---------    | -------------              | 
 | Message type | GET_MORE_SEARCH_DATA       | 
 | Direction    | Nav2 `<- <->` UI             | 
 | Reply        | GET_FULL_SEARCH_DATA_REPLY | 
 | Data:        |                            | 
 | Index        | uint16                     | 
 | Value        | string                     | 

* `Index ` The index of the FullSearchItem in the current sort order. 
* `Value ` The Value of the AdditionalInfo item with type more. 


##### Get Full Search Data From ItemID 

Request full search data using a server string id. Actually not related to search any search reply. 

 | Data type      | type_and_string                        | 
 | ---------      | ---------------                        | 
 | Message type   | GET_FULL_SEARCH_DATA_FROM_ITEMID       | 
 | Direction      | Nav2 `<- <->` UI                         | 
 | Reply          | GET_FULL_SEARCH_DATA_FROM_ITEMID_REPLY | 
 | Data:          |                                        | 
 | Server item id | string                                 | 


##### Get Full Search Data From ItemID Reply 

Returns the full search data. 

 | Data type                              | type_and_data                          | 
 | ---------                              | -------------                          | 
 | Message type                           | GET_FULL_SEARCH_DATA_FROM_ITEMID_REPLY | 
 | Direction                              | Nav2 rightleftrightright UI            | 
 | Reply                                  | NONE                                   | 
 | Data:                                  |                                        | 
 | GuiProtMess class = FullSearchDataMess | data                                   | 


#### Map Messages 

Map messages are messages used for requesting and receiving map images from Nav2. 

##### Get Map 

Get map messages are sent from the UI to Nav2 when the UI wants a map image of some area. 

 | Data type       | type_and_data  | 
 | ---------       | -------------  | 
 | Message type    | GET_MAP        | 
 | Direction       | Nav2 `<- <->` UI | 
 | Reply           | GET_MAP_REPLY  | 
 | Data:           |                | 
 | Bounding Box    | BoundingBox    | 
 | Image width     | uint16         | 
 | Image height    | uint16         | 
 | Viewbox width   | uint16         | 
 | Viewbox height  | uint16         | 
 | Image Format    | uint16         | 
 | Number of items | uint16         | 
 | Items           | MapItem[ ]     | 
 | Number of info  | uint16         | 
 | Infos           | ExtraInfo[ ]   | 

* `Bounding Box ` Describes to the server how to select the bounding box for the map. 
* `Image width ` The image width in pixels. 
* `Image height ` The image height in pixels. 
* `Viewbox width ` The viewbox width in pixels. 
* `Viewbox height ` The viewbox height in pixels. 
* `Image Format ` The file format of the image. Select a value from the ImageFormat enum (see ). 
* `Number of items` The number of items to mark in the map. 
* `Items ` Information on the items to mark. 
* `Number of info` Number of extra information to the map drawing algorithm. 
* `Infos ` Extra instructions to the map drawing algorithm. 

##### Bounding Box 

The bounding box should be one of the non-abstract classes from the BoundingBox hierarchy shown in figure below. 

{{:nav2_ui_prot_boundingboxes2.png?800|}}

* `BoxBox ` The bounding box is defined by two coordinates, the top left corner and the bottom left corner. 
* `VectorBox ` The server calculates a bounding box from the users position, speed, and heading. The client can supply a wished-for real world width of the map image, but the server is free to override it if the width is inappropriate. 
* `RadiusBox ` Defines a circle which is should be included in the map image. The server has some freedom in selecting a bounding box that contains this circle. 
* `RouteBox ` The server calculates a bounding box that can hold the specified route. 

All boundingboxes are serialized to 16 bytes according to the table below. The first two bytes are a instance of the enum BoxType. 

 | BoundingBox           |||||
 | ---------------------------
 | Byte offset                 | BoxBox     | VectorBox        | DiameterBox        | RouteBox | 
 | 0                           | 0x01       | 0x02             | 0x03               | 0x04     | 
 | 2                           | Top Lat    | Position lat     | Position lat       | Route id | 
 | 4                           |            |                  |                    |          | 
 | 6                           | Left Lon   | Position lon     | Position lon       |          | 
 | 8                           |            |                  |                    |          | 
 | 10                          | Bottom Lat | Speed (m/s)      | Map cover diameter | 0        | 
 | 12                          |            | Heading 0        |                    |          | 
 | 14                          | Right Lon  | Real world width | 0                  |          | 
 | 16                          |            |                  |                    |          | 

Note that the heading field in the VectorBox is 8 bits wide, and the 8 bits at offset 13 should be set to 0. 


##### MapItem 

The MapItems can be any of the non-abstract classes from the MapItem hierarchy shown in the figure below. 

{{:nav2_ui_prot_mapitem2.png?400|}}

The type field should hold one of the values from the MapItemType enum (see ). All types but MapRouteItem use the PositionItem class. 

MapItems are serialized as follows. 

 | MapItem     |              |              | 
 | -------     |              |              | 
 | Byte offset | PositionItem | RouteItem    | 
 | 0           | Type         | Route        | 
 | 2           | Latitude     | Route id MSW | 
 | 4           |              |              | 
 | 6           | Longitude    | Route id LSW | 
 | 8           |              |              | 


##### MapInfo 

MapInfo holds information to the map drawing algorithm about what information to include in the map, and also about what colors to use and any transformations to the map, such as rotation. 

 | MapInfo |        | 
 | ------- |        | 
 | Type    | uint16 | 
 | Value   | uint32 | 

* `Type ` One of the values from the MapInfoType enum (see ). 
* `value ` 32 bits packed with information. What information depends on the type value. 
* `Category ` The value identifies a single category, and all items of that category found in the boundingbox should be marked in the map. 
* `TrafficInformation ` The value defines what traffic information should be displayed in the map and how. 
* `Ruler ` The value doesn't matter. If a MapInfo item with this type is present a scale ruler shall be included in the image. 
* `Topographic ` Include topographic information in the map. The value contains parameters controlling this. 
* `MapFormat ` selects palette and similar. 
* `Rotate ` Rotates the map clockwise. The value defines the parts of a lap that the map should be rotated. There are 256 parts on a lap. 

##### Get Map Reply 

 | Data type         | type_and_data  | 
 | ---------         | -------------  | 
 | Message type      | GET_MAP_REPLY  | 
 | Direction         | Nav2 `<- <->` UI | 
 | Reply             | none           | 
 | Data:             |                | 
 | Top Latitude      | int32          | 
 | Left Longitude    | int32          | 
 | Bottom Latitude   | int32          | 
 | Right Longitude   | int32          | 
 | Image width       | uint16         | 
 | Image height      | uint16         | 
 | Real world width  | uint32         | 
 | Real world height | uint32         | 
 | Image Buffer size | uint32         | 
 | Image Buffer type | uint16         | 
 | Image buffer      | uint8[ ]       | 

* `Top Latitude ` The latitude of the maps top left corner. 
* `Left Longitude ` The longitude of the maps top left corner. 
* `Bottom Latitude ` The latitude of the maps bottom right corner. 
* `Right Longitude ` The longitude of the maps bottom right corner. 
* `Image width ` The image width in pixels. 
* `Image height ` The image height in pixels. 
* `Real world width ` The map contents width in meters. 
* `Real world height ` The map contents height in meters. 
* `Image Buffer size ` The size of the image buffer. 
* `Image Buffer type ` The image format (see ). 
* `Image buffer ` The image data. 

#### Vector map messages 

##### Get Vector Map 

Request a vectormap from the server. 

 | Data type                  | type_and_string      | 
 | ---------                  | ---------------      | 
 | Message type               | GET_VECTOR_MAP       | 
 | Direction                  | Nav2 `<- <->` UI       | 
 | Reply                      | GET_VECTOR_MAP_REPLY | 
 | Data:                      |                      | 
 | Id of vector map on server | string               | 


##### Get Vector Map Reply 

Returns the vector map data from server. 

 | Data type       | type_and_length_and_data | 
 | ---------       | ------------------------ | 
 | Message type    | GET_VECTOR_MAP_REPLY     | 
 | Direction       | Nav2 -> UI               | 
 | Reply           | NONE                     | 
 | Data:           |                          | 
 | Vector map data | data                     | 


##### Get Multi Vector Map 

Request a number of vectormaps from the server. 

 | Data type    | type_and_length_and_data   | 
 | ---------    | ------------------------   | 
 | Message type | GET_MULTI_VECTOR_MAP       | 
 | Direction    | Nav2 `<- <->` UI             | 
 | Reply        | GET_MULTI_VECTOR_MAP_REPLY | 
 | Data:        |                            | 
 | Binary data  | data                       | 


##### Get Multi Vector Map Reply 

Returns the vector map data from server. 

 | Data type       | type_and_length_and_data   | 
 | ---------       | ------------------------   | 
 | Message type    | GET_MULTI_VECTOR_MAP_REPLY | 
 | Direction       | Nav2 -> UI                 | 
 | Reply           | NONE                       | 
 | Data:           |                            | 
 | Vector map data | data                       | 


### GuiProt 4 General Settings

This is the specification of all General settings (parameters) used in the Nav2 UI protocol, and their respective Nav2 parameters. 

This document describes the different parameters that can be used within Nav2. Parameters are used for permanent storage of settings, for example saving the favorites on the device. 

The way to use the parameters is pretty straight forward. 
 1.  UI sends a GeneralParameterMessage with the type paramAutoReroute
 2.  The Parameter Module inside Nav2 parses the message and creates a reply
 3.  The reply is sent to the UI. The reply contains the data and the same ID as in the request.

##### Format

The messages are sent to mirror the underlying parameter from Nav2. Each message includes the following fields: 

 | Name         | Size   | 
 | ----         | ----   | 
 | Type         | uint16 | 
 | ParamID      | uint16 | 
 | NumEntries   | uint32 | 
 | rest of data | varies | 

The type can be one of the following: 

 | Type            | Array of | 
 | ----            | -------- | 
 | paramTypeInt32  | int32    | 
 | paramTypeFloat  | float    | 
 | paramTypeString | strings  | 
 | paramTypeBinary | bytes    | 

Message thus contains **NumEntries** entries of type **Type**. 


##### Helper enums

A number of enums have been specified to allow easier definition of the parameters: 


###### Bool

{% highlight cpp %}
   enum Bool {
      false                                  = 0x00,
      true                                   = 0x01,
   };
{% endhighlight %}

###### SortingType

{% highlight cpp %}
   enum SortingType {
      alphabeticalOnName = 0x00,
      distance = 0x01,
      newSort = 0x02,
      invalidSortingType = 0xffff,
   };
{% endhighlight %}

###### YesNoAsk

{% highlight cpp %}
   enum YesNoAsk {
      yes = 0x00,
      no = 0x01,
      ask = 0x02,
      invalidYesNoAsk = 0x03,
   };
{% endhighlight %}

###### BacklightStrategy

{% highlight cpp %}
   enum BacklightStrategy {
      backlight_always_off = 0x00,
      backlight_on_during_route = 0x01,
      backlight_always_on = 0x02,
      backlight_near_action = 0x03,
      backlight_invalid = 0xff,
   };
{% endhighlight %}

###### TurnSoundsLevel

{% highlight cpp %}
   enum TurnSoundsLevel {
      turnsound_mute = 0x00,
      turnsound_min = 0x01,
      turnsound_less = 0x02,
      turnsound_normal = 0x03,
      turnsound_more = 0x04,
      turnsound_max = 0x05,
      turnsound_invalid = 0xff
   };
{% endhighlight %}

###### DistanceMode

{% highlight cpp %}
   enum DistanceMode {
      ModeInvalid = 0,
      ModeMetric = 1,
      ModeImperialYards = 2,
      ModeImperialFeet = 3,
      ModeMetricSpace = 4,
      ModeImperialYardsSpace = 5,
      ModeImperialFeetSpace = 6,
   };
{% endhighlight %}

###### ShowFavoriteInMap

{% highlight cpp %}
   enum ShowFavoriteInMap {
      ShowFavoriteInMapAlways = 0x00,
      ShowFavoriteInMapCityLevel = 0x01,
      ShowFavoriteInMapNever = 0x02,
   };
{% endhighlight %}

###### RouteCostType

{% highlight cpp %}
   enum RouteCostType {
      DISTANCE = 0,
      TIME = 1,
      TIME_WITH_DISTURBANCES = 2,
      INAVLID = 0xff
   };
{% endhighlight %}

###### RouteTollRoads

{% highlight cpp %}
   enum RouteTollRoads {
      TollRoadsAllow = 0,
      TollRoadsDeny = 1,
   };
{% endhighlight %}

###### RouteHighways

{% highlight cpp %}
   enum RouteHighways {
      HighwaysAllow = 0,
      HighwaysDeny = 1,
   };
{% endhighlight %}

###### languageCode

{% highlight cpp %}
   enum languageCode {
      ENGLISH = 0,
      SWEDISH = 1,
      GERMAN = 2,
      DANISH = 3,
      FINNISH = 4,
      NORWEGIAN = 5,
      ITALIAN = 6,
      DUTCH = 7,
      SPANISH = 8,
      FRENCH = 9,
      WELCH = 10,
      PORTUGUESE = 11,
      CZECH = 12,
      AMERICAN_ENGLISH = 13,
      HUNGARIAN = 14,
      GREEK = 15,
      POLISH = 16,
      SLOVAK = 17,
      RUSSIAN = 18,
      SLOVENIAN = 19,
      TURKISH = 20,
      ARABIC = 21,
      SWISS_FRENCH = 22,
      SWISS_GERMAN = 23,
      ICELANDIC = 24,
      BELGIAN_FLEMISH = 25,
      AUSTRALIAN_ENGLISH = 26,
      BELGIAN_FRENCH = 27,
      AUSTRIAN_GERMAN = 28,
      NEW_ZEALAND_ENGLISH = 29,
      CHINESE_TAIWAN = 30,
      CHINESE_HONG_KONG = 31,
      CHINESE_PRC = 32,
      JAPANESE = 33,
      THAI = 34,
      AFRIKAANS = 35,
      ALBANIAN = 36,
      AMHARIC = 37,
      ARMENIAN = 38,
      TAGALOG = 39,
      BELARUSIAN = 40,
      BENGALI = 41,
      BULGARIAN = 42,
      BURMESE = 43,
      CATALAN = 44,
      CROATIAN = 45,
      CANADIAN_ENGLISH = 46,
      SOUTH_AFRICAN_ENGLISH = 47,
      ESTONIAN = 48,
      FARSI = 49,
      CANADIAN_FRENCH = 50,
      GAELIC = 51,
      GEORGIAN = 52,
      GREEK_CYPRUS = 53,
      GUJARATI = 54,
      HEBREW = 55,
      HINDI = 56,
      INDONESIAN = 57,
      IRISH = 58,
      SWISS_ITALIAN = 59,
      KANNADA = 60,
      KAZAKH = 61,
      KHMER = 62,
      KOREAN = 63,
      LAO = 64,
      LATVIAN = 65,
      LITHUANIAN = 66,
      MACEDONIAN = 67,
      MALAY = 68,
      MALAYALAM = 69,
      MARATHI = 70,
      MOLDOVIAN = 71,
      MONGOLIAN = 72,
      NYNORSK = 73,
      BRAZILIAN_PORTUGUESE = 74,
      PUNJABI = 75,
      ROMANIAN = 76,
      SERBIAN = 77,
      SINHALESE = 78,
      SOMALI = 79,
      LATIN_AMERICAN_SPANISH = 80,
      SWAHILI = 81,
      FINNISH_SWEDISH = 82,
      TAMIL = 83,
      TELUGU = 84,
      TIBETAN = 85,
      TIGRINYA = 86,
      CYPRUS_TURKISH = 87,
      TURKMEN = 88,
      UKRAINIAN = 89,
      URDU = 90,
      VIETNAMESE = 91,
      ZULU = 92,
      SESOTHO = 93,
      BASQUE = 94,
      GALICIAN = 95,
      ASIA_PACIFIC_ENGLISH = 96,
      TAIWAN_ENGLISH = 97,
      HONG_KONG_ENGLISH = 98,
      CHINA_ENGLISH = 99,
      JAPAN_ENGLISH = 100,
      THAI_ENGLISH = 101,
      ASIA_PACIFIC_MALAY = 102,
   };
{% endhighlight %}

###### VehicleType

{% highlight cpp %}
   enum VehicleType {
      passengerCar = 0x01,
      pedestrian = 0x02,
      emergencyVehicle = 0x03,
      taxi = 0x04,
      publicBus = 0x05,
      deliveryTruck = 0x06,
      transportTruck = 0x07,
      highOccupancyVehicle = 0x08,
      bicycle = 0x09,
      publicTransportation = 0x0a,
      invalidVehicleType = 0xff
   };
{% endhighlight %}

###### WayfinderType

{% highlight cpp %}
   enumWayfinderType {
      InvalidWayfinderType = -1,
      Trial = 0,
      Silver = 1,
      Gold = 2,
      Iron = 3,
   };
{% endhighlight %}

#### Parameters

The following table shows all the general parameters and their respective Nav2 parameter and types. 

 | GuiProt Parameter            | Type   | Nav2 parameter             | 
 | -----------------            | ----   | --------------             | 
 | paramAutomaticRouteOnSMSDest | Int32  | UC_AutomaticRouteOnSMSDest | 
 | paramAutoReroute             | Int32  | UC_AutoReroute             | 
 | paramAutoTracking            | Int32  | UC_AutoTracking            | 
 | paramBacklightStrategy       | Int32  | UC_BacklightStrategy       | 
 | paramBtGpsAddressAndName     | String | BtGpsAddressAndName        | 
 | paramCategoryIds             | String | CategoryIds                | 
 | paramCategoryNames           | String | CategoryNames              | 
 | paramDistanceMode            | Int32  | UC_DistanceMode            | 
 | paramFavoriteShow            | Int32  | UC_FavoriteShow            | 
 | paramGPSAutoConnect          | Int32  | UC_GPSAutoConnect          | 
 | paramHighways                | Int32  | NSC_RouteHighways          | 
 | paramHttpServerNameAndPort   | String | NSC_HttpServerHostname     | 
 | paramKeepSMSDestInInbox      | Int32  | UC_KeepSMSDestInInbox      | 
 | paramLanguage                | Int32  | Language                   | 
 | paramLastKnownRouteId        | Binary | UC_GUILastKnownRouteId     | 
 | paramLatestNewsChecksum      | Int32  | NSC_LatestNewsChecksum     | 
 | paramLatestShownNewsChecksum | Int32  | UC_LatestShownNewsChecksum | 
 | paramLinkLayerKeepAlive      | Int32  | UC_LinkLayerKeepAlive      | 
 | paramMapLayerSettings        | Binary | UC_MapLayerSettings        | 
 | paramPoiCategories           | Binary | UC_PoiCategories           | 
 | paramPositionSymbol          | Int32  | UC_PositionSymbolType      | 
 | paramSearchStrings           | Binary | UC_GUISearchStrings        | 
 | paramSelectedAccessPointId2  | Int32  | SelectedAccessPointId2     | 
 | paramSelectedAccessPointId   | Int32  | SelectedAccessPointIdReal  | 
 | paramServerNameAndPort       | String | NSC_ServerHostname         | 
 | paramSoundVolume             | Int32  | UC_SoundVolume             | 
 | paramStoreSMSDestInMyDest    | Int32  | UC_StoreSMSDestInMyDest    | 
 | paramTimeDist                | Int32  | NSC_RouteCostType          | 
 | paramTimeLeft                | Int32  | NSC_ExpireVector           | 
 | paramTollRoads               | Int32  | NSC_RouteTollRoads         | 
 | paramTrackingLevel           | Int32  | TR_trackLevel              | 
 | paramTrackingPIN             | Binary | TR_trackPIN                | 
 | paramTransportationType      | Int32  | NSC_TransportationType     | 
 | paramTurnSoundsLevel         | Int32  | UC_TurnSoundsLevel         | 
 | paramUserAndPassword         | String | NSC_UserAndPasswd          | 
 | paramUserTermsAccepted       | Int32  | UC_UserTermsAccepted       | 
 | paramUseSpeaker              | Int32  | UC_UseMainSpeaker          | 
 | paramVectorMapCoordinates    | Binary | UC_VectorMapCoordinates    | 
 | paramVectorMapSettings       | Int32  | UC_VectorMapSettings       | 
 | paramWayfinderType           | Int32  | WayfinderType              | 
 | paramWebPassword             | String | UC_WebPasswd               | 
 | paramWebUsername             | String | UC_WebUser                 | 
 | userRights                   | Int32  | NSC_userRights             | 
 | userTrafficUpdatePeriod      | Int32  | NT_UserTrafficUpdatePeriod | 
 | paramShowNewsServerString    | String | NSC_latestNewsId           | 
 | paramShownNewsChecksum       | String | UC_latestNewsId            | 
 | paramNeverShowUSDisclaimer   | int32  | UC_neverShowUSDisclaimer   | 
 

##### paramAutomaticRouteOnSMSDest

**NumEntries:** 1 

 | Data         | type  | Helper   | offset | 
 | ----         | ----  | ------   | ------ | 
 | Route to SMS | int32 | YesNoAsk | 0      | 

This parameter is set to yes if a route should automatically be calculated when an SMS destination is received, no if it should not, and ask if the UI should ask the user. 


##### paramAutoReroute

**NumEntries:** 1 

 | Data         | type  | Helper | offset | 
 | ----         | ----  | ------ | ------ | 
 | Auto reroute | int32 | Bool   | 0      | 

This parameter is set to true if a route should automatically be calculated when the navigation goes off-track, and false if it should not. 


##### paramAutoTracking

**NumEntries:** 1 

 | Data          | type  | Helper | offset | 
 | ----          | ----  | ------ | ------ | 
 | Auto tracking | int32 | Bool   | 0      | 

This parameter is set to true if the map should try to follow the GPS position, and false if it should not. 


##### paramBacklightStrategy

**NumEntries:** 1 

 | Data               | type  | Helper            | offset | 
 | ----               | ----  | ------            | ------ | 
 | Backlight strategy | int32 | BacklightStrategy | 0      | 

Defines the current BacklightStrategy. 


##### paramBtGpsAddressAndName

**NumEntries:** 3 

 | Data             | type   | Helper                   | offset | 
 | ----             | ----   | ------                   | ------ | 
 | GPS address high | string | uint32 encoded as string | 0      | 
 | GPS address low  | string | uint32 encoded as string | 1      | 
 | GPS name         | string | string                   | 2      | 

This parameter contains a number of strings that can be used to identify the current GPS. 


##### paramCategoryIds

**NumEntries:** varies 

 | Data            | type   | Helper | offset | 
 | ----            | ----   | ------ | ------ | 
 | Category id 1   | string | string | 0      | 
 | Category id 2   | string | string | 1      | 
 | Category id ... | string | string | ...    | 
 | Category id n   | string | string | n      | 

This parameter contains the search category ids, which happens to be strings. 


##### paramCategoryNames

**NumEntries:** varies 

 | Data              | type   | Helper | offset | 
 | ----              | ----   | ------ | ------ | 
 | Category name 1   | string | string | 0      | 
 | Category name 2   | string | string | 1      | 
 | Category name ... | string | string | ...    | 
 | Category name n   | string | string | n      | 

This parameter contains the search category names, localized to the language used when contacting the server. 

##### paramDistanceMode

**NumEntries:** 1 

 | Data         | type  | Helper       | offset | 
 | ----         | ----  | ------       | ------ | 
 | DistanceMode | int32 | DistanceMode | 0      | 

Defines the current DistanceMode. 


##### paramFavoriteShow

**NumEntries:** 1 

 | Data                  | type  | Helper            | offset | 
 | ----                  | ----  | ------            | ------ | 
 | Show favorites in map | int32 | ShowFavoriteInMap | 0      | 


##### paramHighways

**NumEntries:** 1 

 | Data         | type  | Helper        | offset | 
 | ----         | ----  | ------        | ------ | 
 | Use highways | int32 | RouteHighways | 0      | 


##### paramHttpServerNameAndPort

**NumEntries:** varies 

 | Data       | type   | Helper | offset | 
 | ----       | ----   | ------ | ------ | 
 | Server 1   | string | string | 0      | 
 | Server 2   | string | string | 1      | 
 | Server ... | string | string | ...    | 
 | Server n   | string | string | n      | 

The server strings has the format "host:port". 


##### paramKeepSMSDestInInbox

**NumEntries:** 1 

 | Data              | type  | Helper   | offset | 
 | ----              | ----  | ------   | ------ | 
 | Keep SMS in inbox | int32 | YesNoAsk | 0      | 


##### paramLanguage

**NumEntries:** 1 

 | Data          | type  | Helper       | offset | 
 | ----          | ----  | ------       | ------ | 
 | Language code | int32 | languageCode | 0      | 

The current language code. This parameter is only set from the GUI. 


##### paramLastKnownRouteId

**NumEntries:** 1 

 | Data                | type   | Helper | offset | 
 | ----                | ----   | ------ | ------ | 
 | Last known route id | binary | int64  | 0      | 

The last known route id. The data is saved as an unaligned 64bit number. **Currently unused** 


##### paramLatestNewsChecksum

Note: This parameter is now obsolete and must not be used. 

**NumEntries:** 1 

 | Data                 | type  | Helper | offset | 
 | ----                 | ----  | ------ | ------ | 
 | Latest news checksum | int32 | uint32 | 0      | 

The latest news checksum. Sent from the server when the news should be shown. 

**Currently unused **


##### paramLatestShownNewsChecksum

Note: This parameter is now obsolete and must not be used. 

**NumEntries:** 1 

 | Data                       | type  | Helper | offset | 
 | ----                       | ----  | ------ | ------ | 
 | Latest shown news checksum | int32 | uint32 | 0      | 

The latest shown news checksum. Set from the paramLatestNewsChecksum when the news has been shown. 

**Currently unused**


##### paramLinkLayerKeepAlive

**NumEntries:** 1 

 | Data                 | type  | Helper | offset | 
 | ----                 | ----  | ------ | ------ | 
 | Link layer keepalive | int32 | bool   | 0      | 

Indicates if the UI should keep GPRS alive. 


##### paramMapLayerSettings

**NumEntries:** varies 

 | Data               | type   | Helper                 | offset | 
 | ----               | ----   | ------                 | ------ | 
 | Map layer settings | binary | TileMapLayerInfoVector | 0      | 

Binary representation of TileMapLayerInfoVector. Please refer to TileMapLayerInfoVector to understand format. 


##### paramNeverShowUSDisclaimer

**NumEntries:** 1 

 | Data                     | type  | Helper | offset | 
 | ----                     | ----  | ------ | ------ | 
 | Never show US disclaimer | int32 | uint32 | 0      | 

If the parameter is uset or zero, then the normal test for US Disclaimer page should be performed. 
Otherwise, the US Disclaimer page should never be shown. 


##### paramPoiCategories

**NumEntries:** varies 

 | Data i  | type   | Helper | offset | 
 | ------  | ----   | ------ | ------ | 
 | Version | int16  | int16  | 0      | 
 | Name 1  | string | string | 1      | 
 | Id 1    | int32  | int32  | 2      | 
 | Value 1 | int8   | int8   | 3      | 
 | Name 2  | string | string | 4      | 
 | Id 2    | int32  | int32  | 5      | 
 | Value 2 | int8   | int8   | 6      | 
 | ...     |        |        |        | 

Version is currently 5. 


##### paramPositionSymbol

**NumEntries:** 1 

 | Data                   | type  | Helper | offset | 
 | ----                   | ----  | ------ | ------ | 
 | Position symbol in map | int32 | int32  | 0      | 


##### paramSearchStrings

**NumEntries:** varies 

 | Data                    | type   | Helper | offset | 
 | ----                    | ----   | ------ | ------ | 
 | Version                 | int16  | int16  | 0      | 
 | Search string 1         | string | string | 1      | 
 | Search house number 1   | string | string | 2      | 
 | Search city string      | string | string | 3      | 
 | Search city id 1        | string | string | 4      | 
 | Search country string 1 | string | string | 5      | 
 | Search country id 1     | string | string | 6      | 
 | ...                     | string | string | ...    | 

Version is currently 5. 


##### paramSelectedAccessPointId2

**NumEntries:** 1 

 | Data                 | type  | Helper | offset | 
 | ----                 | ----  | ------ | ------ | 
 | Used access point id | int32 | int32  | 0      | 

Can have the two special values of -1 for ask user and -2 for use system default. Set from paramSelectedAccessPointId. Always set to either -1 or -2 on startup. 


##### paramSelectedAccessPointId

**NumEntries:** 1 

 | Data                  | type  | Helper | offset | 
 | ----                  | ----  | ------ | ------ | 
 | Saved access point id | int32 | int32  | 0      | 

Can have the two special values of -1 for ask user and -2 for use system default. paramSelectedAccessPointId2 should always be set to the same value when paramSelectedAccessPointId is received. Always set to either -1 or -2 on first startup. 


##### paramServerNameAndPort

**NumEntries:** varies 

 | Data       | type   | Helper | offset | 
 | ----       | ----   | ------ | ------ | 
 | Server 1   | string | string | 0      | 
 | Server 2   | string | string | 1      | 
 | Server ... | string | string | ...    | 
 | Server n   | string | string | n      | 

The server strings has the format "host:port". 


##### paramShowNewsServerString

**NumEntries:** 1 

 | Data                 | type   | Helper | offset | 
 | ----                 | ----   | ------ | ------ | 
 | Server news checksum | string | string | 0      | 


Used in conjunction with `paramShownNewsChecksum`. 

This parameter contains the checksum string downloaded from the server on param
sync. If it is unset or the empty string, the news should be shown always.
Otherwise, the string should be compared with the `paramShownNewsChecksum`
parameter, and news should only be shown when the two don't match. After a
successful showing of the news page, the `paramShownNewsChecksum` should be set
to the same value as `paramShowNewsServerString`, unless th
`paramShowNewsServerString` is unset or the empty string. 


##### paramShownNewsChecksum

**NumEntries:** 1 

 | Data                   | type   | Helper | offset | 
 | ----                   | ----   | ------ | ------ | 
 | paramShownNewsChecksum | string | string | 0      | 

See paramShowNewsServerString for documentation. 


##### paramSoundVolume

**NumEntries:** 1 

 | Data   | type  | Helper | offset | 
 | ----   | ----  | ------ | ------ | 
 | Volume | int32 | int32  | 0      | 

The value describes the volume in percent from 0 to 100. 


##### paramStoreSMSDestInMyDest

**NumEntries:** 1 

 | Data                       | type  | Helper   | offset | 
 | ----                       | ----  | ------   | ------ | 
 | Save SMS dest in favorites | int32 | YesNoAsk | 0      | 


##### paramTimeDist

**NumEntries:** 1 

 | Data                | type  | Helper        | offset | 
 | ----                | ----  | ------        | ------ | 
 | Route optimized for | int32 | RouteCostType | 0      | 

Sets what the route should be optimized for, time, time and traffic information, or distance. 


##### paramTimeLeft

**NumEntries:** 3 

 | Data                  | type  | Helper | offset | 
 | ----                  | ----  | ------ | ------ | 
 | Days left             | int32 | int32  | 0      | 
 | Transactions left     | int32 | int32  | 1      | 
 | Transaction days left | int32 | int32  | 2      | 


##### paramTollRoads

NumEntries: 1 

 | Data           | type  | Helper         | offset | 
 | ----           | ----  | ------         | ------ | 
 | Use toll roads | int32 | RouteTollRoads | 0      | 


##### paramTrackingLevel

**NumEntries:** 1 

 | Data           | type  | Helper | offset | 
 | ----           | ----  | ------ | ------ | 
 | Tracking level | int32 | int32  | 0      | 


##### paramTrackingPIN

NumEntries: varies 

 | Data     | type   | Helper       | offset | 
 | ----     | ----   | ------       | ------ | 
 | PIN data | binary | TrackPINList | 0      | 

Please refer to TrackPINList for documentation about format. 


##### paramTransportationType

**NumEntries:** 1 

 | Data                | type  | Helper      | offset | 
 | ----                | ----  | ------      | ------ | 
 | Transportation type | int32 | VehicleType | 0      | 


##### paramTurnSoundsLevel

**NumEntries:** 1 

 | Data              | type  | Helper          | offset | 
 | ----              | ----  | ------          | ------ | 
 | Turn sounds level | int32 | TurnSoundsLevel | 0      | 


##### paramUserAndPassword

**NumEntries:** 2 


 | Data     | type   | Helper | offset | 
 | ----     | ----   | ------ | ------ | 
 | Username | string | string | 0      | 
 | Password | string | string | 1      | 

This parameter is only used for debug. 


##### paramUserTermsAccepted

**NumEntries:** 1 

 | Data                | type  | Helper | offset | 
 | ----                | ----  | ------ | ------ | 
 | User terms accepted | int32 | bool   | 0      | 


##### paramUseSpeaker 

**NumEntries:** 1 

 | Data        | type  | Helper | offset | 
 | ----        | ----  | ------ | ------ | 
 | Use speaker | int32 | bool   | 0      | 


##### paramVectorMapCoordinates

**NumEntries:** varies 

 | Data                   | type   | Helper               | offset | 
 | ----                   | ----   | ------               | ------ | 
 | Vector map coordinates | binary | VectorMapCoordinates | 0      | 

Please refer to VectorMapCoordinates for documentation about format. 


##### paramVectorMapSettings 

**NumEntries:** 7 

 | Data                       | type  | Helper | offset | 
 | ----                       | ----  | ------ | ------ | 
 | Version                    | int32 | int32  | 0      | 
 | Cache size                 | int32 | int32  | 1      | 
 | Map type **obsolete**      | int32 | int32  | 2      | 
 | Tracking orientation       | int32 | int32  | 3      | 
 | Favorite show **obsolete** | int32 | int32  | 4      | 
 | Guide mode                 | int32 | int32  | 5      | 
 | Gui mode **obsolete**      | int32 | int32  | 6      | 

Version is currently 1. 


##### paramWayfinderType

**NumEntries:** 1 

 | Data           | type  | Helper        | offset | 
 | ----           | ----  | ------        | ------ | 
 | Wayfinder type | int32 | WayfinderType | 0      | 


##### paramWebPassword

**NumEntries:** 1 

 | Data         | type   | Helper | offset | 
 | ----         | ----   | ------ | ------ | 
 | Web password | string | string | 0      | 

Parameter can be set from client, but is never downloaded from server. Is not saved over restarts. 


##### paramWebUsername

**NumEntries:** 1 

 | Data         | type   | Helper | offset | 
 | ----         | ----   | ------ | ------ | 
 | Web username | string | string | 0      | 

Paramer cannot be set from client. 


##### userRights

**NumEntries:** varies 

 | Data        | type  | Helper     | offset | 
 | ----        | ----  | ------     | ------ | 
 | Rights data | int32 | UserRights | 0      | 

Please refer to UserRights for documentation of format. 


##### userTrafficUpdatePeriod

**NumEntries:** 1 

 | Data                  | type  | Helper | offset | 
 | ----                  | ----  | ------ | ------ | 
 | Traffic update period | int32 | uint32 | 0      | 

The lower 30 bits describe the traffic update time in minutes. Setting the second highest bit means that the setting is not active since the value will be so large (in minutes) that it is impossible to trigger a traffic update. 

### Use cases 

#### Activation 

The current mode of the application is kept in a non-persistent Nav2 parameter called WayfinderType. The value of this parameter should be on from the WayfinderType enum (see ). These enum values corresponds to the various Wayfinder modes as follows: 

 | WayfinderType enum | Wayfinder Mode             | 
 | ------------------ | --------------             | 
 | Trial              | Trial mode                 | 
 | Silver             | Wayfinder Mobile MapGuide  | 
 | Gold               | Wayfinder Mobile Navigator | 

Whenever the Wayfinder server decides that the client should change to another mode, Nav2 changes the WayfinderType parameter and sends it to the GUI. 

As the WayfinderType parameter is not persistent, it's up to the GUI to keep track of the WayfinderType and send it to Nav2 as a parameter at startup. We recommend to keep this state in an ini-file of some kind. 


##### Activation and Upgrade 

The process of changing the status of the users Wayfinder application from Trial to Mobile MapGuide or Mobile Navigator is called Activation. Changing from Mobile MapGuide to Mobile Navigator or extending either kind of subscription is called Upgrading. Both processes use the same messages. 

   - The GUI sends a REQUEST_LICENSE_UPGRADE message (see ) with proper content to Nav2. 
   - Nav2 Replies with a LICENSE_UPGRADE_REPLY message (see ) containing three booleans indicating which of the data sent to Nav2 was accepted by the server. If all three booleans are set, the type argument contains one of the values from the WayfinderType enum (see ). 

##### Reactivate

When a user already has a Wayfinder account but for some reason has reinstalled the application on his mobile phone, he should use the Reactivation option in the Trial menu to bring the application in sync with his server account. 

   - The GUI sends a PARAMETERS_SYNC message (see ) to Nav2. This message has no payload. 
   - The Nav2 contacts the server and replies with a PARAMETERS_SYNC_REPLY message (see ). This message holds an uint32 that should be converted to a WayfinderType enum (see ) value. This enum value should determine the mode the application should run in. 

