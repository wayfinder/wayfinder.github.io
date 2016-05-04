---
title: "Sample Javascript to display map tiles"
date: 2011-01-25T20:05:59+01:00
layout: single
read_time: false
comments: false
share: true
excerpt: ''
---

Parts of the former Wayfinder web development team has extracted some
JavaScript code that might be interesting for the public. The code, together
with HTML and CSS, is available on github at
[Wayfinder-JavaScriptMapClient](http://github.com/wayfinder/Wayfinder-JavaScriptMapClient) and offers basic management
of the tiles that can be downloaded from the Wayfinder Server.

To test, simply download the code from github, and open the index.html file
with your browser. By default the code use Wayfinder Open Source server, with
the limited map coverage that is offered there, please insert your own set of
XML servers in config.js.

It is highly recommended to setup caching of the "Merkator tiles" in the
Wayfinder Server to use this.
