---
title: "hide-em: A personal attention filter for the web"
description: "Hide names, keywords, or phrases on any website. Text-only matching. No AI, no backend - your blocklist stays on your device."
date: 2026-05-20
lastmod: 2026-07-24
image: "hero.png"
tags: ["projects", "browser-extension", "chrome-extension", "firefox-extension", "side-project", "productivity", "privacy"]
links:
  - title: "Add to Chrome - Free"
    description: "Install from the Chrome Web Store"
    website: "https://chromewebstore.google.com/detail/hide-em/nfdkjpjmbahmkehjaadehlncjcfidaaj"
  - title: "Firefox Source"
    description: "Build the Firefox version from source"
    website: "https://github.com/mgelsinger/hide-em-ff"
  - title: "View Source"
    description: "MIT-licensed on GitHub"
    website: "https://github.com/mgelsinger/hide-em"
---

You add a name, keyword, or phrase. Anything matching disappears from any page you visit - YouTube tiles, Reddit cards, search results, articles, feeds. One universal content script. Zero site-specific code.

That's the whole product.

---

{{< youtube yLgz1yb6FKg >}}

---

## What it does

- **Block by name** - a creator, a person, an ex
- **Block by keyword or phrase** - a topic, a spoiler, a product category
- **Block by regex** - for when you need more control
- **Tune each rule** - aliases, whole-word matching, and case sensitivity
- **Works across sites** - YouTube, Reddit, news sites, Twitter/X, Google search results, and other dynamic pages
- **Pause without deleting rules** - reveal the current tab, pause a hostname temporarily, or exclude a domain
- **Manage larger lists** - search, filter, and test a rule against sample text before saving it
- **Import and export rules as JSON** - move your blocklist manually between machines or browsers

The extension finds the nearest content card around any matching text and hides it cleanly. Not just the matching word - the whole tile, post, or article.

The toolbar popup also shows whether the scanner is active and exactly how many items are currently hidden. If a broad rule catches too much, you can reveal the page and pause filtering immediately.

## What it isn't

**Not an AI filter.** There's no model guessing what you probably want to see. Exact text matching, with proper Unicode handling for names with diacritics and other edge cases. If you added the rule, it fires. If you didn't, it doesn't.

**Not a thumbnail blocker.** Text only. If the title doesn't contain the keyword, the card stays visible.

**Not a tracker.** No analytics, telemetry, accounts, or backend. Your blocklist lives in local browser storage and never leaves. The full source is on GitHub - audit it yourself.

**Not a community platform.** Your rules are yours. There's no sharing layer, no suggested blocklists, no social features. JSON import and export is the extent of it.

**Not an auto-suggester.** It will never suggest something to block based on your browsing behavior.

## Why I built it

I kept running into the same problem: I wanted to mute one specific creator on YouTube, one specific topic on Reddit, and one specific name across news sites - and every existing tool was either limited to one site, phoning home, or had quietly added an AI to decide what I "probably" wanted filtered.

hide-em is the simplest version of the tool I wanted. A list of rules, and anything matching disappears. Same engine everywhere.

## Privacy

Your canonical configuration lives in `chrome.storage.local`, not browser-account sync. The background service worker serializes each change, writes it locally, reads it back, and only reports success after verification. A local recovery backup protects against a failed or damaged save.

Temporary tab and hostname pauses use in-memory session storage and disappear when the relevant tab or browser session ends. Moving rules between devices or browsers is an explicit JSON export and import.

Users upgrading from version 1.0 get a one-time migration from the old sync-storage format. After migration, hide-em does not continue using cloud sync. Nothing goes to a server I operate - there is no server, remote code, account, analytics, or telemetry.

## Browser support

The Chrome Web Store build targets Chrome, Edge, Brave, and other compatible Chromium browsers. A separate [Firefox source repository](https://github.com/mgelsinger/hide-em-ff) carries the same local, site-agnostic approach for Firefox 115 and newer. Until its Mozilla Add-ons listing is live, the Firefox version is a build-from-source option.

Source: [github.com/mgelsinger/hide-em](https://github.com/mgelsinger/hide-em)

## FAQ

**Does it work on [specific site]?**
Almost certainly. There's no site-specific code - the scanner runs on everything. If you find a site where it misses something, [open an issue](https://github.com/mgelsinger/hide-em/issues).

**What if it breaks something?**
Use **Reveal all and pause this tab** from the toolbar popup. For deeper troubleshooting, open DevTools and run `__heDebug.kill()` to disable the scanner on the current page immediately without a reload.

**Is it really free?**
Yes. MIT-licensed, no paid tier, no business model. If that changes I'll say so.
