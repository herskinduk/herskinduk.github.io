---
layout: post
title: Heads up! Caching statically included renderings
---

Adding Renderings and Sublayouts in Sitecore can be done in two ways.

*   Dynamically - Renderings/Sublayouts are added via the page designer or layout details dialog.
*   Statically - Renderings/Sublayouts are added in-line in the HTML code for a Sublayout/Layout.

When adding Sublayouts/Renderings dynamically the caching attributes can be assigned to either the Sublayout/Rendering definition item (in /sitecore/layouts/...) - this sets the default caching for this rendering - or it can be set directly on the layout configuration for an item. This is all described in great detail in the documentation.

There is one pitfall though. When statically adding Sublayouts/Renderings (ex. Sublayout path=... /&gt;) they do not pickup the global cache settings defined on the Sublayout/Rendering definition item. You will have to specify the cache settings directly on your tag. Here is an example:

> &lt;sc:Sublayout path=&quot;/layouts/cache.ascx&quot; runat=&quot;server&quot; cacheable=&quot;true&quot; varybydata=&quot;true&quot; /&gt;

There is a hint in the Presentation Component Reference saying that _Sitecore copies caching options from the rendering definition item to the control when you bind a rendering statically to a layout or sublayout using the Developer Center or the Grid Designer_. That&#39;s fine if you have setup your caching parameters **before** you add it and that you are aware that modifying the Sublayout/Rendering definition cache settings will not affect statically added Renderings/Sublayouts already in place.

As it stands the sc:Sublayout tag seems completely detached from Sitecore once added. Perhaps it would have been nice if you could point it to a Sublayout/Rendering definition where it could pick up path, cache settings etc.