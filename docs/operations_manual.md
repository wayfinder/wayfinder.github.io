---
title: MC2 Operations Manual
layout: single
---

## Introduction

_MC2_ is a distributed LBS server, offering functionality like routing,
geocoding, reverse geocoding and user management. Developed in C++, running on
a cluster of computers with a Linux OS. A database (MySQL) is used for managing
users and related information, but the maps are stored in a proprietary
(read-only) database. The system is truly distributed with two categories of
components:

   * **Front-ends;** The connection point for the clients. The front-ends are responsible for interpreting the client request and compiling the complete reply based on internal requests to the back-end modules. The front-ends are also referred to as Interfaces and Servers (since they can be seen as a server from the clients point of view). The two front-ends that are typically used today are handling the 
       - XML protocol (traditionally called the XMLServer) serving the web applications and the Java based clients.
       - NGP, Next Generation Protocol (traditionally called NavigatorServer) serving C++ based clients

   * **Back-ends;** Responsible for carrying out tasks within a specific area, like Map, Search, Route and User. All tasks requiring a significant amount of resources (memory and/or CPU) should be handled by a back-end. There can (and probably are) several instances of each back-end type to share the work load, and each type of back-ends have one _leader_ distributing the tasks between the available modules. The leader is selected via an internal voting mechanism between the available back-ends to handle failures. Internally also referred to as Modules.

A set of running components sharing the same configuration form an MC2
instance.

## Intended audience

Experienced system administrators with good Linux skills.

## Hardware Requirements

The hardware requirements to run MC2 varies depending on the level of service,
the amount of map data and the number of users. For simple demonstration
purposes a desktop class PC with 1 or 2 cores and 2-4 GB of RAM will work
fine. For large scale operations with world wide map coverage and hundreds of
thousands of global users you will need 20+ servers, gigabit switches, load
balancers, etc.

Unfortunately large scale deployments are not possible on any of the big cloud
providers such as AWS due to a lack of multicast support, which is crucial for
the MC2 components with the current design.

## Software Requirements

### Operating system

MC2 can run on several different operating systems. The main target platform
has been clones of Red Hat Enterprise Linux with RHEL 4 being mostly used but
verified and used also on both RHEL 5 and 6. MC2 works very well with CentOS
and has been deployed in production use on CentOS 4.x, which is a good fit since
it allows for easy use of file systems other than ext3.

The recommended way to to install the OS and dependencies for a production
system is to install the minimal *Base* group of packages and then add the
necessary additional packages and configuration.

#### kickstart

At Wayfinder all systems, including development systems, were fully managed by
*WFConf* (see below) and were installed using kickstart. In the WFConf example
you will also find the kickstart environment used, it's included in the
management node role. It consists of a few configuration files and a perl CGI
script. In the configuration file you use the MAC address of a server to
specify the building blocks for the kickstart file to use for it. This allows
for automatically setting root passwords based on the role of the server (eg
dev vs test vs production), disk partitioning scheme based on hard drives
available, etc. On the management / kickstart server there is a full mirror of
the CentOS version including up to date copies of all updates, managed by
*mrepo*. In `/kick` you will find the CentOS files as well as the config files:

   * `/kick/kick-config.pl` - defines where the kickstart files are and what the different installation targets are.
   * `/kick/kick-ethers.pl` - a hash with the MAC address as a key with the value being a hash of the kickstart file parts. There is also a *default* section with the kickstart file parts default values.
   * `/kick/TARGET/ks` - the location of the kickstart template and fragments, where TARGET is the distribution directory, eg c48x64 for CentOS 4.8 x86_64.

`mc2-base.ks` is a regular kickstart file with a twist, some sections are left
out and instead replaced with a placeholder, eg `<ROOT_PASSWORD>`. The name
within brackets is the fragment key, it's used within `kick-ethers.pl` to
assign a certain fragment to that section of the file.

### Other software and libraries

 | Component                 | Where to get it                               | Comment                                                                             | 
 | ---------                 | ---------------                               | -------                                                                             | 
 | **JTC**                   | 3rd party, see below                          | *Used for thread handling including synchronization, "Java Like Threads for C++"*   | 
 | **boost**                 | Included in RHEL/CentOS 4/5/6                 |                                                                                     | 
 | **openssl**               | Included in RHEL/CentOS 4/5/6                 |                                                                                     | 
 | **libtecla**              | 3rd party, see below                          | *Small library for interactive line editing*                                        | 
 | **freetype**              | Included in RHEL/CentOS 4/5/6                 |                                                                                     | 
 | **ImageMagick**           | Included in RHEL/CentOS 4/5/6                 |                                                                                     | 
 | **gd**                    | 2.0.33 or higher, included in RHEL/CentOS 5/6 |                                                                                     |
 | **librsvg2**              | Included in RHEL/CentOS 4/5/6                 |                                                                                     | 
 | **cairo**                 | 1.4.6 or newer, included in RHEL/CentOS 5/6   |                                                                                     |
 | **zlib**                  | Included in RHEL/CentOS 4/5/6                 |                                                                                     | 
 | **postgresql**            | Included in RHEL/CentOS 4/5/6                 |                                                                                     | 
 | **MySQL-shared-standard** | Included in RHEL/CentOS 4/5/6                 |                                                                                     | 
 | **memcached**             | 3rd party, see below                          | *Memcached is only used for experimental features not yet proven in production use* | 
 | **libmemcached**          | 3rd party, see below                          | *Memcached is only used for experimental features not yet proven in production use* | 
 | **XercesC**               | 3rd party, see below                          | *XML parsing library*                                                               |
 | **screen**                | Included in RHEL/CentOS 4/5/6                 |                                                                                     | 
 | **zsh**                   | Included in RHEL/CentOS 4/5/6                 |                                                                                     | 

