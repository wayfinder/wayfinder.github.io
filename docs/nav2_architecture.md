---
title: Nav2 Architecture
layout: single
---

## GuiProt 4 General Parameters

### General 

This is the specification of all General parameters used in the Nav2 UI protocol, and their respective Nav2 parameters. 

This document describes the different parameters that can be used within Nav2.
Parameters are used for permanent storage of settings, for example saving the
favorites on the device. 

The way to use the parameters is pretty straight forward. 
1.  UI sends a `GeneralParameterMessage` with the type `paramAutoReroute`
2.  The Parameter Module inside Nav2 parses the message and creates a reply
3.  The reply is sent to the UI. The reply contains the data and the same ID as in the request.

#### Format

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


#### Helper enums

A number of enums have been specified to allow easier definition of the parameters: 


##### Bool

{% highlight cpp %}
   enum Bool {
      false                                  = 0x00,
      true                                   = 0x01,
   };
{% endhighlight %}

##### SortingType

{% highlight cpp %}
   enum SortingType {
      alphabeticalOnName = 0x00,
      distance = 0x01,
      newSort = 0x02,
      invalidSortingType = 0xffff,
   };
{% endhighlight %}

##### YesNoAsk

{% highlight cpp %}
   enum YesNoAsk {
      yes = 0x00,
      no = 0x01,
      ask = 0x02,
      invalidYesNoAsk = 0x03,
   };
{% endhighlight %}

##### BacklightStrategy

{% highlight cpp %}
   enum BacklightStrategy {
      backlight_always_off = 0x00,
      backlight_on_during_route = 0x01,
      backlight_always_on = 0x02,
      backlight_near_action = 0x03,
      backlight_invalid = 0xff,
   };
{% endhighlight %}

##### TurnSoundsLevel

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

##### DistanceMode

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

##### ShowFavoriteInMap

{% highlight cpp %}
   enum ShowFavoriteInMap {
      ShowFavoriteInMapAlways = 0x00,
      ShowFavoriteInMapCityLevel = 0x01,
      ShowFavoriteInMapNever = 0x02,
   };
{% endhighlight %}

##### RouteCostType

{% highlight cpp %}
   enum RouteCostType {
      DISTANCE = 0,
      TIME = 1,
      TIME_WITH_DISTURBANCES = 2,
      INAVLID = 0xff
   };
{% endhighlight %}

##### RouteTollRoads

{% highlight cpp %}
   enum RouteTollRoads {
      TollRoadsAllow = 0,
      TollRoadsDeny = 1,
   };
{% endhighlight %}

##### RouteHighways

{% highlight cpp %}
   enum RouteHighways {
      HighwaysAllow = 0,
      HighwaysDeny = 1,
   };
{% endhighlight %}

##### languageCode

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

##### VehicleType

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

##### WayfinderType

{% highlight cpp %}
   enum WayfinderType {
      InvalidWayfinderType = -1,
      Trial = 0,
      Silver = 1,
      Gold = 2,
      Iron = 3,
   };
{% endhighlight %}

### Parameters

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
 

#### paramAutomaticRouteOnSMSDest

**NumEntries:** 1 

 | Data         | type  | Helper   | offset | 
 | ----         | ----  | ------   | ------ | 
 | Route to SMS | int32 | YesNoAsk | 0      | 

This parameter is set to yes if a route should automatically be calculated when an SMS destination is received, no if it should not, and ask if the UI should ask the user. 


#### paramAutoReroute

**NumEntries:** 1 

 | Data         | type  | Helper | offset | 
 | ----         | ----  | ------ | ------ | 
 | Auto reroute | int32 | Bool   | 0      | 

This parameter is set to true if a route should automatically be calculated when the navigation goes off-track, and false if it should not. 


#### paramAutoTracking

**NumEntries:** 1 

 | Data          | type  | Helper | offset | 
 | ----          | ----  | ------ | ------ | 
 | Auto tracking | int32 | Bool   | 0      | 

This parameter is set to true if the map should try to follow the GPS position, and false if it should not. 


#### paramBacklightStrategy

