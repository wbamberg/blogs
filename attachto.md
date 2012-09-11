We're pleased to announce a new feature for `page-mod` module, that will be introduced in Add-on SDK 1.11: The `attachTo` PageMod's option!

If you're interested in trying this feature out now, you can do so by checking out the `master` branch of the addon-sdk repository:

git clone git://github.com/mozilla/addon-sdk.git

Or download the current master's zip archive:

https://github.com/mozilla/addon-sdk/zipball/master

# What PageMod does

As the Add-on SDK documentation says, with PageMod you can dynamically modify the content of certain pages: the add-on developer supplies a set of rules to select the desired subset of web pages *based on their URL.*

Here the first important thing: a PageMod affects *any* document where the URL matches the rules supplied. Any. So, if your rule matches a specific hostname and path for example, and the topmost document that satisfy the rule has ten iframes inside, with relative URL, then your PageMod is applied eleven times. And probably that's not what you expected.

But that's not all: *"A page mod does not modify its pages until those pages are loaded or reloaded. In other words, if your add-on is loaded while the user's browser is open, the user will have to reload any open pages that match the mod for the mod to affect them."*

That's annoying, isn't it? 

# How to solve?

## Top and frames documents

Prior the Add-on SDK 1.11, the only way to applying the PageMod only on topmost document was… not using PageMod. Basically use `tabs` module to attach a content script in the desired subset of pages, simulating partially – and manually – what PageMod does:

    var { MatchPattern } = require("match-pattern");
    var tabs = require("tabs");

    var script = require("self").data.url("content.js");
    var pattern = new MatchPattern("*");

    tabs.on("ready", function onReady(tab) {
      if (pattern.test(tab.url))
        tab.attach({contentScriptFile: script});
    });

Unfortunately, beside the boilerplate code, there are some downside: `tabs` doesn't support `contentStyle*`, and if you want actually to target the frames instead of the topmost document, you can't. You can still use PageMod, and in the content script have a check like the follow:

    if (window.frameElement) {
      // loaded in a frame
    } else {
      // loaded in the topmost document
    }

But it means that the PageMod has to be attached and executed to every documents, and for memory consumption that is not suggested.

## Existing documents

To applying the PageMod on existing documents, you have a similar approach. This time using both `PageMod` and `tabs` module together:

    var { MatchPattern } = require("match-pattern");
    var { PageMod } = require("page-mod");
    var tabs = require("tabs");

    var script = require("self").data.url("content.js");
    var pattern = new MatchPattern("*");

    // PageMod will affects any new documents
    PageMod({
      include: "*",
      contentScriptFile: script
    });

    // So, we're using `tabs` module to applying the content
    // script to existing documents.
    for (var i = 0; i < tabs.length; i++)
      if (pattern.test(tabs[i].url))
        tabs[i].attach({contentScriptFile: script});

However, as we already saw, a content script attached in that way is not exactly equivalent to the PageMod's one. For instance, the content script will be attach only to the tab's document, ignoring any frames' document that could match the same `include` pattern.

# attachTo FTW!

The `attachTo` option is introduced to give to the developer more flexibility about their PageMods. With `attachTo`, if you want to applying your PageMod only on topmost document, you can simply have:

    var { PageMod } = require("page-mod");
    var script = require("self").data.url("content.js");

    PageMod({
      include: "*",
      contentScriptFile: script,
      attachTo: ["top"]
    });

Likewise, if you want to attach your PageMod on all documents – topmost, frames, and even existing ones - you will have:

    var { PageMod } = require("page-mod");
    var script = require("self").data.url("content.js");

    PageMod({
      include: "*",
    contentScriptFile: script,
    attachTo: ["top", "frames", "existing"]
    });

???

# But wait, there is more!

As you saw `attachTo` accepts an array of strings, with three possible values: `"existing"`, `"top"` and `"frames"`. And you can combine them, to open a whole new set of behaviors for PageMods. Here a comprehensive list:

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

As you probably noticed, `"existing"` alone is not listed. That's because `attachTo`, when specified, requires at least one value as documents' target (`"top"`, `"frames"`).

# A touch of style

The current implementation of `contentStyle*` is independent from the `contentScript*` and therefore `attachTo`. Because the style is registered as global stylesheet, it will always applied to all documents despite what the `attachTo` specify. We're working on address those differences. If you are interested, you can have a look at our [contentStyle issues](https://github.com/mozilla/addon-sdk/wiki/contentStyle-issues) wiki page.

# Conclusion

I personally believe that this feature is a great step forward for PageMods. But "with great power comes great responsibility": a PageMod that can be attached to existing documents means that you could have a scenario where is attached twice: An user could install your add-on, disabled it and re-enable it.

So, play safe! If you add some DOM elements to a page, just check they're not already there, or clean up what you have done when your PageMod is destroyed.

But don't be worried and stay tuned! In one of my next posts I will talk about how to clean up your PageMod's changes nicely.

# Bugs Reference

[Bug 708190 - page-mod should be able to apply to existing tabs automatically](http://bugzil.la/708190)

[Bug 684047 - page-mod: contentScripts being injected in all frames of a page](http://bugzil.la/684047)