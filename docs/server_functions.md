---
title: MC2 Functions
layout: single
---

## Overview of the code layout for the Wayfinder Server

The source code organization. The source code is organized as modules according to overview descriptions.

![source code overview](/images/mc2_source_code_tree_overview.png)

### Configuration files 

The configuration files are described in the [MC2 operations manual](../operations_manual).

## Documentation of functions in the Wayfinder Server 

### Search

The search in MC2 can search for names in the map data whereas it be streets, points of interest(POI), shops restaurants etc., or areas.
Searching can be done in areas, restricted by name, with whole countries as the upper limit of areas to search in.
The matching is done the by SearchModule backend for each map part and the result is then merged in the frontend and sent to the client.
From a development perspective the frontends calls the `SearchParserHandler::compactSearch()` function with the input from the client.

#### Selecting what areas to search in

This can be done by name, a string, called `where`, that limits the results to the ones located in areas, normally cities, with a matching name.

#### String search

The string search matches streets, POIs and other items that are located in areas.
The text input is called %what%.

#### Category search

Categories are groupings of POIs into groups and subgroups. The client can download a category tree. An example from the tree is The top category Food and drink with the subcategory Restaurants which in turn has subcategories like Pizzeria.
If a category that has subcategories is selected all categories in that branch is selected e.i. the subcategories, and their subcategories.


#### Headings

For searches the results are returned in different sections depending on what type of search hit it is. One such section is addresses another is places and it can also be an external search provider, see below.
These sections are called Headings and a Heading can be set in the search input to limit the results to one heading. This is normally done when requesting more the next part of the search list.
In the newest version of the search interface, called One Search, all results are returned in one list and doesn't not have any Heading concept.


### External search

External searches provides additional results for searches from 
external providers.

#### Google local search 

The Wayfinder server has an integration with Google local search but to use
it you must have an agreement with Google. See
http://code.google.com/apis/ajaxsearch/local.html for details.
When you have permission open the mc2 property file `Server/bin/mc2.prop` and
set the property `ENABLE_GOOGLE_LOCAL_SEARCH_INTEGRATION` to true under
the External Search Integrations section.

#### Qype

Qype provides millions of reviews and images of places all over the world.
To enable Qype searches you must first get permission and an API key from
Qype. See http://apidocs.qype.com/ for details about usage of Qype.
Then enable it in the mc2 property file `Server/bin/mc2.prop` by setting
the `ENABLE_QYPE_INTEGRATION` property to true and setting the
`QYPE_CONSUMER_KEY` to the Qype API key.


### External connections

#### App Store 

There is a beta version of an Apple App Store integration. Be warned the 
integration has not been fully tested, only with a sandbox test product.
Clients with a client type that ends with "-apps" are identified as App
Store clients and are checked in `ParserAppStoreHandler::checkService`.
To make the iPhone client an App Store client set the client type, 
to `wf-iphone-apps` from `navclientsettings.txt`, and the client will get
a list of App Store IDs when making something that requires purchased 
product. The products and the operations, search or route for example, are
set in the ParserAppStoreHandler. You needs to add appStoreID in 
ParserAppStoreHandler and change from sandbox to real when testing is done.
    
#### PosPush

There is a way to push positions received from clients to another server
depending on some matching of the user.
Set this in `ParserPosPush.cpp` using example there. The actual sending
is done by the CommunicationModule but the content is made in ParserPosPush.
    

### Copyright

The copyright shown in client consists of two parts, first the copyright of
the application and then the copyright of the map data.
#### Application copyright

The application copyright is set in CopyrightHandler, 
`Server/Shared/src/CopyrightHandler.cpp` where `m_copyrights.m_copyrightHead` 
is set twice, for vector maps and image maps.
If the copyright is too wide in the image maps there is in 
`Server/Shared/Drawing/src/MapDrawer.cpp` a unused section where the copyright
can be shortened, at the declaration of "string newCopyright".
#### Map copyright

The Map copyright, used by CopyrightHandler to make the full copyright string,
is made during the map generation, please see the Map Generation section. But
there is a default set in `Server/MapGen/src/OldGenericMapHeader.cpp` and in
`Server/Shared/src/GenericMapHeader.cpp` if no copyright bounding boxes are used
during the map generation.

### Users

#### Hardware Keys

A hardware key is a unique identifier for the device that the Wayfinder client application is running on.

#### Regions

There are regions and region groups, regions are normally countries and region 
groups are a set of regions. The regions and region groups are defined in
`Server/bin/Scripts/region_ids.xml`. The ident for the regions is used 
during the map generation when areas are grouped together into regions.

#### Rights

Rights is the thing controlling what services and areas users has access to.
The type of rights are defined in `UserEnums.h`. The first values: `TRIAL`, `SILVER`,
`GOLD`, of enum `userRightLevel` is only used with `UR_WF`, from the second enum
`userRightService`, which is the main right giving access to the Wayfinder server
for clients. 

The first section in userRightLevel are the different levels of 
the Wayfinder right, trial silver gold, and variants of the Wayfinder right, 
iron, lithium, cesium, wolfram, that is for other products than the Navigator.
With `UR_DEMO` which is a flag in between.
The `userRightService` enum is mostly add-ons except for the UR_WF which is
the service for Navigator clients, `UR_MYWAYFINDER` is the access to the
web sites and `UR_XML` is access to the XML API.

