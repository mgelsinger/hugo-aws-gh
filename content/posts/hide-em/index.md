---
title: "hide-em: A personal attention filter for the web"
description: "Hide names, keywords, or phrases on any website. Text-only matching. No AI, no backend - your blocklist stays on your device."
date: 2026-05-20
image: "hero.png"
tags: ["projects", "chrome-extension", "side-project", "productivity"]
links:
  - title: "Add to Chrome - Free"
    description: "Install from the Chrome Web Store"
    website: "https://chromewebstore.google.com/detail/REPLACE-WITH-YOUR-ID"
    image: "https://github.com/mgelsinger/hide-em/raw/main/icons/icon-128.png"
  - title: "View Source"
    description: "MIT-licensed on GitHub"
    website: "https://github.com/mgelsinger/hide-em"
---

You add a name, keyword, or phrase. Anything matching disappears from any page you visit - YouTube tiles, Reddit cards, search results, articles, feeds. One universal content script. Zero site-specific code.

That's the whole product.

---

## What it does

- **Block by name** - a creator, a person, an ex
- **Block by keyword or phrase** - a topic, a spoiler, a product category
- **Block by regex** - for when you need more control
- **Works on every site** - YouTube, Reddit, news sites, Twitter/X, Google search results, anything
- **Import and export rules as JSON** - move your blocklist between machines

The extension finds the nearest content card around any matching text and hides it cleanly. Not just the matching word - the whole tile, post, or article.

## What it isn't

**Not an AI filter.** There's no model guessing what you probably want to see. Exact text matching, with proper Unicode handling for names with diacritics and other edge cases. If you added the rule, it fires. If you didn't, it doesn't.

**Not a thumbnail blocker.** Text only. If the title doesn't contain the keyword, the card stays visible.

**Not a tracker.** No analytics, telemetry, accounts, or backend. Your blocklist lives in your browser's own storage and never leaves. The full source is on GitHub - audit it yourself.

**Not a community platform.** Your rules are yours. There's no sharing layer, no suggested blocklists, no social features. JSON import and export is the extent of it.

**Not an auto-suggester.** It will never suggest something to block based on your browsing behavior.

## Why I built it

I kept running into the same problem: I wanted to mute one specific creator on YouTube, one specific topic on Reddit, and one specific name across news sites - and every existing tool was either limited to one site, phoning home, or had quietly added an AI to decide what I "probably" wanted filtered.

hide-em is the simplest version of the tool I wanted. A list of rules, and anything matching disappears. Same engine everywhere.

## Privacy

Your blocklist lives in `chrome.storage.sync`, which means it follows your Chrome sign-in across your own devices through Google's standard browser sync. Nothing goes to any server I operate - there is no server. No remote code is loaded. The extension has no backend.

Source: [github.com/mgelsinger/hide-em](https://github.com/mgelsinger/hide-em)

## FAQ

**Does it work on [specific site]?**
Almost certainly. There's no site-specific code - the scanner runs on everything. If you find a site where it misses something, [open an issue](https://github.com/mgelsinger/hide-em/issues).

**What if it breaks something?**
Open DevTools and run `__heDebug.kill()` - disables the scanner on the current page immediately, no reload needed.

**Is it really free?**
Yes. MIT-licensed, no paid tier, no business model. If that changes I'll say so.
