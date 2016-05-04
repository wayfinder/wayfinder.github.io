---
title: Creation of the Core and decision making
layout: single
---

### Decision

When the Core is created by the Environment, it must be as light-weight as
possible and must not take any decisions on actions without the UI/Environment
explicitly stating so.

Examples of actions that are not allowed are:

*  Opening of files
*  Connections to networks
*  Synchronization of favorites
*  Starting of Threads

The Core is however free to (and should) take follow-up decisions to achieve a goal requested by the UI/Environment.

Examples of allowed decisions are:

* Switch-overs to new servers due to the current server being offline
* Fall-backs to alternative persistent storage methods due to lack of capabilities or permissions

### Motivation


At least on the Android platform, we can expect the Core to be created on the
event dispatcher.  The Core creation must therefor be slimmed to a bare minimum
of object creation to ensure that it's quick and efficient and can be run on
for example the event dispatcher without problems.

Additionally the Core cannot predict what parts of the Core will actually be in
use by the application. Therefor we will leave it up to the UI or Environment
to request actions as appropriate.

## Core method callbacks to UI

### Decision

When the Core is created by the UI, the UI will provide the Core with a stub
implementation that decides which thread will invoke the callbacks to the UI.
The Core will guarantee that the provided method will be used for all callbacks
to the UI.

### Motivation

For the UI, there is much that can be gained if the callback is performed in
the event dispatcher thread. Populating a view will be faster and easier to do
since there is no need for the UI to reschedule such a task back onto the
dispatcher.

However, there are also situations where the event dispatcher is not available
or even bad to use due to demands on platforms or products. One example is if
the Core will be used on a BlackBerry background task where there is no event
dispatcher at all.

Because of this, the decision on how the callbacks will be made will be up to
the UI. The Core will in turn guarantee that the provided method will be used
for all callbacks to the UI.


## Core method invocations

### Decision

No exposed methods in the Core may block. Any calls that end up in blocking
code (such as I/O operations or server communication) must be done asynchronous
with use of callbacks to the UI. Any exposed synchronous methods must return
quickly.

Each method will as parameter take the listener that will be invoked upon
completion (or failure) of the request. Example code:
 
{% highlight java %}
    private final WorkScheduler iScheduler;
    private final CallbackHandler iHander;
    private final RequestIDAllocater iIDAllocater;
    
    public int doSearch(SearchRequest aRequest, final SearchListener aListener){
        // instead of doing the search immediately, we will instead create
        // a runnable or an object implementing runnable and schedule in on
        // the threadpool
        Runnable r = new Runnable() {
            public void run() {
                // this run method will actually perform the search
                // search is done here
                final SearchReply sr = new SearchReply();
                
                // we now have the SearchReply. Instead of calling the listener
                // immediately, we will again create a runnable that will
                // invoke the listener. This runnable will be scheduled on
                // the CallBackHandler and then run by whichever thread the
                // UI want's to
                Runnable rBack = new Runnable() {
                    public void run() {
                        aListener.searchDone(sr);
                    }
                };
                iHander.callOnEventDispatcher(rBack);
            }
        };
        iScheduler.schedule(r);
        
        return iIDAllocater.getNextRequestID();
{% endhighlight %}

### Motivation

Since the calls to the Core will be very much driven by the UI and the user
interaction, we can expect most (if not all) Core calls to be invoked by the
event dispatcher.

Since a blocking call on the event dispatcher will result in a non-responsive
UI, blocking methods will not be present in the Core interfaces against the UI.

Please note that the internal methods in the Core and the Core ↔ Platform
interaction may still contain blocking calls, as long as the UI never know they
are there.

## Objects and data shared between UI and Core

### Decision

All objects shared between the UI and Core must either be immutable or one-shot
objects, with a preference for immutable objects. 

If a mutable object is used, it must be considered gone once it has passed into the realm of the other part.

All objects passed to the other part must be considered ”easy to use” without
too much in-depth knowledge of the inner workings. The owning module (package)
is allowed to exploit the inner structure of data objects as long as
concurrency correctness is preserved. Only in rare,  performance critical, and
well documented, circumstances should the object change state after it has
passed the line of demarcation. This is subject to normal best-practices for
encapsulation.

### Motivation

Due to the nature of the Core being disconnected completely from the UI, we
will not be able to rely on the notion that the other part knows what the first
part is doing.

We must also be prepared for a situation where the Core (or parts of it) will
be shipped to a third party who will use it without any knowledge of how the
Core works internally.

Due to this any data that is passed between the Core and the UI must be in a
state that allows the other party to freely use it as it sees best.

## Permissions

### Decisions

All methods in the Core that runtime will be disallowed due to permission
restrictions in the currently running platform will still be available for use,
but will result in the callback containing an error (SecurityException for
synchronous methods).

Since lack of permissions can influence what the user can actually do, it will
be the responsibility of the UI to ”grey out” or hide any actions not allowed
by the current level of permissions.

An interface for checking the current level of permissions will be made
available via the Platform component.

### Motiviation

While the pre-installed versions of our clients will often have the permission
problem ”removed” by carrier signing, the problem will still remain for both
non-carrier clients and (at least) the BlackBerry platform which does not offer
the possibility to remove security by signing.

