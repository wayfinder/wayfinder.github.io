---
title: Tools
layout: single
---

### ngpmaker

`ngpmaker` is a tool for making and manipulating ngp packets that can be sent to a NavigatorServer, see the `navserver_prot` documentation in `docs/Design` (in the Wayfinder-Server repository). It is used by the interface test, in `Tests/InterfaceTest`.

Upon startup ngpmaker asks some questions, first is the name of the output file. After that it asks for a file, this is either a captured or saved packet from a previous session, just hit enter if you don't want to load a file. Then ngpmaker asks for a param file, this is a file with a `NavRequest` line from a NavigatorServer log file. These lines are printed by the NavigatorServer whenever it receives a request packet from a client. If no file was loaded ngpmaker asks some questions for information for the packet header. After that ngpmaker asks for a command. You can hit enter to get help with the available commands which works at all levels, for example command h(eader) gives another set of commands. Exit header with d(done).

Together with the ngp documentation one can make a request from scratch by adding parameters or making changes to an existing one.

ngpmaker can be started by giving the `--ngpmaker` argument to NavigatorServer or by running the standalone tool ngpmaker found in `Server/Servers/Tools/ngpmaker`.