The `UR_VERSION_LOCK`, `UR_ADD_NON_SUP_RIGHTS` and `UR_DISABLE_POI_LAYER` services 
are special services. The first one is not as a normal right, it says up to
which client version the user may use, the version is stored in the region 
of the User Right. The version lock is defined in `navclientsettings.txt`.
`UR_ADD_NON_SUP_RIGHTS` enables an administration user, a user with edit user 
right bool set to true, to add rights with comment, called origin as it is 
where the right comes from, starting with something other than "SUP: " and 
the last one is a debug right that turns off POIs in the vector maps.

The userRightLevel and userRightService are combined together in the class
URType to form the type of right. It can also hold masks for rights, for 
example `TSG_MASK` for userRightLevel with `UR_WF` from userRightService.
The URType with a region and a duration forms a User Right, see `UserRight.h`,
which is the main User Right container.

There are old ways to control users access that aren't used any more. The 
first one is the "WAP, SMS, Html, Navigator, XML and Operator service bools
in the user with the Valid until date as end date. The other one is the region
access that gave access to a region for a specified time.

The demo client types gets free access by the `ParserThreadGroup::addExtraRights`
function that adds virtual rights that is not stored in the database but is 
available when making access checks. The demo clients are identified by the
brand field set in `navclientsettings.txt` to `DEMO`.

### Activation Codes

Activation Codes are a text that the end user inputs in the application and gets some extra rights.
The Activation Code entered by the user is looked up in the SQL table WFActivationCodes and if not used,
this is indicated by empty MC2UIN field, the Rights field is applied to the user account.
In the WFActivationCodes table the fields ActivationCode, Rights and Server are used when getting an activation code and when applying the Rights to the user's account.

When an Activation Code is used the fields MC2UIN, IP, UseTime, UserAgent, UserInput and UpdateTime are filled in by the server.

The WFActivationCodes table:

 | Field          | Type         | 
 | -----          | ----         | 
 | ActivationCode | VARCHAR(20)  | 
 | MC2UIN         | BIGINT       | 
 | Info           | VARCHAR(255) | 
 | Rights         | MC2BLOB      | 
 | server         | VARCHAR(255) | 
 | IP             | VARCHAR(15)  | 
 | UseTime        | INT          | 
 | UserAgent      | VARCHAR(200) | 
 | UserInput      | MC2BLOB      | 
 | UpdateTime     | INT          | 

New Activation Codes are added to the SQL table like this.

	INSERT INTO WFActivationCodes ( ActivationCode, Info, Rights ) VALUES ( 'TestCode', 'Test code addedd 20100311 by X', 'GOLD(12m,2097152),TRAFFIC(12m,2097152)' )

#### Rights as string

The rights as string is an representation of a list of User Right with URType, region and a duration formated as 
`SERVICE(DURATION,REGIONID)` with a comma between each User Right. Example: `GOLD(12m,2097152),TRAFFIC(12m,2097152)`.
All right services as string are in `ParserActivationHandler.cpp`.


### Regression test documentation

The regression test system is located in the `RegressionTests/InterfaceTest` 
directory.
To run the regression tests automatically there is a `crontest.sh` that only 
prints output if any test fails.
Before running tests you must first run `make` in `RegressionTests/InterfaceTest/tools` and have the ngpmaker target in the path, see compilation of MC2.
To add new tests add to either `NS/nstest.pl` or `XMS/xmltests.pl`, see examples
there.
The regression tests are dependent on the running server instance and the
map data, also optionally enabled external search providers, for its tests
and results. This means that the results will differ if a different map
source is used. The server interface settings are in the beginning of 
`servertest.pl`.

####  Set up test users: 

*  NS - Create a nstestuser with binary of tye IMEI with the value `nstestuser`. In `NS/tools/ngp_grep` update the `elsif ( $arg eq '--testuser' )` block: change 460809085 to the UIN of the nstestuser user.

*  XML - Create a user with XML right and set login and password in servertest.pl.

### MC2 unit tests documentation

The unit tests are located in various places in the tree in Tests directories 
next to the code they should test. 
Test framework is defined in `Shared/UnitTest/include/MC2UnitTest.h`. Read the 
documentation there.
To add a new Tests directory create the directory and add a wscript file in 
it, see `Shared/Tests/wscript` for example of how the wscript should look like.
Then add the new Tests directory to nearest wscript above the new Tests 
directory.
The unit tests are run by calling `./waf check`


### Cellid

The Wayfinder Server has the possibility to lookup cell ids against the OpenCellID database.
But you must first get permission and api key from opencellid.org before enabling OpenCellID 
and setting the API key. The two settings are `ENABLE_OPENCELLID_INTEGRATION` and `OPENCELLID_KEY`
in `mc2.prop`.

### Memcached in UserModule

The UserModule that is responsible for storing user related data can use
Memcached to cache user data in memory speeding up repeated user data retrieval.
This requires that Memcached is installed and running on the hosts set in the mc2.prop property DEFAULT_MEMCACHED_SERVERS.
Also the compilation of UserModule needs changing, in the root add the 
following lines in `Makefile.variables`:
    CXXFLAGS     += -DPARALLEL_USERMODULE
wscript in configure:
     conf.env.append_value( 'CXXFLAGS', '-DPARALLEL_USERMODULE' )

