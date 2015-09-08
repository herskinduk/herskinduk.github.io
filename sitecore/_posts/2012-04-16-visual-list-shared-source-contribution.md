---
layout: post
title: Visual List Sitecore Shared Source contribution
---

On a recent project I used the [Visual List field](http://trac.sitecore.net/FieldTypes/wiki/VisualList)&nbsp;from the Sitecore Shared Source repository. The [FieldTypes project](http://trac.sitecore.net/FieldTypes/) has been around for a while so I was&nbsp;surprised to find that the field type was missing support for languages and item versions and had poor integration with the Content Editor's field validation facility.

There are no other means of maintaining a list of images in Sitecore that offers the same degree of usability for the editors (drag and drop reordering, list of thumbnails). Luckily only minor changes where needed to address the issues. I have committed [my fix](http://trac.sitecore.net/FieldTypes/changeset?old_path=%2Ftrunk%2Fsitecore+modules%2FOutercore.FieldTypes%2FVisualList&amp;old=42&amp;new_path=%2Ftrunk%2Fsitecore+modules%2FOutercore.FieldTypes%2FVisualList&amp;new=42) back to the Sitecore Shared Source repository.

![Visual List field](http://static1.herskind.co.uk/%5Epub=NjM0NzAyMTEzNjYwMDAwMDAw%5Eh=241%5Ew=725%5Ecrop=1/~/media/Images/Projects/VisualList.ashx)