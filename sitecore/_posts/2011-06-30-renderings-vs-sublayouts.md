---
layout: post
title: Renderings vs. Sublayouts
---

This is a question that I'm confronted with again and again - and just today it surfaced again: _When should we use&nbsp;Sublayouts&nbsp;and when should we use&nbsp;XSL&nbsp;Renderings?&nbsp;Which is better?_

I guess my answer has always been a bit vague; it depends on what you are trying to do! Use a screwdriver for screws and a hammer for nails.

I have always used XSLT&nbsp;renderings for simple presentation and I think it's an elegant way of mixing markup with presentation logic. Obviously there is threshold where the presentation logic becomes too complex and is unmanageable in&nbsp;XSLT.

Rule of thumb: If you start contemplating XslHelper&nbsp;methods - you should probably consider doing a&nbsp;Sublayout&nbsp;instead.

Sublayouts&nbsp;is the way to go when there is any form of interaction (think POST packs and&nbsp;query string processing). One side effect of&nbsp;sublayouts&nbsp;is the temptation of using ASP.Net controls and the horrible markup they entail.

Performance wise I seem to recall that&nbsp;Sublayouts&nbsp;take approximately half the time to render than a&nbsp;XSL&nbsp;Rendering doing a comparable job. I must admit that this is not my biggest concern - as most performance issues can be mitigated with a good caching strategy.

An interesting contender is the&nbsp;[Razor for&nbsp;Sitecore](http://trac.sitecore.net/RazorForSitecore "Razor for Sitecore")&nbsp;module that allows you to render Razor views. For pure presentation it looks to be much more&nbsp;slick than messing around with&nbsp;ListView's&nbsp;etc.

Another thing to consider is also the skill level needed for the different techniques - it may be easier for a frontend developer to learn XSLT whereas an experienced ASP.Net developer would feel more comfortable doing web user controls.

I think there is none of the techniques that is a clear winner - they both have strenghts and weaknesses - so make the choice base on what you are trying to build.