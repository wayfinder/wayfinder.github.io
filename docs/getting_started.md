---
title: Getting Started
layout: single
---

> The beginning is the most important part of the work.  
 -- Plato

## MC2 Server Development

### Setting up a system to compile MC2

MC2 in runs under most posix compliant systems but only Centos 4 is tested and 
verified to compile and work properly.

If you don't use the image above you have to have the following libraries, 
and their dependencies, installed. Some of these can be found in as RPMs in
both source and binary format for RHEL/CentOS 4 i386/x86_64 and RHEL6 i686
here: [wayfinder-rpms](https://github.com/wayfinder/wayfinder-rpms)

For more details refer to the [operations manual](../operations_manual/).

*  JTC (Java Like Threads for C++)
*  boost, boost-devel
*  openssl, openssl-devel 
*  libtecla, libtecla-devel
*  freetype, freetype-devel (Freetype 2)
*  ImageMagick, ImageMagick-devel
*  gd, gd-devel (GD Graphics Library 2.0)
*  librsvg2, librsvg2-devel
*  cairo, cairo-devel
*  zlib, zlib-devel
*  postgresql, postgresql-devel,
*  MySQL-shared-standard, MySQL-server-standard. MySQL-client-standard
*  memcached, libmemcached
*  XercesC, XercesC-devel
*  gtk2, gtk2-devel
*  gtkmm24, gtkmm24-devel

When building MC2 you have two choices either `make` or `waf`, see sections below 
for instructions for each of them.

#### Building with WAF

The root `wscript` file contains the compile options and subdirectories.
The configuration is currently only for Linux, a platform check can easily be
added in the configure method and the options for the different platforms set 
only for the right platform.
The other `wscript` files in the subdirectories adds options, that applies to it and 
all subdirectories it adds, subdirectories and targets.
Several `wscript`files adds targets by calling methods in the `waftools/servertool.py`
that are for common types of build targets.

When running waf the build command is called by default, other important 
commands are:

* `configure`  
 set up for current host
* `check`  
  run unit tests

Run `./waf --help` to see all available commands.

To set up waf call:

    ./waf configure

To compile with waf call, change 2 to the number of parallel jobs:

    ./waf -j2`

To compile only some targets set the `--targets` parameter, example:

    ./waf -j2 --targets=target[,target]

The resulting files are in the same path as the wscript adding the target 
but under the output directory located at the root of the tree.

##### To compile with distcc in waf

Add option `--distcc` to configure call, example `./waf configure --distcc`.
You also have to set the environment variable `DISTCC_HOSTS`, example:
`DISTCC_HOSTS="host1 host2"`
This requires that `distcc` is installed on all hosts, the one that `waf` is run on and all hosts in `DISTCC_HOSTS`.

#### Building with Make

`Makefile.variables` contains the options and settings per platform. 
Supported platforms are Solaris, Centos 4, Centos 6, Red Hat Enterprise Linux 4, Red Hat Enterprise Linux 6, FreeBSD,
CYGWIN_NT-5.0 and Darwin.
The only actively tested and maintained one is Centos 4.

The different `Makefile` files in the root:

* `Makefile.bin` contains rules for building binaries from cpp files.
* `Makefile.common` is the Makefile included from Makefiles in subdirectories.
* `Makefile.docs` contains rules for making doc++ documentation, see Building Documentation section for how to build documentation.
* `Makefile.lib` contains rules for building libraries.
* `Makefile.shared` contains shared rules for making dependencies and making object files from cpp files.
* `Makefile.subdirs` contains rules for subdirectories for different targets.


To compile with make run: `make -j2`. Change 2 to the number of parallel jobs,
one per core is recommend.  To compile only some targets set the ONLY
parameter: `make -j2 ONLY=target[,target]`

##### Building a release version

To make a release build without debugging information, symbols and with
optimizations: `make -j2 release=on`

##### To compile with distcc using make

Change `CXX` to `CXX=distcc g++` in `Makefile.variables`.
You also have to set the environment variable `DISTCC_HOSTS`, example:
`DISTCC_HOSTS="host1 host2`.
This requires that distcc is installed on all hosts, the one that make is run on and all hosts in `DISTCC_HOSTS`.

#### Building and running MC2 without multicast

To run on one one computer with no network compile with `SINGLE_VERSION`
defined, run: `make VERSION=single` or using `waf`:

    ./waf configure --single
    ./waf -j2`</code>`
    
Use the `mc2.prop.single` property file, the section starting at `MAP_LEADER_IP`
to `SMS_AVAILABLE_PORT` is the one setting IPs to localhost. That and the
removal of the ´MODULE_NET` section is the only difference compared with `mc2.prop`.

#### Building source code documentation

##### Doxygen documentation

Doxygen documentation from documentation in header files.
Run `doxygen doxy.conf` and the output is put in the `doxydocs` directory.

##### Doc++ documentation 

Run `make docs` 

The output is put in a `docs` directory next to each corresponding include directory in the tree.

## Java Library Development

First you need to decide which platform you wish to run. See matrix below for the recommended repos. Please note that the majority of all the functionality requires the support of the MC2 server. If you are completely new to the Java Core, it's recommended that you start off by taking a look at the Standard Java configuration since it can be run without having to install on a mobile device.

 | Platform      | Core repos                             | Client repos                        | 
 | --------      | ----------                             | ------------                        | 
 | Android       | Java-Core AND Java-Core-PAL-Android    | Android-Navigator OR Android-Locate | 
 | BlackBerry    | Java-Core AND Java-Core-PAL-BlackBerry | X                                   | 
 | Standard Java | Java-Core AND Java-Core-PAL-JSE        | X                                   | 


### Java Core

All development is intended to be done using the Eclipse IDE, however it should be fully possible to use any development environment of your choice. Please note that while the Core library can be built from within the IDE without ever invoking ant, it's highly recommended that you use ant for releases in order to benefit from the extra optimizations done by the bytecode optimization steps.

#### Prerequisites

Installation of required tools:

   * [Java Standard Edition JDK](http://java.sun.com) 1.6.0_16 or later. It's highly recommended that you use the Sun distributions.
   * [Apache Ant](http://ant.apache.org) 1.7.1 or later.
   * [Proguard](http://sourceforge.net/projects/proguard) 4.3 (please note that 4.4 will not work since files will be  corrupted, later versions have not been tested)

Installation of optional tools:

   * [Eclipse IDE for Java developers](http://www.eclipse.org) 3.5 (optional, but all configurations in this document are done with Eclipse in mind)
   * [JUnit](http://www.junit.org) 4.6 (bundled with Eclipse, but separate installation required if run from the build script).
   * [FindBugs](http://findbugs.sourceforge.net) 1.3.9 or later
            
Since the repos themselves contain auto configuration settings for Eclipse, it's highly recommended that the inital contact with the code is done using the Eclipse development environment. It's great for the environment and ok for you!

#### Config Files for Apache Ant

Setup of development environment:

* Download the core repository and place it accessible.
* Read the example developer config in `<REPO>/etc/develconfig/bob/`
* Create a folder in `<REPO>/etc/develconfig/` with the same name as your login (eg the name reported in ant as `${user.name}`)
* Copy the file `<REPO>/etc/develconfig/bob/antproperties_paths.bobs_computer.txt` to the new folder and rename it to reflect the name of your computer, eg substitute `bobs_computer` with the name of your own computer. 
* Edit the paths inside the file to reflect the paths on your computer.

Example:  
My login name is `pelle` and my computername is `serenity`. The folder and file will thus be:

´<REPO>/etc/develconfig/pelle/antproperties_paths.serenity.txt`

#### Test the ant script

Test run the installation by going into the `<REPO>/buildfiles/` folder and type: `ant clean deploy´
   
Expect the full build process to take a few minutes (depending on your computer). If successful, the assigned dist folder (from the properties file) will contain the following:

 | Subfolder    | Filename                                             | Debug printouts | Description                                                | 
 | ---------    | --------                                             | --------------- | -----------                                                | 
 | doc          | core-dev-javadoc.zip                                 | N/A             | Contains the Javadoc for the Core SDK library              | 
 | lib          | core-dev-vm_1_1_preverified-debug.jar                | Yes             | Library targeted towards the J2me or BlackBerry platform.  | 
 | lib          | core-dev-vm_1_1_preverified-release.jar              | No              | Library targeted towards the J2me or BlackBerry platform.  | 
 | lib          | core-dev-vm_1_5-debug.jar                            | Yes             | Library targeted towards the Android platform.             | 
 | lib          | core-dev-vm_1_5-release.jar                          | No              | Library targeted towards the Android platform.             | 
 | lib          | core-dev-vm_1_6-debug.jar                            | Yes             | Library targeted towards Java Standard/Enterprise Edition. | 
 | lib          | core-dev-vm_1_6-release.jar                          | No              | Library targeted towards Java Standard/Enterprise Edition. | 
 | proguard_map | core-dev-obfusc_mapping-vm_1_1_preverified-debug.map | N/A             | Proguard obfuscation mapping                               | 
 | proguard_map | core-dev-obfusc_mapping-vm_1_5-debug.map             | N/A             | Proguard obfuscation mapping                               | 
 | proguard_map | core-dev-obfusc_mapping-vm_1_6-debug.map             | N/A             | Proguard obfuscation mapping                               | 
      
Files marked `vm_1_1_preverfied` are prepared for usage on J2me or BlackBerry, eg class file format 45 (Java 1.1) with microedition preverification.
      
Files marked `vm_1_5` are prepared for usage on Android, eg class file format 49 (Java 1.5).
    
Files market `vm_1_6` are prepared for usage on the Standard/Enterprise edition of Java 6.

Files marked `-debug` contain debug printouts

Files marked `-release` have all debug printouts removed
      

**You are now ready to start coding!** 

It's recommended that you start off by reading the documents outlining the high-level architecture and then dig into the Javadoc starting with the documentation for the class `com.wayfinder.core.Core.java`.


### Java Core - Platform Support Libraries

While Java Core is written in a way that allows it to compile and work on all platforms, it requires the support of platform libraries to work for a specific platform. Compiling the platform libraries is a little bit more tricky since it requires the PAL interfaces that are located inside the Core repo.


### Java Core - Android Platform Abstraction Layer

#### Prerequisites

Installation of required tools:

   * [Java Standard Edition JDK](http://java.sun.com) 1.6.0_16 or later. It's highly recommended that you use the Sun distributions.
   * [Apache Ant](http://ant.apache.org) 1.7.1 or later.
   * [The Android SDK](http://developer.android.com/sdk/index.html) set up using Android 1.5. (If later version is used, modify row 40 in build.xml)


The PAL repositories contain the current PAL interfaces as a libfile, but if you make any modifications o 

#### Config Files for Apache Ant

Setup of development environment:

*  Download the core repository and place it accessible.
*  Read the example developer config in `<REPO>/etc/develconfig/bob/`
*  Create a folder in `<REPO>/etc/develconfig/` with the same name as your login (eg the name reported in ant as `${user.name}´)
*  Copy the file `<REPO>/etc/develconfig/bob/antproperties_paths.bobs_computer.txt` to the new folder and rename it to reflect the name of your computer, eg substitute `bobs_computer` with the name of your own computer. 
*  Edit the paths inside the file to reflect the paths on your computer.

