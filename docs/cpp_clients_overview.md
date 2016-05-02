---
title: C++ clients, overview
layout: single
---

Given the long history of the C/C++ clients in this project, the configuration has become rather complex. This document aim to describe the overview of the components, how they are related and where to find more information.

The following table describes the configuration of the clients included in the released source code:

 | Client           | UI/front-end | Core library | 
 | ------           | ------------ | ------------ | 
 | S60 Navigator    | Nav 2        | x            | 
 | iPhone Navigator |              | Core V.2     | 

##### S60 Navigator

S60 Navigator is a stand alone navigation client, built on the Nav2 repository.

##### iPhone Navigator

The iPhone Navigator is built on the Core v.2 LBS API. The iPhone Navigator
client provides turn-by-turn navigation, searching and storing favourites.

The iPhone Navigator consists of several views, the image below shows the
different views that can be found in the client.

![iphoneviews](/images/iphoneviews.jpg)

The functionality can more or less be grouped into 6 groups.

* The start screen of the client, does not contain any specific functionality more than giving the user a couple choices. (blue color)
* A simple about page, consists of HTML code for showing information. (pink color)
* There are a view used for allowing the user for making a number of settings, both application related and account related. (gray color)
* There are a number views corresponding to the search functionality in the client. The search functionality consists of a view for entering a search string, choose the country to search within, display previous searches and show the search result. (green color)
* The client also includes functionality for storing places, which in fact is stored search result. Search results can therefor be added as a "My Place" and edited and removed. (purple color)
* The client contain two views for navigation, one is the route overview, displaying the route on a 2D map and the second one is the view that is used during driving, showing the route in 2D or 3D. In this view it is also possible to switch between day and night mode. The last view also display distance to next turn and current street name. (yellow view)

