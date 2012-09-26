
Defining a lifecycle for the SDK's APIs

The Jetpack team met up a few weeks ago, and one of the things we talked
about was a process for defining and communicating the stability of our
APIs, and for deprecating and eventually removing APIs that don't work as
well as we'd like. It's important for developers to know which APIs they
can rely on, and exactly what compatibility promises the SDK makes about
a given API. On the other side, it's important that the core team can
continue to improve the SDK, and sometimes this will mean removing or
making incompatible changes to its APIs.

Until now, we've not had a very well-defined lifecycle for SDK APIs.
We've said, broadly, that APIs in the [`addon-kit` package](https://addons.mozilla.org/en-US/developers/docs/sdk/latest/packages/addon-kit/index.html)
are "supported", meaning we will not change them "unless absolutely
necessary", while APIs in the lower-level [api-utils package](https://addons.mozilla.org/en-US/developers/docs/sdk/latest/packages/api-utils/index.html)
are not fully stabilized, meaning that we do expect to make incompatible
changes to them.

But [some new modules in `addon-kit` have been marked as
"experimental"](https://addons.mozilla.org/en-US/developers/docs/sdk/latest/packages/addon-kit/simple-prefs.html),
while some APIs in addon-kit, such as the global `postMessage()` and `on()` functions in
content scripts, have been deprecated for a long time. Without a process
for promoting modules from "experimental" status to "supported" status,
or for removing deprecated APIs










* what we have now
- problems
  - promises aren't clear
  - can't remove deprecated modules/functions
  - hard to communicate individual differences
  - packagelessness breaks it all!
- examples
- packageless future

* where we're going?
- module stability definition from nodejs
- stored in module metadata
- will (somehow) be exposed in module's documentation
- deprecation process for modules: here
- modules we're planning to deprecate

* still a work in progress