Example: My login name is `pelle` and my computername is `serenity`. The folder and file will thus be:

`<REPO>/etc/develconfig/pelle/antproperties_paths.serenity.txt`

#### Test the ant script

Test run the installation by going into the `<REPO>/buildfiles/` folder and type: `ant clean make_pal_lib´
   
As the library is fairly small, the build process should be completed rather quickly. If successful, the assigned `dist` folder (from the properties file) will contain the following:

 | Subfolder | Filename                  | Description                                                            | 
 | --------- | --------                  | -----------                                                            | 
 | lib       | pal-android_1.5r3-dev.jar | The platform abstraction layer implementation for the Android platform | 


### Java Core - BlackBerry Platform Abstraction Layer

#### Prerequisites

Installation of required tools:

* [Java Standard Edition JDK](http://java.sun.com) 1.6.0_16 or later. It's highly recommended that you use the Sun distributions.
* [Apache Ant](http://ant.apache.org) 1.7.1 or later.
* [BlackBerry Ant Tools](http://bb-ant-tools.sourceforge.net)
* [The BlackBerry SDK](http://na.blackberry.com/eng/developers). Please note that to compile a jarfile the buildsystem only requires access to the net_rim_api.jar library.

Note! The BlackBerry PAL was developed using the vanilla SDK. For those of you out there who are partners with RIM and have access to more internal SDKs, no guarantees are given. ;)


The PAL repositories contain the current PAL interfaces as a libfile, but if you make any modifications o 

#### Config Files for Apache Ant

Setup of development environment:

* Download the core repository and place it accessible.
* Read the example developer config in `<REPO>/etc/develconfig/bob/`
* Create a folder in `<REPO>/etc/develconfig/` with the same name as your login (eg the name reported in ant as `${user.name}´)
* Copy the file `<REPO>/etc/develconfig/bob/antproperties_paths.bobs_computer.txt` to the new folder and rename it to reflect the name of your computer, eg substitute `bobs_computer` with the name of your own computer. 
* Edit the paths inside the file to reflect the paths on your computer.

