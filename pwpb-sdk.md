Firefox 20 introduces major changes to the "private browsing" feature, which
will have an effect on add-ons developed using the SDK. This blog post
explains what the change is, how the SDK is handling it, and what add-on
developers will need to do as a result. In summary:

* if your add-on uses the `private-browsing` API, then you must **repack it**
when SDK 1.14 is released on March 12, if you want your add-on to work
properly on Firefox 20.
* whether or not your add-on uses the `private-browsing` API, you must
**update it** if you want your add-on to be able to see private windows on
Firefox 20.

## What's per-window private browsing? ##

Before Firefox 20, private browsing is a global property of the entire browser.
When the user enters private browsing, the existing browsing session is
suspended and a new blank window opens. This window is private, as are any
other windows opened until the user chooses to exit private browsing, at which
point all private windows are closed and the user is returned to the original
non-private session.

Firefox 20 introduces **per-window private browsing**. This means that private
browsing status is a property of an individual window. The user enters private
browsing by opening a new, private, window. When they do this, any existing
non-private windows are kept open, so the user will typically have both
private and non-private windows open at the same time.

## How does the SDK handle this change? ##

Under the old, global, private browsing model, add-ons can handle private
browsing as a simple binary condition: while private browsing is active,
don't store any user data. The SDK's `private-browsing` API supports this by
offering an `isActive` property to check whether the browser is in private
browsing mode, alongside `start` and `stop` events to be notified when
private browsing starts and stops.

All add-ons get to see private browsing sessions, so checking that add-ons
respect private browsing is a task for reviewers.

Here's an add-on that stores the titles of tabs the user loads, unless the
browser is in private browsing mode:

    var simpleStorage = require("simple-storage");

    if (!simpleStorage.storage.titles)
      simpleStorage.storage.titles = [];

    require("tabs").on("ready", function(tab) {
      if (!require("private-browsing").isActive) {
        console.log("storing...");
        simpleStorage.storage.titles.push(tab.title);
      }
      else {
        console.log("not storing, private data");
      }
    });

The first SDK release to target Firefox 20 is SDK 1.14. This release
updates the API to support per-window private browsing, and makes
another important change: by default, SDK 1.14-based add-ons won't see
any private windows.

### Repacking your add-on ###

Hiding private windows by default means that add-ons developers
don't need to update their code in order to respect per-window private browsing: all they need to do is repack their add-ons using SDK 1.14. The old API will log deprecation warnings, but will behave as if the user never enters
private browsing: 

* `isActive` will always be `false`, and `start` and `stop`
will never be triggered.
* the add-on will never see private windows,
or objects such as tabs that are associated with private windows
* page-mods will not be matched for private windows

You must repack your add-on though! If you don't, it may not
function correctly, and will leak user private data, because private
windows will not be hidden.

### Updating your code ###

If you want to see private windows, you'll need to set the
following key in your ["package.json"](dev-guide/package-spec.html)
file:

<pre>
"permissions": {"private-browsing": true}
</pre>

Once you do that, you'll see private windows, so if you store user data,
you'll need to use the new API to respect private browsing.

SDK 1.14 replaces the existing API with a new function `isPrivate()`
that takes an object - a window, tab, or worker - as a parameter,
and returns `true` iff the object is a private window or is associated
with a private window. So to update the add-on above, we could do
something like this:

    var simpleStorage = require("simple-storage");

    if (!simpleStorage.storage.titles)
      simpleStorage.storage.titles = [];

    require("tabs").on("ready", function(tab) {
      if (!require("private-browsing").isPrivate(tab)) {
        console.log("storing...");
        simpleStorage.storage.titles.push(tab.title);
      }
      else {
        console.log("not storing, private data");
        // do something else...
      }
    });

### Working with Firefox 19 ###

SDK 1.14 bridges the gap between the old global private browsing,
in Firefox 19, and the new per-window private browsing, in Firefox 20.

Since SDK 1.14 needs to support both versions, the new private-browsing
API is designed to work with global private browsing. When running on
Firefox 19, `isPrivate()` will return `true` if and only if the user has
global private browsing enabled.

## Summary ##

If you have an add-on built with an earlier version of the SDK,  this
section summarises your options when SDK 1.14 comes out.

### If you do nothing ###
* on Firefox 19 your add-on will continue to work fine: it
will get results for `isActive`, `start`, and `stop` that track global
private browsing, and you will be able to use them to avoid storing user
private data.
* on Firefox 20 your add-on might not work at all. If it
does, it will see private windows, but the old private-browsing API will
not ever tell you that they are private, so you may leak user private data.

### If you repack with SDK 1.14 ###
If you just repack your add-on but leave the code unchanged:
* on Firefox 19 your add-on will continue to work as before:
it will get results for `isActive`, `start`, and `stop` that track  global
private browsing, and you will be able to use them to avoid  storing user
private data.
* on Firefox 20 any of the old private-browsing functions
*(`isActive`, `start`, and `stop`) will log deprecation warnings. Your
add-on won't see any private windows or objects, such as tabs, that are
associated  with them (it will behave as if these windows just don't exist).
`isActive` will always be false, and `start`, and `stop` will never fire.

### If you update your add-on ###
If you update your add-on, by setting the "private-browsing" flag, and
updating your code to use the new `isPrivate()` API:
* on Firefox 19 your add-on will work fine: `isPrivate` will map
on to global private browsing by returning `true` if and only if the user
is in  global private browsing mode.
* on Firefox 20 you'll see private windows, and `isPrivate`
will tell you whether it's OK to  store user data associated with windows,
tabs, and workers.
