# Defining a lifecycle for the Add-on SDK's APIs #

Until now we've not had a very well-defined lifecycle for APIs in the Add-on SDK. We've said, broadly, that APIs in the [addon-kit package](https://addons.mozilla.org/en-US/developers/docs/sdk/latest/packages/addon-kit/index.html) are "supported", meaning we won't change them "unless absolutely necessary", while APIs in the lower-level [api-utils package](https://addons.mozilla.org/en-US/developers/docs/sdk/latest/packages/api-utils/index.html) are "not fully stabilized", meaning that we expect to make incompatible changes to them.

There are some exceptions to this broad rule: some newer modules in addon-kit, like [`simple-prefs`](https://addons.mozilla.org/en-US/developers/docs/sdk/latest/packages/addon-kit/simple-prefs.html), have been marked as "experimental", and we've deprecated a few APIs in addon-kit, such as the global `postMessage()` and `on()` functions in content scripts. More recently we've landed a lot of changes to the SDK's internals, and as a result several modules in api-utils are effectively obsolete and should no longer be used.

Without a process for promoting modules from "experimental" to "supported", for deprecating APIs, or for removing deprecated APIs, things tend to stagnate:

* new modules stay "experimental" for longer than they should, meaning developers can't rely on them not to change unexpectedly
* developers can't distinguish between new and obsolete APIs (should we use [`window-utils`](https://addons.mozilla.org/en-US/developers/docs/sdk/latest/packages/api-utils/window-utils.html) or [`window/utils`](https://addons.mozilla.org/en-US/developers/docs/sdk/latest/packages/api-utils/window/utils.html))?
* deprecated APIs hang around indefinitely, increasing the support burden and making the SDK more confusing to use

When the Jetpack team met up a few weeks ago one of the things we talked about was a process for defining and communicating the stability of our APIs, and for deprecating and eventually removing obsolete APIs.

The lifecycle we've drafted has two main components:

* a "stability index", that defines how stable each module is
* a deprecation process that is intended to enable the SDK to remove or change APIs when necessary, while giving developers enough time to update their code.

## Stability Index ##

The stability index is adopted from [node.js](http://nodejs.org/api/documentation.html). Each module is assigned one of six values:

1. Deprecated
2. Experimental
3. Unstable
4. Stable
5. API Frozen
6. Locked

The stability index for each module is written into that module's metadata structure:
* [`page-mod` is `stable`](https://github.com/mozilla/addon-sdk/blob/master/packages/addon-kit/lib/page-mod.js#L9)
* [`addon-page` is `experimental`](https://github.com/mozilla/addon-sdk/blob/master/packages/addon-kit/lib/simple-prefs.js#L7)

For the time being, we'll only use three of these values:

* **Experimental**: this means that the module is not yet stabilized. You can try it out and provide feedback, but we may change or remove it in future versions without having to pass through a formal deprecation process.
* **Stable**: this means that the module is a fully-supported part of the SDK. We will avoid breaking backwards compatibility unless absolutely necessary. If we do have to make backwards-incompatible changes, we will go through the formal deprecation process.
* **Deprecated**: we plan to change this module, and backwards compatibility should not be expected. Don't start using it, and plan to migrate away from this module to its replacement.

In future releases, the SDK will read these values and expose them in the docs: in fact, the [SDK will eventually remove the "package" structure entirely](https://github.com/mozilla/addon-sdk/wiki/JEP-packageless), and then `cfx docs` will organize modules according to their stability rather than by package.

We'll periodically review APIs that are marked as "experimental" and, if possible, raise bugs to promote them to "stable".

Right now, we've raised bugs to stabilize some "experimental" APIs in addon-kit:
* [the `addon-page` module](https://bugzilla.mozilla.org/show_bug.cgi?id=790320)
* [the `simple-prefs` module](https://bugzilla.mozilla.org/show_bug.cgi?id=790323)
* [page script access to messaging](https://bugzilla.mozilla.org/show_bug.cgi?id=790328).

## Deprecation Process ##

We've drafted a deprecation process on the [Jetpack Wiki](https://wiki.mozilla.org/Jetpack/Module_Deprecation_Process). In summary, to remove or change any "stable" API, we'll:

* develop and document alternatives for any APIs we wish to deprecate
* communicate the deprecation, and support developers in migrating
to the alternative
* start issuing warnings for code that uses deprecated APIs
* keep deprecated APIs for at least three releases (18 weeks)
after deprecation, and longer if they are still seeing enough use.

Once the process is ready to go, we'll [deprecate a number of obsolete modules in api-utils](https://bugzilla.mozilla.org/show_bug.cgi?id=787075).

## What's Next? ##

This process is itself still in the "experimental" state! If you have any feedback about how we can make it better, we'd love to hear from you in comments to this post or via any of the usual channels:

* [the Jetpack discussion group/mailing list](http://groups.google.com/group/mozilla-labs-jetpack)
* [the #jetpack IRC channel](http://mibbit.com/?channel=%23jetpack&server=irc.mozilla.org)
