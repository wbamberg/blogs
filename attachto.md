We're pleased to announce a new feature for the `page-mod` module that's coming in Add-on SDK 1.11: the `attachTo` option!

# What PageMod does

With PageMod you can dynamically modify the content of certain pages: the add-on developer supplies a set of rules to select the desired subset of web pages based on their URL.

Here the first important thing: a PageMod affects *any* document whose URL matches the rules supplied. Any. So if your rule matches a specific hostname and path, and the topmost document that satisfies the rule includes ten iframes using a relative URL, then your PageMod is applied eleven times. And probably that's not what you expected.

But that's not all: [*"A page mod does not modify its pages until those pages are loaded or reloaded. In other words, if your add-on is loaded while the user's browser is open, the user will have to reload any open pages that match the mod for the mod to affect them"*](https://addons.mozilla.org/en-US/developers/docs/sdk/latest/packages/addon-kit/page-mod.html). That's annoying, isn't it? 

# Workarounds

## Excluding frames

Before the `attachTo` option, the only way to attach a PageMod to only the topmost document was… to not use PageMod. Instead, use the `tabs` module to attach a content script to the desired subset of pages, simulating partially – and manually – what PageMod does.
Unfortunately, beside the boilerplate code, there are some downsides: `tabs` doesn't support `contentStyle*`, and if you want  to target the frames instead of the topmost document, you can't.

Alternatively, you can still use PageMod, and in the content script have a check to figure out if the document is loaded in a frame or not, like testing [window.frameElement](https://developer.mozilla.org/en-US/docs/DOM/window.frameElement) for example.
But this means that the PageMod has to be attached to and executed in every document, which can have a bad effect on memory consumption.

## Attaching to existing documents

To attach the PageMod to existing documents, you can have a similar approach, this time using both `PageMod` and `tabs` together. You basically create your regular `PageMod`, and iterating the tabs already opened with `tabs` module to attach the same content script, if the document's URL satisfies the rule specified.

However, as we already saw, a content script attached in that way is not exactly equivalent to the PageMod's one: you can't use `contentStyle*`, and the content script will be attached only to the tab's document, ignoring any sub frames.

# attachTo FTW!

This new option addresses all these issues.
With `attachTo`, if you want to attach your PageMod only to the topmost document, you can simply have:

    var { PageMod } = require("page-mod");
    var script = require("self").data.url("content.js");

    PageMod({
      include: "*",
      contentScriptFile: script,
      attachTo: ["top"]
    });

Likewise, if you want to attach your PageMod to all documents – topmost, frames, and existing ones - you will have:

    var { PageMod } = require("page-mod");
    var script = require("self").data.url("content.js");

    PageMod({
      include: "*",
    contentScriptFile: script,
    attachTo: ["top", "frames", "existing"]
    });

# But wait, there's more!

As you saw, `attachTo` accepts an array of strings with three possible values: `"existing"`, `"top"`, and `"frames"`. You can combine them, to open a whole new set of behaviors for PageMods. Here's a comprehensive list:

	// Applied to: new top documents
	// Not applied to: frames' documents; existing documents
	attachTo: ["top"]

	// Applied to: new frames' documents
	// Not applied to: top documents; existing documents
	attachTo: ["frames"]

	// Applied to: new top and frames' documents
	// Not applied to: existing documents
	// NOTE: it's equivalent to omit `attachTo` option at all
	attachTo: ["top", "frames"]

	// Applied to: all top documents
	// Not applied to: frames' documents
	attachTo: ["top", "existing"]

	// Applied to: all frames' documents
	// Not applied to: top documents
	attachTo: ["frames", "existing"]

	// Applied to: all documents
	attachTo: ["top", "frames", "existing"]

As you probably noticed, `"existing"` alone is not listed. That's because `attachTo`, when specified, requires at least one value to determine the documents' target (`"top"`, `"frames"`).

# Style-challenged

The current implementation of `contentStyle*` is independent from the `contentScript*` and therefore `attachTo`. Because the style is registered as a global stylesheet, it will always be applied to all documents, whatever `attachTo` specifies. We're working on addressing this difference. If you are interested, you can have a look at our [contentStyle issues](https://github.com/mozilla/addon-sdk/wiki/contentStyle-issues) wiki page.

# Cleaning up

I personally believe that this feature is a great step forward for PageMods. But "with great power comes great responsibility": a PageMod that can be attached to existing documents means that you could have a scenario where it is attached twice: a user could install your add-on, disable it and then re-enable it.

So play safe! If you add some DOM elements to a page, check they're not already there, or clean up what you have done when your PageMod is destroyed.

If this sounds complicated, don't worry! In my next post I'll describe how to clean up your PageMod's changes nicely.

# Try it out

If you're interested in trying this feature out now, check out the `master` branch of the addon-sdk repository:

git clone git://github.com/mozilla/addon-sdk.git

Or download the current master's zip archive:

https://github.com/mozilla/addon-sdk/zipball/master

# Bugs Reference

[Bug 708190 - page-mod should be able to apply to existing tabs automatically](http://bugzil.la/708190)

[Bug 684047 - page-mod: contentScripts being injected in all frames of a page](http://bugzil.la/684047)