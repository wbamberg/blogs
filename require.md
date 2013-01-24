In version 1.12 we changed the structure of the SDK, removing the idea of
"packages" and relocating all the SDK's modules under the "lib" directory.
This changed (and greatly simplified) the `require()` syntax used to import
modules.

This change would break backwards compatibility, so we implemented a shim
to ensure that most common usages of `require()` would be unaffected by
the change. But there are edge cases where you might run into trouble unless
you update to the new syntax. This post outlines some of these edge cases,
and highlights the way developers should use `require()` in future.

* what's "require"?
* how did it work? - complexly!
* what's changed?
* backwards compatibility
* problems
* how to use require()

## What's `require()`? ##

With the SDK, you can import objects from other modules using the `require()`
statement. This is how you use objects supplied by the SDK itself, like
page-mod and panel, and how you use objects supplied by the wider community,
like menuitems and toolbarbuttons. You can also structure your own code into
separate modules, then use `require()` to import objects from these local
modules.

The `require()` statement takes a single argument, which tells the SDK where
to find the module we need. In version 1.12, we changed the way the SDK
interprets this argument.

## How did `require()` work before? ##

Before version 1.12 of the SDK, the algorithm used to find modules was based
on the idea of **"packages"**. A package is a collection of modules. Most of
the modules in the SDK belonged to either the `addon-kit` or the `api-utils`
packages. An add-on was itself a package, and community-developed modules like
menuitems were delivered in packages of their own. A package declares
a dependency on another packages, if it wants to use modules from that package.

The algorithm was [pretty complicated](https://addons.mozilla.org/en-US/developers/docs/sdk/1.11/dev-guide/guides/module-search.html#SDK%20Search%20Rules),
but in its most basic form: the SDK would build a list of packages, always
starting with the package that contained the module doing the `require()`
and followed by the dependencies of that package. The SDK would treat the
argument to `require()` as a path to a module file, and would search all
packages in its list for a matching file, starting at the package's "lib"
directory.

So suppose an add-on called "my-addon" declares an additional dependency,
on the "menuitems" package. It would have four packages in the search list:
`["my-addon", "addon-kit", "api-utils", "menuitems"]`. If a module in
"my-addon" contains a require statement like:

`require("some-module")`

The SDK will search the following paths:

my-addon/lib/some-module.js
addon-kit/lib/some-module.js
api-utils/lib/some-module.js
menuitems/lib/some-module.js

This means that add-ons can import modules from all four packages using only
the module name as an argument:

`require("my-local-module"); // load from my-addon`
`require("panel");           // load from addon-kit`
`require("match-pattern");   // load from api-utils`
`require("menuitems");       // load from menuitems`

This seems like a nice feature, but it's tricky, because you can't tell just
from looking at a `require()` statement which module will be imported. If
module names clash, unexpected modules might be imported.

## What's changed? ##

In version 1.12 of the SDK we removed the concept of "packages". All SDK
modules were relocated under a new "lib" directory directly under the SDK
root. Along with this, a much simpler algorithm for importing from modules
was implemented:

* to import objects from SDK modules, specify the full path to the module
starting from, but not including, the "lib" directory:

    `require("panel");
    `require("page-mod/match-pattern");

* to import objects from modules in your add-on, specify a path relative
to the importing module:

    `require("./my-module");`
    `require("./subdirectory/another-module");`

## Backwards compatibility ##

Obviously, this change would break every SDK add-on in existence. To
prevent this we added a file, "mapping.json", which maps old-style
`require()` statements to their new counterparts:

    "panel": "sdk/panel"

With this in place, most users of the SDK should be unaffected by the change.



