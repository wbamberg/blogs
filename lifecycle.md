# Defining a lifecycle for the Add-on SDK's APIs #

Until now, we've not had a very well-defined lifecycle for APIs in the Add-on SDK.
We've said, broadly, that APIs in the [addon-kit package](https://addons.mozilla.org/en-US/developers/docs/sdk/latest/packages/addon-kit/index.html)
are "supported", meaning we will not change them "unless absolutely
necessary", while APIs in the lower-level [api-utils package](https://addons.mozilla.org/en-US/developers/docs/sdk/latest/packages/api-utils/index.html)
are "not fully stabilized", meaning that we do expect to make incompatible
changes to them.

But [some newer modules in addon-kit have been marked as
"experimental"](https://addons.mozilla.org/en-US/developers/docs/sdk/latest/packages/addon-kit/simple-prefs.html),
while we have deprecated a few APIs in addon-kit, such as the global `postMessage()` and `on()` functions in
content scripts. More recently we've landed a lot of changes to the
SDK's internals, and as a result several modules in api-utils
are effectively obsolete and should no longer be used.

Without a process for promoting modules from
"experimental" status to "supported" status, for deprecating APIs,
or for removing deprecated APIs, things tend to stagnate.

* new modules stay "experimental" for
longer than they should, meaning developers can't use them with confidence

* developers can't distinguish between new and obsoleted APIs
(should we use
[`window-utils`](https://addons.mozilla.org/en-US/developers/docs/sdk/latest/packages/api-utils/window-utils.html)
or [`window/utils`](https://addons.mozilla.org/en-US/developers/docs/sdk/latest/packages/api-utils/window/utils.html))?

* deprecated APIs hang around indefinitely, increasing the
support burden and making the SDK more confusing to use

Without a clear statement of the compatibility promises that the
SDK makes to developers, it's difficult to use any APIs with confidence.

So when the Jetpack team met up a few weeks ago one of the things
we talked about was a process for defining and communicating the stability
of our APIs, and for deprecating and eventually removing obsolete APIs.

The lifecycle we've drafted has two main components:

* a "stability index", that defines how stable each module is

* a deprecation process that's intended to enable the SDK to remove
or change APIs when necessary, while giving developers enough time
to update their code.

## Stability Index ##

The stability index is adopted from [node.js](http://nodejs.org/api/documentation.html).
Each module is assigned one of six values:

1. *Deprecated*: This feature is known to be problematic, and changes are planned. Do not rely on it. Use of the feature may cause warnings. Backwards compatibility should not be expected.
2. *Experimental*: This feature was introduced recently, and may change or be removed in future versions. Please try it out and provide feedback. If it addresses a use-case that is important to you, tell the node core team.
3. *Unstable*: The API is in the process of settling, but has not yet had sufficient real-world testing to be considered stable. Backwards-compatibility will be maintained if reasonable.
4. *Stable*: The API has proven satisfactory, but cleanup in the underlying code may cause minor changes. Backwards-compatibility is guaranteed.
5. *API Frozen*: This API has been tested extensively in production and is unlikely to ever have to change.
6. *Locked*: Unless serious bugs are found, this code will not ever change. Please do not suggest changes in this area; they will be refused.

The stability index for each module is written into that module's metadata structure:
* [`page-mod` is `stable`](https://github.com/mozilla/addon-sdk/blob/master/packages/addon-kit/lib/page-mod.js#L9)
* [`addon-page` is `experimental`](https://github.com/mozilla/addon-sdk/blob/master/packages/addon-kit/lib/simple-prefs.js#L7)

In future releases, the SDK's documentation system will read these values
and expose them: in fact, the [SDK will eventually remove the "package"
structure entirely](https://github.com/mozilla/addon-sdk/wiki/JEP-packageless),
and then `cfx docs` will organize modules according to their stability rather
than by package.

We'll periodically review APIs that are marked as "experimental" or "unstable"
and, if possible, raise bugs to promote them to a more stable state.

Right now, we've raised bugs to stabilize some "experimental" APIs in addon-kit:
* [the `addon-page` module](https://bugzilla.mozilla.org/show_bug.cgi?id=790320)
* [the `simple-prefs` module](https://bugzilla.mozilla.org/show_bug.cgi?id=790323)
* [page script access to messaging](https://bugzilla.mozilla.org/show_bug.cgi?id=790328).

## Deprecation Process ##

We've drafted a deprecation process on the
[Jetpack Wiki](https://wiki.mozilla.org/Jetpack/Module_Deprecation_Process).
To remove or change any "stable" API, we'll:

* develop and document alternatives for any APIs we wish to deprecate
* communicate the deprecation, and support developers in migrating
to the alternative
* start issuing warnings for code that uses deprecated APIs
* keep deprecated APIs for at least three releases (18 weeks)
after deprecation, and longer if they are still seeing enough use.

Once the process is ready to go, we'll [deprecate a number of obsolete
modules in api-utils](https://bugzilla.mozilla.org/show_bug.cgi?id=787075).

## What's Next? ##

This process is itself still in the "experimental" state! If you have any
feedback about how we can make it better, we'd love to hear from you by
comments to this post or via any of the usual channels:

* [the Jetpack discussion group/mailing list](http://groups.google.com/group/mozilla-labs-jetpack)
* [the #jetpack IRC channel](http://mibbit.com/?channel=%23jetpack&server=irc.mozilla.org)