Example: My login name is `pelle` and my computername is `serenity`. The folder and file will thus be:

`<REPO>/etc/develconfig/pelle/antproperties_paths.serenity.txt`

#### Test the ant script

Test run the installation by going into the `<REPO>/buildfiles/` folder and type: `ant clean make_pal_jar`
   
As the library is fairly small, the build process should be completed rather quickly. If successful, the assigned dist folder (from the properties file) will contain the following:

 | Subfolder | Filename                     | Description                                                               | 
 | --------- | --------                     | -----------                                                               | 
 | lib       | pal-blackberry_4.6.0-dev.jar | The platform abstraction layer implementation for the BlackBerry platform | 


#### Important notes regarding the BlackBerry PAL

The above command builds the library as a jar file containing preverified classfiles for JVM 1.1, eg the micro edition format. While this is the method we used when making clients (by later incorporating all libraries into a single application), we've also recognized the need to build standalone cod library files.

In order to build a fully working COD file of the library, it's possible to run it using: `ant clean make_pal_cod`
   
Please observe that when this text was written, it's only possible to do so on the windows platform since the required RAPC compiler ONLY works in the windows environment.

### Java Core - Standard Java Edition Platform Abstraction Layer

This is the platform abstraction layer implementation for Java Standard Edition. While never actually tested, there should not be a problem running this library on the Enterprise Edition of Java.