If a newer version is needed or if it's not available in the different versions
of RHEL/CentOS you can find the packages as RPMs in both source and binary
format for RHEL/CentOS 4 i386/x86_64 and RHEL6 i686 here: [wayfinder-rpms](https://github.com/wayfinder/wayfinder-rpms)

Additional packages, such as gtkmm24, is needed to compile some tools like MapEditor, but these are not necessary to operate the server.

MC2 has support for several different databases but the only one that's up to
date and has had any large scale production use is MySQL. By default both MySQL
and PostgreSQL support is compiled and linked.

### Setting up

#### Network

It's recommended to use a separate dedicated ethernet interface or at least a
dedicated VLAN for the internal MC2 communication. Allocate private IP address
space for this and setup the default multicast route to use the same
interface/VLAN in
`/etc/sysconfig/network/static-routes/`

	
	any net 224.0.0.0 netmask 240.0.0.0 dev eth0.2


Note that MC2 uses the host name for the server to find the IP address to use
for communication, so make sure that the host names resolves to the IP adress
space you've setup for the internal communication.

The interconnecting switch should have IGMP snooping enabled, if not all
switch ports will be flooded with all of the multicast traffic.

It is not recommended to use DHCP for IP address assignment in a production
environment for other purposes than initial installation.

#### Database

MySQL needs to be configured to use InnoDB. InnoDB tuning is not within the
scope of this document and is not necessary for small scale production use.
A minimal MySQL config could look like this:

```
[client]
port                    = 3306
socket                  = /var/lib/mysql/mysql.sock

[mysqld]
port                    = 3306
socket                  = /var/lib/mysql/mysql.sock
datadir                 = /var/lib/mysql
skip-locking
max_connections         = 300
table_cache             = 600
set-variable            = key_buffer=32M
set-variable            = max_allowed_packet=1M
set-variable            = table_cache=128
set-variable            = sort_buffer=2M
set-variable            = net_buffer_length=8K
set-variable            = myisam_sort_buffer_size=8M

### innodb ###

innodb_data_file_path = ibdata1:10M:autoextend
innodb_file_per_table
innodb_data_home_dir = /var/lib/mysql/
innodb_log_group_home_dir = /var/lib/mysql/
innodb_log_arch_dir = /var/lib/mysql/
set-variable = innodb_mirrored_log_groups=1
set-variable = innodb_log_files_in_group=4
set-variable = innodb_log_file_size=5M
set-variable = innodb_log_buffer_size=16M
innodb_flush_log_at_trx_commit=1
innodb_log_archive=0
set-variable = innodb_buffer_pool_size=32M
set-variable = innodb_additional_mem_pool_size=4M
set-variable = innodb_file_io_threads=6
set-variable = innodb_lock_wait_timeout=50
```

Create at least one database and grant usage to at least one dedicated user.
It's recommended to use separate databases and users for the MC2 UserModule
and one for InfoModule to easily keep track of connections but it works fine
with a single database and user. Here's an example with a single DB and user,
please note that giving acccess for the user using a wildcard host like in the
example is not recommended for production use unless you limit access to the
database in other ways.

{% highlight sql %}
CREATE DATABASE mc2db;
GRANT ALL PRIVILEGES ON mc2db.* TO mc2@"%" IDENTIFIED BY 'secretpasword';
{% endhighlight %}

Both UserModule and InfoModule will create the tables they need automatically
on first startup.

MapModule also connects to a database in case dynamic POI info fields are
used. You can use the user and database already setup also for MapModule in
case you don't need this feature. MapModule only performs reads from the
database.  

### Internal caching web proxy

The mercator bitmap tile handling in MC2 requires a web proxy to work. The outside URL is rewritten by the frontend and requested towards the proxy, the particular bitmap is then either fetched from the proxy cache or the proxy will request it from the frontend using the rewritten URL. The proxy needs to be configured to cache requests which look dynamic, ie includes a question mark and the proxy URL needs to be set in `mc2.prop` using the setting called `INTERNAL_SQUID_URL`. Wayfinder used *Squid* but any web proxy that's correctly configured should work fine. Example setting:

	
	INTERNAL_SQUID_URL=http://127.0.0.1:3128/


## Configuration

There are many different configuration files used for different purposes in
MC2. Apart from the major configuration file `mc2.prop` there is also
configuration files for major features, for client configurations and for
managing instances.

### mc2.prop

The `mc2.prop` file is the main configuration file for MC2. Here you will find
most of the settings within MC2 and the configuration file itself documents
what settings are available and what they are used for. In this section we
will go into some detail on the most important ones.

The `mc2.prop` format is a simple ASCII file based on key/value pairs. It also
supports simple expansion of variables (keys) within the value by enclosing a
key name in curly brackets. Example:

	
	FOO = Hello
	BAR = {FOO} World


Will assign `Hello` to the key `FOO` and `Hello World` to the key `BAR`.

#### Basics

##### MAP_PATH and MODULE_CACHE_PATH

`MAP_PATH` sets the path to the set of *M3* maps you want to use. Simply point
this to the directory containing the maps. `MODULE_CACHE_PATH` sets the path
to the module map cache, which should normally be generated in advance. The
recommended way to set this is to do it based on the `MAP_PATH`:

	
	MAP_PATH = /maps/world-2010-06-01
	MODULE_CACHE_PATH = {MAP_PATH}/cache


##### *_NET - Multicast network

Setup which multicast network and ports to use by setting the different `*_NET`
variables. Best practice is to set the network to use with one variable and
point the rest to it. For advanced setups where some modules are shared among
many instances you would use more than one.

Here's a simple example of setting the `*_NET` variables:

	
	DEV_NET = 1
	
	GFX_MODULE_NET = {DEV_NET}
	TILE_MODULE_NET = {DEV_NET}
	INFO_MODULE_NET = {DEV_NET}
	MAP_MODULE_NET = {DEV_NET}
	USER_MODULE_NET = {DEV_NET}
	ROUTE_MODULE_NET = {DEV_NET}
	SEARCH_MODULE_NET = {DEV_NET}
	EXTSERVICE_MODULE_NET = {DEV_NET}
	COMM_MODULE_NET = {DEV_NET}
	EMAIL_MODULE_NET = {DEV_NET}
	SMS_MODULE_NET = {DEV_NET}


An example where you might share components is in a development environment
where a set of servers are running a development instance based on the latest
development code branch. A developer working on a certain module can the use
these components while testing his changes and will then not have to start all
of the other necessary modules. In this example the developer is working with
/TileModule/

Here's an advanced example of setting the `*_NET` variables:

	
	DEV_NET = 1
	MY_TEST_NET = 2
	
	GFX_MODULE_NET = {DEV_NET}
	TILE_MODULE_NET = {MY_TEST_NET}
	INFO_MODULE_NET = {DEV_NET}
	MAP_MODULE_NET = {DEV_NET}
	USER_MODULE_NET = {DEV_NET}
	ROUTE_MODULE_NET = {DEV_NET}
	SEARCH_MODULE_NET = {DEV_NET}
	EXTSERVICE_MODULE_NET = {DEV_NET}
	COMM_MODULE_NET = {DEV_NET}
	EMAIL_MODULE_NET = {DEV_NET}
	SMS_MODULE_NET = {DEV_NET}


There are several settings for limiting the memory used by each module, please
refer to the sample `mc2.prop` for more information. Play around with the
settings to get a feeling for the effect they have.

##### Tile cache

When MC2 generates the vectorized map data used by the clients it needs to be
cached in the file system. The `TILE_MAP_CACHE` is used to control where the
cache is stored. Best practice is to put this on a separate file system,
preferably using an LVM logical volume, or by using a loop mounted file. use
*ReiserFS* for this file system if available since the cache consists of many
small files. The reason for keeping the cache in a separate file system is that
it makes purging the cache quick and easy, simple unmount it, reformat and
mount it again. The cache can quickly ramp up to contain a great amount of
files, to make this work with a reasonable performance also for ext2 or ext3
file systems MC2 employs a simple scheme where a hierarchy of folders is used
to put the cached tile files in.

##### Database settings

There are three modules in MC2 that uses a connection to a database, the most
important one is *UserModule* which uses the database to store all user
related data. *InfoModule* uses the data for persistent storage of traffic
information and speed cameras, *MapModule* uses it to fetch dynamic
information field in POIs (Points of Interest).

All three modules are configured the same way, in these examples we will
configure only the *UserModule*. Simply replace the `USER_` prefix with either
`INFO_` or `POI` for the other two module types.

*Simple example using MySQL driver*

	
	USER_SQL_DRIVER    = mysql
	USER_SQL_HOST      = db1
	USER_SQL_DATABASE  = dev_user
	USER_SQL_USER      = mc2user
	USER_SQL_PASSWORD  = secretpassword
	USER_SQL_CHARENCODING = UTF-8


A more advanced example would use the `mysqlrepl` driver for all database
accesses, which supports a very simple failover mechanism when MySQL
replication is being used. This example also demonstrates the recommended
practice of using helper variables to set the driver and host.

Advanced example using the MySQL replication driver:

	
	# Default SQL settings
	DEFAULT_SQL_DRIVER    = mysqlrepl
	DEFAULT_SQL_HOST      = dbmaster,dbslave
	
	# User Module specific settings
	USER_SQL_DRIVER    = {DEFAULT_SQL_DRIVER}
	USER_SQL_HOST      = {DEFAULT_SQL_HOST}
	USER_SQL_DATABASE  = prod_user
	USER_SQL_USER      = mc2produser
	USER_SQL_PASSWORD  = verysecretpassword
	USER_SQL_CHARENCODING = UTF-8

Note that this method was used for a while in production at Wayfinder to fail
over from a MySQL master to a slave, but we later created functionality to deal
with failover in a more transparent way to allow several different writers to
the database to switch consistenly in case of a failure. The `mysqlrepl`
approach has a risk of a temporary issue causing a partial failover.

#### Maps and map sets

MC2 supports multiple sets of maps. This is used when you have geographically
separated sets of maps. This could be one map set for North and South America,
one for Europe, Asia, Africa and one for Australia and New Zealand. To
configure for three map sets set `MAP_SET_COUNT` to *3* and set the
corresponding `MAP_PATH` variables.

Example where three map sets is used:

	
	MAP_SET_COUNT = 3
	MAP_PATH_0 = /maps/americas-2010-06-01
	MAP_PATH_1 = /maps/europe_africa_asia-2010-06-10
	MAP_PATH_2 = /maps/australia_nz-2010-06-15
	MODULE_CACHE_PATH_0 = {MAP_PATH_0}/cache0
	MODULE_CACHE_PATH_1 = {MAP_PATH_1}/cache1
	MODULE_CACHE_PATH_2 = {MAP_PATH_2}/cache2


You will also need to configure multiple map using modules
(Map/Search/Route/Info),
at least one running module per map set, use the `--mapSet` command line
option for this.

**NOTE:** It is a recommended practice to put the module specific map cache files in directories named according to the map set since they need to be generated and used with the same map set number!

### mc2control

The individual MC2 components are managed using a simple scheme based on the
*GNU screen* tool. This allows for easy starting and stopping of all components
on a server while still retaining the possibility to easily look at the
component output as well as controlling it.

The overall settings for an instance is in `mc2control.settings`. It contains
a few settings you will probably not have to use (some are deprecated, some
were only used for development environments). The minimal configuration needs
to properly set two of these; `mc2screenprefix` and `mc2BinDir`.

An example of a `mc2control.settings` file:

```
mc2screenprefix=myinstance
mc2Dir="/tmp"
mc2BinDir=/mc2/bin-1.0.1
sourceMapDir=""
mapDir=""
tmpNewMapDir=""
tmpOldMapDir=""
logDir=/tmp
installDir=""
```

This will use the `/mc2/bin-1.0.1` directory as the working directory for this
instance and use the string `myinstance` to set the screen name.

The host names of all the servers running components for an instance is put in
the `mc2control.hosts` file.

```
dev-1
dev-2
dev-3
```

You then have one `mc2control.HOST` file for each server, where HOST is the
hostname corresponding to a line in `mc2control.hosts`. This file is actually
a *screenrc* file, so it can also include screen settings, here's an example
for the server *dev-1*, `mc2control.dev-1`.

```
setenv LOGFILE_PATH /logs/
screen -t shell /bin/zsh
screen -t Map ./runBin /mc2/bin-1.0.1/MapModule -p /mc2/etc/mc2.prop
screen -t Srch ./runBin /mc2/bin-1.0.1/SearchModule -p /mc2/etc/mc2.prop
screen -t Route ./runBin /mc2/bin-1.0.1/RouteModule -p /mc2/etc/mc2.prop
screen -t Gfx ./runBin /mc2/bin-1.0.1/GfxModule -p /mc2/etc/mc2.prop
screen -t Email ./runBin /mc2/bin-1.0.1/EmailModule -p /mc2/etc/mc2.prop
screen -t Tile ./runBin /mc2/bin-1.0.1/TileModule -p /mc2/etc/mc2.prop
screen -t Ext ./runBin /mc2/bin-1.0.1/ExtServiceModule -p /mc2/etc/mc2.prop
screen -t XML ./runBin /mc2/bin-1.0.1/XMLServer -p /mc2/etc/mc2.prop
hardstatus alwayslastline "%w"
detach
```

The above example uses the `runBin` wrapper to run all components, for details
have a look a the *Running MC2* section. It uses binaries located in
`/mc2/bin-1.0.1` and includes a basic set of modules and one frontend.
`LOGFILE_PATH` sets the location of all log files (used in `runBin`) and the
`hardstatus` line tells screen to always include a line at the bottom of your
terminal showing the windows in the screen session.

The `mc2control` script is used to control the instance, please refer to the
section *Running MC2* for more information.

#### Using multiple mc2controls

If you are running several instances or large instances on a group of servers
it's often advisable to also have multiple mc2control instances. A typical
configuration could be one mc2control per MC2 instance, or as recommended
separate the components per role into different MC2 instance. A best practice
is to put the frontends (Servers) in one, map using modules in a second and
other shared modules in a third. If you're using map sets we advice having one
mc2control for each map set. To use multiple instances of mc2control you
simply create a symlink that reflects the name:

{% highlight bash %}
ln -s mc2control mc2control-prod
{% endhighlight %}

If you split it up depending on role as described above and the MC2 instance
is named prod it would typically look like this:

{% highlight bash %}
ln -s mc2control mc2control-prodservers
ln -s mc2control mc2control-prodmap
ln -s mc2control mc2control-prodcommon
{% endhighlight %}

The configuration files reflects the names, so they would for instance be
named `mc2control-prodservers.hosts`, `mc2control-prodservers.host1` and
`mc2control-prodservers.settings`.

### MC2Conf

Handling mc2control files manually with more than a handful of servers and
instances can quickly become a mess. To make this a bit easier you can use the
simple tool called MC2Conf. Just like WFConf it's written in perl and instead
of it having a configuration file you create a a perl script that uses
functions from the `MC2Conf.pm` module.

The configuration has a list of all the servers in your server cluster, and
then sections for each mc2control instance you want to create. You typically
have one of these configuration file for each environment (dev / test /
preprod / prod) which in turn has several instances as described above.

Here's a small example with only a few servers:

{% highlight perl %}
#!/usr/bin/perl -w

# config for test environment

use MC2Conf;
use strict;
use POSIX qw(strftime);

my $timestamp = strftime "%Y%m%d%H%M%S", localtime;
my %all_nodes = (
   'server1' => { 'fqdn' => 'server1', },
   'server2' => { 'fqdn' => 'server2', },
   'server3' => { 'fqdn' => 'server3', },
   'server4' => { 'fqdn' => 'server4', },
   'server5' => { 'fqdn' => 'server5', },
   'server6' => { 'fqdn' => 'server6', },
);

# it can be convenient to specify lists of servers if they are configured and used differently:

my @frontend_nodes = gen_nodes('server', (1..3));
my @backend_only_nodes = gen_nodes('server', (4..6));
my @nodes = keys %all_nodes;

# common variables

my $version = '1.0.1';
my $path = "/mc2/bin-$version";
my $nav_params = '';
my $log_path = "/logs/";
my $default_server_params  = '--client_settings=' . $path . '/navclientsettings.txt --server_lists=' .
                             $path . '/namedservers.txt --minnumberthreads=30 --maxnumberthreads=30';
my $default_nav_params     = '--boxtype=1 --categories=' . $path . '/wfcat/ ' . $default_server_params;
my $default_testenv_nav_params      = $default_nav_params;
my $default_testenv_nav_http_params = $default_nav_params . ' --httpport=8080';
my $default_testenv_xml_params      = "--port=11199 --unsecport=19911 " .  $default_server_params;

# the different mc2control instances

# Common Modules
my $instance='test-com';
my $log_prefix = 'test-com';
my $mc2_prop='/mc2/etc/mc2-testenv.prop';
create_hosts($instance, @nodes);
create_settings($instance, $path);
my $CONFIG = open_and_add_header($instance, $log_path, @nodes);
write_all($CONFIG,        module(add_path($path, 'GfxModule'),   "-p $mc2_prop", $log_prefix, undef, 'Gfx'));
write_all($CONFIG,        module(add_path($path, 'EmailModule'), "-p $mc2_prop", $log_prefix, undef, 'Email'));
write_all($CONFIG,        module(add_path($path, 'TileModule'),  "-p $mc2_prop", $log_prefix, undef, 'Tile'));
write_one($CONFIG, 'server5', module(add_path($path, 'UserModule'),          "-p $mc2_prop -r 20", $log_prefix, undef, 'User'));
write_one($CONFIG, 'server6', module(add_path($path, 'UserModule'),          "-p $mc2_prop -r 10", $log_prefix, undef, 'User'));
write_many($CONFIG, \@backend_only_nodes, module(add_path($path, 'ExtServiceModule'),    "-p $mc2_prop", $log_prefix, undef, 'Ext'));
write_many($CONFIG, \@backend_only_nodes, module(add_path($path, 'CommunicationModule'), "-p $mc2_prop", $log_prefix, undef, 'Com'));
add_footer_and_close($CONFIG);

# Europe/Africa Modules (map set #0)
my $instance='test-com';
$instance='test-modemea';
$log_prefix = 'test-modemea';
create_hosts($instance, @backend_only_nodes);
create_settings($instance, $path);
$CONFIG = open_and_add_header($instance, $log_path, @nodes);
write_all($CONFIG, module(add_path($path, 'MapModule'),    "-p $mc2_prop --mapSet=0", $log_prefix, undef, 'Map'));
write_all($CONFIG, module(add_path($path, 'SearchModule'), "-p $mc2_prop --mapSet=0", $log_prefix, undef, 'Srch'));
write_all($CONFIG, module(add_path($path, 'RouteModule'),  "-p $mc2_prop --mapSet=0", $log_prefix, undef, 'Rte'));
write_one($CONFIG, 'server4', module(add_path($path, 'InfoModule'), "-p $mc2_prop --mapSet=0 -r 20", $log_prefix, undef, 'Info'));
write_one($CONFIG, 'server5', module(add_path($path, 'InfoModule'), "-p $mc2_prop --mapSet=0 -r 10", $log_prefix, undef, 'Info'));
add_footer_and_close($CONFIG);

# North America Modules (map set #1)
$instance='test-modamericas';
$log_prefix = 'test-modamericas';
create_hosts($instance, @backend_only_nodes);
create_settings($instance, $path);
$CONFIG = open_and_add_header($instance, $log_path, @nodes);
write_all($CONFIG, module(add_path($path, 'MapModule'),    "-p $mc2_prop --mapSet=1", $log_prefix, undef, 'Map'));
write_all($CONFIG, module(add_path($path, 'SearchModule'), "-p $mc2_prop --mapSet=1", $log_prefix, undef, 'Srch'));
write_all($CONFIG, module(add_path($path, 'RouteModule'),  "-p $mc2_prop --mapSet=1", $log_prefix, undef, 'Rte'));
write_one($CONFIG, 'server4', module(add_path($path, 'InfoModule'), "-p $mc2_prop --mapSet=1 -r 20", $log_prefix, undef, 'Info'));
write_one($CONFIG, 'server5', module(add_path($path, 'InfoModule'), "-p $mc2_prop --mapSet=1 -r 10", $log_prefix, undef, 'Info'));
add_footer_and_close($CONFIG);

# Frontends
$instance='test-srv';
$log_prefix = 'test-srv';
create_hosts($instance, @frontend_nodes);
create_settings($instance, $path);
$CONFIG = open_and_add_header($instance, $log_path, @nodes);
write_all($CONFIG, server(add_path($path, 'NavigatorServer'), $default_testenv_nav_http_params . " -p $mc2_prop", $log_prefix . '-HT', undef, 'NavHT'));
write_all($CONFIG, server(add_path($path, 'NavigatorServer'), $default_testenv_nav_params . " -p $mc2_prop", $log_prefix, undef, 'Nav'));
write_all($CONFIG, server(add_path($path, 'XMLServer'),       $default_testenv_xml_params . " -p $mc2_prop", $log_prefix, undef, 'XML'));
add_footer_and_close($CONFIG);
{% endhighlight %}

You simply run the config/script to generate the files. You can also use the
helper script called `sync.sh` to run it and synchronize the configuration to
all servers. If a symlink called `sync.sh` pointing to `conf.sh` is created you
can run `sync.sh` to only sync all of the mc2control files. `çonf.sh` needs a
config file called `çluster.conf` that should be in `/mc2/etc`.

**NOTE:** `conf.sh` and `cluster.conf` is available in the Operations repository at: http://github.com/wayfinder/Wayfinder-Server-Operations

### Other configuration files

#### Categories

The complete category handling within MC2 is not within the scope of this
document. Unfortunately there are several different methods to provide
categories to client depending on their age. This is a list of all category
related configuration files:

   * `poi_category_tree.xml`
   * `ConfigFiles/Categories/wfcat.txt`
   * `ConfigFiles/Categories/category_tree_default_configuration.xml`
   * `ConfigFiles/Categories/category_tree_region_configuration.xml`
   * `wfcat/*`

#### Client settings (`navclientsettings.txt`)

Each client that is allowed to talk to a frontend is assigned a client type.
These are configured in the `navclientsettings.txt` file. This file has a CSV
format with lots of fields per entry, a lot of these are old and deprecated.
You set things such as the initial rights assigned to a user account, length of
a trial, map coverage for a trial, etc. The file itself has lots of details in
the comments. To view it in a more easily readable HTML format you can use the
`navclientsettings_show.pl` CGI script.

#### Region definitions

You can define regions that can later be used in user rights, these are
configured in the `region_ids.xml` file. The file will let you create a region
such as /Scandinavia/, which would include Sweden, Denmark, Norway and Finland.

#### Map colors

The colors used by clients when rending vector (tile) map data is controlled in these XML files:

   * tile_map_colors.xml     
   * tile_map_night_colors.xml
   * tile_map_colors_v2.xml 
   * tile_map_night_colors_v2.xml

## Running MC2

### Managing the MC2 binaries

When you want to run MC2 on a non-development server the recommended way to
get a deployable set of files is to use the
`tools/packaging/install_prod_bin.sh` in the server repository. Set `MC2VER`
and run it and you will get a `bin-x.x.x` directory with everything necessary.

`install_prod_bin.sh` *example including output and bin directory listing*

{% highlight bash %}
$ MC2VER=1.0.1 tools/packaging/install_prod_bin.sh
BASEDIR: /devel/ckk/mc2-opensource
BIN: bin-centos4-x86_64
Installing MC2 in /devel/ckk/mc2-opensource/bin-1.0.1, using /devel/ckk/mc2-opensource as the source
  
  - Creating directories
  - Installing modules
  - Installing servers
  - Installing utilities
  - Installing scripts
  - Copying configuration files, etc except mc2.prop
  - Copying html files etc
  - Copying files for XMLServer
  - Copying other files
  - Making symlinks, patching scripts, etc
/devel/ckk/mc2-opensource/bin-1.0.1 /devel/ckk/mc2-opensource
/devel/ckk/mc2-opensource
$ ls bin-1.0.1
CommunicationModule*  InfoModule*               namedservers.txt       tile_map_colors_v2.xml
ConfigFiles/          isab-mc2.dtd@             navclientsettings.txt  tile_map_colors.xml
EmailModule*          KeyedData/                NavigatorServer*       tile_map_night_colors_v2.xml
ExtServiceModule*     map_generation-mc2.dtd    poi_category_tree.xml  tile_map_night_colors.xml
Fonts/                MapModule*                poi_scale_ranges.xml   TileModule*
GfxModule*            mc2control*               public.dtd@            TrafficServer*
HtmlFiles/            mc2control-modules@       region_ids.xml         UserModule*
httpd.pem             mc2controlscreenstarter*  RouteModule*           wfcat/
httpdwrap*            mc2control-servers@       runBin*                XML/
httpdwrap.c           mc2.prop@                 SearchModule*          XMLServer*
httpfile*             MessageTemplate/          SMSModule*
Images/               ModuleTestServer*         Supervisor*
{% endhighlight %}

### mc2control

Use the `mc2control` command or if you have multiple instances the
corresponding symlink to it (see above). The most common commands are:
`start`, `stop`, and `restart`.  All of these commands accepts an optional
hostname which will then only stop/start/restart on that particular host.

There is also a very useful but unfortunately disabled command called
`inscreenrestart` which will without any downtime restart a frontend. It's
disabled since the required shutdown scripts and functionality is not in the OSS
version. But if these missing pieces are added the functionality is still
there in `mc2control`

### Component control

The components have a direct control interface with a limited set of actions.
The most common one used is the `shutdown` command, which will cleanly shut
down the component. You can also press *Ctrl-D* to initiate a shutdown.
Related to this is the `quit` command which will cause an immediat exit of the
process without a clean shut down and the `abort` command which works like
`quit` with the addition that it tries to provoke a core dump.

There are other commands available, depending on the component type. Use the
`help` command for a list. This is what it looks like for the MapModule:
`
	
	MapModule help
	
	  help         - This help
	  quit         - quit immediately [exit(0)]
	  shutdown     - Shutdown this component gracefully (also Ctrl-D)
	  abort        - Abort this component (immediate shutdown with core dump)
	  config       - Configuration details
	  status       - Overall status of this component
	  heapstatus   - Displays information about the heap
	  set rank     - Set the rank of this module
	  vote         - Initiate a vote
	  queuestat    - Display Module queue status
	  queuedump    - Display Module queue dump
	  mapstatus    - Displays data about the module's maps


And this is the `help` output for XMLServer:

	
	XMLServer help
	
	  help         - This help
	  quit         - quit immediately [exit(0)]
	  shutdown     - Shutdown this component gracefully (also Ctrl-D)
	  abort        - Abort this component (immediate shutdown with core dump)
	  config       - Configuration details
	  status       - Overall status of this component
	  heapstatus   - Displays information about the heap


Two useful commands for modules are `set rank` and `vote`; `set rank` changes
the rank used when voting for a leader (higher is better) and `vote` will
initiate a vote. You can use this if you in a controlled fashion want to switch
the module leader role away from a server, simply do a `set rank` of a higher
value (0 is default) and then use `vote`.

### runBin

The `runBin` wrapper is typically used to run a component. It will restart the
component if it is shutdown or restarted and also handles the logging. If
`wftee` is available it will be used to also enable automatic log rotation (see
below).

### Supervisor

`Supervisor` is a simple tool that visualizes the current state of the backend
modules. It will show basic data for each module showing queue lenght,
processing time, load averages and a list of loaded modules. If you have few
servers and modules simply run `Supervisor`, if you have a lot run `Supervisor
-m -c` to group each module type and hide the complete list of maps (it will
show a count instead). There is also some keyboard commands you can use while
Supervisor runs.

`Supervisor` also has a *raw* mode which you can use to process the data in
scripts. In the *operations* repository you'll find a perl module and a few
scripts that uses the raw mode, including one which generate *rrdtool* RRDs for
the data and one which monitors the different values and alerts you if the
configured thresholds are exceeded.

### log files (wftee)

`wftee` is an extended version of `tee(1)` which will ignore temporary write
errors due to a full file system for instance. It can also perform automatic
log rotation and compression.

`wftee` *usage*

	Usage:
	./operations/wftee/wftee [-v] [-d] [-c] [-m path] file_template
	
	  -v - turn on verbose mode
	  -d - turn on debug messages
	  -c - turn on compression
	  -m - set directory to move files to after rotation
	
	file_template is a filename with optional strftime(3) format specifiers
	and is rotated when the expanded name changes. Use double % characters to include
	the data but not use it for rotation
	  example: log_file_%Y%m%d.txt will name the log file according to todays
	  date, e.g. log_file_20100615.txt, and automatically rotate it every day,
	  the next file name would be log_file_20100616.txt
	  Too also include the hour and minute in the file name but still rotate
	  when the date changes use log_file_%Y%m%d%%H%%M.txt


Compile `wftee` by running `make`, strip it using `strip` and put it in
`/usr/local/bin/wftee` on your servers.

### Database maintenance

There are a few tables in the database which you will have to do some
maintenance on:

 | **ISABDebit**              | Details on user transactions              | *Clean this regularly if not used, it will grow pretty large otherwise and it contains user location data*                                  | 
 | -------------              | ----------------------------              | ------------------------------------------------------------------------------------------------------------                                  | 
 | **ISABRouteStorage**       | Route data cache                          | *Clean this regularly, it grows a lot otherwise. If a route is not found here it will be recalculated using data from `ISABRouteStorage`* | 
 | **ISABRouteStorageCoords** | Start and end points for stored routes    | *Clean this occasionally, but only during periods with little traffic and not as often as `ISABRouteStorage`*                             | 
 | **changelog tables**       | Changelog tables for various other tables | *Clean these occasionally when the grow to large. Only useful to track changes, not used in any other ways.*                                | 

## Recommended configurations

The hardware and software configuration of a cluster depends on a few
variables, the most important ones are the number of users, the usage patterns
for the apps and/or users and the map coverage.

The total amount of memory you need is heavily dependent on the map coverage.
For complete map coverage of the world using one of the major commercial
providers you will need a lot more than a smaller map set using OSM data (at
least currently). Easiest way to determine the memory needs is to test loading
the data. Make sure to allow for redundancy, have at least memory capacity
corresponding to two servers as spares.

For large scale production instances we recommend putting frontends and
backends on dedicated servers instead of mixing frontend and backend
components on the same server. 

If you have specific loads that you need to handle, such as a large amount of
bitmap generation, just add more copies of `GfxModule`. If noone ever uses
routing on a certain cluster instance you do not have to run any copies of
`RouteModule`.

The database you use can become a bottleneck, tune it and scale up the
hardware if needed. See also the section below on large scale use.

### Development

You can have a smaller development environment if you use a smaller set of
maps. Remember that several developers working on different parts in the same
branch of the code can share running components by setting the multicast net
property (see above).

### Testing

It's recommended to have full map coverage for the testing environment, but
unless it's used for full-scale simulated load testing it can still be a lot
smaller than a production cluster. 

Make sure that it's big enough to give your testers or testing community a
performance that will match the user experience when running in the production
environment.

### Production

Make sure that you allow for growth, monitor response times in both frontend
and backends and add hardware early. Use tools such as *ganglia* or *munin* to
also monitor low-level metrics.

Since MC2 components are separate components that you typically run many of on
the same servers multiple CPU cores will give you better performance.

Always use at least gigabit ethernet in a production environment.
 
## User account administration

###  Introduction 

The web based user administration site is not ready for release but will
hopefully be available soon, this section will then be updated.

###  Setting up 

*site content and configuration*

###  Using it  

*Description of the included functionality*

###  User rights 

*Basic information about user rights*

## Monitoring

We recommend using *ganglia* for monitoring low level metrics. You can use
`ganglia_fs_mon` and the corresponding `cron.d` file available in the
*Operations* repo to also monitor file system metrics.

The `nagios_ganglia_source.pl` is nagios monitoring plugin that can be used to
monitor any metric available in ganglia. This script is also available in the
*Operations* repo.

## Backup

It's easy to backup an MC2 cluster, make sure that you have offsite copies of
the map data and the binaries and then backup the database using eg
`xtrabackup` (http://www.percona.com/docs/wiki/percona-xtrabackup:start)
Combined with using automated installations (eg kickstart + WFConf) and you
will have a complete backup and recovery system in place.

## Large scale use

### Load balancing

Load balance across multiple frontends using LVS (Linux Virtual Server) or a
commercial IP load balancer. Wayfinder used LVS/keepalived in production use
for several years with great success. Note that the load balancer must support
session stickiness for full functionality and best performance since some
features require that the clients talks to the same server (eg precaching map
data for routes).

### memcached and multiple database read slaves

There is experimental code in MC2 to use MySQL replication slaves for reading
from UserModule availables and also to do caching using memcached. This has
never been used in a production environment and requires a setup that can
migrate IPs for both MySQL and memcached when doing failovers.

### WFConf

WFConf is a very simple Configuration Management system that is quite generic
but was written and used only within Wayfinder. In general we would recommend
that you use eg [Puppet](http://www.puppetlabs.com/) or
[Ansible](http://www.ansible.com) instead, but WFConf is a simple and
lightweight alternative for managing a homogenous collection of servers with
the same operating system. WFConf is not a requirement to install and operate
an MC2 server, you can skip this section if you want to.

WFConf is written in perl but all of the actions needed to be taken to
configure a machine is handled by shell script code that WFConf generates. The
WFConf configuration file is actually perl code, which is a simple and quick
approach but unfortunately makes troubleshooting a bit difficult.

WFConf allows for applying almost all of the configuration needed on a server
to run MC2. Only some basic boot strapping needs to be performed during a
kickstart install for it to work. It can manage LVM based file systems, users,
groups, files, packages etc.

#### Basics

In WFConf you describe different roles a certain server can have. To do this
you start by defining different configurations. An example covering a lot of 
the functionality is a web server config, in this case requiring a dedicated 
file system for content, packages for Apache httpd, PHP and a bunch of config 
files.

*web server config*

	
	$config{'web_server'} = <<EOF
	packages('httpd-2.0.63', 'mod_ssl', 'php', 'php-gd', 'php-mysql');
	file_system('sysvg/www', '/var/www/sites', '15G', 'root.root', '755', 'ext3', '150000');
	nofile('/etc/httpd/conf.d/welcome.conf');
	file('php/php.ini',                 '/etc/php.ini',                          '644', 'root.root');
	file('apache/site-example.conf',    '/etc/httpd/conf.d/site-example.conf',   '644', 'root.root');
	file('apache/mime.types',           '/etc/mime.types',                       '644', 'root.root');
	file('logrotate/httpd',             '/etc/logrotate.d/httpd',                '644', 'root.root');
	file('apache/certs/the-web-server-cert.cer',  '/etc/httpd/conf/the-web-web-server.cer',    '644', 'root.root');
	file('apache/certs/the-web-server-cert.key',  '/etc/httpd/conf/the-web-web-server.key',    '644', 'root.root');
	# Start apache if not already running
	service('httpd', 'on', 'start');
	# Reload apache if it was already running
	service('httpd', 'on', 'reload');
	EOF
	;


You would typically also have a base config that is shared among all roles,
in this example everything we want a generic server to have configured:

	
	$config{'generic_server'} = <<EOF
	group('mc2', 500);
	user('mc2', 500, '/mc2', '/bin/zsh', 'mc2', undef);
	file_system('sysvg/mc2',   '/mc2',             '2G',    'mc2.mc2',     '1755', 'ext3',     '150000');
	file_system('sysvg/wayf',  '/wayf',            '128M',  'mc2.mc2',     '1755', 'ext3',     '150000');
	packages('bind','bind-chroot','ntp','OpenIPMI-tools', 'vim-enhanced', 'strace', 'sysstat');
	dir('/etc/wayf', '755', 'root.root');
	# SSH host keys
	dir('/root/.ssh', '700', 'root.root');
	file('ssh/keys.$cluster/$host/ssh_host_key',         '/etc/ssh/ssh_host_key',         '600', 'root.root');
	file('ssh/keys.$cluster/$host/ssh_host_dsa_key',     '/etc/ssh/ssh_host_dsa_key',     '600', 'root.root');
	file('ssh/keys.$cluster/$host/ssh_host_rsa_key',     '/etc/ssh/ssh_host_rsa_key',     '600', 'root.root');
	file('ssh/keys.$cluster/$host/ssh_host_key.pub',     '/etc/ssh/ssh_host_key.pub',     '644', 'root.root');
	file('ssh/keys.$cluster/$host/ssh_host_dsa_key.pub', '/etc/ssh/ssh_host_dsa_key.pub', '644', 'root.root');
	file('ssh/keys.$cluster/$host/ssh_host_rsa_key.pub', '/etc/ssh/ssh_host_rsa_key.pub', '640', 'root.root');
	file('ssh/keys.$cluster/root/authorized_keys',       '/root/.ssh/authorized_keys',    '644', 'root.root');
	file('ssh/keys.$cluster/root/id_dsa',                '/root/.ssh/id_dsa',             '600', 'root.root');
	file('ssh/keys.$cluster/root/id_dsa.pub',            '/root/.ssh/id_dsa.pub',         '644', 'root.root');
	file('ssh/sshd_config',                              '/etc/ssh/sshd_config',          '644', 'root.root');
	# wfconf init file
	file('wfconf_hg.init',                               '/etc/init.d/wfconf',             '755', 'root.root');
	# Updatedb configuration file
	file('updatedb/updatedb.conf',          '/etc/updatedb.conf',     '644', 'root.root');
	# Network related
	file('network/hosts.$cluster',          '/etc/hosts',             '644', 'root.root');
	file('network/resolv.conf.$cluster',    '/etc/resolv.conf',       '644', 'root.root');
	# Yum repos
	file('yum/Wayfinder-CentOS.repo', '/etc/yum.repos.d/Wayfinder-CentOS.repo', '644', 'root.root');
	file('yum/Wayfinder-EL4.repo',    '/etc/yum.repos.d/Wayfinder-EL4.repo',    '644', 'root.root');
	# NTP
	file('ntp/ntp.conf',     '/etc/ntp.conf',             '644', 'root.root');
	file('ntp/step-tickers', '/etc/ntp/step-tickers',     '644', 'root.root');
	# unpackaged tools
	file('bin/wftee.$arch',                 '/usr/local/bin/wftee',                            '755', 'root.root');
	# sysctl
	file('misc/sysctl.conf',     '/etc/sysctl.conf',             '644', 'root.root');
	# logrotate
	file('logrotate/logrotate.conf', '/etc/logrotate.conf',      '644', 'root.root');
	# mail / sendmail
	file('mail/aliases',         '/etc/aliases',                 '644', 'root.root');
	file('mail/sendmail.mc.$cluster', '/etc/mail/sendmail.mc', '644', 'root.root');
	# ganglia
	file('ganglia/gmond.conf.$cluster',          '/etc/gmond.conf',                  '644', 'root.root');
	file('ganglia/cron.ganglia_fs_mon', '/etc/cron.d/ganglia_fs_mon',       '644', 'root.root');
	file('ganglia/ganglia_fs_mon.sh',   '/usr/local/bin/ganglia_fs_mon.sh', '755', 'root.root');
	perms('/usr/bin/gmetric', '511', 'root.root');
	service('gmond', 'on', 'restart');
	# turn on named
	service('named', 'on', 'restart');
	# Turn of kudzu
	service('kudzu', 'off', 'stop');
	# Turn on loading of the IPMI local device driver
	service('ipmi', 'on', 'start');
	# Make sure that ssh is on and restart it to catch any changes
	service('sshd', 'on', 'restart');
	# Make sure that ntpd is on and restart it to sync the time
	service('ntpd', 'on', 'restart');
	service('sendmail', 'on', 'restart');
	# CentOS default yum repo defs are inside the centos-release RPM
	nofile('/etc/yum.repos.d/CentOS-Base.repo');
	nofile('/etc/yum.repos.d/CentOS-Media.repo');
	# generate new aliases
	shell('newaliases');
	EOF
	;


The above example includes even more WFConf functionality, such as adding
users and running additional commands through a shell.

The configs are then used to define a role. A web server role could be defined
lke this given the above configs:

	
	$role_configs{'web_server'}               = [
	     'generic_server', 'web_server',
	];


You then assign roles to individual servers, in this case the two servers
called web1 and web2 are assigned the web server role:

	
	$roles{'web_server'}          = ['web1', 'web2', 'web3'];


At this stage the documentation for WFConf is very limited, please have a look
at the available example config in the `<ops repo>` for more complete examples.

As mentioned above WFConf needs some minimal boot strapping to work, how much
depends a little bit on what you use in your configuration. For example if you
want to use reiserfs or XFS on CentOS you need to make sure that the necessary
kernel, modules and tools are available. This is a complete example to
bootstrap WFConf, taken from the `%post` section of a kickstart file for
CentOS 4.7 x86_64:

*WFConf bootstrapping*

	%post
	(
	# Mount /kick so we can access the config files
	mkdir /tmp/kick
	echo "* Mounting kickstart:/kick"
	mount kickstart:/kick /tmp/kick
	echo "* Setting up the network and hostname"
	# fix hostname, removes 'int-' prefix assigned when installing
	sed -i -e 's/int-//g' /etc/sysconfig/network
	source /etc/sysconfig/network
	echo "

	* Fix yum/rpm"
	cp -v /tmp/kick/wfconf/yum/Wayfinder-CentOS.repo /etc/yum.repos.d/Wayfinder-CentOS.repo
	cp -v /tmp/kick/wfconf/yum/Wayfinder-EL4.repo /etc/yum.repos.d/Wayfinder-EL4.repo
	cp -v /tmp/kick/wfconf/yum/Wayfinder-3rd.repo /etc/yum.repos.d/Wayfinder-3rd.repo
	rm /etc/yum.repos.d/CentOS-Base.repo
	rpm --import /usr/share/doc/centos-release-4/RPM-GPG-KEY-centos4
	rpm --import /tmp/kick/wfconf/rpm/RPM-GPG-KEY-wayfinder
	echo "* Installing CentOS plus kernel"
	echo "  - Removing old normal kernel"
	rpm -e kernel
	echo "  - Updating kernel-smp with centosplus kernel"
	rpm -Uv /tmp/kick/dist/src/centos-4.7-x86_64/centosplus/kernel-smp-2.6.9-78.0.13.plus.c4.x86_64.rpm
	rpm -Uv /tmp/kick/dist/src/centos-4.7-x86_64/centosplus/kmod-xfs-smp-0.4-1.el4.2.6.9_78.0.13.plus.c4.x86_64.rpm
	echo "* Installing additional critical packages"
	echo "  - Installing ReiserFS tools"
	rpm -Uv /tmp/kick/dist/src/centos-4.7-x86_64/centosplus/reiserfs-utils-3.6.19-2.4.1.x86_64.rpm
	echo "  - Installing XFS tools"
	rpm -Uv /tmp/kick/dist/src/centos-4.7-x86_64/centosplus/xfsprogs-2.9.4-1.el4.centos.x86_64.rpm
	echo "* wfconf config provisioning"
	echo " - Copying current wfconf tree to /tmp"
	cp -a /tmp/kick/wfconf /tmp
	echo " - Add service file and enable it"
	install --verbose -C --mode 755 --preserve-timestamps --owner root --group root \
	/tmp/kick/wfconf/wfconf_hg.init /etc/init.d/wfconf
	chkconfig --add wfconf
	echo "* Unmounting kickstart:/kick"
	umount /tmp/kick
	rmdir /tmp/kick
	) 2>&1 | tee /root/kick-post.log > /dev/console