**NumEntries:** 1 

 | Data               | type  | Helper            | offset | 
 | ----               | ----  | ------            | ------ | 
 | Backlight strategy | int32 | BacklightStrategy | 0      | 

Defines the current BacklightStrategy. 


#### paramBtGpsAddressAndName

**NumEntries:** 3 

 | Data             | type   | Helper                   | offset | 
 | ----             | ----   | ------                   | ------ | 
 | GPS address high | string | uint32 encoded as string | 0      | 
 | GPS address low  | string | uint32 encoded as string | 1      | 
 | GPS name         | string | string                   | 2      | 

This parameter contains a number of strings that can be used to identify the current GPS. 


#### paramCategoryIds

**NumEntries:** varies 

 | Data            | type   | Helper | offset | 
 | ----            | ----   | ------ | ------ | 
 | Category id 1   | string | string | 0      | 
 | Category id 2   | string | string | 1      | 
 | Category id ... | string | string | ...    | 
 | Category id n   | string | string | n      | 

This parameter contains the search category ids, which happens to be strings. 


#### paramCategoryNames

**NumEntries:** varies 

 | Data              | type   | Helper | offset | 
 | ----              | ----   | ------ | ------ | 
 | Category name 1   | string | string | 0      | 
 | Category name 2   | string | string | 1      | 
 | Category name ... | string | string | ...    | 
 | Category name n   | string | string | n      | 

This parameter contains the search category names, localized to the language used when contacting the server. 

#### paramDistanceMode

**NumEntries:** 1 

 | Data         | type  | Helper       | offset | 
 | ----         | ----  | ------       | ------ | 
 | DistanceMode | int32 | DistanceMode | 0      | 

Defines the current DistanceMode. 


#### paramFavoriteShow

**NumEntries:** 1 

 | Data                  | type  | Helper            | offset | 
 | ----                  | ----  | ------            | ------ | 
 | Show favorites in map | int32 | ShowFavoriteInMap | 0      | 


#### paramHighways

**NumEntries:** 1 

 | Data         | type  | Helper        | offset | 
 | ----         | ----  | ------        | ------ | 
 | Use highways | int32 | RouteHighways | 0      | 


#### paramHttpServerNameAndPort

**NumEntries:** varies 

 | Data       | type   | Helper | offset | 
 | ----       | ----   | ------ | ------ | 
 | Server 1   | string | string | 0      | 
 | Server 2   | string | string | 1      | 
 | Server ... | string | string | ...    | 
 | Server n   | string | string | n      | 

The server strings has the format "host:port". 


#### paramKeepSMSDestInInbox

**NumEntries:** 1 

 | Data              | type  | Helper   | offset | 
 | ----              | ----  | ------   | ------ | 
 | Keep SMS in inbox | int32 | YesNoAsk | 0      | 


#### paramLanguage

**NumEntries:** 1 

 | Data          | type  | Helper       | offset | 
 | ----          | ----  | ------       | ------ | 
 | Language code | int32 | languageCode | 0      | 

The current language code. This parameter is only set from the GUI. 


#### paramLastKnownRouteId

**NumEntries:** 1 

 | Data                | type   | Helper | offset | 
 | ----                | ----   | ------ | ------ | 
 | Last known route id | binary | int64  | 0      | 

The last known route id. The data is saved as an unaligned 64bit number. **Currently unused** 


#### paramLatestNewsChecksum

Note: This parameter is now obsolete and must not be used. 

**NumEntries:** 1 

 | Data                 | type  | Helper | offset | 
 | ----                 | ----  | ------ | ------ | 
 | Latest news checksum | int32 | uint32 | 0      | 

The latest news checksum. Sent from the server when the news should be shown. 

**Currently unused **


#### paramLatestShownNewsChecksum

Note: This parameter is now obsolete and must not be used. 

**NumEntries:** 1 

 | Data                       | type  | Helper | offset | 
 | ----                       | ----  | ------ | ------ | 
 | Latest shown news checksum | int32 | uint32 | 0      | 

The latest shown news checksum. Set from the paramLatestNewsChecksum when the news has been shown. 