Please note that this layer was primarily used for development and testing purposes and was not intended to be used for any products. Due to this (and the fact that most desktop machines lack support for some things)m, not everything is implemented (such as the positioning system).

#### Prerequisites

Installation of required tools:

* [Java Standard Edition JDK](http://java.sun.com) 1.6.0_16 or later. It's highly recommended that you use the Sun distributions.
* [Apache Ant](http://ant.apache.org) 1.7.1 or later.

The PAL repositories contain the current PAL interfaces as a lib file.

#### Config Files for Apache Ant

Setup of development environment:

* Download the core repository and place it accessible.
* Read the example developer config in `<REPO>/etc/develconfig/bob/`
* Create a folder in `<REPO>/etc/develconfig/` with the same name as your login (eg the name reported in ant as `${user.name}´)
* Copy the file `<REPO>/etc/develconfig/bob/antproperties_paths.bobs_computer.txt` to the new folder and rename it to reflect the name of your computer, eg substitute `bobs_computer` with the name of your own computer. 
* Edit the paths inside the file to reflect the paths on your computer.

Example: My login name is `pelle` and my computername is `serenity`. The folder and file will thus be:

`<REPO>/etc/develconfig/pelle/antproperties_paths.serenity.txt`

#### Test the ant script

Test run the installation by going into the `<REPO>/buildfiles/` folder and type: `ant clean make_pal_lib`
   
As the library is fairly small, the build process should be completed rather quickly. If successful, the assigned dist folder (from the properties file) will contain the following:

 | Subfolder | Filename            | Description                                                             | 
 | --------- | --------            | -----------                                                             | 
 | lib       | pal-jse_1.6-dev.jar | The platform abstraction layer implementation for Java Standard Edition | 


## Java Client Development

### Clients - Android Navigator and Locate

As many of you know, development using the Android SDK revolves very heavily around using Eclipse. As a result, all of the Android workspaces from Wayfinder are all made with Eclipse in mind. That being said there's actually nothing preventing you from using another IDE for developing or inspecting the code inside the repositories below.

#### Prerequisites

If you are a veteran Android developer, chances are high that you already have the below installed on your machine.

Installation of required tools:

* [Java Standard Edition JDK](http://java.sun.com) 1.6.0_16 or later. It's highly recommended that you use the Sun distributions.
* [Apache Ant](http://ant.apache.org) 1.7.1 or later.
* [Eclipse IDE for Java developers](http://www.eclipse.org) 3.5 (optional, but all configurations in this document are done with Eclipse in mind)
* [The Android SDK](http://developer.android.com/sdk/index.html) set up using Android 1.5. (If later version is used, modify row 40 in build.xml)
* [Eclipse ADT Plugin](https///dl-ssl.google.com/android/eclipse/) for Android development


#### Building / testing in Eclipse

If all tools above are installed properly, it's enough to clone the repo using your favorite Eclipse Git plugin. For convenience reasons, the client repos comes preloaded with ready versions of the Core and Android PAL libraries - once cloned you're good to go! You should be able to create and install a version of the applications from inside Eclipse. For detailed information on this process we refer you to the Android developer site. (hint, hint - select "Run as Android application"... ;))

Please note that files that are (and should) be autogenerated such as the R files are _NOT_ present in the repo - Eclipse will generate these on the first build.

#### Building / testing from command prompt

Setup of development environment:

* Download the core repository and place it accessible.
* Read the example developer config in `<REPO>/userconfig/bob/` *Note the difference in path names from the core libraries*
* Create a folder in `<REPO>/userconfig/` with the same name as your login (eg the name reported in ant as `${user.name}`)
* Copy the file `<REPO>/userconfig/bob/antproperties_paths.bobs_computer.txt` to the new folder and rename it to reflect the name of your computer, eg substitute `bobs_computer` with the name of your own computer. 
* Edit the paths inside the file to reflect the paths on your computer.

Example: My login name is `pelle` and my computername is `serenity`. The folder and file will thus be:

`<REPO>/userconfig/pelle/antproperties_paths.serenity.txt`

Test run the installation by going into the repo root and run: `ant debug`

#### Building a release version

This is done using the script called `build-release.xml`, to invoke `ant` and use it, run: `ant -f build-release.xml release`

## C++ Client Development

### S60 Navigator

The S60 Navigator can be built for both S60v5 and S60v3, please note that STL is used within the code hence the OpenC/C++ plugin must be installed when using S60v3.

#### Prerequisites

*  S60 SDK 
     * For S60v5, the recommended SDK version is the Nokia N97 SDK, note that OpenC/C++ is included by default in S60v5 SDK.
     * For S60v3, the recommended SDK version is either "S60 3rd Edition, Feature Pack 2 SDK" or "S60 3rd Edition, Feature Pack 1 SDK". Please note that "Open C/C++ Plug-in" has to be installed in order to be able to compile for 3rd edition.

*  Carbide C++, the version of Carbide that has been used when creating the project is "Carbide.c++ Version 2.3.0".

*  OpenC/C++ 
     * For S60 3rd edition SDKs the OpenC/C++ plugin has to be installed separately since it is not included in the SDK by default.
     * For S60 5th edition the OpenC/C++ plugin is included by default in the SDK hence the plugin does not need to be separately installed.

#### Importing the Navigator project

 1.  Check out the Navigator repository from github, it is called Wayfinder-S60-Navigator
 2.  In Carbide, select File -> Import -> Symbian OS -> Symbian OS Bld.inf file. Select the Bld.inf file that is located in `<REPO>/group/bld.inf`.
 3.  In "Symbian OS SDKs", select the platforms you wish to build for then press "Next".
 4.  In "MMP Selection" Click "Select all" and then press "Next". 
 5.  Name the project and then click "Finish".

#### Building the Navigator project

To build the Navigator is very simple, just select which configuration you wish to build by clicking the hammer icon in the menu bar in Carbide and then start the build.

#### Creating sisx file

First you need to be aware of that the Navigator requires some capabilities in the certificate in order to function. Below is a list of all the capabilities that are required:

*  LocalServices
*  Location
*  NetworkServices
*  ProtServ
*  ReadDeviceData
*  ReadUserData
*  SwEvent
*  WriteDeviceData
*  WriteUserData

Of course you will need a certificate and key that grants use of the above capabilities in order to successfully build, install and run Navigator.

Add your key and certificate to the pkg file located in the `sis` folder. They key and certificate should be added to line number 9, by changing `YOURKEY.key` to your key and `YOURCER.cer` to your certificate.

The next step will be to set up Carbide to generate the .sisx file, this is done by doing the following:

1.  Select Properties for the Navigator project (Project -> Properties).
2.  In the left panel, select "Carbide.c++" -> "Build Configurations", click the "SIS builder" tab
3.  Under "Active Configuration" select the configuration you wish to generate the .sisx file for then click the "Add" button.
4.  The PKG File to use can be found in `<REPO>/sis/Navigator_gcce.pkg`
5.  Enter name of the Output file name if you wish.
6.  Under "Signing Options", select "Sign sis file with certificate/key pair" and fill in the required fields below.
7.  Press "OK" button.

When the above list has been completed, just recompile for the configuration you selected for the sis builder settings. The .sisx file should now be generated and ready for installation.

#### Known Issues

*  Problems when building for "Phone release".  
 Carbide is by default using the optimization flag "-O2" when building for "Phone release". This leads to some serious problems. During the startup of the application, we initialize all the modules. When doing this there is a Kern-Exec 3 panic thrown when trying to do "new NavServerCom()". No real solution for this has yet been found. We do however have a workaround which requires to change the optimization flag from "-O2" to "-O1". This can be done by doing the following (the information was found at "http://wiki.forum.nokia.com/index.php/Changing_the_GCC-E_optimization_level_to_speed_up_your_application"):
     * Open the file `epoc32/tools/compilation_config/GCCE.mk`
     * Change `REL_OPTIMISATION=-O2 -fno-unit-at-a-time" to "REL_OPTIMISATION=-O1 -fno-unit-at-a-time`.

### Core Version 3

This C++ Core version is the start of a complete rewrite of the old CppCore. Unfortunately this version is not completed. However it is a very good starting point for developers that would like to start using the binary navigator protocol when communicating with the server. The fundamentals do exist, the basic structure has been implemented according to the design. You can find a both a test client and a regression test client that can send requests to the server and parse the reply. Below is a bit more detailed explanation of how to use this new version of core.

#### Cloning the repositories

Clone the repo called Wayfinder-CppCore-v3 from github.

#### Description of the repository

Below is a short description of the five different folders that are included in the repository:

* **Core**  
 This folder contains the code of the actual modules and the server communication.

* **CoreAPI**  
 The idea is to keep the API:s between client and core in this folder. Currently there do not exist any API:s.

* **Common**  
 In this folder we keep the shared code between the different parts of Core V.3. For example, we do keep classes such as WFString, Buffer etc here.

* **PAL**  
 Here is the folder where we keep code that is platform specific in any way. For example we have the thread implementation and network implementation here. The majority of the files, or at least the functions that are visible from the outside, are written in C. 

* **ngplib**  
 This is where we keep the generated request and reply classes. All the classes here should be generated by running the supplied script "generator_cpp.py". The script creates requests and replies based on the xml-documentation of the Nav-protocol.

Note that this library is not fully complete. However, most of the requests and replies can be generated in the correct way.

#### Compiling

Compiling this new version of the C++ Core is done by running `make`. Currently, the only supported platform is Linux.

Before compiling, make sure to go into the folder of `ngplib` and run the command: `./generator_cpp.py CC`

There are a few different targets which can be compiled:

*  Core: `make core`
*  Common: `make common`
*  PAL: `make pal`
*  ngplib: `make ngp`
*  All four targets above: `make libs´
*  Core Test Client: `make coretestclient`
*  Core Regression Test Client: `make coreregtestclient`
*  Everything: `make all`

Please note that you should be located in the root of the main repository when running `make`.

##### Running clients

*  Core Test Client: `./Core/CoreTestClient/CoreTestClient`
*  Core Regression Test Client: `./Core/RegTest/CoreRegTest`
    
#### Known Issues

If build fails, change the version of the g++ compiler you are using. Change to g++ version 4.1, that should to the trick.
This is due to the source code not supporting compilers later than this version.

### Core Version 2

This is the stable version of the Core libraries. On this version Wayfinder has successfully created Navigation clients and Locate clients to e.g Symbian, iPhone and Windows Mobile. To the libraries there is a corresponding API, documentation can be found in the docs folder in the Wayfinder-CppCore-v2 repository.

The repository also includes test clients for a number of platforms. The normal test client contains a simple map. We also provide a test client for Linux, this contains examples for most of the functionality provided in the API.

In addition to the test client, the repository also contains test clients for running regression tests. These tests are used for, partly make sure old functionality works and, partly for test driven development.

#### Clone the repository

Clone the repository called Wayfinder-CppCore-v2 from github.

#### Compile for Linux

##### Requirements

* CMake, version 2.8 or later.

##### How to build

If you have the Imagination Technologies OpenGLES v.1 SDK installed in some system path such as `/usr/lib`, you can enable opengl support in the library by uncommenting the following line in CMakeLists.txt: `#SET(OPENGL_CLIENT YES)`
Then decide if you want text rendered with pango or freetype. If you want the former, you don't need to do anything. For freetype you first need harfbuzz. A convinience script has been made (note that as harfbuzz changes this script might not work any more) to make things easier: `sh copy_harfbuzz.sh` then edit the `CMakeLists.txt`file and uncomment `#SET(OPENGL_PAL_TEXT_RENDERING YES)`

Then follow the below steps to build Core version 2 and also the Linux testclient and the regression tester:

* In the root directory of the source code, run command: `cmake .`
* Run command: `make`

When the build has been successfully completed, the libraries can be found in the folder named lib.

##### Run the test clients

When running make, the test client and the regression test client for Linux will also be built. To run the test client go to the folder `cpp/Targets/WFAPITestClient` and run:  ` ./WFAPITestClient`. To run the regression test client go to the folder cpp/Targets/RegressionTests and run the command `./RegressionTester`

##### Know Issues

If building fails, change the version of the g++ compiler you are using. Change to g++ version 4.1, that should to the trick.
This is due to the source code not supporting compilers later than this version.

#### Compile for Symbian

##### Prerequisites

*  S60 SDK 
     * For S60v5, the recommended SDK version is the Nokia N97 SDK, note that OpenC/C++ is included by default in S60v5 SDK.
     * For S60v3, the recommended SDK version is either "S60 3rd Edition, Feature Pack 2 SDK" or "S60 3rd Edition, Feature Pack 1 SDK". Please note that "Open C/C++ Plug-in" has to be installed in order to be able to compile for 3rd edition.

*  Carbide C++, the version of Carbide that has been used when creating the project is "Carbide.c++ Version 2.3.0".

*  OpenC/C++ 
     * For S60 3rd edition SDKs the OpenC/C++ plugin has to be installed separately since it is not included in the SDK by default.
     * For S60 5th edition the OpenC/C++ plugin is included by default in the SDK hence the plugin does not need to be separately installed. 

##### Import the Core version 2 project into Carbide

*  Start Carbide
*  Choose File -> Import..
*  Choose Symbian OS and and then Symbian OS Bld.inf file in the window that has opened. Press Next >.
*  Browse to the folder group in the root of the source code tree and choose the `bld.inf` file and press Next >.
*  Choose your preferred build configuration, we recommend Nokia_N97_SDK_v1.0. Then press Next >.
*  In the MMP Selection dialogue, make sure all mmp files are selected and then press Next >.
*  Choose your project name and root directory for the project, then press Finish.

##### Build the project

To build is very simple, just select which configuration you wish to build by clicking the hammer icon in the menu bar in Carbide and then start the build. 

##### How to build the S60 Test client for Symbian

The repository also includes a test client for Series 60. Follow the steps below to build it.

##### Import the test client

* Start Carbide
* Choose File -> Import.. 
* Choose Symbian OS and and then Symbian OS Bld.inf file in the window that has opened. Press Next >.
* Browse to the folder containg the bld.inf file for the test client, it is located in the folder cpp/Targets/S60WFAPITestClient/WFAPITestClient/group. Select the bld.inf file and press Next >.
* Choose your preferred build configuration, we recommend Nokia_N97_SDK_v1.0. Then press Next >.
* In the MMP Selection dialogue, make sure all mmp files are selected and then press Next >.
* Choose your project name and root directory for the project, then press Finish.

##### Build the test client

To build is very simple, just select which configuration you wish to build by clicking the hammer icon in the menu bar in Carbide and then start the build.

##### Run the test client

* Choose Run -> Debug Configurations
* In the dialogue, double click on Symbian OS Emulation
* In the text box labeled “Process to launch:” change the last part of the text, “WFAPITestClient.exe” to “epoc.exe”. This will start the emulator.
* Press Apply and then press Debug.
* Enter the phones main menu, go to applications and choose WFAPITestClient
* When the test client has started you should see the map over Paris.
* (If no map is shown, you may need to add a working IAP in the emulator.)

##### How to build the regression test client for Symbian

The repository also includes a regression test client for Series 60. Follow the steps below to build it.

##### Import the regression test client

*  Start Carbide
*  Choose File -> Import.. 
*  Choose Symbian OS and and then Symbian OS Bld.inf file in the window that has opened. Press Next >.
*  Browse to the folder containg the `bld.inf´ file for the test client, it is located in the folder `cpp/Targets/RegressionTests/S60/RegressionTester/group`. Select the bld.inf file and press Next >.
*  Choose your preferred build configuration, we recommend Nokia_N97_SDK_v1.0. Then press Next >.
*  In the MMP Selection dialogue, make sure all mmp files are selected and then press Next >.
*  Choose your project name and root directory for the project, then press Finish.

##### Build the test client

To build is very simple, just select which configuration you wish to build by clicking the hammer icon in the menu bar in Carbide and then start the build.
##### Run the test client

*  Choose Run -> Debug Configurations
*  In the dialogue, double click on Symbian OS Emulation
*  Press Debug. 

#### Compile for iPhone in Xcode

##### Prerequisites

   * CMake, version 2.8 or later.
   * MacOSX 10.6
   * iPhone SDK 3.1.3

##### How to build

In a terminal, go to the main directory of the source tree and run: `cmake . -G Xcode`

The xcode project is now created. Open it in xcode and build it from menu build->build

The library files is placed in `lib/[Debug|Release]/lib*.a` as usual.
To build the IPWFAPITestClient, open the project located in `cpp/Targets/IPWFAPITestClient/TestClient/` press build and go. Done.

#### Windows Mobile support

The Core version 2 also supports Windows Mobile.
However at the moment, there is no support in the `CMakeLists.txt` files to build for Windows Mobile.  This needs to be added to be able to build.

There is also a test client included for Windows Mobile. This need to be compiled with the Vincent libraries to work.

### Iphone Navigation Client

To build the full C++ iPhone Navigator, place the cppcore tree on the same level as the navigator tree. This is needed for the navigation project to find the header files from Core V.2 which are needed to build the client. Open the project in `wf-iphone-navigation/WFNavigation/`.

#### Clone the repository

Clone the repository called Wayfinder-iPhone-Navigator from github.

#### Prerequisites

* Xcode, version 3.1.3
* Successfully builds of Core V.2 libraries

#### Before build

Please check the project settings, under the tab build. Go to the label named Library Search path. Make sure it contains the correct path to the Core V.2 libraries. The path is different depending on if you choose to build for simulator or device. If the path is incorrect you will not be able to build the navigation client.

#### Build

Just press build and run to build the client.

##### Building for device

If building for device a correct provisioning profile is needed.

### Iphone PowerSearch, version 2

This repository contains the source code for the PowerSearch client version 2. It was released on App Store and the version 3 was renamed to Locate. The client shares the map engine with the Wayfinder-S60-Navigator and thereby it is needed when building the client.

PowerSearch is a search application, searching around the users position.

#### Clone the repository

Clone the repository called Wayfinder-iPhone-PowerSearch from github.
Clone the repository called Wayfinder-S60-Navigator from github.
Make sure they are downloaded to the same folder.

#### Prerequisites

* Xcode, version 3.1.3

#### Build

Open the project file located in Wayfinder-iPhone-PowerSearch/PowerSearch folder. Then simply press build and run to build the client. Make sure both repositories are located in the same folder, otherwise the PowerSearch client will not be able to locate the necessary files in the Wayfinder-S60-Navigator repository.

##### Building for device

If building for device a correct provisioning profile is needed.

#### Known issues

When making a search the client is crashing. This is due to that the client receives search replies from the servers in two rounds. The second round is disabled on the open source servers and the client can not handle when receiving an empty reply.