**Currently unused**


#### paramLinkLayerKeepAlive

**NumEntries:** 1 

 | Data                 | type  | Helper | offset | 
 | ----                 | ----  | ------ | ------ | 
 | Link layer keepalive | int32 | bool   | 0      | 

Indicates if the UI should keep GPRS alive. 


#### paramMapLayerSettings

**NumEntries:** varies 

 | Data               | type   | Helper                 | offset | 
 | ----               | ----   | ------                 | ------ | 
 | Map layer settings | binary | TileMapLayerInfoVector | 0      | 

Binary representation of TileMapLayerInfoVector. Please refer to TileMapLayerInfoVector to understand format. 


#### paramNeverShowUSDisclaimer

**NumEntries:** 1 

 | Data                     | type  | Helper | offset | 
 | ----                     | ----  | ------ | ------ | 
 | Never show US disclaimer | int32 | uint32 | 0      | 

If the parameter is uset or zero, then the normal test for US Disclaimer page should be performed. 
Otherwise, the US Disclaimer page should never be shown. 


#### paramPoiCategories

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


#### paramPositionSymbol

**NumEntries:** 1 

 | Data                   | type  | Helper | offset | 
 | ----                   | ----  | ------ | ------ | 
 | Position symbol in map | int32 | int32  | 0      | 


#### paramSearchStrings

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


#### paramSelectedAccessPointId2

**NumEntries:** 1 

 | Data                 | type  | Helper | offset | 
 | ----                 | ----  | ------ | ------ | 
 | Used access point id | int32 | int32  | 0      | 

Can have the two special values of -1 for ask user and -2 for use system default. Set from paramSelectedAccessPointId. Always set to either -1 or -2 on startup. 


#### paramSelectedAccessPointId

**NumEntries:** 1 

 | Data                  | type  | Helper | offset | 
 | ----                  | ----  | ------ | ------ | 
 | Saved access point id | int32 | int32  | 0      | 

Can have the two special values of -1 for ask user and -2 for use system default. paramSelectedAccessPointId2 should always be set to the same value when paramSelectedAccessPointId is received. Always set to either -1 or -2 on first startup. 


#### paramServerNameAndPort

**NumEntries:** varies 

 | Data       | type   | Helper | offset | 
 | ----       | ----   | ------ | ------ | 
 | Server 1   | string | string | 0      | 
 | Server 2   | string | string | 1      | 
 | Server ... | string | string | ...    | 
 | Server n   | string | string | n      | 

The server strings has the format "host:port". 


#### paramShowNewsServerString

**NumEntries:** 1 

 | Data                 | type   | Helper | offset | 
 | ----                 | ----   | ------ | ------ | 
 | Server news checksum | string | string | 0      | 


Used in conjunction with paramShownNewsChecksum. 

This parameter contains the checksum string downloaded from the server on param sync. If it is unset or the empty string, the news should be shown always. Otherwise, the string should be compared with the paramShownNewsChecksum parameter, and news should only be shown when the two don't match. After a successful showing of the news page, the paramShownNewsChecksum should be set to the same value as paramShowNewsServerString, unless th paramShowNewsServerString is unset or the empty string. 


#### paramShownNewsChecksum

**NumEntries:** 1 

 | Data                   | type   | Helper | offset | 
 | ----                   | ----   | ------ | ------ | 
 | paramShownNewsChecksum | string | string | 0      | 

See paramShowNewsServerString for documentation. 


#### paramSoundVolume

**NumEntries:** 1 

 | Data   | type  | Helper | offset | 
 | ----   | ----  | ------ | ------ | 
 | Volume | int32 | int32  | 0      | 

The value describes the volume in percent from 0 to 100. 


#### paramStoreSMSDestInMyDest

**NumEntries:** 1 

 | Data                       | type  | Helper   | offset | 
 | ----                       | ----  | ------   | ------ | 
 | Save SMS dest in favorites | int32 | YesNoAsk | 0      | 


#### paramTimeDist

**NumEntries:** 1 

 | Data                | type  | Helper        | offset | 
 | ----                | ----  | ------        | ------ | 
 | Route optimized for | int32 | RouteCostType | 0      | 

Sets what the route should be optimized for, time, time and traffic information, or distance. 


#### paramTimeLeft

**NumEntries:** 3 

 | Data                  | type  | Helper | offset | 
 | ----                  | ----  | ------ | ------ | 
 | Days left             | int32 | int32  | 0      | 
 | Transactions left     | int32 | int32  | 1      | 
 | Transaction days left | int32 | int32  | 2      | 


#### paramTollRoads

NumEntries: 1 

 | Data           | type  | Helper         | offset | 
 | ----           | ----  | ------         | ------ | 
 | Use toll roads | int32 | RouteTollRoads | 0      | 


#### paramTrackingLevel

**NumEntries:** 1 

 | Data           | type  | Helper | offset | 
 | ----           | ----  | ------ | ------ | 
 | Tracking level | int32 | int32  | 0      | 


#### paramTrackingPIN

NumEntries: varies 

 | Data     | type   | Helper       | offset | 
 | ----     | ----   | ------       | ------ | 
 | PIN data | binary | TrackPINList | 0      | 

Please refer to TrackPINList for documentation about format. 


#### paramTransportationType

**NumEntries:** 1 

 | Data                | type  | Helper      | offset | 
 | ----                | ----  | ------      | ------ | 
 | Transportation type | int32 | VehicleType | 0      | 


#### paramTurnSoundsLevel

**NumEntries:** 1 

 | Data              | type  | Helper          | offset | 
 | ----              | ----  | ------          | ------ | 
 | Turn sounds level | int32 | TurnSoundsLevel | 0      | 


#### paramUserAndPassword

**NumEntries:** 2 


 | Data     | type   | Helper | offset | 
 | ----     | ----   | ------ | ------ | 
 | Username | string | string | 0      | 
 | Password | string | string | 1      | 

This parameter is only used for debug. 


#### paramUserTermsAccepted

**NumEntries:** 1 

 | Data                | type  | Helper | offset | 
 | ----                | ----  | ------ | ------ | 
 | User terms accepted | int32 | bool   | 0      | 


#### paramUseSpeaker 

**NumEntries:** 1 

 | Data        | type  | Helper | offset | 
 | ----        | ----  | ------ | ------ | 
 | Use speaker | int32 | bool   | 0      | 


#### paramVectorMapCoordinates

**NumEntries:** varies 

 | Data                   | type   | Helper               | offset | 
 | ----                   | ----   | ------               | ------ | 
 | Vector map coordinates | binary | VectorMapCoordinates | 0      | 

Please refer to VectorMapCoordinates for documentation about format. 


#### paramVectorMapSettings 

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


#### paramWayfinderType

**NumEntries:** 1 

 | Data           | type  | Helper        | offset | 
 | ----           | ----  | ------        | ------ | 
 | Wayfinder type | int32 | WayfinderType | 0      | 


#### paramWebPassword

**NumEntries:** 1 

 | Data         | type   | Helper | offset | 
 | ----         | ----   | ------ | ------ | 
 | Web password | string | string | 0      | 

Parameter can be set from client, but is never downloaded from server. Is not saved over restarts. 


#### paramWebUsername

**NumEntries:** 1 

 | Data         | type   | Helper | offset | 
 | ----         | ----   | ------ | ------ | 
 | Web username | string | string | 0      | 

Paramer cannot be set from client. 


#### userRights

**NumEntries:** varies 

 | Data        | type  | Helper     | offset | 
 | ----        | ----  | ------     | ------ | 
 | Rights data | int32 | UserRights | 0      | 

Please refer to UserRights for documentation of format. 


#### userTrafficUpdatePeriod

**NumEntries:** 1 

 | Data                  | type  | Helper | offset | 
 | ----                  | ----  | ------ | ------ | 
 | Traffic update period | int32 | uint32 | 0      | 

The lower 30 bits describe the traffic update time in minutes. Setting the second highest bit means that the setting is not active since the value will be so large (in minutes) that it is impossible to trigger a traffic update. 


